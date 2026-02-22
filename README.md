# website

## 1. Problem
- Small businesses (salons, restaurants, clinics, service providers, professionals) waste time answering calls/WhatsApp manually for **bookings, inquiries, and service requests**.
- Existing solutions are either:
  - Too **complex** (require Meta/Facebook onboarding, technical setup).
  - Too **generic** (multichannel CRMs not designed for SMB needs).
- Result: lost leads, missed bookings, overworked staff.


## 2. Solution
- **Frictionless WhatsApp automation platform**: pay → get a WhatsApp Business number → start.
- Industry-specific **templates**:
  - Booking flow (Google Calendar sync).
  - Lead qualification (lawyers, architects, consultants).
  - Service dispatch (plumbers, cleaners).
- **Dashboard**: see all conversations + bookings/leads/jobs in one place.
- **Top support**: onboarding + live help, tuned for non-technical SMB owners.

### BOOKING PRODUCT
We will start with this solution
- ONBOARDING:
  - Registro: Email
  - Vinculación con Google Calendar
  - Configuración Asistida: El profesional sube una foto de su lista de precios actual. La IA (aprovechando la visión de Kimi k2.5) extrae los servicios, precios y duraciones automáticamente.
  - Credit card --> onboarding charge
  - Asignación de Número: Nosotros proporcionamos un número +34 nuevo.
  - QR link
2. Objetivos PrincipalesA. Experiencia del Cliente Final (Usuario de WhatsApp)Naturalidad: El cliente debe sentir que habla con un asistente humano muy eficiente. Nada de "Pulsa 1".Omnipresencia: Capacidad de agendar 24/7 sin que el profesional tenga que tocar el móvil.Reducción del "No-Show": Sistema automático de recordatorios y confirmaciones con lenguaje persuasivo.B. Experiencia del Profesional (Peluquero/Fisio)Onboarding Instantáneo: Integración con Google Calendar en dos clics y conexión de un número de WhatsApp dedicado (Opción A de nuestra charla anterior) para separar vida personal de profesional.Cero Mantenimiento: El profesional solo mira su calendario. Si necesita mover una cita, lo hace en Google Calendar y el bot se encarga de avisar al cliente y renegociar el hueco.Control por Voz/Chat: El profesional puede "hablarle" a su propio bot para bloquear horas o añadir citas manuales mientras trabaja.3. Arquitectura Técnica y Stack (Propuesta 2026)3.1. Capa de Conectividad (El "Cuerpo")Motor de WhatsApp: Utilizaremos Baileys. Al ser una librería de ingeniería inversa del protocolo de WhatsApp Web, nos permite una densidad de instancias por servidor mucho mayor que Puppeteer.Aislamiento de Sesiones: Cada cliente tendrá un contenedor ligero o un proceso aislado que gestione su Auth State de WhatsApp, permitiendo que si una sesión cae, no afecte al resto de la red.3.2. Capa de Inteligencia (El "Cerebro")Orquestador: Implementaremos un Agent Swarm (Enjambre de Agentes) usando el SDK de Kimi k2.5 por su eficiencia en costes.Agente de Triaje: Analiza el mensaje entrante. ¿Es una cita nueva? ¿Es una cancelación? ¿Es una duda sobre el precio?Agente de Calendario: Traduce el lenguaje natural a consultas de la API de Google (ej: "martes tarde" -> timeMin: 2026-02-24T16:00:00Z).Agente de Cierre: Redacta la respuesta final buscando la confirmación explícita.3.3. Integración de CalendarioGoogle Calendar API: Uso intensivo de freeBusy para comprobar disponibilidad en tiempo real.Lógica de Buffers: El sistema añadirá automáticamente "tiempos de limpieza" entre citas (configurables por el profesional) para evitar el solapamiento.4. Definición de Servicios y Lógica de NegocioEl sistema debe soportar una matriz de servicios compleja que el profesional configurará en su panel de administración (diseñado por Isabel):ServicioDuración EstimadaRequisitos de IACorte rápido20 minConfirmación inmediata.Tinte y Peinado120 minRequiere verificar huecos largos; no ofrecer si falta poco para el cierre.Fisio (Sesión)60 minPreguntar si es la primera vez (requiere más tiempo de ficha).

## 3. Target Market
- Initial geography: **Spain** (WhatsApp penetration ~90%, but only ~27% SMB adoption).
- Expansion: **Europe, LATAM, India, Brazil** (similar SMB + WhatsApp-heavy markets).
- Segments:
  - Booking-based: salons, clinics, restaurants.
  - Lead-based: lawyers, architects, consultants.
  - Dispatch-based: plumbers, electricians, cleaners.


## 4. Unique Value Proposition
- “Your **24/7 receptionist** on WhatsApp — set up instantly, without tech headaches.”
- Unlike competitors:
  - **Not multichannel bloat** (focus only on WhatsApp).
  - **Instant onboarding** (we provision number + templates for them).
  - **Vertical templates** (ready-to-use, no flow building needed).


## 5. Competitors
- **360dialog** – frictionless WABA, but developer-focused, no SMB dashboard.
- **Tidio / Respond.io** – good UI, but multichannel, not WhatsApp-first, limited SMB flows.
- **WATI, Callbell, Gupshup** – offer inboxes, but lack easy onboarding + SMB simplicity.
- **Opportunity**: combine frictionless onboarding + templates + strong UX.


## 6. Business Model (PENDING)
- **Pricing**:
  - Setup fee: €49–€99 (covers number provisioning, onboarding).
  - Monthly subscription:
    - Starter (€29): 100 conv./month, 1 template.
    - Pro (€59): 500 conv./month, booking + calendar integration.
    - Premium (€99+): unlimited conv., multiple templates, advanced support.
- Costs: Meta conversation fees (passed through or included in plan).
- Upsells: multi-location, integrations (Stripe, CRM), white-label for agencies.


## 7. Go-to-Market (PENDING)
- **Pilot clients**: personal network (3 salons, 1 restaurant, 1 architect).
- **Acquisition**:
  - Direct outreach in Spain SMB verticals.
  - Local digital agencies → resell under their brand.
  - WhatsApp Ads (“Click to WhatsApp”) for lead generation.
- **Expansion**: focus on WhatsApp-heavy markets (Spain → LATAM → EU/India).

## 8. Risks & Mitigation (PENDING)
- **Onboarding friction (Meta/Facebook requirements)** → Use your-owned WABA for MVP, migrate big clients later.
- **Compliance (spam, misuse)** → Strict opt-in, monitor quality scores.
- **Competition from bigger players** → Differentiate with SMB simplicity + support.

## 8. Risks & Mitigation (PENDING)

