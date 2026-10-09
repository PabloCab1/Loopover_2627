# Especificación — Tarea 3: Heurísticas

| | |
|---|---|
| **Clases** | `Heuristicas`, `HeuristicasPBD`, `PBD` |
| **Depende de** | [Tarea 1 — `Estado`](tarea-1-estado.md) (lectura de fichas); [Tarea 2](tarea-2-busqueda.md) (consumo vía `ToIntFunction<Estado>`) |
| **Estado** | ⬜ Pendiente |
| **Fuente de partida** | Esqueletos de `src/Heuristicas.java`, `src/HeuristicasPBD.java`, `src/PBD.java` y el enum `Heuristica` de `ComandoBusqueda` |

## 1. Objetivo

Proporcionar funciones heurísticas `h: Estado → int` para las estrategias
informadas, con dos niveles: geométricas de bajo coste (toroide) y *pattern
databases* de 8 piezas (PBD), distinguiendo explícitamente cuáles son
**admisibles** (garantizan óptimalidad en A*) y cuáles no.

## 2. Requisitos comunes a toda heurística

1. `h(estado) ≥ 0` siempre, de tipo `int`.
2. `h(resuelto) == 0`.
3. **Admisibilidad** (solo las marcadas como tal): `h(e) ≤ coste óptimo(e)`
   para todo estado alcanzable.
4. Coste: `O(16)` por llamada, **sin asignaciones** en el bucle caliente (A*
   llama a `h` por cada sucesor generado).
5. Estrategias no informadas ignoran `h` → prueban con `-h CERO`.

### Análisis de admisibilidad (marco común)

Una acción encadenada realiza **dos** desplazamientos:
- fila: cambia la coordenada `x` de 4 piezas en ±1 → la suma de distancias
  horizontales puede bajar **≤ 4**;
- columna: cambia la coordenada `y` de 4 piezas en ±1 → la suma de distancias
  verticales puede bajar **≤ 4**.

En un toroide 4x4 la distancia por dimensión `d = min(|Δ|, 4 − |Δ|)` cambia en
≤ 1 por pieza y dimensión. **Descenso máximo por acción: 8.** De ahí que
cotas del tipo `⌈Σdist / 8⌉` sean admisibles y la suma sin dividir, no.

## 3. `Heuristicas` (estáticas)

| Método | Definición propuesta | Admisible |
|---|---|---|
| `heuristicaManhattanToroidal(Estado)` | `S = Σ_i distToroide(ficha(i), casilla i)`, con `distToroide` = `min(|Δf|, 4−|Δf|) + min(|Δc|, 4−|Δc|)`. Devuelve `S` (versión fuerte). | **No** (ver §7.1) |
| `heuristicaManhattanAdmisible(Estado)` | `⌈S / 8⌉` con la misma `S`. | **Sí** (§2) |
| `heuristicaPermutaciones(Estado)` | Cota por estructura de ciclos de la permutación: con `c` ciclos (fijos incluidos), `⌈(16 − c) / 8⌉`. Cada acción afecta ≤ 8 piezas ⇒ puede acercar el estado a la identidad en ≤ 8 «unidades». | **Sí** (propuesta; ver §7.2) |

Donde `ficha(i)` está en `(i/4, i%4)` y su casilla objetivo es
`(f/4, f%4)` (estado resuelto: ficha `f` en casilla `f`).

Ejemplo (estado `00+` sobre el resuelto, del caso A1):

```
pieza en (0,0)=12 → objetivo (3,0): dr=min(3,1)=1, dc=0   → 1
pieza en (0,1)=0  → objetivo (0,0): dr=0, dc=min(1,3)=1    → 1
pieza en (1,0)=3  → objetivo (0,3): dr=1, dc=min(3,1)=1    → 2
… resto de piezas bien colocadas o con dc=1 …
S = Σ distToroide  (valor exacto a fijar con los tests)
```

## 4. `PBD` — Pattern Database de 8 piezas

| Miembro | Contrato |
|---|---|
| `static final int TAMANNO = 32_432_400` | Nº de entradas. **Análisis: `16P8 / 16 = 32.432.400`** → posicionamientos ordenados de 8 piezas en 16 casillas, divididos entre las **16 traslaciones globales del toroide** (simetría: desplazar todo el tablero no cambia las distancias). Ver §7.4 — *hipótesis a confirmar con el fichero*. |
| `PBD(String fichero)` / `PBD(String fichero, int minimo)` | Carga el fichero en memoria (o *memory-map*); `IOException` si no existe/lee mal. |
| `static PBD desdeRecurso(String nombre, int minimo)` | Carga desde recurso de classpath (`/pdb8.dat`, `/pdb8_impares.dat`). |
| `int valor(long tablero)` | Indexa el bitboard → entrada → valor heurístico. |
| `int valor(Estado)` | Delega en `valor(estado.bitboard())`. |
| `int minimo()` | Mínimo almacenado (para normalizar). |

Indexado (a determinar, §7.4): extraer las 8 piezas del patrón, calcular su
posición en el tablero, producir un índice en `[0, TAMANNO)`.

## 5. `HeuristicasPBD`

Carga **ambos** PBD en el constructor (`IOException` si faltan recursos → el
CLI ya lo reporta como *Error de configuración*).

| Recurso | Constante | Contenido |
|---|---|---|
| `/pdb8.dat` | `RECURSO_PARES` | BD de las 8 piezas «pares» |
| `/pdb8_impares.dat` | `RECURSO_IMPARES` | BD de las 8 piezas «impares» |

