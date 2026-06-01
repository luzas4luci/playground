# Playbook de cadencia · PreZero Líderes
> Versión 1 · base para implementación

---

## Quién habla en cada canal

| Canal | Voz | Tono |
|-------|-----|------|
| WhatsApp | **Lorena** (Customer Success) | Humano, directo, empático |
| Chat in-app | **Aurya** | Cercano, informal, motivador |
| Push | Notificación del sistema | Corto, accionable |

---

## Variables de plantilla

| Variable | Valor |
|----------|-------|
| `{Nombre}` | Nombre del líder |
| `{Centro}` | Nombre del centro (ej. "Talavera") |
| `{Diana}` | Diana del tier (ej. "7/7", "6/7", "5/7") |
| `{Días}` | Días con warmup esta semana |
| `{Semanas}` | Nº de semanas consecutivas cumpliendo/fallando |

---

## Diagrama de flujo entre fases

```
Nominación (isLeader: true)
  └─ L0a · Recién nominado
       ├─ → L1  en cuanto reporta 1er warmup grupal
       └─ → L0b  21d sin reportar warmup (pausa · revisión trimestral)

L1 · Activo
  ├─ → L2  8 sem consecutivas en diana del tier
  └─ → L3  3 sem consecutivas bajo umbral  (o 2 sem sin reportar nada)

L2 · Consolidado
  └─ → L1  3 sem consecutivas perdiendo diana
       (L2 nunca cae directo a L3 · doble pared de colchón)

L3 · En riesgo
  ├─ → L1  recupera dentro de la ventana de rescate
  └─ → L0b  rescate fallido (L3a: sin retomar en 14d · L3b: sin retomar en 21d)
```

---

## Tiers de centro · Diana operativa

| Tier | Diana | Umbral (L3 activa) | Umbral crítico (L3b) |
|------|-------|---------------------|----------------------|
| Mantener | 7/7 | 5/7 | 3/7 |
| Cerrar gap | ≥6/7 | 4/7 | 2/7 |
| Recuperar | ≥5/7 | 3/7 | 1/7 |

---

---

# L0a · Recién nominado

**Objetivo:** Conseguir el primer warmup grupal en ≤21 días.
**Tiempo:** Días naturales desde la nominación.
**Ventana total:** 21 días.

---

## Día 1 · WhatsApp de bienvenida

| | |
|--|--|
| **Canal** | WhatsApp |
| **Quién** | Lorena |
| **Cuándo** | Día 1 desde nominación |
| **Completa** | Manual (marcar enviado) |

**Mensaje:**
> Hola {Nombre}, soy Lorena del equipo de PreZero. Alex me ha dicho que eres el nuevo líder de warmup en {Centro} 💪 Estoy aquí para ayudarte a arrancar. ¿Tienes un momento esta semana para hacer el primer warmup con tu equipo? Si tienes cualquier duda de cómo funciona, me escribes aquí y te ayudo.

---

## Día 3 · Push de activación

| | |
|--|--|
| **Canal** | Push |
| **Cuándo** | Día 3 desde nominación |
| **Condición** | Solo si tiene notificaciones activas |
| **Sin push** | No bloqueante · se marca automáticamente |
| **Completa** | Manual (marcar enviado) |

**Texto push:**
> {Nombre}, tu equipo te espera 💪 ¿Hoy es el día del primer warmup?

---

## Día 7 · Aviso 2

| | |
|--|--|
| **Cuándo** | Día 7 desde nominación |
| **Bloqueo previo** | Día 3 completado o marcado no bloqueante |

**Árbol de canal (prioridad: chat → push → WhatsApp):**

```
1. ¿Tiene chat in-app activo?
   └── SÍ → Aurya in-app  ← automático por mensaje de Aurya

2. ¿Tiene push activo?
   └── SÍ → Push  ← MANUAL

3. ¿Tiene WhatsApp?
   └── SÍ → WhatsApp Lorena  ← MANUAL
```

**Mensaje Aurya (chat in-app):**
> Hola {Nombre} 👋 ¿Cómo está yendo la semana en {Centro}? Lorena me dijo que eres el nuevo líder de warmup. Si quieres, cuéntame cómo está el equipo y te ayudo a preparar el primero.

**Texto push:**
> {Nombre}, ¿ya organizaste el primer warmup? ¡Tu equipo lo está esperando! 🔥

**Mensaje WhatsApp Lorena:**
> Hola {Nombre}, te escribo de nuevo para ver cómo va la semana. ¿Has podido organizar algún warmup con el equipo? Si hay algo que te lo esté poniendo difícil (horarios, participación, lo que sea), me cuentas y lo vemos juntos.

---

## Día 14 · Aviso 3

| | |
|--|--|
| **Cuándo** | Día 14 desde nominación |
| **Bloqueo previo** | Día 7 completado |

