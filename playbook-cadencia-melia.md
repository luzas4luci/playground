# Playbook de cadencia Melia · Fisify
> Extraído del código fuente. Base para replicar en PreZero líderes.

---

## Mapa de fases y transiciones

```
Registro
  └─ Fase 0 · Sin contacto
       ├─ → Fase 1  cuando el paciente responde por chat, tiene cita o no-show
       └─ → "Sin interés"  cuando >30 días lab. desde primer chat Y aviso final hecho

Fase 1 · Contacto iniciado
  ├─ → Fase 2  cuando hay contacto clínico (no chat) o cita confirmada ya pasada
  └─ (bucle interno con seguimientos semanales y estados de cita)

Fase 2 · Esperando programa
  └─ → Fase 3  cuando se asigna plan de rehabilitación

Fase 3 · Programa sin empezar   (0 sesiones)
  └─ → Fase 4  cuando se registra la primera sesión

Fase 4 · Realizando sesiones   (1–9 sesiones)
  └─ → Fase 5  cuando se completan 10 sesiones

Fase 5 · Pendiente revalorización
  └─ → Fase 6  cuando planUpdatedAt > lastSessionAt (renovación de programa)

Fase 6 · Programa renovado · Fidelización
  (cadencia idéntica a Fase 4, ancla distinta)

Abandonados  ← manual desde cualquier fase
  └─ recontacto a los 2 meses
```

**Clasificación en código** (`classifyPhase` + `classifyColumn`):
- Sin plan + sin contacto + sin cita → Fase 0
- Sin plan + chat/cita/no-show → Fase 1
- Sin plan + contacto clínico (no-chat) o cita confirmada pasada → Fase 2
- Con plan + 0 sesiones → Fase 3
- Con plan + 1–9 sesiones → Fase 4
- Con plan + 10 sesiones, sin renovación → Fase 5
- Con plan + renovado (planUpdatedAt > lastSessionAt) → Fase 6

---

## Fase 0 · Sin contacto

**Objetivo:** Conseguir el primer contacto efectivo del fisio (Alex).  
**Tiempo:** Días laborables desde el registro del paciente.  
**Ancla:** Registro → primer chat del fisio.

### Paso 0 — Primer chat (día lab. 1–2)

| Campo | Valor |
|-------|-------|
| Cuándo | A partir del día lab. 1 desde registro |
| Canal | Chat (app) |
| Quién | Alex |
| Cómo completa | **Automático**: se detecta el primer mensaje del fisio en el chat |
| Urgencia | Día lab. 1: aviso · >2 días: crítico |

**Texto:**
> Hola {Nombre}, soy Alex, tu fisio de {Empresa}. He visto tu registro en la app y quería preguntarte cómo estás ahora para valorar tu caso y ayudarte a empezar cuanto antes.

*Antes del día lab. 1: estado `waiting`, sin acción.*

---

### Paso 1 — Push de activación (día lab. 3)

| Campo | Valor |
|-------|-------|
| Cuándo | Día lab. 3 desde el primer chat |
| Canal | Push |
| Quién | Aurya |
| Condición | Solo si tiene notificaciones activas |
| Cómo completa | **Manual** (marcar enviado) |
| Sin push | **No bloqueante** — se marca automáticamente y pasa al siguiente |

**Texto push:**
> {Nombre}, Alex empieza a pensar que le estás ignorando ¿Le contestas y le quitamos el drama?

*Este paso bloquea el día 7 hasta completarse o no aplicar.*

---

### Paso 2 — Aviso 2 (día lab. 7)

**Árbol de decisión (canal):**

```
¿Leyó el primer chat?
├── SÍ → Chat (Alex)  ← automático por 2.º mensaje del fisio
│         "¿Te pillo en mal momento o es que no sabías muy bien qué contestarme?"
│
└── NO
    ├── ¿Tiene push activo?
    │   └── SÍ → Push (Aurya)  ← MANUAL
    │             "{Nombre}, Alex empieza a pensar que le estás ignorando
    │              ¿Le contestas y le quitamos el drama?"
    │
    ├── ¿Tiene WhatsApp?
    │   └── SÍ → WhatsApp  ← MANUAL
    │             "Hola {Nombre}, Soy Alex, tu fisio de {Empresa},
    │              te mandé un mensaje por la App pero como seguramente no
    │              te estén llegando te escribo por aquí. Te decía que
    │              teniendo en cuenta tu caso con un dolor de {dolor}/10,
    │              merece la pena valorarte bien y empezar lo más pronto
    │              posible. ¿Cómo estás ahora?"
    │
    └── Ninguno → Email  ← MANUAL
                  "Alex está esperando tu respuesta."
```

