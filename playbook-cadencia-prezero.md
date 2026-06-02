# Playbook de comunicaciones · PreZero Líderes
> Versión 4 · revisado

---

## Arquitectura

```
CAPA BASE · Todos los usuarios PreZero (F0–F7 tipo Meliá)
    │
    └─→ CAPA LÍDER (solo centros viables bucket A)
            L0 → L1 → L2
                  ↕
                  L3 (intervención)
```

Un líder recibe **ambas capas**: sigue siendo usuario con plan individual + recibe las comunicaciones de líder encima.

---

## Canales y quién escribe

| Canal | Voz | Tono | Completa |
|-------|-----|------|----------|
| Chat in-app | **Aurya** | Cercano, informal | Automático (mensaje detectado) o manual |
| Push | Sistema | Corto, accionable | Manual (marcar enviado) |
| WhatsApp | **Lorena** (CS) | Humano, directo | Manual (marcar enviado) |

**Prioridad de canal:** Chat in-app → Push → WhatsApp

---

## Estados del resolver

| Estado | Significado |
|--------|-------------|
| `pending` | Hay una acción que hacer ahora mismo |
| `waiting` | La siguiente acción no toca todavía |
| `complete` | Las comunicaciones de esta fase están completas |

---

## Variables de plantilla

| Variable | Valor |
|----------|-------|
| `{Nombre}` | Nombre del líder |
| `{Centro}` | Nombre del centro |
| `{Diana}` | Diana del tier (ej. "7/7") |
| `{Días}` | Días con warmup esta semana |
| `{Semanas}` | Nº semanas consecutivas |

---

---

# L0 · No activo

**Definición:** Designado por Alex · aún no ha organizado su primer warmup grupal.

**Sub-estados:**
- `L0a` · recién nominado → seguimiento activo 21 días
- `L0b` · intentado → activación fallida, sin seguimiento, espera revisión trimestral

**Salida a L1:** Primer warmup grupal reportado.
**Salida a L0b:** 21 días sin warmup + aviso final (day-20) completado.

---

## Resolver L0a

```
input:
  nominatedAt           ← fecha de nominación (isLeader: true)
  leader.cadence.l0.firstWhatsAppAt
  leader.cadence.l0.manualActions[]   ← { stage, channel, actionKey, completedAt }
  leader.chatInApp.lastAuryaMessageAt ← para auto-detección
  leader.pushEnabled
  leader.hasWhatsApp
  leader.chatInApp.lastReadAt         ← para condición leído/no leído

output:
  { kind: 'pending' | 'waiting' | 'complete', stage, channel, message, requiresManual }
```

**Stages en orden:**
`first-whatsapp` → `day-3` → `day-7` → `day-14` → `day-20` → `noInterest`

**Timing:** Días naturales (no laborables) desde la fecha de nominación.

---

## Stage 1 · first-whatsapp (Día 1)

| | |
|--|--|
| **Cuándo** | Día 1 desde nominación |
| **Antes del día 1** | `kind: waiting` |
| **A partir del día 1** | `kind: pending` |
| **Canal** | WhatsApp — Lorena |
| **Completa** | Manual (marcar enviado) |

**Mensaje WhatsApp Lorena:**
> Hola {Nombre}, soy Lorena del equipo de Fisify. Alex me ha dicho que eres el nuevo líder de warmup en {Centro} 💪 Estoy aquí para ayudarte a arrancar. ¿Cuándo crees que podría ser el primer warmup con tu equipo esta semana? Si tienes cualquier duda, escríbeme aquí.

---

## Stage 2 · day-3 (Día 3)

| | |
|--|--|
| **Cuándo** | Día 3 desde nominación |
| **Bloqueo previo** | `first-whatsapp` completado |
| **Canal** | Push |
| **Condición** | Solo si `pushEnabled: true` |
| **Sin push** | **No bloqueante** — se marca automáticamente y pasa al siguiente |
| **Completa** | Manual (marcar enviado) |

**Texto push:**
> {Nombre}, tu equipo te espera 💪 ¿Hoy puede ser el primer warmup?

---

## Stage 3 · day-7 (Día 7)

| | |
|--|--|
| **Cuándo** | Día 7 desde nominación |
| **Bloqueo previo** | `day-3` completado o marcado no bloqueante |
| **Completa** | Automático (mensaje Aurya detectado) o manual |

