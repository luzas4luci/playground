# Briefing · Implementación PreZero · Cadencia Líderes

> Este documento es el punto de partida para implementar el sistema de líderes de PreZero.
> Contiene todo lo que Claude necesita saber para ayudarte paso a paso.

---

## Qué estamos construyendo

Un sistema de dos capas para gestionar líderes de warmup en centros PreZero:

1. **Capa de fases** — clasificar a cada líder en L0/L1/L2/L3 según su comportamiento
2. **Capa de comunicaciones** — enviar los mensajes correctos según la fase del líder

Estas capas van en orden. Primero las fases, luego las comunicaciones encima.

---

## Referencia del producto

Tenemos dos documentos de referencia en este repositorio:
- `playbook-cadencia-prezero.md` — todos los mensajes, lógica y timing de comunicaciones
- `briefing-implementacion-prezero.md` — este fichero, el plan de implementación

Hay también una repo de referencia `fisify-eyes` que tiene implementado el mismo patrón para el proyecto Melia. Antes de escribir código nuevo, leer cómo está hecho allí.

---

## Contexto del negocio

- **PreZero**: empresa de gestión de residuos industrial con 12 centros operativos
- **Líder de warmup**: empleado designado por Alex (responsable) para organizar warmups grupales diarios en su centro
- **Warmup grupal**: sesión de calentamiento que el líder organiza para su equipo
- **Diana**: objetivo semanal de días con warmup según el tier del centro
- **Alex**: responsable que designa líderes, activa sub-estados manualmente, hace revisiones trimestrales
- **Lorena**: Customer Success, voz humana en WhatsApp
- **Aurya**: asistente IA dentro de la app, voz del chat in-app

---

## BLOQUE 1 · Fases (empezamos aquí)

Las fases determinan en qué estado está cada líder. Son propiedades del usuario en PostHog que hay que mantener actualizadas.

### Las 4 fases

```
L0 · No activo      → designado pero aún no ha organizado ningún warmup grupal
L1 · Activo         → organiza warmups con regularidad
L2 · En racha       → 8+ semanas consecutivas cumpliendo diana
L3 · En riesgo      → lleva semanas sin cumplir o sin reportar
```

### Cómo se determina cada fase

**L0 — se activa cuando:**
- `isLeader: true` en el perfil del usuario
- `groupWarmupCount === 0` (nunca ha reportado un warmup grupal)

Sub-estados:
- `L0a` · recién nominado, seguimiento activo 21 días
- `L0b` · intentado, 21 días sin warmup, sin seguimiento hasta revisión trimestral

**L1 — se activa cuando:**
- `isLeader: true`
- Ha reportado al menos 1 warmup grupal
- No cumple criterios de L2 ni L3

Sub-estados (cuadrante 2×2, modula el tono de los mensajes, no la fase):
- `L1a · Líder Ejemplar` — warmup ✅ + rehab plan activo con Alex ✅
- `L1b · Líder Activo` — warmup ✅ + sin rehab plan ❌

**L2 — se activa cuando:**
- `isLeader: true`
- `consecutiveWeeksInDiana >= 8`

Sub-estados:
- `L2 base` · mantiene su rutina (por defecto)
- `L2 activo` · movilizador, Alex lo activa manualmente (`isActiveMobilizer: true`)

**L3 — se activa cuando:**
- `isLeader: true`
- `consecutiveWeeksBelowUmbral >= 3` O `weeksWithoutReport >= 2`

Sub-estados (se asignan al entrar y no cambian):
- `L3a · intervención suave` — tiene rehab plan activo con Alex al entrar
- `L3b · intervención directa` — sin rehab plan al entrar
- `L3c · sin retoma` — ventana de rescate agotada → vuelve a L0b

### Tiers y dianas (fijos, no varían por centro)

| Tier | Diana | Umbral crítico |
|------|-------|----------------|
| Mantener | 7/7 días/semana | <5/7 |
| Cerrar gap | ≥6/7 días/semana | <4/7 |
| Recuperar | ≥5/7 días/semana | <3/7 |

### Campos necesarios en el perfil del líder