---

### Paso 3 — Aviso 3 (día lab. 14)

**Árbol de decisión (canal):**

```
¿Tiene push activo?
├── SÍ → Push (Aurya)  ← MANUAL
│         "Hola {Nombre}, te recuerdo que todavía tienes pendiente
│          responder a Alex. ¿Cogemos un minuto hoy para ello?"
│
└── NO
    ├── ¿Tiene WhatsApp?
    │   └── SÍ → WhatsApp  ← MANUAL
    │             "¿Te pillo en mal momento o es que no sabías muy bien
    │              qué contestarme?"
    │
    └── NO → Chat (Alex)  ← automático por 3.er mensaje
              Si aviso 2 fue leído:
                "¿Te pillo en mal momento o es que no sabías muy bien
                 qué contestarme?"
              Si aviso 2 NO fue leído:
                "Te escribo de nuevo para saber si te viene bien que
                 valoremos tu caso y empecemos cuanto antes."
```

---

### Paso 4 — Aviso final / cierre (día lab. 20)

**Canal:** Siempre MANUAL. Requiere marcar enviado.

```
¿Tiene WhatsApp?
├── SÍ → WhatsApp + Email  ← MANUAL
│
└── NO → Chat + Email  ← MANUAL
```

**Mensaje WhatsApp (con dolor):**
> {Nombre}, como no he tenido respuesta tuya estos días, voy a pausar tu servicio por ahora, ya que prefiero no estar mandándote mensajes si no es buen momento para ti. Que sepas que si en cualquier momento empeora tu dolor o quieres empezar a tratarlo, me escribes por el chat y lo retomamos al instante. Un abrazo!

**Mensaje WhatsApp (sin dolor):**
> {Nombre}, como no he tenido respuesta tuya estos días, voy a pausar tu servicio por ahora. Que sepas que si en cualquier momento quieres empezar a cuidar tu salud o tienes alguna lesión o dolor, me escribes por el chat y lo retomamos al instante. Te escribo dentro de unos meses por si entonces sí te encaja. Un abrazo!

**Mensaje Chat (con dolor):**
> {Nombre}, veo que no tienes interés en este servicio ahora mismo y no pasa nada. Te daremos de baja en unos días pero si en cualquier momento empeora tu dolor o quieres empezar a tratarlo, me escribes por el chat y podrás volver a empezar con el servicio.

**Mensaje Chat (sin dolor):**
> {Nombre}, veo que no tienes interés en este servicio ahora mismo y no pasa nada. Te daremos de baja en unos días pero si en cualquier momento quieres volver o empiezas con cualquier dolor, me escribes por el chat y podrás volver a empezar con el servicio.

**Email (siempre, igual con o sin WhatsApp):**
> Hola {Nombre}, Desde Fisify queremos respetar tu tiempo, así que vamos a pausar tu servicio de fisioterapia por ahora. No queremos llenarte la bandeja si no es el momento. Lo importante: el servicio sigue disponible para ti cuando quieras. Para reactivarlo, basta con un mensaje a Alex por el chat de la app o por WhatsApp y lo retomamos al momento, sin trámites. Te volveremos a escribir en unos meses por si entonces te encaja. Un abrazo, El equipo de Fisify.

---

### Columna "Sin interés"
Condición automática doble:
1. >30 días lab. desde el primer chat del fisio, Y
2. El aviso final (día 20) está completado

---

## Fase 1 · Contacto iniciado

**Objetivo:** Avanzar hacia cita o asignación de programa.  
**Tiempo:** Días de **calendario** (24h, no laborables).  
**Intervalo entre seguimientos:** 7 días calendario.

### Cómo se cuenta el progreso

La función `getWeeklySpacedPhaseOneMessages` toma todos los mensajes del fisio posteriores al primer mensaje del paciente, y filtra para contar solo **uno por cada ventana de 7 días**. Los marcados manualmente (WhatsApp) también cuentan.

### Sub-estados