**Árbol de canal — prioridad: chat → push → WhatsApp**

```
1. ¿Tiene chat in-app activo?
   └── SÍ → Aurya (auto-detectado por mensaje posterior al día 7)
              Leído prev.:   mensaje A
              No leído prev: mensaje B

2. ¿Tiene push activo?
   └── SÍ → Push (manual)
              Leído prev.:   mensaje C
              No leído prev: mensaje D

3. ¿Tiene WhatsApp?
   └── SÍ → WhatsApp Lorena (manual)
              Leído prev.:   mensaje E
              No leído prev: mensaje F
```

**A · Aurya · leído:**
> {Nombre}, vi que leíste el mensaje de Lorena 👀 ¿Te pillo en mal momento o es que la semana ha estado muy cargada en {Centro}?

**B · Aurya · no leído:**
> Hola {Nombre} 👋 Lorena te escribió la semana pasada pero parece que no llegó bien. Solo quería saber cómo va el tema del primer warmup en {Centro}. ¿Cuándo crees que podría ser?

**C · Push · leído:**
> {Nombre}, leíste el mensaje pero aún no hemos arrancado. ¿Esta semana puede ser el primero? 🔥

**D · Push · no leído:**
> {Nombre}, Lorena lleva una semana esperando 😏 ¿Le das al equipo el primer warmup esta semana?

**E · WhatsApp Lorena · leído:**
> Hola {Nombre}, vi que leíste mi mensaje. ¿Qué tal va? Si hay algo que te esté frenando para organizar el warmup en {Centro}, cuéntame — lo vemos juntos.

**F · WhatsApp Lorena · no leído:**
> Hola {Nombre}, te escribí la semana pasada por la app pero creo que no te llegó. Solo quería saber cómo va el warmup en {Centro} — ¿hay algo que te esté bloqueando para arrancarlo?

---

## Stage 4 · day-14 (Día 14)

| | |
|--|--|
| **Cuándo** | Día 14 desde nominación |
| **Bloqueo previo** | `day-7` completado |
| **Completa** | Automático o manual |

**A · Aurya · leído:**
> {Nombre}, llevamos dos semanas y todavía no hemos podido arrancar el warmup grupal en {Centro}. Es normal que cueste al principio — después va solo. ¿Qué es lo que más te está frenando ahora mismo?

**B · Aurya · no leído:**
> {Nombre}, te hemos mandado un par de mensajes estas semanas. Sé que estás liado, pero el primer warmup es el más difícil — después el equipo lo pide solo. ¿Qué es lo que más te está costando?

**C · Push · leído:**
> {Nombre}, dos semanas sin warmup grupal en {Centro}. ¿Hoy puede ser el día? 💪

**D · Push · no leído:**
> {Nombre}, Lorena sigue esperando 📲 Solo necesitamos un primer warmup esta semana.

**E · WhatsApp Lorena · leído:**
> Hola {Nombre}, soy Lorena. Han pasado dos semanas y todavía no hemos arrancado con el warmup en {Centro}. No te escribo para presionarte — quiero entender qué está pasando. ¿Tienes 5 minutos para contarme?

**F · WhatsApp Lorena · no leído:**
> Hola {Nombre}, soy Lorena de Fisify. Te he mandado un par de mensajes estas semanas. Sé que hay épocas muy cargadas, pero quería escribirte antes de cerrar el mes. El warmup grupal en {Centro} todavía no ha arrancado — ¿qué está pasando?

---

## Stage 5 · day-20 (Día 20) · Cierre

| | |
|--|--|
| **Cuándo** | Día 20 desde nominación |
| **Bloqueo previo** | `day-14` completado |
| **Completa** | Manual — último paso antes de L0b |

**A · Aurya · leído:**
> {Nombre}, vamos a dejarlo aquí por ahora. Alex lo verá en la próxima revisión trimestral. Si en cualquier momento quieres retomar el rol de líder de warmup en {Centro}, escríbeme aquí y lo activamos al instante.

**B · Aurya · no leído:**
> {Nombre}, no hemos podido conectar estas semanas. Lo dejamos aquí por ahora — el rol sigue siendo tuyo si quieres retomarlo. Escríbeme cuando estés listo.

**C · Push · leído:**
> {Nombre}, lo dejamos aquí por ahora. El rol sigue siendo tuyo — retómalo cuando quieras.

