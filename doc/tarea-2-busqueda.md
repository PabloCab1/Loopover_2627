# Especificación — Tarea 2: Motor de búsqueda

| | |
|---|---|
| **Clases** | `Busqueda`, `Nodo`, `Frontera`, `Visitados` |
| **Depende de** | [Tarea 1 — `Estado`](tarea-1-estado.md) |
| **Estado** | ⬜ Pendiente |
| **Fuente de partida** | Esqueletos de `src/Busqueda.java`, `src/Nodo.java`, `src/Frontera.java`, `src/Visitados.java` y uso en `src/ComandoBusqueda.java` |

## 1. Objetivo

Implementar un motor de búsqueda genérico sobre el espacio de estados de
`Estado` que soporte las seis estrategias del CLI `solve`, con límites de
profundidad, de nodos en memoria (SMA*) y de estados visitados, y que reporte
estadísticas.

## 2. API de `Busqueda`

### Constructores (jerarquía acumulativa)

```java
Busqueda(Estado inicial, Estrategia e, int profundidadMaxima)
Busqueda(..., int maxNodosArbol)                    // capacidad de la frontera (SMA*)
Busqueda(..., ToIntFunction<Estado> heuristica)
Busqueda(..., long maxVisitados)                    // 0 = ilimitado
```

El CLI siempre usa el último:
`new Busqueda(actual, estrategia, profundidadMaxima, capacidad != null ? capacidad : 10_000_000, heuristica, maxVisitados)`.

Contratos:
- `estrategia == A_ESTRELLA_ACOTADA && maxNodosArbol <= 0` →
  `IllegalArgumentException("A* acotada requiere indicar el maximo de nodos del arbol con -c")`
  (el chequeo también se hace en `ComandoBusqueda` antes de construir).
- `profundidadMaxima < 0`, heurística `null` cuando la estrategia es
  informada, o `maxVisitados < 0` → `IllegalArgumentException`.
- El cronómetro (`tiempoMs`) arranca en la construcción.

### Consultas

| Método | Contrato |
|---|---|
| `Nodo buscar()` | Ejecuta la búsqueda. Devuelve el nodo solución (hoja con `Estado.esResuelto()`), o `null` si se agota la profundidad, se alcanza `maxVisitados` o no hay solución. |
| `int nodosExpandidos()` | Nodos a los que se generaron sucesores. |
| `int estadosVisitados()` | `Visitados.tamano()` — estados únicos registrados. |
| `long tiempoMs()` | Milisegundos de pared desde la construcción hasta el fin de `buscar()`. |
| `boolean limiteVisitadosAlcanzado()` | `true` si `buscar()` terminó porque `maxVisitados > 0 && estadosVisitados() >= maxVisitados`. |

## 3. Semántica de las estrategias

| Estrategia | Prioridad de selección | Óptimo | Notas |
|---|---|---|---|
| `PROFUNDIDAD` | LIFO (pila) | No | Respeta `profundidadMaxima`. |
| `ANCHURA` | FIFO (cola) | Sí (coste unitario) | Respeta `profundidadMaxima`. |
| `COSTO_UNIFORME` | `g` creciente | Sí | |
| `VORAZ` | `h` creciente | No | Con `h = 0` degenera en anchura por valores. |
| `A_ESTRELLA` | `f = g + h` creciente | Sí **si `h` es admisible** | Con `h = 0` equivale a costo uniforme. |
| `A_ESTRELLA_ACOTADA` | `f` creciente con **frontera acotada** a `maxNodosArbol` (SMA*) | Sí (con reinserciones) | Usa `Frontera.eliminarPeorHojaNoRaiz` y `Nodo.respaldarMinimo`. |

- Las estrategias **no informadas ignoran la heurística** (`-h CERO` es la
  opción recomendada para probarlas).
- Con `-h CERO`, `A_ESTRELLA` = `COSTO_UNIFORME` (costes todos 1.0): ambas
  deben devolver soluciones de **igual longitud** que `ANCHURA`.
- Desempate: por orden de generación (insertar después → peor prioridad),
  para que `ANCHURA` sea FIFO y `PROFUNDIDAD` sea LIFO de forma estable.