**Árbol de canal:**

```
1. ¿Tiene chat in-app activo?
   └── SÍ → Aurya in-app  ← automático

2. ¿Tiene push activo?
   └── SÍ → Push  ← MANUAL

3. ¿Tiene WhatsApp?
   └── SÍ → WhatsApp Lorena  ← MANUAL
```

**Mensaje Aurya (chat in-app):**
> {Nombre}, llevamos dos semanas desde que te nombraron líder. Es normal que arrancar cueste — los primeros warmups son los más raros. Pero una vez que el equipo engancha, va solo. ¿Qué es lo que más te está frenando ahora mismo?

**Texto push:**
> {Nombre}, quedan pocos días para que el primer warmup cuente 🕐 ¿Hoy puede ser?

**Mensaje WhatsApp Lorena:**
> Hola {Nombre}, soy Lorena. Sé que arrancar no siempre es fácil, pero quería escribirte antes de que se acabe el mes. Si el problema es el equipo, los horarios o simplemente no saber cómo empezar, dímelo y lo resolvemos juntos. Tú tienes el rol, solo falta dar el primer paso.

---

## Día 21 · Cierre / paso a L0b

| | |
|--|--|
| **Cuándo** | Día 21 desde nominación |
| **Bloqueo previo** | Día 14 completado |
| **Completa** | Manual · último paso antes de marcar L0b |

**Árbol de canal:**

```
1. ¿Tiene chat in-app activo?
   └── SÍ → Aurya in-app  ← automático

2. WhatsApp Lorena  ← MANUAL (siempre, como cierre)
```

**Mensaje Aurya (chat in-app):**
> {Nombre}, vamos a dejarlo aquí por ahora. Alex revisará tu caso en la próxima revisión trimestral. Si en cualquier momento quieres retomar el rol, un mensaje aquí y lo activamos al instante.

**Mensaje WhatsApp Lorena:**
> Hola {Nombre}, te escribo para cerrar el loop de este mes. Como no hemos podido arrancar con los warmups, vamos a pausar por ahora. Pero el rol sigue siendo tuyo si quieres retomarlo — solo escríbeme y lo reactivamos. Alex lo verá en la próxima revisión. Un saludo.

> 🔴 **Acción:** Marcar como L0b · intentado.

---

---

# L1 · Activo

**Objetivo:** Sostener los warmups grupales dentro de la diana del tier durante 8 semanas consecutivas para consolidar en L2.
**Tiempo:** Semanas de reporte.
**Ancla de seguimiento:** Último reporte semanal de warmups.

El cuadrante L1 modula el **tono** pero no la **estructura**:

| Warmups | Plan propio | Lectura | Modulación |
|---------|-------------|---------|------------|
| Sí | Sí | L1 pleno · ideal | Playbook base sin cambio |
| Sí | No | Cumple pero no es ejemplo | Push extra semanal · "predica con el ejemplo" |
| No | Sí | Líder dormido | Baja a L3 · tag rescate suave |
| No | No | Caído | Baja a L3 · tag rescate completo |

---

## Track A · Seguimiento semanal (cada 7 días)

**Cuándo:** Cada semana de reporte, si el líder está en diana.
**Canal principal:** Chat in-app (Aurya) · automático si hay mensaje posterior al reporte.

**Mensaje Aurya (semana normal, en diana):**
> {Nombre}, {Días}/7 esta semana 💪 El equipo está respondiendo. ¿Algo que quieras ajustar para la semana que viene?

**Mensaje Aurya (semana con plan propio, sin warmup grupal — L1 "cumple no es ejemplo"):**
> {Nombre}, esta semana hiciste tu plan personal pero el warmup grupal no salió. El equipo nota cuando el líder lleva al grupo — ¿qué pasó esta semana?

---

## Track B · Señal de caída (reactivo)

Se activa cuando el líder cae bajo umbral durante **1 semana aislada** (no consecutiva).

> ⚠️ Este trigger es cross-fase y también aplica en L2. Se llama "Tu equipo está en racha · no lo cortes hoy".

**Condición:** Líder en tier Mantener (7/7) cae a 5/7 en 1 semana aislada.
**Frecuencia máxima:** 1 vez cada 14 días.
**Canal:** Push + chat in-app.

**Push:**
> {Nombre}, esta semana bajó un poco el ritmo. ¿Todo bien en {Centro}?

**Mensaje Aurya (chat in-app):**
> Oye {Nombre}, vi que esta semana bajó el warmup grupal. No pasa nada si fue algo puntual — ¿qué pasó? Así vemos si hay algo que ajustar o si fue solo una semana rara.

---

## Semana 2 consecutiva bajo umbral · Aviso

