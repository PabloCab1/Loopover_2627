# Índice de documentos — Loopover_2627

Proyecto desarrollado bajo **Spec-Driven Development (SDD)**: cada tarea se
especifica antes de implementarse, y la spec se mantiene como fuente de verdad
del contrato que el código debe cumplir. Además de las specs, la carpeta
`doc/` acumula documentación técnica por tema (estado, búsqueda, heurísticas…).

## Especificaciones (SDD)

| Documento | Contenido | Estado |
|---|---|---|
| [tarea-1-estado.md](tarea-1-estado.md) | Contrato de `Estado`: bitboard, acciones, sucesores, errores y criterios de aceptación | ⬜ Pendiente |
| [tarea-2-busqueda.md](tarea-2-busqueda.md) | Motor de búsqueda, `Nodo`, `Frontera`, `Visitados`; SMA* y límites | ⬜ Pendiente |
| [tarea-3-heuristicas.md](tarea-3-heuristicas.md) | Manhattan toroidal, permutaciones, Pattern Databases | ⬜ Pendiente |

Leyenda: ⬜ Pendiente · 🟡 En curso · ✅ Implementada y verificada

## Documentación técnica

| Documento | Contenido |
|---|---|
| [01-modelo-del-puzzle.md](01-modelo-del-puzzle.md) | Modelo del puzzle: toroide sin huecos, celdas `00..33`, mecánica de 2 pasos, propiedades del grupo de movimientos. Incluye la guía de modelado (`imagenes/guia-modelo-4x4.png`) |
| [02-estado-y-bitboard.md](02-estado-y-bitboard.md) | `Estado.java` a fondo: memoria bit a bit, máscaras, codificación de acciones, trazas de rotación, validaciones, casos de prueba y checklist de implementación |
| — | *Reservados para Tareas 2 y 3:* `03` frontera/min-max, `04` SMA*, `05` heurísticas, `06` PBD, `07` visitados y estrategias, `08` CLI, `09` dependencias, `10` auditoría de tareas, `11` baterías/entregas, `12` correcciones y regresiones |

## Otros documentos

| Documento | Contenido |
|---|---|
| [referencias.md](referencias.md) | Repositorios y fuentes externas sobre Loopover |

## Flujo de trabajo

1. **Especificar**: redactar/actualizar la spec de la tarea en `doc/tarea-N-*.md`.
2. **Implementar**: satisfacer los contratos de la spec en `src/`
   (la guía técnica `NN-*.md` de la tarea orienta la implementación).
3. **Verificar**: ejecutar los criterios de aceptación (comandos CLI) de la spec.
4. **Actualizar**: marcar la tarea ✅ en este índice y anotar decisiones tomadas.

## Convenciones

- Las specs citan `clase:método` tal y como aparecen en `src/` (p. ej.
  `Estado:aplicar`); son la **fuente de verdad**. Los documentos técnicos
  numerados dan profundidad y ejemplos, y enlazan a la spec.
- Toda validación falla con `IllegalArgumentException` y mensaje descriptivo
  (salvo indicación en contraria).
- Los Javadoc de `src/` son la referencia de partida de cada spec; si código y
  spec divergen, se decide explícitamente y se anota en *Decisiones*.
- Estado del documento: se marca en la tabla de especificaciones (no aquí).
