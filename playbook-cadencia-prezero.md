# Playbook de cadencia · PreZero Líderes
> Versión 2 · documento vivo · pendiente cierre L3

---

## Arquitectura

```
CAPA BASE · Todos los usuarios PreZero (F0–F7 tipo Meliá)
    │
    └─→ CAPA LÍDER (solo centros viables bucket A)
            L0 → L1 → L2
                  ↕
                  L3 (ciclo de rescate)
```

Un líder recibe **ambas capas**: sigue siendo usuario con plan individual + recibe la cadencia líder encima.

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
| `complete` | La cadencia de esta fase está completa |

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
- `L0a` · recién nominado → cadencia activa 21 días
- `L0b` · intentado → activación fallida, sin cadencia, espera revisión trimestral

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
> {Nombre}, vamos a pausar la cadencia por ahora. Alex lo revisará en la próxima revisión trimestral. Si en cualquier momento quieres retomar el rol de líder de warmup en {Centro}, escríbeme aquí y lo activamos al instante.

**B · Aurya · no leído:**
> {Nombre}, no hemos podido conectar estas semanas. Vamos a pausar por ahora — el rol sigue siendo tuyo si quieres retomarlo. Escríbeme cuando estés listo.

**C · Push · leído:**
> {Nombre}, cerramos el ciclo de activación. El rol sigue siendo tuyo — reactívalo cuando quieras.

**D · Push · no leído:**
> {Nombre}, pausamos por ahora. El warmup en {Centro} te espera cuando quieras retomarlo.

**E · WhatsApp Lorena · leído:**
> Hola {Nombre}, soy Lorena. Vamos a dejar aquí la activación por ahora — hemos intentado conectar varias veces y entiendo que ahora no es el momento. El rol de líder de warmup sigue siendo tuyo: cuando quieras retomarlo, me escribes aquí y lo activamos al instante. Alex lo verá en la próxima revisión trimestral. Un saludo.

**F · WhatsApp Lorena · no leído:**
> Hola {Nombre}, soy Lorena de Fisify. He intentado escribirte estas semanas para arrancar con el warmup en {Centro}, pero no hemos podido conectar. Lo entiendo — hay épocas muy cargadas. Vamos a pausar por ahora, pero el rol sigue siendo tuyo. Cuando quieras retomarlo, me escribes y lo activamos al momento. Un saludo.

---

## noInterestEligible · → L0b

**Condición:** Día 21+ sin warmup reportado Y `day-20` completado.
**Acción:** Marcar como `L0b · intentado`. Sin cadencia activa. Espera revisión trimestral.

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
[L0b]             Intentado · sin cadencia · revisión trimestral
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
| ✅ | ✅ | Líder pleno · ideal | Playbook base sin cambio |
| ✅ | ❌ | Cumple pero no es ejemplo | Push semanal extra · "predica con el ejemplo" |
| ❌ | ✅ | Líder dormido | → L3a |
| ❌ | ❌ | Caído | → L3b |

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

## Track B · Mantenimiento semanal (recurrente)

**Lógica clave:** Si el líder está en diana → `kind: waiting`, no se manda nada.
Solo se activa cuando cae bajo diana.

```
Cada semana de reporte:
├── ≥ diana del tier           → kind: waiting (sin mensaje)
├── < diana, semana 1 aislada  → kind: pending · push + chat Aurya (trigger reactivo)
├── < diana, 2 sem seguidas    → kind: pending · WA Lorena
└── < diana, 3 sem seguidas    → exit a L3
```

### Trigger reactivo — 1 semana bajo diana (aislada)

**Frecuencia máxima:** 1 vez cada 14 días.
**Canal:** Push + chat Aurya.

**Push:**
> {Nombre}, esta semana bajó el ritmo en {Centro}. ¿Todo bien?

**Mensaje Aurya:**
> Oye {Nombre}, vi que esta semana el warmup bajó un poco en {Centro}. Si fue algo puntual no pasa nada — ¿qué pasó? Así vemos si hay que ajustar algo o si fue solo una semana rara.

**Versión "líder pleno" (✅ warmup + ✅ plan):** igual que arriba.
**Versión "cumple pero no ejemplo" (✅ warmup + ❌ plan):**
> {Nombre}, esta semana el equipo notó que el warmup bajó. Tú eres el referente en {Centro} — cuando el líder va, el equipo va. ¿Qué pasó?

---

### 2 semanas seguidas bajo diana

**Canal:** WhatsApp Lorena + chat Aurya.

