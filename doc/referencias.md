# Referencias — Loopover

Investigación externa realizada para este proyecto (repositorios, artículos y
herramientas relacionadas con el puzzle Loopover y con búsqueda en espacio de
estado).

## El juego

- **<https://loopover.xyz/>** — juego original de **carykh**; define el puzzle
  de rejilla con filas/columnas circulares.
- **Kata de Codewars «Loopover»** — <https://www.codewars.com/kata/5c1d796370fee68b1e000611>
  (autor: jaybruce1998). Especificación canónica del formato de movimientos
  (`R0`/`L0` filas, `U2`/`D2` columnas), tamaños de 2x2 a 9x9 y estados
  irresolubles (devolver `null`). Nota: Codewars distribuye el reto con
  licencia BSD-2-Clause.
- **DEV Community — Daily Challenge #176** — <https://dev.to/thepracticaldev/daily-challenge-176-loopover-3b23>
  (versión redistribuida del reto de Codewars).
- **Simon Tatham's «Sixteen»** — implementación del mismo mecanismo en
  *Simon Tatham's Portable Puzzle Collection*.
- **OpenProcessing** — sketch original del reto:
  <https://www.openprocessing.org/sketch/576328>.

## Repositorios

### Relevancia alta (Java / mismos algoritmos que el proyecto)

| Repositorio | Notas |
|---|---|
| [`torchlight/loopsolver`](https://github.com/torchlight/loopsolver) | Solver **Java** 4x4 y 5x5 con **IDA***; documenta métrica STM, notación tipo SiGN y heurísticas/PBD en `heuristics.md`. Referencia directa para las Tareas 2 y 3 (búsqueda informada + pattern databases). |
| [`coolcomputery/Loopover-Brute-Force-and-Improvement`](https://github.com/coolcomputery/Loopover-Brute-Force-and-Improvement) | **Java**; BFS por fases (*block-building*), reducciones por simetría, cotas de God's number. Buen material sobre estructuras de estados y exploración exhaustiva. |
| [`alexyuisingwu/sliding-puzzle-database-generator`](https://github.com/alexyuisingwu/sliding-puzzle-database-generator) | **Java**; generador de *disjoint pattern databases* siguiendo a **Korf y Felner**. Base para la Tarea 3 (`PBD`, `HeuristicasPBD`). |
| [`xerotolerance/Loopover`](https://github.com/xerotolerance/Loopover) | **Java 11**; solución al kata de Codewars con generación de movimientos. |
| [`Seanspoons/N-Puzzle-Solver`](https://github.com/Seanspoons/N-Puzzle-Solver) | **Java**; A* con open list (priority queue) y closed list (HashMap), variantes informadas/no informadas. Patrón clásico aplicable a `Busqueda`. |

### Relevancia media/baja (otros lenguajes / otros enfoques)

| Repositorio | Notas |
|---|---|
| [`wezik/loopover-solver`](https://github.com/wezik/loopover-solver) | **Rust** (antes Java); solver CLI, notación `R1`/`L0`/`U3`/`D0`. |
| [`Martan03/loopover`](https://github.com/Martan03/loopover) | **Rust**; implementación TUI del juego. |
| [`JonathanAlderson/LoopOver`](https://github.com/JonathanAlderson/LoopOver) | **Python**; solver + OCR con Tesseract para leer el tablero. |
| [`AndreiToroplean/loopover`](https://github.com/AndreiToroplean/loopover) | **Python**; solución al kata de Codewars, matemática del puzzle. |
| [`zachs18/loopoversolver`](https://github.com/zachs18/loopoversolver) | **Python**; solver interactivo para el sketch de OpenProcessing. |
| [`janispritzkau/loopover`](https://github.com/janispritzkau/loopover) | **Vue/PWA**; juego completo (interesante por la mecánica de movimientos). |
| [`coolcomputery/Loopover-NRG-Upper-Bounds`](https://github.com/coolcomputery/Loopover-NRG-Upper-Bounds) | Variante *no-regrip*; BFS + simulated annealing para cotas. |
| [`coolcomputery/Loopover-Multi-Phase-God-s-Number`](https://github.com/coolcomputery/Loopover-Multi-Phase-God-s-Number) | Deprecado; sustituido por el repo de block-building. |

### Gists y código suelto

- [`torchlight`: Loopover O(n log² n) solver](https://gist.github.com/torchlight/d01b79fa6120cd5854d1a282795fd94a) —
  solver asintóticamente óptimo en STM para tamaños arbitrarios (Shell sort con
  gaps 3-smooth); también un solver 2xn por divide y vencerás.

## Artículos y resultados

- **SpeedSolving — *Loopover god's number upper bounds: 4×4, asymptotics, etc.***
  — <https://www.speedsolving.com/threads/loopover-gods-number-upper-bounds-4%C3%974-asymptotics-etc.75180/>
  Hilo técnico con resultados clave (actualizado a 2023):
  - **God's number 4x4 = 18 STM** (Tomasz Rokicki, con coset solver); 4x4 MTM = 14.
  - 5x5 ≤ 42 STM, 6x6 ≤ 86 STM; cotas generales m×n: `O(N log² N)`.
  - Métricas: **STM** (*single-tile metric*, la del juego) vs **MTM**
    (*multi-tile metric*). Este proyecto usa STM (coste 1 por celda desplazada).
  - Heurísticas descritas: Manhattan, *walking distance* y *enhanced MD*.
  - Representación del tablero como **entero de 64 bits con 4 bits por celda**
    —exactamente el planteamiento de `Estado` en este proyecto— y uso de tablas
    de poda (*pattern databases*) con simetrías.
- **SpeedSolving — *Loopover NRG god's number upper and lower bounds***
  — <https://www.speedsolving.com/threads/loopover-nrg-gods-number-upper-and-lower-bounds.84819/>
- **Korf, R. E. y Felner, A.** — *Disjoint pattern databases*; base teórica de
  `PBD`/`HeuristicasPBD` (ver también el repositorio de
  `alexyuisingwu` más arriba, que lo implementa en Java).

## Notación de movimientos (comparativa)

| Fuente | Formato | Ejemplo |
|---|---|---|
| Este proyecto (`Estado`) | `fila` `columna` `signo` | `01+`, `33-` |
| Codewars / dev.to | dirección + índice | `R0`, `U2` |
| `wezik/loopover-solver` | dirección + índice | `R1`, `D0` |
| `torchlight/loopsolver` (SiGN-like) | índice 1-based + dirección + nº celdas | `1U2`, `1L2'` |
