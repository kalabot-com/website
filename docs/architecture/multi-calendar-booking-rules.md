# Multi-Calendar Booking Rules

## Purpose

Define deterministic rules for creating appointment slots and validating bookings when a business connects multiple Google calendars.

The objective is to:

- maximize availability visibility;
- prevent double-bookings across all connected calendars;
- keep behavior predictable for users and support;
- remain resilient when one provider is degraded.

## Main Product Rules

1. A client can connect from 1 to many calendars.
2. Every calendar belongs to exactly one employee.
3. We only support Google Calendar.
4. All connected calendars belong to the same main Google account.

### Clarifications

- Example: a hair salon with 3 employees requires 3 calendars (one per employee).
- Employee availability is computed from that employee's assigned calendar only, unless explicit cross-employee blocking rules are introduced later.
- Any non-Google calendar is out of scope for this version.

## Hair Salon Example (3 Employees)

### Setup

- Business: hair salon
- Employees: `Alice`, `Bruno`, `Carla`
- Connected calendars (Google, same main account):
  - `calendar_alice@group.calendar.google.com` -> `Alice`
  - `calendar_bruno@group.calendar.google.com` -> `Bruno`
  - `calendar_carla@group.calendar.google.com` -> `Carla`

Rule: bookings for an employee are written only to that employee's mapped calendar.

### Employee Metadata Required

Every employee must include:

- `employeeId`
- `displayName`
- `calendarId` (Google calendar external ID)
- `active` flag
- `offeredServiceIds` (optional but recommended for filtering)

Service assignment rules:

- If `offeredServiceIds` is set, the employee is eligible only for those services.
- If `offeredServiceIds` is empty, employee can perform all services.
- At least one eligible employee must exist for a service, otherwise no slots are returned.

### Schedules

Each employee has an independent schedule that can differ from others.

Examples:

- `Alice`: full day (`09:00-18:00`)
- `Bruno`: mornings only (`09:00-13:00`)
- `Carla`: afternoons only (`13:00-18:00`)

Schedule model requirements:

- Base weekly schedule per employee (day-of-week working windows)
- Date-level overrides per employee (vacation, exceptions, temporary shifts)
- Weekly changes are supported by updating future overrides or replacing the base schedule with an effective date

Slot generation rule:

- A slot exists only if an employee is simultaneously:
  - scheduled as working at that time;
  - not busy in their own Google calendar;
  - eligible for the requested service.

## Scope

This document defines booking-time rules only:

- calendar and employee assignment;
- busy/free aggregation;
- slot generation constraints;
- conflict checks at reservation and confirmation time;
- fallback behavior when one or more calendars are unavailable.

Out of scope:

- external provider OAuth/provisioning details;
- UI microcopy and visual design;
- reminder/notification rules.

## Terms

- **Employee calendar**: the Google calendar assigned to a specific employee, used for availability checks and booking writes.
- **Availability source**: working hours, overrides, buffers, and service duration constraints.
- **Hold**: temporary reservation with TTL before final confirmation.
- **Conflict**: any overlap with unavailable time according to precedence rules below.

## Calendar Assignment Rules

Each connected Google calendar must be assigned to one employee:

1. Employee mapping is mandatory (`calendarId` -> `employeeId`)
2. Disabled calendars are excluded from booking computations

### Assignment rules

- If an employee has no active calendar, they are not bookable.
- If no mappable employee calendars are active, public booking is disabled with a configuration error.
- Disabled calendars are kept connected but excluded from slot and conflict logic.
- A connected calendar maps to exactly one employee record.
- An employee can have only one active calendar for booking in this version.

## Normalization Rules

All provider events must be normalized into a unified interval model before processing.

Required normalized fields:

- `startAtUtc`, `endAtUtc`
- `sourceCalendarId`
- `transparency` (`OPAQUE` blocks time, `TRANSPARENT` does not)
- `status` (cancelled events are ignored)
- `allDay` flag
- `updatedAt`

Normalization constraints:

- Store and compute in UTC.
- Convert all-day events using the calendar's timezone boundaries.
- Ignore cancelled events.
- Treat unknown transparency as blocking by default (fail-safe).

## Availability Composition

Candidate slots are built per employee from this pipeline:

1. Start from employee base weekly working hours (business timezone).
2. Apply employee date overrides (closed dates, custom windows).
3. Subtract breaks/unavailability windows.
4. Subtract busy intervals from the employee's mapped Google calendar.
5. Apply pre/post buffers around busy intervals and around generated appointments.
6. Enforce service duration and slot interval granularity.
7. Drop slots that violate min notice / max advance booking window.
8. Keep only slots from employees eligible for the requested service.

Then aggregate valid employee slots into the customer-facing availability list.

## Busy Aggregation Rules

- Busy intervals are computed per employee calendar (not unioned across all employees).
- Overlapping or touching intervals are merged per employee before subtraction.
- Transparent events do not block.
- Tentative events are configurable:
  - default: block (safer to avoid collisions);
  - optional mode: do not block.
- All-day events block the entire local day after timezone conversion.

## Conflict Detection Rules