**Mensaje Aurya:**
> {Nombre}, llevan dos semanas con el warmup por debajo de lo esperado en {Centro}. ¿Hay algo que esté pasando en el equipo o en la operativa que lo esté frenando?

**Mensaje WhatsApp Lorena:**
> Hola {Nombre}, soy Lorena. He visto que las últimas dos semanas el warmup grupal en {Centro} ha bajado. ¿Puedo ayudarte con algo? Cuéntame qué está pasando y lo vemos juntos.

---

### Trigger cross-fase · "Tu equipo está en racha"

**Condición:** Líder tier Mantener (7/7) cae a 5/7 en 1 semana aislada.
**Frecuencia:** Máximo 1 cada 14 días.
**Canal:** Push + chat Aurya.

**Push:**
> Tu equipo está en racha · no la cortes hoy 🔥

**Mensaje Aurya:**
> {Nombre}, esta semana bajó un poco pero el equipo lleva una racha buena en {Centro}. Hoy es buen día para el warmup — no rompas la inercia.

---

## Transición L1 → L2 · Momento especial (8 semanas en diana)

**Canal:** Push + WhatsApp Lorena + in-app especial (modal/badge · pendiente spec con Inhar).

**Push:**
> {Nombre}, 8 semanas seguidas en diana 🏆 Ya eres Líder Consolidado en {Centro}.

**Mensaje WhatsApp Lorena:**
> {Nombre}! 🎉 8 semanas seguidas cumpliendo el objetivo en {Centro}. Eso no es casualidad — el equipo ya lo tiene interiorizado. Enhorabuena, estás en un nivel diferente. Alex lo sabe y lo valora.

**In-app:** Modal de celebración / badge — especificación pendiente con Inhar.

---

---

# L2 · Consolidado

**Definición:** 8+ semanas consecutivas cumpliendo diana. Rutina estable sin empuje.
**Salida a L1:** 3 semanas consecutivas perdiendo diana (L2 nunca cae directo a L3).

**Sub-estados:**
- `L2 base` · Consolidado estándar — mantiene rutina personal
- `L2 activo` · Movilizador — mantiene rutina + moviliza al equipo (Alex activa manualmente)

---

## Seguimiento quincenal · Reconocimiento

**Canal:** Chat in-app (Aurya).
**Frecuencia:** Cada 2 semanas.

**Mensaje Aurya (L2 base · Consolidado):**
> {Nombre}, semana a semana 💪 El warmup en {Centro} sigue en marcha. Llevas {Semanas} semanas siendo un referente para el equipo. ¿Algo nuevo que quieras contarme?

**Mensaje Aurya (L2 activo · Movilizador):**
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

## Señal de caída en L2 (trigger reactivo)

Misma lógica que L1 Track B. L2 tiene doble pared antes de L3:
1. Primera caída → vuelve a L1 (con cadencia activa semanal)
2. Solo desde L1 puede llegar a L3

---

---

# L3 · En riesgo

> ⚠️ **Definición de sub-estados pendiente de aclaración.**
>
> Hay una inconsistencia entre las fichas de diseño y el documento de arquitectura. Antes de escribir los mensajes de L3, necesito confirmar cuál es la definición correcta de L3a, L3b y L3c.
>
> **Opción A (imágenes/fichas):**
> - L3a = rescate suave · líder con rehab plan activo al entrar en L3
> - L3b = rescate completo · líder sin rehab plan al entrar
> - L3c = rescate fallido · 21d sin retomar
>
> **Opción B (documento de arquitectura):**
> - L3a = PLENO (warmups ✅ + reporta ✅ + plan prevention)
> - L3b = PARCIAL (warmups ✅ + falta una pata)
> - L3c = CLÍNICO (warmups ✅ + rehab con Alex)

---

## Pendiente de definir

- [ ] Definición correcta de sub-estados L3 (opción A vs B)
- [ ] Mensajes de rescate L3a y L3b (se escriben tras resolver lo anterior)
- [ ] Videollamada en L3b — Lorena propone por WhatsApp (¿días disponibles?)
- [ ] In-app especial en transición L1→L2 (modal/badge/animación) · con Inhar
- [ ] Denominador de asistencia (% equipo presente como criterio)
- [ ] Frecuencia diana warmups/semana (¿varía por centro?)
- [ ] Track A solo vs Track A + Track B en L2 (reconocimiento base vs. gamificación)
- [ ] Criterio operativo para L2 activo · qué señales mira Alex
