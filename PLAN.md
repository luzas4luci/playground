# Plan de implementación · PreZero Cadencia Líderes

> Actualiza este fichero conforme vayas completando pasos.

---

## Estado actual
> Última actualización: 2 junio 2026

---

## FASE 0 · Preparación ✅

- [x] Playbook de comunicaciones completo (`playbook-cadencia-prezero.md`)
- [x] Briefing técnico para implementación (`briefing-implementacion-prezero.md`)
- [x] Repo `cadencia-prezero` creada en `lucia-fisify`
- [x] Repo `fisify-eyes` (Melia) clonada en local

---

## FASE 1 · Entender Melia · Hoy

- [ ] Abrir `fisify-eyes` en Cursor
- [ ] Entender estructura de carpetas
- [ ] Localizar el resolver de cadencia (función que devuelve `{ kind, stage, channel, message }`)
- [ ] Entender cómo fluye un usuario desde registro hasta mensaje
- [ ] Entender cómo se marcan las acciones manuales (`CadenceManualAction`)
- [ ] Entender cómo se conecta el resolver con Customer.io

---

## FASE 2 · Schema de datos · Hoy o mañana

- [ ] Definir todos los campos necesarios en PostHog (ver lista en `briefing-implementacion-prezero.md`)
- [ ] Confirmar con Inhar qué campos ya existen y cuáles hay que añadir
- [ ] Decidir dónde vive la lógica (mismo repo que Melia, repo nueva, o monorepo)

---

## FASE 3 · Lógica de fases

- [ ] Función `getLeaderPhase(user)` → devuelve `L0a | L0b | L1 | L2 | L3`
- [ ] Cron semanal que evalúa transiciones para todos los líderes
- [ ] Tests: líder que sube (L0→L1→L2), líder que cae (L1→L3→L1), líder que no arranca (L0→L0b)

---

## FASE 4 · Resolver L0a

- [ ] Crear `l0aCadence.ts` con estructura de resolver
- [ ] Añadir mensajes (6 variantes por stage: leído/no leído × 3 canales)
- [ ] Test: dado un líder con `nominatedAt` hace X días, devuelve el stage correcto
- [ ] Conectar con Customer.io para envíos reales

---

## FASE 5 · Resolver L1

- [ ] Track A: onboarding único (días 1, 3, 7 desde primer warmup)
- [ ] Push diario: si no reportó warmup hoy → push recordatorio
- [ ] Track B: seguimiento semanal reactivo (1 sem → push+Aurya, 2 sem → WA Lorena, 3 sem → L3)
- [ ] Trigger "en racha" para tier Mantener

---

## FASE 6 · Resolver L2

- [ ] Seguimiento semanal silencioso (misma lógica que L1 Track B, tono suave)
- [ ] Reconocimiento quincenal (Aurya, cada 2 semanas en diana)
- [ ] Email mensual automático (primer lunes de mes)

---

## FASE 7 · Resolver L3

- [ ] Ventana única 21 días: Día 0 → 7 → 14 → 21
- [ ] Exit automático si reporta warmup en cualquier punto → L1
- [ ] Cierre día 21 → L0b

---

## FASE 8 · Transición L1 → L2

- [ ] Push + WA Lorena al llegar a 8 semanas en diana
- [ ] In-app especial (modal/badge) · pendiente spec con Inhar

---

## NO hacer en V1

- Read tracking (leído/no leído) — pendiente implementación técnica
- Criterio automático para L2 activo — Alex lo activa manualmente
- Email mensual — puede esperar a V2
- In-app modal L1→L2 — pendiente con Inhar

---

## Decisiones tomadas

| Decisión | Valor |
|----------|-------|
| Diana Mantener | 7/7 |
| Diana Cerrar gap | 6/7 |
| Diana Recuperar | 5/7 |
| Denominador warmup | Reporte del líder (no % equipo) |
| L2 activo | Alex activa manualmente (`isActiveMobilizer: true`) |
| L3 sub-estados | Ninguno — ventana única 21 días |
| Push diario L1 | Sí, si no ha reportado warmup ese día |