| Método | Definición | Admisible |
|---|---|---|
| `valorPares(Estado)` | consulta BD de pares | **Sí** (subproblema: `coste(entero) ≥ coste(patrón)`) |
| `valorImpares(Estado)` | consulta BD de impares | **Sí** |
| `valorMaximo(Estado)` | `max(pares, impares)` | **Sí** |
| `valorSuma(Estado)` | `pares + impares` | **No** (§6) |
| `funcion(Tipo)` | devuelve la `ToIntFunction<Estado>` correspondiente | — |

`Tipo`: `PARES_8`, `IMPARES_8`, `PBD_8` (= `valorMaximo`, por defecto),
`PBD_8_SUMA` (= `valorSuma`).

## 6. Por qué `PBD_8_SUMA` no es admisible

Un PBD de un subconjunto de piezas es admisible porque cualquier solución
global resuelve ese subconjunto en ≤ sus movimientos. La **suma** de dos PBD
disjuntas es admisible **solo si ninguna acción afecta piezas de ambos
patrones a la vez**. Aquí una acción desplaza una fila completa y una columna
completa (7-8 piezas distintas), de modo que casi siempre toca piezas pares e
impares: **un solo movimiento reduce ambas columnas**, y `pares + impares`
puede superar el coste real → A* dejaría de garantizar óptimalidad (como
advierte el CLI: *«admisible falso, no garantiza óptimo»*). `max` sí es
admisible porque `max(a,b) ≤ coste` se sigue de `a ≤ coste` y `b ≤ coste`.

## 7. Decisiones abiertas

1. **Fórmula de `heuristicaManhattanToroidal`**: ¿`S` cruda (fuerte, no
   admisible) o `⌈S/8⌉`? El CLI solo marca `PBD_8_SUMA` como no admisible, lo
   que sugiere que **`MANHATTAN` por defecto debe ser admisible** → en ese caso
   `heuristicaManhattanToroidal` devolvería `⌈S/8⌉` y
   `heuristicaManhattanAdmisible` una cota más conservadora (p. ej. con
   distancias **no** envolventes o `⌈S/8⌉` sobre Manhattan clásica).
   *Decisión pendiente de fijar antes de implementar; ambos casos quedan
   cubiertos por los tests de admisibilidad §8.*
2. **`heuristicaPermutaciones`**: fórmula exacta por confirmar; alternativas
   candidatas: `⌈nFueraDeLugar / 8⌉` (más débil) o la basada en ciclos
   (propuesta). Requisito mínimo: ≥0, 0 en el resuelto y admisible.
3. **Reparto pares/impares**: ¿piezas numeradas pares `{0,2,…,14}` vs impares
   `{1,3,…,15}`, o primera/mitad `{0..7}` / `{8..15}`? Afecta al indexado de
   las BD; ambos repartos tienen 8 piezas.
4. **Esquema de indexado y formato de `*.dat`**: la hipótesis
   `16P8/16` (traslaciones del toroide) se confirmará al inspeccionar los
   ficheros. Define: orden de las piezas del patrón, endianness y tipo de
   entrada (¿`int`? 32.432.400 × 4 B ≈ 130 MB por BD; ¿`byte`/`short` con
   `minimo`?).
5. **⚠️ Ficheros `pdb8.dat` / `pdb8_impares.dat` ausentes del repo** y no
   existe generador en el esqueleto. Opciones: (a) implementar un generador
   (BFS inverso desde el patrón resuelto, ~32M estados × 32 acciones),
   (b) obtener las BD de un repositorio externo (`torchlight/loopsolver`
   publica sus PBD), (c) entregar Tarea 3 solo con las heurísticas de
   geometría toroidal y dejar las PBD condicionadas a disponer de los
   ficheros. **Requiere decisión del profesor/equipo.**
6. **Rendimiento**: decidir carga en memoria (*heap* vs `MappedByteBuffer`) al
   construir `HeuristicasPBD`; la primera llamada no debe reabrir ficheros.

## 8. Criterios de aceptación

```bash
# C1: propiedades básicas (todas las heurísticas existentes)
#     h ≥ 0 y h(resuelto) == 0. Verificación con -v sobre el estado resuelto:
java -jar target/loopover.jar solve -s 00010203040506070809101112131415 -h MANHATTAN
# → "(ya resuelto)"   [no debe fallar ni imprimir nada raro]

# C2: admisibilidad sobre estados de coste óptimo conocido (= 1)
#     Para cada una de las 32 acciones a: S_a = a(resuelto) se resuelve en 1
#     movimiento, luego toda heurística admisible debe devolver ≤ 1.
#     Heurísticas a comprobar: MANHATTAN_ADMISIBLE, PARES_8, IMPARES_8,
#     PBD_8, PERMUTACIONES.
#     (Manualmente o con un pequeño driver de pruebas que recorra
#      Estado.sucesores() del estado resuelto.)

# C3: optimalidad con heurística admisible
#     S = estado de B1 (generado con 3 acciones):
#     A* -h MANHATTAN_ADMISIBLE debe devolver una solución tan corta como
#     ANCHURA -h CERO (mismo número de movimientos).

# C4: PBD_8 vs PBD_8_SUMA
#     -h PBD_8 garantiza optimalidad; -h PBD_8_SUMA puede devolver soluciones
#     más largas (comportamiento documentado, no es un fallo).

# C5: recurso PBD ausente → error controlado (exit 1)
java -jar target/loopover.jar solve -s <S> -h PBD_8
# stderr: "Error de configuración: <IOException>"   (cuando no estén los .dat)

# C6: rendimiento: A* con MANHATTAN sobre un estado de profundidad ~10
#     debe terminar en segundos; h se evalúa sin asignaciones (sin basura
#     por llamada — comprobar con -Xmx256m).
```