**D · Push · no leído:**
> {Nombre}, lo dejamos aquí por ahora. El warmup en {Centro} te espera cuando quieras retomarlo.

**E · WhatsApp Lorena · leído:**
> Hola {Nombre}, soy Lorena. Vamos a dejarlo aquí por ahora — hemos intentado conectar varias veces y entiendo que ahora no es el momento. El rol de líder de warmup sigue siendo tuyo: cuando quieras retomarlo, me escribes aquí y lo activamos al instante. Alex lo verá en la próxima revisión trimestral. Un saludo.

**F · WhatsApp Lorena · no leído:**
> Hola {Nombre}, soy Lorena de Fisify. He intentado escribirte estas semanas para arrancar con el warmup en {Centro}, pero no hemos podido conectar. Lo entiendo — hay épocas muy cargadas. Lo dejamos aquí por ahora, pero el rol sigue siendo tuyo. Cuando quieras retomarlo, me escribes y lo activamos al momento. Un saludo.

---

## noInterestEligible · → L0b

**Condición:** Día 21+ sin warmup reportado Y `day-20` completado.
**Acción:** Marcar como `L0b · intentado`. Sin seguimiento activo. Espera revisión trimestral.

---

## Resumen visual L0a

```
Nominación (isLeader: true)
  │ Día 1
  ▼
[first-whatsapp]  WhatsApp Lorena · bienvenida          ← manual
  │ Día 3
  ▼
[day-3]           Push activación (si push, si no: skip) ← manual / non-blocking
  │ Día 7
  ▼
[day-7]           Chat/Push/WA · leído vs no leído       ← auto o manual
  │ Día 14
  ▼
[day-14]          Chat/Push/WA · tono más directo        ← auto o manual
  │ Día 20
  ▼
[day-20]          Chat/Push/WA · cierre                  ← manual
  │ Día 21+
  ▼
[L0b]             Intentado · sin seguimiento · revisión trimestral
```

---

---

# L1 · Activo

**Definición:** Ha reportado al menos 1 warmup grupal. Organiza con regularidad.
**Salida a L2:** 8 semanas consecutivas en diana del tier.
**Salida a L3:** 3 semanas consecutivas bajo umbral, o 2 semanas sin reportar nada.

**Cuadrante 2×2 — modula el tono, no la estructura:**

| Warmups | Plan propio | Lectura | Modulación |
|---------|-------------|---------|------------|
| ✅ | ✅ | Líder pleno · ideal | Mensajes base sin cambio |
| ✅ | ❌ | Cumple pero no es ejemplo | Push diario extra · "predica con el ejemplo" |
| ❌ | ✅ | Líder dormido | → L3 |
| ❌ | ❌ | Caído | → L3 |

---

## Push diario · Recordatorio warmup

**Lógica:** Cada día mientras el líder está en L1, si aún no ha reportado warmup ese día → Push.
Si ya reportó → nada.

```
Cada día en L1:
  Si warmup reportado hoy     → kind: waiting (sin mensaje)
  Si no ha reportado hoy      → kind: pending · Push recordatorio
```

**Condición:** Solo si `pushEnabled: true`.

**Texto push (líder pleno — warmup ✅ + plan ✅):**
> {Nombre}, ¿warmup hoy en {Centro}? 💪

**Texto push (cumple pero no es ejemplo — warmup ✅ + plan ❌):**
> {Nombre}, el equipo espera que arranques tú. ¿Warmup hoy en {Centro}?

---

## Track A · Onboarding (días 1–7 desde el primer warmup)

Se ejecuta **una sola vez** al entrar en L1. Objetivo: celebrar, fijar expectativas, establecer diana.

### Día 1 · Celebración del primer warmup

| | |
|--|--|
| **Canal** | Chat in-app (Aurya) |
| **Completa** | Automático (mensaje detectado) |

**Mensaje Aurya:**
> ¡{Nombre}! 🎉 Primer warmup grupal en {Centro} hecho. Así se empieza. La diana esta semana es {Diana} — dime cómo va y cualquier cosa que necesites, aquí estoy.

---

### Día 3 · Check-in rápido

| | |
|--|--|
| **Canal** | Push |
| **Condición** | Solo si `pushEnabled` |
| **Sin push** | No bloqueante |
| **Completa** | Manual |