| Estado | Cuándo | Canal | Urgencia |
|--------|--------|-------|----------|
| `waiting-response` | Último mensaje es del paciente, sin responder | Chat | <2d: aviso / ≥2d: crítico |
| `initial-contact` | Fisio ya respondió (1 mensaje contado) | Chat | — |
| `followup-1` | 7d desde contacto inicial, sin cita | Chat | Aviso |
| `followup-2` | 7d desde seguimiento 1 | Chat | Aviso |
| `followup-3` | 7d desde seguimiento 2 | Chat | Crítico |
| `appointment-requested` | Cita/videollamada solicitada, sin confirmar | — | — |
| `appointment-confirmed` | Cita confirmada, no ha pasado aún | — | — |

### Mensajes de seguimiento

**Waiting-response:**
> Hola {Nombre}, soy Alex. He visto tu mensaje y quería contestarte para entender bien cómo estás y ver el siguiente paso contigo.

**Seguimiento 1:**
> Hola {Nombre}, soy Alex. Te escribo para hacer seguimiento de lo que me contaste y ver si avanzamos con el siguiente paso.

**Seguimiento 2:**
> Hola {Nombre}, soy Alex. Te vuelvo a escribir para saber cómo sigues y si quieres que lo revisemos con una cita o videollamada.

**Seguimiento 3:**
> Hola {Nombre}, soy Alex. Cierro este seguimiento por aquí: si quieres, podemos agendar una cita o videollamada para revisar tu caso bien.

### Reglas de cita
- Mientras hay cita activa (solicitada o confirmada futura) → el paciente **permanece en Fase 1**, no avanza la cadencia.
- Cuando la cita confirmada ya pasó → pasa a **Fase 2**.
- Si cita marcada como "no-show" → vuelve a Fase 1, sub-estado `initial-contact`.

---

## Fase 2 · Esperando programa

**Objetivo:** Asignar plan de rehabilitación lo antes posible.  
**No hay pasos de cadencia** — solo SLA de asignación.

| Días lab. desde contacto | Estado |
|--------------------------|--------|
| < 1 | OK |
| 1–2 | Aviso amarillo — revisar y asignar hoy |
| > 2 | **Crítico** — asignar o reasignar fisio |

---

## Fase 3 · Programa sin empezar

**Objetivo:** Que el paciente complete la primera sesión.  
**⚠️ Timing relativo:** Cada paso se calcula desde la fecha real del paso anterior, no desde el inicio del programa. Si un paso se hace tarde, el siguiente se retrasa proporcionalmente.  
**Ancla inicial:** Primer chat del fisio tras el programa.

### Paso 0 — Chat inicial (día lab. 0)

| Campo | Valor |
|-------|-------|
| Cuándo | Inmediatamente al asignar programa |
| Canal | Chat (app) |
| Quién | Alex |
| Cómo completa | **Automático**: primer chat del fisio tras el programa |

**Texto:**
> {Nombre}, te he dejado tu programa en la app. Revísalo cuando puedas y me dices si algo no te encaja.

---

### Paso 1 — Push recordatorio (día lab. +2)

| Condición | Acción |
|-----------|--------|
| Tiene push | Push Aurya — **MANUAL** |
| Sin push | No bloqueante — se salta |

**Texto push:**
> {Nombre}, tu programa lleva 2 días esperándote 👀 ¿Empezamos hoy?

---

### Paso 2 — Seguimiento sin push (día lab. +3)

*Solo si NO tiene push. Relativo al paso anterior.*

```
¿Leyó el chat del paso 0?
├── SÍ → Chat (Alex)  ← automático
│         "¿Que tal con los ejercicios? ¿Has podido empezar?"
│
└── NO
    ├── ¿Tiene WhatsApp?
    │   └── SÍ → WhatsApp  ← MANUAL
    │             "Hola {Nombre}, soy Alex, tu fisio de {Empresa}.
    │              Te dejé tu programa en la app hace un par de días
    │              pero veo que aún no has empezado.
    │              ¿Si tienes algún problema con la App avísame y te ayudo!"
    │
    └── NO → Chat  ← automático (mismo texto que WhatsApp)
```

---

### Paso 3 — Recordatorio 2 (día lab. +5)

```
¿El paciente ha respondido desde que empezó el bloque?
├── SÍ → Chat personalizado (Alex)  ← automático
│
└── NO
    ├── ¿Tiene push?
    │   └── SÍ → Push (Aurya)  ← MANUAL
    │             "{Nombre}, los ejercicios no se hacen solos
    │              (lo hemos intentado, no funciona 🥲). 10 min y empiezas."
    │
    └── NO → No bloqueante — se salta
```