**Cuándo:** 2 semanas seguidas bajo umbral (no crítico aún).
**Canal:** Chat in-app + WhatsApp Lorena.

**Mensaje Aurya (chat in-app):**
> {Nombre}, llevan dos semanas con el warmup por debajo de lo esperado en {Centro}. ¿Hay algo que esté pasando en el equipo o en la operativa que lo esté frenando?

**Mensaje WhatsApp Lorena:**
> Hola {Nombre}, soy Lorena. He visto que las últimas dos semanas el warmup grupal ha bajado un poco. ¿Puedo ayudarte con algo? Cuéntame qué está pasando y lo vemos juntos.

---

## Transición L1 → L2 · Momento especial (8 sem en diana)

**Cuándo:** 8 semanas consecutivas cumpliendo diana del tier.
**Canales:** Push + WhatsApp Lorena + in-app especial (modal / badge · por definir con Inhar).

**Push:**
> {Nombre}, 8 semanas seguidas en diana 🏆 ¡Eso es consolidación! Ya eres Líder Consolidado en {Centro}.

**Mensaje WhatsApp Lorena:**
> {Nombre}!! 🎉 8 semanas seguidas cumpliendo el objetivo en {Centro}. Eso no es casualidad — es que el equipo ya lo ha interiorizado. Enhorabuena, estás en un nivel diferente. Alex lo sabe y lo valora.

**In-app:** Modal de celebración / badge (especificación pendiente con Inhar).

---

---

# L2 · Consolidado

**Objetivo:** Mantener la rutina con apoyo mínimo. Detectar caída antes de que llegue a L3.
**Cadencia:** Quincenal (cada 2 semanas).

---

## Seguimiento quincenal · Reconocimiento

**Canal:** Chat in-app (Aurya).

**Mensaje Aurya:**
> {Nombre}, semana a semana 💪 El warmup en {Centro} sigue en marcha. Llevas {Semanas} semanas siendo un referente. ¿Algo nuevo con el equipo que quieras contarme?

---

## Email mensual de resultados

**Canal:** Email · automático.
**Cuándo:** Primer lunes de cada mes.

**Asunto:** Resumen de warmup · {Centro} · {Mes}

**Cuerpo:**
> Hola {Nombre},
>
> Aquí tienes el resumen de warmup de {Centro} en {Mes}:
>
> - Días con warmup grupal: **{Total días}**
> - Media semanal: **{Media}/7**
> - Semanas consecutivas en diana: **{Semanas}**
>
> El equipo lleva una racha sólida. Sigue así.
>
> Un saludo,
> El equipo de Fisify

---

## Señal de caída en L2 (trigger reactivo · igual que L1)

**Condición:** 1 semana por debajo del tier diana.
**Canal:** Push + chat in-app.

*(Mismos mensajes que Track B de L1)*

---

## Transición L2 → L1 (3 sem perdiendo diana)

No hay mensaje de aviso progresivo — la bajada es automática a L1 cuando se cumplen 3 semanas.
En L1 vuelve a recibir cadencia activa semanal.

> L2 **nunca cae directo a L3** — pasa por L1 primero (doble pared de colchón).

---

---

# L3a · Rescate suave

**Disparador:** 3 semanas consecutivas bajo umbral (con rehab plan activo al entrar).
**Ventana:** 14 días para retomar.
**Tono:** Empático · pregunta diagnóstica · no agresivo.
**Objetivo:** Entender qué pasó y reactivar los warmups grupales.

---

## Día 0 · Entrada a L3a

**Canal:** Chat in-app (Aurya) + WhatsApp Lorena.

**Mensaje Aurya (chat in-app):**
> {Nombre}, he notado que las últimas semanas el warmup grupal ha bajado bastante en {Centro}. No te escribo para presionarte — a veces hay semanas malas, cambios de turno, cosas de la operativa que se complican. ¿Qué está pasando?

**Mensaje WhatsApp Lorena:**
> Hola {Nombre}, soy Lorena. Alex me ha dicho que las últimas semanas han sido complicadas en {Centro} con el warmup. ¿Tienes 10 minutos esta semana para contarme qué está pasando? Quiero entenderlo antes de hacer nada.

---

## Día 7 · Seguimiento L3a

**Canal:** Chat in-app (Aurya).

**Mensaje Aurya:**
> {Nombre}, ¿cómo va la semana? ¿Has podido organizar algún warmup grupal? Si hay algo que esté bloqueando al equipo, cuéntame — a veces hay que ajustar el formato, el horario o simplemente recuperar el hábito de a poco.

---

## Día 14 · Límite L3a

**Sin retomar → escala a L3b.**

**Canal:** WhatsApp Lorena + push.

**Push:**
> {Nombre}, llevamos dos semanas sin warmup grupal en {Centro}. ¿Hablamos esta semana?