Conflict checks happen twice:

1. **At hold creation**: reject if requested interval overlaps computed busy state.
2. **At confirmation**: re-fetch fresh calendar deltas and re-check overlap before writing event.

Conflict outcome:

- If conflict exists, confirmation fails with `SLOT_NO_LONGER_AVAILABLE`.
- If no conflict, write to the selected employee calendar, then mark booking confirmed.

## Employee Selection Rules (No Employee Preference)

When client does not request a specific employee:

1. Filter employees by service eligibility.
2. Filter by schedule validity at requested time.
3. Filter by calendar availability at requested time.
4. If multiple employees remain, apply deterministic tie-break:
   - lowest active load in the selected day (least assigned bookings first);
   - if tied, earliest next availability;
   - if still tied, stable order by `employeeId`.

If no rule preference is configured by tenant, this default policy applies.

When client explicitly requests an employee:

- Only that employee is evaluated.
- If unavailable, return no slot for that exact request (no silent fallback).

## Precedence and Determinism

When rules disagree, precedence order is:

1. Hard closures and manual unavailability
2. Existing busy events (all calendars in scope)
3. Buffer rules
4. Service duration/granularity constraints
5. Display/order preferences

Any hard constraint violation means the slot is invalid.

## Timezone and DST Rules

- Source of truth for customer-facing slots is business/location timezone.
- Internal storage and cross-provider comparisons are UTC.
- DST transitions:
  - nonexistent local times (spring forward) are never offered;
  - duplicated local times (fall back) are disambiguated by UTC instant;
  - slot IDs must be derived from UTC timestamps, not local clock strings.

## Write Rules for Confirmed Bookings

- Confirmed appointments are written only to the selected employee calendar.
- Event write idempotency key: `bookingId` (or deterministic external UID).
- If provider write succeeds but local transaction is uncertain, reconciliation job must upsert by idempotency key.
- If provider write fails, booking remains unconfirmed and hold is released/expired.

## Degraded and Failure Modes

### Single employee calendar unavailable

- Default policy: fail closed for affected profile (do not offer new slots).
- Optional policy flag: fail open with warning if stale cache is within allowed freshness window.

### Primary calendar unavailable

- Do not confirm new bookings.
- Existing bookings remain visible from local DB.
- Retry queue handles delayed writes only for bookings still in `PENDING_CONFIRMATION`.

### Provider rate limits

- Use cached busy snapshots with freshness TTL for slot browsing.
- Force live re-check at confirmation whenever possible.

## Caching and Freshness

- Busy snapshot cache key includes profile + date range + employee-calendar config version.
- Recommended browse freshness: 30-120 seconds.
- Confirmation step must bypass stale cache or require freshness under strict threshold.
- Any employee-calendar assignment/config change invalidates affected cache entries immediately.

## Data Model Requirements

Minimum entities/fields required:

- `ConnectedCalendar`:
  - `id`, `ownerId`, `provider` (`GOOGLE` only), `externalCalendarId`, `timezone`, `status`, `employeeId`, `googleAccountId`
- `Employee`:
  - `id`, `displayName`, `active`, `offeredServiceIds`
- `EmployeeSchedule`:
  - `employeeId`, `weeklyWindows`, `dateOverrides`, `effectiveFrom`
- `BookingProfile`:
  - working hours, overrides, buffers, min/max notice, service durations
- `BookingHold`:
  - `startAtUtc`, `endAtUtc`, `expiresAt`, `status`, `employeeId`, `calendarId`
- `Booking`:
  - `startAtUtc`, `endAtUtc`, `status`, `employeeId`, `calendarId`, `calendarEventId`, `idempotencyKey`

## API Behavior (Contract-Level)

- Slot API returns only currently valid slots for the requested service/profile/date range.
- Hold API may return `SLOT_NO_LONGER_AVAILABLE` if race detected.
- Confirm API must be idempotent by client request key and booking/hold identity.
- Conflict and provider errors should map to stable, user-safe error codes.

## Observability

Track metrics:

- slot generation latency
- hold-to-confirm conversion
- conflict rate at confirmation
- provider API error rate by provider
- stale-cache usage rate

Structured logs should include:

- booking/profile IDs
- employee-calendar assignment hash/version
- date range queried
- conflict reason category

## Testing Matrix

Must-have automated tests:

- multiple employee calendars with overlapping busy events
- transparent vs opaque events
- all-day events across timezone boundaries
- DST forward/backward boundary days
- race condition between slot view and confirmation
- provider timeout on browse vs confirmation
- primary write idempotency/retry behavior

## Default Product Policy

Unless explicitly overridden per tenant:

- one active Google calendar per active employee
- tentative events block time
- degraded reads fail closed
- confirmation always performs live re-check
- provider is always Google Calendar
- all calendars must share the same `googleAccountId`
- if no employee is requested, select by lowest load, then earliest next availability, then `employeeId`

## Future Extensions

- weighted availability (soft conflicts)
- multi-resource booking (staff + room + equipment)
- split writes (write to multiple destination calendars)
- tenant-specific policy packs