---

### Paso 4 — Recordatorio 3 (día lab. +7)

```
¿Tiene push?
├── SÍ → Push (Aurya)  ← MANUAL
│         "¿Hoy empezamos? No nos conviene que Alex se enfade 😏"
│
└── NO
    ├── ¿Hay WhatsApp previo sin respuesta del paciente?
    │   └── SÍ → WhatsApp  ← MANUAL
    │             "{Nombre}, ¿como vas? Si el programa no te encaja
    │              por lo que sea (tiempo, ejercicios, formato),
    │              prefiero saberlo y ajustarlo a que se quede ahí parado.
    │              ¿Qué te está costando más?"
    │
    └── NO → No bloqueante — se salta
```

---

### Paso 5 — Dificultad (día lab. +10)

| Condición | Acción |
|-----------|--------|
| Tiene push | Push Alex — **MANUAL** |
| Sin push | No bloqueante |

**Texto:**
> {Nombre}, veo que todavía no hemos podido empezar. Si el programa no te encaja por lo que sea (tiempo, ejercicios, formato), prefiero saberlo y ajustarlo a que se quede ahí parado. ¿Qué te está costando más?

---

### Paso 6 — Rescate (día lab. +14)

```
¿Hay WhatsApp previo sin respuesta?
├── SÍ → WhatsApp  ← MANUAL
│         "Llevo dos semanas sin respuesta {Nombre} 😔, ¿está todo bien?"
│
└── NO
    ├── ¿Tiene push + WhatsApp?
    │   └── SÍ → Push + WhatsApp  ← MANUAL (ambos juntos)
    │             Push: "El programa se está oxidando. ¡Venga que lo más
    │                    difícil es empezar! 😏"
    │             WA:   "Hola {Nombre}, soy Alex, tu fisio de {Empresa}.
    │                    Te dejé tu programa en la app hace un par de semanas
    │                    pero veo que aún no has empezado.
    │                    ¿Si tienes algún problema con la App avísame?"
    │
    ├── ¿Tiene push (sin WhatsApp)?
    │   └── SÍ → Push (Aurya)  ← MANUAL
    │             "El programa se está oxidando. ¡Venga que lo más
    │              difícil es empezar! 😏"
    │
    └── NO → No bloqueante — se salta
```

---

### Paso 7 — Comprobación (día lab. +17)

| Condición | Acción |
|-----------|--------|
| Tiene push | Push Alex — **MANUAL** |
| Sin push | No bloqueante |

**Texto:**
> Llevo dos semanas sin respuesta {Nombre} 😔, ¿está todo bien? 😏

---

### Paso 8 — Pausa del servicio (día lab. +21)

**Siempre MANUAL. Último paso.**

```
¿Tiene WhatsApp?
├── SÍ → Chat + WhatsApp + Email  ← MANUAL
└── NO → Chat + Email  ← MANUAL
```

**Mensaje chat (con dolor):**
> {Nombre}, llevas 3 semanas sin empezar el programa y entiendo que no es el mejor momento. Voy a pausar el servicio por ahora. Cuando quieras retomarlo, ya sea porque empeora tu {ZONA_DOLOR} o porque te apetezca darle una oportunidad, me escribes por el chat y lo activamos al instante. Cuídate.

**Mensaje chat (sin dolor):**
> {Nombre}, veo que ahora no es el momento para meterle al programa y no pasa nada. Voy a pausar el servicio. Si más adelante te apetece retomarlo o te surge cualquier molestia, me escribes y lo reactivamos al momento. Un abrazo.

**Email:** (idéntico al de Fase 0 — pausar + cómo reactivar)

---

### Resumen visual Fase 3

```
Programa asignado
  │ inmediato
  ▼
[Día 0]  Chat de Alex sobre el programa        ← auto por chat del fisio
  │ +2 días lab.
  ▼
[Día 2]  Push Aurya (solo con push)            ← manual
  │ +3 días lab. desde [0] (solo sin push)
  ▼
[Día 3]  Chat/WhatsApp sin push                ← auto o manual
  │ +5 días lab. desde anterior
  ▼
[Día 5]  Push Aurya o chat si respondió        ← manual / auto
  │ +7 días lab. desde anterior
  ▼
[Día 7]  Push Aurya o WhatsApp ajuste          ← manual
  │ +10 días lab. desde anterior
  ▼
[Día 10] Push Alex (qué cuesta)                ← manual
  │ +14 días lab. desde anterior
  ▼
[Día 14] Rescate push+WhatsApp                 ← manual
  │ +17 días lab. desde anterior
  ▼
[Día 17] Push Alex comprobación                ← manual
  │ +21 días lab. desde anterior
  ▼
[Día 21] Pausa chat + email                    ← manual
```