**Mensaje WhatsApp Lorena:**
> {Nombre}, han pasado dos semanas desde que empezamos el rescate y todavía no hemos podido retomar el ritmo. Necesito que hablemos — ¿tienes un hueco esta semana para una llamada rápida con Alex? Es para ver si podemos ayudarte o si el rol necesita revisarse.

> 🔴 **Acción:** Si no retoma → marcar L3b.

---

---

# L3b · Rescate completo

**Disparador:** Desde L1 sin rehab plan · o escalada desde L3a sin retomar en 14d.
**Ventana:** 21 días para retomar.
**Tono:** Directo · urgente · con oferta de videollamada.
**Objetivo:** Retomar warmups o cerrar el rol.

---

## Día 0 · Entrada a L3b

**Canal:** WhatsApp Lorena (directo y sin rodeos) + push.

**Push:**
> {Nombre}, el warmup en {Centro} lleva semanas parado. Necesitamos hablar.

**Mensaje WhatsApp Lorena:**
> Hola {Nombre}, soy Lorena. Voy al grano: el warmup grupal en {Centro} lleva varias semanas por debajo del mínimo y necesitamos resolverlo. ¿Puedes hacer una videollamada con Alex esta semana? No es para revisar lo que pasó — es para ver cómo seguimos. Dime cuándo y lo organizo.

---

## Día 7 · Seguimiento L3b

**Canal:** Chat in-app (Aurya) + push.

**Push:**
> {Nombre}, ¿pudiste hablar con Alex? El equipo de {Centro} te necesita esta semana.

**Mensaje Aurya (chat in-app):**
> {Nombre}, ¿cómo va? Si la semana pasada no pudiste hacer la llamada con Alex, dime cuándo puedes y lo buscamos. También puedes contarme aquí qué está pasando si lo prefieres.

---

## Día 14 · Aviso crítico

**Canal:** WhatsApp Lorena.

**Mensaje WhatsApp Lorena:**
> {Nombre}, llevamos dos semanas en rescate y no hemos podido retomar el ritmo ni hablar con Alex. Entiendo que puede haber circunstancias fuera de tu control, pero necesito saber si quieres seguir con el rol. ¿Me escribes antes del {fecha}?

---

## Día 21 · Cierre L3b → L3c / L0b

**Sin retomar → L3c (rescate fallido) → L0b**.

**Canal:** WhatsApp Lorena.

**Mensaje WhatsApp Lorena:**
> {Nombre}, cerramos este ciclo aquí. Como no hemos podido retomar el warmup grupal en {Centro}, Alex va a revisar el rol en la próxima revisión trimestral. El rol puede volver a ser tuyo si quieres retomarlo — solo escríbeme. Un saludo.

> 🔴 **Acción:** Marcar L3c → L0b. Sin cadencia activa. Espera revisión trimestral.

---

---

# Resumen visual de mensajes por fase

```
L0a · Recién nominado (21 días)
  Día 1   WhatsApp Lorena    Bienvenida + primer paso
  Día 3   Push               Activación (si push activo)
  Día 7   Chat/Push/WA       Aviso 2 según canal disponible
  Día 14  Chat/Push/WA       Aviso 3 · tono más directo
  Día 21  Chat + WA          Cierre → L0b

L1 · Activo (semanal)
  Cada sem   Chat Aurya      Seguimiento semanal en diana
  1 sem baja Push + Chat     Señal de caída reactiva
  2 sem baja Chat + WA       Aviso · diagnóstico
  8 sem OK   Push + WA + App Celebración → L2

L2 · Consolidado (quincenal)
  Cada 2 sem Chat Aurya      Reconocimiento
  Mensual    Email           Resumen de resultados
  1 sem baja Push + Chat     Señal de caída reactiva

L3a · Rescate suave (14 días)
  Día 0   Chat + WA          Diagnóstico · ¿qué pasó?
  Día 7   Chat               Seguimiento
  Día 14  Push + WA          Límite → L3b si no retoma

L3b · Rescate completo (21 días)
  Día 0   Push + WA          Directo · oferta videollamada
  Día 7   Push + Chat        Seguimiento
  Día 14  WA                 Aviso crítico
  Día 21  WA                 Cierre → L3c / L0b
```

---

# Pendiente de definir

- [ ] Mensajes distintos si leyó / no leyó el anterior (cuando read tracking esté disponible)
- [ ] Wording de videollamada L3b · capacidad operativa de Alex
- [ ] In-app especial en transición L1→L2 (modal / badge / animación) · con Inhar
- [ ] Criterio operativo para re-nominación trimestral desde L3c · con Alex
- [ ] Track A solo vs Track A + Track B en L2 (reconocimiento base vs. gamificación)
- [ ] Implementación del cron semanal que evalúa transiciones · con Inhar