## 4. `Nodo`

### Constructores y estado

- `Nodo(Estado)` — raíz: `padre = null`, `profundidad = 0`, `costoAcumulado = 0`.
- `Nodo(Nodo padre, int accion)` — `costoAcumulado = padre.costoAcumulado() + 1.0f`.
- `Nodo(Nodo padre, int accion, float costoAccion)` — coste arbitrario (prepara
  costes no uniformes).

### Atributos y métodos

| Método | Significado |
|---|---|
| `estado()`, `padre()`, `accion()`, `profundidad()` | Datos del nodo. En la raíz, `accion()` es indefinido (convención: `-1`). |
| `costoAcumulado()` | `g`: suma de costes de la raíz al nodo. |
| `fijarHeuristica(int h)` / `valor()` | `h` (≥ 0) y el **valor de prioridad** `f` según estrategia: `g + h` (A*), `g` (coste uniforme), `h` (voraz), `profundidad` (anchura/profundidad). |
| `fijarValor(int v)` | Fija el valor de prioridad directamente (usado por `Busqueda` al expandir). |
| `marcarExpandido()` / `completado()` | El nodo ya generó todos sus sucesores. |
| `agotado()` | El nodo no tiene más hijos por generar/reexpandir (podado por profundidad, frontera llena o todos los hijos ya registrados). |
| `restarHijo()` | Decrementa el contador de hijos vivos (al eliminar un hijo de la frontera en SMA*). |
| `reexpandir()` | Deshace `completado`/`agotado` para que SMA* vuelva a generar sus hijos. |
| `respaldarMinimo(int v)` | Guarda el mejor valor descendido de un hijo eliminado, para **propagar el fallo hacia arriba** (SMA*): el padre no puede tener valor menor que el mínimo de sus hijos. |
| `esRaiz()` | `padre == null` (nunca se elimina de la frontera). |
| `compareTo(Nodo)` | Orden natural **ascendente por `valor`** (menor = mejor); desempate por orden de inserción. |
| `camino()` | `ObjectList<Nodo>` de la raíz al nodo (fastutil), para reconstruir la solución. |
| `caminoComoTexto()` | Acciones del camino como texto, separadas por comas, p. ej. `00+,01-,33+` (formato de `-a` de `verify`). Vacío si es la raíz. |
| `equals`/`hashCode` | Por estado + profundidad + acción (o simplemente identidad); decisión abierta §7. |

## 5. `Frontera` (heap min-max)

| Método | Contrato |
|---|---|
| `Frontera(int maxNodos)` | Capacidad; `maxNodos > 0`. |
| `int tamano()` / `boolean vacia()` | Tamaño actual. |
| `boolean contiene(Nodo)` | Pertenencia (búsqueda eficiente, no lineal). |
| `void insertar(Nodo)` | Respeta el orden de `compareTo`; si está lanza `IllegalStateException` (SMA* debe podar antes de insertar). |
| `Nodo extraerMejor()` | Extrae el mínimo (raíz del heap); vacía → `null`. |
| `Nodo eliminarPeorHojaNoRaiz()` | Extrae la **hoja con mayor `valor`** que no sea la raíz, **en O(log n)** usando que un heap min-max mantiene el máximo localizable; mantiene la invariante de que solo las hojas del árbol de búsqueda (nodos no expandidos con 0 hijos vivos) son candidatas. **Marcado «Tarea 3» en el esqueleto: ver decisión abierta §7.3.** |

## 6. `Visitados`

| Método | Contrato |
|---|---|
| `boolean registrarSiMejor(long estado, float valor)` | `true` si el estado no estaba registrado, o si su nuevo valor es **estrictamente menor** (mejor) que el registrado; en ese caso **actualiza** el valor. `false` si ya estaba con un valor ≤ `valor`. |
| `int tamano()` | Nº de estados únicos registrados. |

Implementación prevista: `Long2FloatOpenHashMap` de fastutil (clave = bitboard).

Uso en `buscar()`: para cada sucesor, `registrarSiMejor(sucesor.bitboard(), valor)`
decide si el nodo se inserta en la frontera. La raíz se registra con su valor.