**Texto push:**
> {Nombre}, ¿cómo va el equipo en {Centro} esta semana? 💪

---

### Día 7 · Primera revisión semanal

| | |
|--|--|
| **Canal** | Chat in-app (Aurya) |
| **Completa** | Automático o manual |

**Mensaje Aurya (hizo diana):**
> {Nombre}, primera semana completada 💪 {Días}/7 días con warmup. Así se consolida el hábito. ¿Algo que quieras ajustar para la semana que viene?

**Mensaje Aurya (no hizo diana):**
> {Nombre}, primera semana con {Días}/7. No está mal para empezar — la diana es {Diana}. ¿Qué pasó los días que no salió? Así vemos si hay algo que ajustar.

---

## Track B · Seguimiento semanal (recurrente)

**Lógica clave:** Si el líder está en diana → `kind: waiting`, no se manda nada.
Solo se activa cuando cae bajo diana.

**Cuando está bajo diana: 2 touchpoints separados dentro de la misma semana.**
El segundo se cancela automáticamente si el líder reporta un warmup entre medias.

```
Cada semana de reporte:
├── ≥ diana del tier
│     → kind: waiting (sin mensaje)
│
├── < diana, semana 1
│     Día de cierre de semana   → Push (señal inmediata)
│     Día +3                    → Aurya chat (seguimiento · se cancela si reporta warmup)
│     Frecuencia máxima: 1 vez cada 14 días
│
├── < diana, 2 semanas seguidas
│     Día de cierre             → Aurya chat
│     Día +2                    → WhatsApp Lorena
│
└── < diana, 3 semanas seguidas → exit a L3
```

---

### 1 semana bajo diana · Día de cierre

**Canal:** Push

> {Nombre}, esta semana bajó el ritmo en {Centro}. ¿Todo bien?

---

### 1 semana bajo diana · Día +3 (si no reportó warmup)

**Canal:** Chat in-app (Aurya)

**Aurya (líder pleno — warmup ✅ + plan ✅):**
> Oye {Nombre}, vi que esta semana el warmup bajó un poco en {Centro}. Si fue algo puntual no pasa nada — ¿qué pasó? Así vemos si hay que ajustar algo o si fue solo una semana rara.

**Aurya (cumple pero no es ejemplo — warmup ✅ + plan ❌):**
> {Nombre}, esta semana el equipo notó que el warmup bajó. Tú eres el referente en {Centro} — cuando el líder va, el equipo va. ¿Qué pasó?

---

### 2 semanas seguidas bajo diana · Día de cierre

**Canal:** Chat in-app (Aurya)

> {Nombre}, llevan dos semanas con el warmup por debajo de lo esperado en {Centro}. ¿Hay algo que esté pasando en el equipo o en la operativa que lo esté frenando?

---

### 2 semanas seguidas bajo diana · Día +2

**Canal:** WhatsApp Lorena

> Hola {Nombre}, soy Lorena. He visto que las últimas dos semanas el warmup grupal en {Centro} ha bajado. ¿Puedo ayudarte con algo? Cuéntame qué está pasando y lo vemos juntos.

---

### Trigger cross-fase · "Tu equipo está en racha"

**Condición:** Líder tier Mantener (7/7) cae a 5/7 en 1 semana aislada.
**Frecuencia:** Máximo 1 cada 14 días.
**Canal:** Push + chat Aurya (mismo espaciado: push inmediato, Aurya día +3).

**Push:**
> Tu equipo está en racha · no la cortes hoy 🔥

**Aurya (día +3, si no reportó warmup):**
> {Nombre}, esta semana bajó un poco pero el equipo lleva una racha buena en {Centro}. Hoy es buen día para el warmup — no rompas la inercia.

---

## Transición L1 → L2 (8 semanas en diana)

**Canal:** Push + WhatsApp Lorena + in-app especial (modal/badge · pendiente spec con Inhar).

**Push:**
> {Nombre}, 8 semanas seguidas en diana 🏆 El equipo de {Centro} ya lo tiene.

**Mensaje WhatsApp Lorena:**
> {Nombre}! 🎉 8 semanas seguidas cumpliendo el objetivo en {Centro}. Eso no es casualidad — el equipo ya lo tiene interiorizado. Enhorabuena, estás en un nivel diferente. Alex lo sabe y lo valora.

