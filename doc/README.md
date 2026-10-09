# Documentación — Loopover_2627

Proyecto desarrollado bajo **Spec-Driven Development (SDD)**: cada tarea se
especifica antes de implementarse, y la spec se mantiene como fuente de verdad
del contrato que el código debe cumplir.

## Índice de especificaciones

| Spec | Alcance | Estado |
|---|---|---|
| [Tarea 1 — Estado](tarea-1-estado.md) | Bitboard, acciones, sucesores, utilidades de texto | ⬜ Pendiente |
| [Tarea 2 — Búsqueda](tarea-2-busqueda.md) | Motor de búsqueda, `Nodo`, `Frontera`, `Visitados` | ⬜ Pendiente |
| [Tarea 3 — Heurísticas](tarea-3-heuristicas.md) | Manhattan toroidal, permutaciones, Pattern Databases | ⬜ Pendiente |

Leyenda: ⬜ Pendiente · 🟡 En curso · ✅ Implementada y verificada

## Otros documentos

- [Referencias](referencias.md) — repositorios y fuentes externas sobre Loopover.

## Flujo de trabajo

1. **Especificar**: redactar/actualizar la spec de la tarea en `doc/tarea-N-*.md`.
2. **Implementar**: satisfacer los contratos de la spec en `src/`.
3. **Verificar**: ejecutar los criterios de aceptación (comandos CLI) de la spec.
4. **Actualizar**: marcar la tarea ✅ en este índice y anotar decisiones tomadas.

## Convenciones

- Las specs citan `clase:método` tal y como aparecen en `src/` (p. ej.
  `Estado:aplicar`).
- Toda validación falla con `IllegalArgumentException` y mensaje descriptivo
  (salvo indicación en contraria).
- Los Javadoc de `src/` son la referencia de partida de cada spec; si código y
  spec divergen, se decide explícitamente y se anota en *Decisiones*.