---

## Fase 4 · Realizando sesiones (1–9 sesiones)

**Objetivo:** Sostener el ritmo hasta completar el bloque de 10.  
**Ancla:** `lastSessionAt` (se reinicia con cada sesión).  
**Dos tracks independientes — la señal tiene prioridad.**

---

### Track A · Mensaje por señal (día lab. +1)

Se activa cuando ocurre cualquier señal en los datos de ejercicio:

| Señal | Condición | Prioridad |
|-------|-----------|-----------|
| Dolor en sesión | `feedback.hasPain === true` | Alta |
| Subida de dolor | `latestPainLevel > previousPainLevel` | Alta |
| Esfuerzo muy alto | `feedback.difficulty >= 8` | Media |
| Valoración baja | `feedback.rating <= 2` | Media |

**Cuándo:** 1 día laborable después de la señal.

**Canal:**
- WhatsApp si disponible
- Chat (app) si no

**Mensajes según señal:**

**Subida de dolor:**
> {Nombre}, he visto que tu dolor ha subido desde el último registro. Antes de que sigas, dime dónde lo notas y con qué ejercicio para ajustarlo.

**Dolor en sesión:**
> {Nombre}, he visto que marcaste dolor en la última sesión. Antes de que sigas, dime dónde lo notas y con qué ejercicio para ajustarlo.

**Valoración baja:**
> {Nombre}, he visto que valoraste la última sesión con una nota baja. Cuéntame qué no fue bien para poder ajustarlo.

**Esfuerzo muy alto:**
> {Nombre}, he visto que la última sesión te requirió un esfuerzo muy alto. Dime qué parte cuesta más y lo adaptamos para que puedas seguir sin forzar.

**Cómo completa:** Chat del fisio posterior al target (auto) o marcar manual.  
Si hay señal pendiente, este track **bloquea el seguimiento semanal**.

---

### Track B · Seguimiento semanal (días 7–10 de calendario)

⚠️ Este track usa **días de calendario**, no laborables.

| Evento | Días naturales desde ancla |
|--------|---------------------------|
| Ventana abre | Día 7 |
| Deadline | Día 10 |
| Si completa → nuevo ciclo | Día 7 desde fecha de ese seguimiento |

**Canal:**
- WhatsApp si disponible
- Chat (app) si no

**Texto:**
> {Nombre}, ¿qué tal va esta semana con los ejercicios? Si algo te molesta, te resulta demasiado fácil o se te está haciendo cuesta arriba, dime y lo ajusto.

**Cómo completa:** Chat del fisio dentro de la ventana (auto) o marcar manual.

**Urgencia:**
- Antes del día 7: `waiting` — sin acción
- Días 7–10: `pending` — acción en ventana
- Después del día 10: `pending` con retraso — **crítico**

---

### Resumen visual Fase 4

```
Sesión realizada (ancla)
  │
  ├─ [Señal] Dolor/esfuerzo/rating bajo
  │      │ +1 día lab.
  │      ▼
  │   Mensaje personalizado Alex   ← auto o manual
  │   (bloquea el seguimiento semanal)
  │
  └─ [Día 7 calendario]
       Ventana seguimiento semanal  ← auto o manual
       [Día 10: deadline]
       │
       ▼ desde fecha de ese seguimiento
       Vuelta al inicio del ciclo
```

---

## Fase 5 · Pendiente revalorización

**Objetivo:** Revalorar al paciente y decidir continuidad.  
**No hay cadencia automática.** Solo SLA.

| Días lab. desde sesión 10 | Estado |
|---------------------------|--------|
| < 2 | OK — programar revalorización |
| 2–7 | Aviso — realizar revalorización y feedback |
| > 7 | **Crítico** — contactar para cerrar bloque |

**Acción disponible:** Marcar revalorización realizada (canal: mensaje en app).

---

## Fase 6 · Programa renovado · Fidelización