**In-app:** Modal de celebración / badge — especificación pendiente con Inhar.

---

---

# L2 · En racha

**Definición:** 8+ semanas consecutivas cumpliendo diana. Rutina estable.
**Salida a L1:** 3 semanas consecutivas perdiendo diana (L2 nunca cae directo a L3).

**Sub-estados:**
- `L2 base` · mantiene su propia rutina
- `L2 activo` · movilizador — mantiene rutina + moviliza al equipo (Alex activa manualmente)

---

## Seguimiento semanal silencioso

**Igual que L1 Track B, pero con tono más suave. El líder en racha no necesita urgencia — necesita continuidad.**

```
Cada semana:
├── ≥ diana
│     → kind: waiting (sin mensaje)
│
├── < diana, semana 1
│     Día de cierre             → Aurya · check-in suave (no alarmista)
│     Día +3                    → WhatsApp Lorena (si no reportó warmup)
│     Frecuencia máxima: 1 vez cada 14 días
│
├── < diana, 2 semanas seguidas
│     Día de cierre             → Aurya + Push
│     Día +2                    → WhatsApp Lorena
│
└── < diana, 3 semanas seguidas → vuelve a L1 (con seguimiento semanal activo)
```

### 1 semana bajo diana · Día de cierre

**Canal:** Chat in-app (Aurya)

**L2 base:**
> {Nombre}, ¿qué tal esta semana en {Centro}? Vi que el warmup bajó un poco — si hay algo que esté complicando el ritmo, cuéntame.

**L2 activo:**
> {Nombre}, esta semana el warmup bajó un poco en {Centro}. ¿Todo bien? Cuando el equipo te ve, cambia el ambiente — cuéntame si hay algo que esté complicando las cosas.

---

### 1 semana bajo diana · Día +3 (si no reportó warmup)

**Canal:** WhatsApp Lorena

> Hola {Nombre}, soy Lorena. Vi que esta semana el warmup bajó un poco en {Centro}. ¿Hay algo en lo que pueda ayudarte? No te escribo para presionarte — solo quiero saber si está todo bien.

---

### 2 semanas seguidas bajo diana · Día de cierre

**Canal:** Chat in-app (Aurya) + Push

**Push:**
> {Nombre}, dos semanas seguidas bajo el objetivo en {Centro}. ¿Hablamos?

**Aurya:**
> {Nombre}, llevan dos semanas con el warmup por debajo de lo esperado en {Centro}. No es la tendencia que llevabas — ¿qué está pasando? Cuéntame y lo vemos juntos.

---

### 2 semanas seguidas bajo diana · Día +2

**Canal:** WhatsApp Lorena

> Hola {Nombre}, soy Lorena. Dos semanas seguidas con el warmup por debajo en {Centro}. Quiero entender qué está pasando antes de que se complique más. ¿Tienes un momento esta semana?

---

## Reconocimiento quincenal

**Canal:** Chat in-app (Aurya).
**Frecuencia:** Cada 2 semanas · solo cuando está en diana.

**Mensaje Aurya (L2 base):**
> {Nombre}, semana a semana 💪 El warmup en {Centro} sigue en marcha. Llevas {Semanas} semanas siendo un referente para el equipo. ¿Algo nuevo que quieras contarme?

**Mensaje Aurya (L2 activo):**
> {Nombre}, lo que estás haciendo en {Centro} va más allá del warmup — el equipo te sigue porque confía en ti. Llevas {Semanas} semanas. Alex lo ve y lo valora. ¿Cómo podemos hacer que sea aún más fácil para ti?

---

## Email mensual de resultados

**Canal:** Email · automático.
**Cuándo:** Primer lunes de cada mes.

**Asunto:** Tu impacto en {Centro} · {Mes}

**Cuerpo:**
> Hola {Nombre},
>
> Aquí tienes el resumen de warmup de {Centro} en {Mes}:
>
> · Días con warmup grupal: **{Total días}**
> · Media semanal: **{Media}/7**
> · Semanas consecutivas en diana: **{Semanas}**
>
> El equipo tiene una racha sólida. Sigue así.
>
> Un saludo,
> Lorena · Fisify

---

---

# L3 · En riesgo

**Definición:** Los warmups caen bajo umbral del tier durante 3 semanas consecutivas, o el líder no reporta 2 semanas seguidas.
**Salida a L1:** Retoma warmups en cualquier punto de la ventana.
**Salida a L0b:** Sin retoma en 21 días → cierre.