## 7. Decisiones abiertas

1. **Estrategias informadas y heurística no admisible**: con
   `MANHATTAN` por defecto (Tarea 3), `A_ESTRELLA` puede no ser óptimo. La
   solución solo debe ser *correcta* (llega al resuelto); la optimalidad se
   exige con heurísticas admisibles (`CERO`, `MANHATTAN_ADMISIBLE`, `PBD_8`).
2. **Tie-breaking**: orden de inserción (§3) para reproducir FIFO/LIFO.
3. **`eliminarPeorHojaNoRaiz` está etiquetado «Tarea 3»** pero `A_ESTRELLA_ACOTADA`
   (Tarea 2) lo necesita. **Decisión: implementarlo ya en Tarea 2** (heap
   min-max completo) y reservar para la Tarea 3 solo el uso intensivo de la
   frontera acotada. Alternativa: en Tarea 2, `A_ESTRELLA_ACOTADA` aborta con
   `null` cuando la frontera se llena.
4. **Estados sin solución**: no hay comprobación de paridad. Si un estado es
   inalcanzable, `buscar()` devuelve `null` por profundidad/límite y el CLI
   informa del límite alcanzado. Opcional (mejora): detectar el agotamiento
   real de la frontera y distinguirlo de los límites.
5. **Memoria de `Nodo`**: mantener vivos los padres (referencias) es suficiente
   para `camino()`; `Visitados` no referencia nodos, solo bitboards.

## 8. Criterios de aceptación

Primero se construye un estado de prueba y su solución se valida de ida y
vuelta (round-trip):

```bash
# B1: generar un estado de partida (resuelto + 3 acciones conocidas)
java -jar target/loopover.jar verify -s 00010203040506070809101112131415 -a 00+,12-,33+
# → S

# B2: A* con h=0 resuelve y la solución es válida (round-trip con verify)
java -jar target/loopover.jar solve -s <S> -e A_ESTRELLA -h CERO -v
# → lista de acciones; luego:
java -jar target/loopover.jar verify -s <S> -a <lista devuelta>
# → 00010203040506070809101112131415

# B3: optimalidad relativa: la solución encontrada tiene ≤ 3 movimientos
#     (la secuencia usada para generar S ya es una solución de 3)

# B4: consistencia entre estrategias (h=0): ANCHURA, COSTO_UNIFORME y
#     A_ESTRELLA -h CERO devuelven soluciones de la misma longitud

# B5: todas las estrategias devuelven soluciones válidas (round-trip)
for e in PROFUNDIDAD ANCHURA COSTO_UNIFORME VORAZ A_ESTRELLA; do
  java -jar target/loopover.jar solve -s <S> -e $e -h CERO
done

# B6: SMA* acotada en memoria
java -jar target/loopover.jar solve -s <S> -e A_ESTRELLA_ACOTADA -c 1000 -h CERO -v
# → solución válida con la frontera limitada a 1000 nodos

# B7: -c obligatorio para A_ESTRELLA_ACOTADA (exit 1)
java -jar target/loopover.jar solve -s <S> -e A_ESTRELLA_ACOTADA
# stderr: "Error de configuración: A* acotada requiere indicar el maximo de nodos del arbol con -c"

# B8: límite de profundidad (exit 1); -p 0 nunca expande la raíz
java -jar target/loopover.jar solve -s <S> -e ANCHURA -p 0
# stderr: "No se encontró solución dentro del límite de profundidad 0."

# B9: límite de estados visitados (exit 1); -m 1 aborta en el primer registro
java -jar target/loopover.jar solve -s <S> -e A_ESTRELLA -h CERO -m 1
# stderr: "Búsqueda abortada: se alcanzó el límite de 1 estados visitados. ..."

# B10: estado ya resuelto
java -jar target/loopover.jar solve -s 00010203040506070809101112131415
# → "(ya resuelto)"

# B11: -v reporta estadísticas coherentes:
#     profundidad = nº de acciones devueltas,
#     estadosVisitados ≥ nodosExpandidos ≥ 0, tiempo ≥ 0
```