**Lógica idéntica a Fase 4.**  
**Única diferencia:** el ancla de tiempo cambia.

| Fase | Ancla |
|------|-------|
| Fase 4 | `lastSessionAt` |
| Fase 6 | `planUpdatedAt ?? lastSessionAt` |

Renovar el programa **reinicia el reloj** del seguimiento semanal desde cero.

---

## Columna Abandonados

**Manual en todos los casos.**  
Se mueve aquí cuando se completó toda la cadencia y el paciente no responde.

| Evento | Acción |
|--------|--------|
| Marca manual | Paciente pasa a Abandonados |
| +2 meses | Card aparece como acción pendiente — recontactar |

---

## Tabla resumen: canales y detección por fase

| Fase | Canales | Auto-detectable |
|------|---------|-----------------|
| 0 | Chat, Push, WhatsApp, Email | Chat (mensajes del fisio) |
| 1 | Chat, WhatsApp | Chat (mensajes del fisio) |
| 2 | — (solo asignar programa) | — |
| 3 | Chat, Push, WhatsApp, Email, Push+WhatsApp, Chat+WhatsApp+Email | Chat (mensajes del fisio) |
| 4/6 señal | Chat, WhatsApp | Chat del fisio posterior a la señal |
| 4/6 semanal | Chat, WhatsApp | Chat del fisio en ventana día 7–10 |
| 5 | Chat | Manual únicamente |

---

## Datos que necesitas modelar por paciente

Estos son los campos clave del objeto `SeguimientoPatient` que alimentan todo:

```
// Clasificación de fase
hasRehabPlan / hasPreventionPlan
lastFisioContactAt / lastFisioContactChannel
pendingAppointmentAt / pendingAppointmentStatus / pendingAppointmentScheduledAt
lastNoShowAppointmentAt
sessionsCompleted
lastSessionAt
planUpdatedAt / planStartedAt

// Cadencia por fase (objeto cadence)
cadence.phase0.firstPhysioChatAt
cadence.phase0.manualActions[]          ← acciones manuales marcadas
cadence.phase1.patientFirstChatAt
cadence.phase1.physioMessagesAfterPatient[]
cadence.phase1.manualActions[]
cadence.phase2 (mensajes del programa, lecturas, si respondió)
cadence.phase4.manualActions[]
cadence.phase5.manualActions[]

// Señales de ejercicio (Fase 4/6)
latestPainLevel / previousPainLevel
lastSessionFeedback.hasPain
lastSessionFeedback.difficulty
lastSessionFeedback.rating

// Canales de contacto
notifications.enabled                   ← tiene push activo
contactPhone                            ← tiene WhatsApp

// Estado especial
abandoned.active / abandoned.contactAfterAt
chatInbox.resolved / chatInbox.messageAt
lastChatAt / lastChatFromPhysio / lastChatContent
```

---

## Estructura de CadenceManualAction

Así se guardan las acciones completadas manualmente:

```typescript
interface CadenceManualAction {
  stage: string;         // 'first-chat' | 'day-3' | 'day-7' | etc.
  channel: string;       // 'push' | 'whatsapp' | 'message' | etc.
  actionKey: string;     // 'day-3:push' | 'day-7:whatsapp' | etc.
  status: 'done';
  completedAt: string;   // ISO timestamp
  messagePreview?: string;
}
```

El `actionKey` es `${stage}:${channel}` y es el identificador único de cada acción dentro de una fase.

---

## Cómo replicarlo para PreZero líderes

| Concepto Melia | Equivalente PreZero |
|----------------|---------------------|
| Registro del paciente | Nominación del líder (`isLeader: true`) |
| Primer chat del fisio | Primer WhatsApp de Lorena |
| Respuesta del paciente | Primer warmup grupal reportado |
| Fase 0 (sin contacto) | L0a — recién nominado |
| "Sin interés" / L0b | L0b — intentado (21 días sin warmup) |
| Fase 1 (contacto activo) | L1 — Activo |
| Fase 3 (programa sin empezar) | L1 con warmup pero sin rehab plan |
| Fase 4/6 (sesiones activas) | L1/L2 con warmup regular |
| Abandonados | L3c — rescate fallido |
| `sessionsCompleted` | `warmupGroupalDiasEnDiana` por semana |
| `latestPainLevel` | Caída de días de warmup bajo umbral del tier |
| Seguimiento semanal | Seguimiento semanal por tier |