**Ventana única: 21 días. Sin sub-estados.**

---

## Resolver L3

```
input:
  l3EntryAt              ← fecha de entrada a L3
  leader.cadence.l3.manualActions[]
  leader.chatInApp.lastAuryaMessageAt
  leader.warmupReportedAt ← si reporta warmup → exit a L1

output:
  { kind: 'pending' | 'waiting' | 'complete', stage, channel, message }
```

---

### Día 0 · Entrada · Diagnóstico

| | |
|--|--|
| **Cuándo** | Inmediatamente al entrar en L3 |
| **Canal** | Chat in-app (Aurya) + WhatsApp Lorena |
| **Completa** | Automático (chat) o manual (WA) |

**Mensaje Aurya:**
> {Nombre}, he notado que las últimas semanas el warmup grupal ha bajado en {Centro}. No te escribo para presionarte — sé que hay semanas complicadas. ¿Qué está pasando? Cuéntame y lo vemos juntos.

**Mensaje WhatsApp Lorena:**
> Hola {Nombre}, soy Lorena. He visto que las últimas semanas el warmup en {Centro} ha bajado. ¿Tienes un momento esta semana para contarme qué está pasando? Quiero entenderlo antes de hacer nada.

---

### Día 7 · Seguimiento

| | |
|--|--|
| **Cuándo** | Día 7 desde entrada a L3 |
| **Canal** | Chat in-app (Aurya) |
| **Completa** | Automático o manual |

**Mensaje Aurya:**
> {Nombre}, ¿cómo va la semana? ¿Has podido organizar algún warmup grupal en {Centro}? Si hay algo que esté bloqueando al equipo — horarios, participación, lo que sea — cuéntame. A veces hay que ajustar el formato y ya está.

---

### Día 14 · Aviso crítico

| | |
|--|--|
| **Cuándo** | Día 14 desde entrada a L3 |
| **Canal** | Push + WhatsApp Lorena |
| **Completa** | Manual |

**Push:**
> {Nombre}, llevamos dos semanas sin warmup grupal en {Centro}. ¿Hablamos esta semana?

**Mensaje WhatsApp Lorena:**
> Hola {Nombre}, han pasado dos semanas y todavía no hemos podido retomar el ritmo en {Centro}. Necesito saber si quieres seguir con el rol. ¿Tienes un hueco esta semana — aunque sea 10 minutos — para hablar con Alex? Lo organizo yo. Solo dime cuándo.

---

### Día 21 · Cierre → L0b

| | |
|--|--|
| **Cuándo** | Día 21 desde entrada a L3 |
| **Canal** | WhatsApp Lorena |
| **Completa** | Manual · último paso |

**Mensaje WhatsApp Lorena:**
> Hola {Nombre}, lo dejamos aquí. Como no hemos podido retomar el warmup grupal en {Centro}, Alex va a revisar el rol en la próxima revisión trimestral. El rol puede volver a ser tuyo si quieres retomarlo — solo escríbeme y lo reactivamos. Un saludo.

🔴 **Acción:** Marcar → L0b. Sin seguimiento activo. Espera revisión trimestral.

---

## Resumen visual L3

```
Warmups caen bajo umbral (3 sem) o sin reporte (2 sem)
  │
  ▼
L3 · Intervención (21 días)
  Día 0:  Chat Aurya + WA Lorena  (diagnóstico)
  Día 7:  Chat Aurya              (seguimiento)
  Día 14: Push + WA Lorena        (aviso crítico)
  Día 21: WA Lorena               (cierre → L0b)

  Si retoma warmup en cualquier punto → exit a L1
  Si no retoma en 21 días → L0b · revisión trimestral
```

---

---

## Pendiente de definir

- [ ] In-app especial en transición L1→L2 (modal/badge/animación) · con Inhar
- [ ] Denominador de asistencia (% equipo presente como criterio)
- [ ] Frecuencia diana warmups/semana por tier (¿varía por centro?)
- [ ] Track A solo vs Track A + Track B en L2 (reconocimiento base vs. gamificación)
- [ ] Criterio operativo para L2 activo · qué señales mira Alex
- [ ] Read tracking disponible → actualizar mensajes leído/no leído con variantes reales
