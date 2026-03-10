# Manual Mode and Resilience

## Context

Kalabot needs a deterministic way to move between automation and human operation without losing messages, duplicating replies, or confusing operators. This document defines:

- `bot_enabled` semantics and ownership of the conversation
- Queue and retry behavior for outbound work
- Timeout-to-human escalation when automation is unhealthy
- Operator UX states for clear and safe intervention

---

## 1. Core Principles

- **No message loss:** every inbound message is persisted before processing.
- **Single active owner:** at any time, either bot or human is the primary responder.
- **Human can take over anytime:** any human intervention immediately pauses bot replies.
- **Fail safe to human:** on prolonged uncertainty, route to manual mode.
- **Idempotent retries:** all outbound sends and downstream actions must be safe to replay.
- **Transparent UX:** operators always see why a conversation is in its current state.

---

## 2. Conversation Ownership Model

### 2.1 `bot_enabled` semantics

`bot_enabled` is a conversation-level switch that controls whether automation is allowed to generate and send replies.


| `bot_enabled` | Meaning                                     | Who can send automatic replies |
| ------------- | ------------------------------------------- | ------------------------------ |
| `true`        | Automation is active for the conversation   | Bot                            |
| `false`       | Automation is disabled for the conversation | Human only                     |

When any human sends a message in the thread (operator or business user), the system must atomically:

1. set `bot_enabled=false`
2. cancel/suppress pending bot sends for that conversation
3. mark `manual_mode_reason=operator_forced`

### 2.2 Global manual modes

In addition to conversation-level `bot_enabled`, there are two global override scopes:

| Flag | Scope | Who can toggle | Effect |
|---|---|---|---|
| `tenant_global_manual` | Single client account | Client and internal operator | Bot disabled for all conversations of that client |
| `platform_global_manual` | All client accounts | Internal admin only | Bot disabled for all conversations in the platform |

These flags do not erase per-conversation state. They override execution while active.

### 2.3 Effective ownership precedence

Automation eligibility is computed using strict precedence:

1. if `platform_global_manual=true` -> bot disabled
2. else if `tenant_global_manual=true` -> bot disabled
3. else if `conversation.bot_enabled=false` -> bot disabled
4. else -> bot enabled

Effective expression:

`effective_bot_enabled = !platform_global_manual && !tenant_global_manual && conversation.bot_enabled`

When `effective_bot_enabled=false`, new bot generation and outbound bot sends must not run.


### 2.4 Related runtime flags

`bot_enabled` alone is not enough to explain transient states. Runtime flags are needed for observability and UX:


| Field                  | Type               | Purpose                                                                                          |
| ---------------------- | ------------------ | ------------------------------------------------------------------------------------------------ |
| `manual_mode_reason`   | enum nullable      | Why automation is off (`operator_forced`, `timeout_to_human`, `provider_outage`, `policy_block`) |
| `automation_health`    | enum               | Current health (`healthy`, `degraded`, `down`)                                                   |
| `last_bot_attempt_at`  | timestamp nullable | Last attempt to process or send a bot reply                                                      |
| `human_taken_over_at`  | timestamp nullable | First moment a human took ownership                                                              |
| `recovery_eligible_at` | timestamp nullable | Earliest time auto-recovery can re-enable bot                                                    |

For global controls, also track:

| Field | Type | Purpose |
|---|---|---|
| `tenant_global_manual` | boolean | Client-wide manual override |
| `tenant_global_manual_reason` | text nullable | Client-provided reason or operator note |
| `platform_global_manual` | boolean | Platform-wide manual override |
| `platform_global_manual_reason` | text nullable | Incident/change reason |


---

## 3. State Model

System behavior should be represented as explicit states, not inferred from logs.


| State                       | Conditions                                               | Allowed owner                            |
| --------------------------- | -------------------------------------------------------- | ---------------------------------------- |
| `auto_active`               | `bot_enabled=true`, health healthy                       | Bot                                      |
| `auto_degraded`             | `bot_enabled=true`, health degraded                      | Bot with guarded retries                 |
| `pending_handoff`           | Handoff requested, awaiting business-user acceptance      | Bot (guarded)                            |
| `manual_active`             | `bot_enabled=false`                                      | Human                                    |
| `manual_recovery_candidate` | `bot_enabled=false`, system healthy for stability window | Human until explicit/automatic re-enable |


### 3.1 Transition rules

1. `auto_active -> auto_degraded` when transient failures exceed warning threshold.
2. `auto_degraded -> pending_handoff` when timeout-to-human threshold is reached.
3. `pending_handoff -> manual_active` only after the business user accepts manual mode.
4. `manual_active -> manual_recovery_candidate` when health is restored and stable.
5. `manual_recovery_candidate -> auto_active` only if:
  - the business user explicitly re-enables the bot, and
  - no unresolved critical errors remain, and
  - no outage flag is active for the channel.

---

## 4. Queue and Retry Strategy

### 4.1 Queue contracts

At minimum, keep separate queue categories:

- `inbound_processing`: classify/route inbound messages
- `bot_generation`: model inference and response generation
- `outbound_delivery`: provider send operations

Each job includes:

- `job_id` (UUID)
- `conversation_id`
- `dedupe_key` (idempotency key)
- `attempt`
- `max_attempts`
- `next_retry_at`

### 4.2 Retry policy

Use bounded exponential backoff with jitter.

- Initial delay: 2s
- Backoff factor: x2
- Max delay cap: 2m
- Max attempts:
  - Generation failures: 3
  - Provider send failures: 5