```typescript
// Identidad
isLeader: boolean
nominatedAt: Date
tier: 'mantener' | 'cerrar-gap' | 'recuperar'
leaderPhase: 'L0a' | 'L0b' | 'L1a' | 'L1b' | 'L2' | 'L3a' | 'L3b' | 'L3c'
isActiveMobilizer: boolean  // solo L2, Alex activa manualmente

// Warmup tracking
warmup.groupLastReportedAt: Date
warmup.groupWeeklyDays: number        // días con warmup grupal esta semana
warmup.consecutiveWeeksInDiana: number
warmup.consecutiveWeeksBelowDiana: number
warmup.consecutiveWeeksBelowUmbral: number
warmup.weeksWithoutReport: number
warmup.totalGroupCount: number

// Rehab plan
hasRehabPlan: boolean    // plan personal creado por Alex en FisifyStudio

// Comunicaciones
chatInApp.lastAuryaMessageAt: Date
chatInApp.lastReadAt: Date
pushEnabled: boolean
hasWhatsApp: boolean
```

### Lógica de transición entre fases

```
Cada semana (cron semanal):

L0a:
  Si reporta warmup grupal → L1
  Si día 21+ sin warmup → L0b

L1:
  Si consecutiveWeeksInDiana >= 8 → L2
  Si consecutiveWeeksBelowUmbral >= 3 → L3 (L3a si hasRehabPlan, L3b si no)
  Si weeksWithoutReport >= 2 → L3 (L3a si hasRehabPlan, L3b si no)

L2:
  Si consecutiveWeeksBelowDiana >= 3 → L1 (nunca cae directo a L3)

L3a / L3b:
  Si reporta warmup grupal → L1
  Si ventana agotada (L3a: 14d, L3b: 21d) sin retoma → L3c → L0b

L3c:
  Sin seguimiento hasta revisión trimestral de Alex
```

---

## BLOQUE 2 · Comunicaciones (después de las fases)

Una vez las fases están funcionando, añadimos la capa de comunicaciones.

El patrón es el mismo que en Melia: **resolvers**. Funciones puras que reciben el estado del líder y devuelven qué comunicación hacer ahora.

```typescript
// Patrón del resolver
input:  { datos del líder, fechas, acciones completadas }
output: { kind: 'pending' | 'waiting' | 'complete', stage, channel, message, requiresManual }
```

- `pending` → hay algo que enviar ahora
- `waiting` → aún no toca
- `complete` → esta fase está completa

Las acciones manuales (WhatsApp, push manual) se marcan con:
```typescript
CadenceManualAction {
  stage: string
  channel: string
  actionKey: '${stage}:${channel}'
  status: 'done'
  completedAt: Date
}
```

### Orden de implementación de resolvers

1. **`l0aCadence.ts`** — el más lineal, 5 stages en 21 días. Empieza aquí.
2. **`l1Cadence.ts`** — Track A (onboarding único) + Track B (semanal recurrente)
3. **`l2Cadence.ts`** — seguimiento silencioso semanal + reconocimiento quincenal
4. **`l3Cadence.ts`** — L3a (14d) y L3b (21d) con escalada entre sub-estados

Todos los mensajes, variantes y timing están en `playbook-cadencia-prezero.md`.

### Canales y prioridad

| Canal | Quién | Completa |
|-------|-------|----------|
| Chat in-app | Aurya (IA) | Automático (mensaje detectado) |
| Push | Sistema | Manual (marcar enviado) |
| WhatsApp | Lorena (CS humana) | Manual (marcar enviado) |

Prioridad cuando hay árbol de canal: chat in-app → push → WhatsApp

---

## Orden de implementación completo

```
Semana 1:
  [ ] Leer resolver de Melia en fisify-eyes — entender el patrón
  [ ] Definir schema de campos en PostHog/base de datos
  [ ] Construir lógica de clasificación de fases (Bloque 1)
  [ ] Construir cron semanal de transiciones

Semana 2:
  [ ] Resolver L0a + mensajes
  [ ] Conectar L0a con Customer.io / sistema de envío

Semana 3:
  [ ] Resolver L1 (Track A + Track B)
  [ ] Resolver L2

Semana 4:
  [ ] Resolver L3a + L3b
  [ ] Tests de escenarios completos (líder que sube, líder que cae, líder rescatado)
```

---

## Lo que NO hay que construir en V1

- In-app especial para transición L1→L2 (modal/badge) — pendiente con Inhar
- Read tracking (leído/no leído) — pendiente de implementación técnica
- Criterio automático para L2 activo — Alex lo activa manualmente por ahora
- Email mensual de resultados — puede esperar a V2

---

## Cómo trabajar conmigo en Cursor

Soy nueva en esto. Necesito que:
1. Me expliques qué hace cada fichero que leas antes de modificarlo
2. Vayas fase a fase — no empecemos L1 hasta que L0 esté completo y lo entienda
3. Cuando crees un fichero nuevo, explícame su estructura antes de rellenarlo
4. Si hay decisiones de diseño, pregúntame antes de elegir