### 4.3 Non-retryable errors

Immediately fail and mark `manual_mode_reason=policy_block` or equivalent for:

- Invalid destination or hard provider validation failures
- Safety/policy hard blocks
- Authentication/configuration errors requiring operator action

### 4.4 Idempotency requirements

Retries must not duplicate user-visible sends.

- Every outbound send uses a stable `dedupe_key` passed through provider metadata when possible.
- Before send, check existing delivery record by `dedupe_key`.
- If record exists in terminal success state, drop retry as already delivered.

---

## 5. Timeout-to-Human

Timeout-to-human is the automatic fallback that protects customer experience when automation cannot produce a reliable response in time.

### 5.1 Trigger conditions

Escalate to `pending_handoff` when one of the following is true:

- Automation has been unavailable for 30 minutes continuously (critical threshold)
- Consecutive automation failures exceed threshold (example: 3 within the 30-minute window)
- Provider outage signal is active for the channel

### 5.2 Escalation actions (atomic)

1. Lock conversation row.
2. Set state to `pending_handoff`.
3. Set `manual_mode_reason=timeout_to_human` (or outage reason).
4. Persist handoff-required event in timeline.
5. Notify operator queue/UI and business owner contact channels.

If any step fails, retry handoff transaction until committed exactly once.

### 5.3 Acceptance flow for manual mode

Manual mode is activated only after the business user accepts the handoff request.

1. Send handoff request to business owner via WhatsApp and email.
2. If accepted, set `bot_enabled=false`, set state `manual_active`, and stamp `human_taken_over_at`.
3. While waiting for acceptance, remain in `pending_handoff`; bot may continue only in guarded mode if healthy.
4. Once in `manual_active`, bot stays disabled until the business user explicitly re-enables it.

### 5.4 Customer-facing behavior

When handoff is triggered:

- Do not send speculative bot replies after switch.
- Optionally send a single fallback acknowledgment template.
- Route subsequent inbound messages directly to operator inbox.

---

## 6. Operator UX States

Operator UI should expose explicit state chips and actions.


| UX state                  | Backend condition           | Operator actions                         |
| ------------------------- | --------------------------- | ---------------------------------------- |
| `Bot Active`              | `auto_active`               | Disable bot, monitor thread              |
| `Bot Degraded`            | `auto_degraded`             | Force manual mode, retry failed step     |
| `Handing Off`             | `pending_handoff`           | Wait (read-only), view progress          |
| `Manual`                  | `manual_active`             | Reply as human, optionally re-enable bot |
| `Manual (Recovery Ready)` | `manual_recovery_candidate` | Re-enable bot now, keep manual mode      |


### 6.1 Manual override rules

- Operator "Disable bot" is immediate and authoritative.
- Operator can request "Enable bot", but activation requires business-user approval.
- Business user "Enable bot" requires:
  - no unresolved critical errors
  - not in outage global flag
- Client can enable `tenant_global_manual` at any time and it applies to all their conversations.
- Only internal admin can enable `platform_global_manual` for all clients.
- If both global modes are active, platform mode takes precedence operationally.

### 6.2 Business user contact requirements

- Business WhatsApp number is mandatory and stored as private contact for escalation and operational notices.
- Valid email is mandatory for login and is used as secondary escalation channel.
- Manual-mode acceptance prompts are sent to both channels.

### 6.3 Guardrails in UI

- Prevent double-send while provider delivery is in-flight.
- Show last failure reason and retry countdown.
- Show whether recovery is blocked by policy or outage.
- Show takeover owner and whether manual mode is awaiting acceptance.
- Show manual mode scope badge: `Conversation`, `Client Global`, or `Platform Global`.

---

## 7. Observability and Alerts

### 7.1 Required metrics

- Handoff rate (`auto -> manual`) per channel
- Mean time to handoff
- Mean time to recovery (`manual -> auto`)
- Retry counts by failure type
- Duplicate-send prevention hit rate

### 7.2 Alerts

- Elevated handoff rate above baseline
- Outbound queue lag above SLA
- Provider error spike with same root cause
- Conversations stuck in `pending_handoff` without business response beyond 30 minutes

### 7.3 Audit trail

Every ownership transition and manual override must be auditable with:

- actor (`system` or `operator_id`)
- previous state
- new state
- reason
- timestamp

---

## 8. Acceptance Criteria

Implementation is complete when:

1. No duplicate outbound messages are observed under retry load testing.
2. Conversations enter `pending_handoff` after 30 minutes of continuous automation unavailability.
3. Operator can always see and change ownership state with clear reason codes.
4. Transition to `manual_active` requires business-user acceptance.
5. Once manual is active, only explicit business-user action can re-enable bot.
6. All transitions are visible in timeline/audit logs.
7. `tenant_global_manual` disables bot execution for all conversations of that client.
8. `platform_global_manual` disables bot execution for all clients.
9. Precedence is enforced exactly as defined by `effective_bot_enabled`.

---

## 9. Resolved Decisions

- Business communication channels for operational events are WhatsApp (primary) and email (secondary).
- Business WhatsApp private phone is required in account data model.
- Valid email is required for login and escalation.
- Critical automation SLA threshold is 30 minutes of continuous unavailability.
- Entering `manual_active` requires explicit business-user acceptance.
- Returning to bot mode requires explicit business-user re-enable action.
- Client-global manual mode is supported through `tenant_global_manual`.
- Platform-global manual mode is supported through `platform_global_manual` (admin-only).

