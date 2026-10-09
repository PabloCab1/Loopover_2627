# 02 — `Estado`: bitboard, acciones y rotaciones

| | |
|---|---|
| **Clase** | `src/Estado.java` (Tarea 1) |
| **Contrato formal** | [tarea-1-estado.md](tarea-1-estado.md) — la spec manda; este documento da profundidad |
| **Modelo** | [01-modelo-del-puzzle.md](01-modelo-del-puzzle.md) |

## 1. Mapa de la clase

| Grupo | Miembros |
|---|---|
| Constantes públicas | `LADO = 4`, `NUM_CASILLAS = 16`, `BITS_POR_CASILLA = 4`, `MASCARA_FICHA = 15L`, `NUM_ACCIONES = 32`, `ACCIONES[32]` (inicializada, no modificar) |
| Constantes privadas | `BITBOARD_RESUELTO`, máscaras de acción, `MASCARA_FILA_TABLERO`, `MASCARA_COLUMNA_TABLERO` |
| Campo | `private final long bitboard` |
| Constructores | `Estado(long)`, `Estado(String)`, `Estado(int[])` |
| Consultas | `bitboard()`, `ficha(int)`, `ficha(int,int)`, `esResuelto()` |
| Sucesores | `sucesores()`, `aplicar(int)` |
| Texto de acciones | `accionComoTexto(int)`, `accionDesde(String)` |
| Igualdad | `equals(Object)`, `hashCode()`, `toString()` |
| Privados | `construirBitboard`, `desplazarFila`, `desplazarColumna`, `comprobarCasilla`, `validarAccion`, `esDigito` (los tres últimos ya están hechos) |

## 2. Memoria: el bitboard

`long` de 64 bits; **cada casilla ocupa 4 bits (un nibble)**:

```
casilla i  →  bits [i·4, i·4+3]      (fila = i/4, columna = i%4)
casilla 0  →  bits [0,3]  (los más bajos)      casilla 15 → bits [60,63]
```

En hexadecimal, el `long` se escribe de nibble 15 (izquierda) a nibble 0
(derecha):

```
Estado resuelto  = 0xFEDCBA98_76543210L
                     ││││││││ ││││││││
nibble (casilla):    F E D C B A 9 8 7 6 5 4 3 2 1 0
```

Es decir: en el estado resuelto, el nibble `i` contiene el valor `i`
(ficha `i` en casilla `i`). Su representación en texto es
`00010203040506070809101112131415`.

Invariante de validez: las 16 fichas de un estado válido son una
**permutación** de `0..15` (sin duplicados, ninguna fuera de rango).

### Mascaras derivadas

| Constante | Valor | Origen |
|---|---|---|
| `MASCARA_FICHA` | `0xF` | 1 nibble |
| `MASCARA_FILA_TABLERO` | `0xFFFF` | 16 bits = 4 nibbles contiguos |
| desplazada a fila `f` | `0xFFFFL << (f·16)` | cada fila ocupa 16 bits |
| `MASCARA_COLUMNA_TABLERO` | `0x000F000F000F000F` | nibble de columna 0 en las 4 filas (separados 16 bits) |
| desplazada a columna `c` | `… << (c·4)` | alinear con la columna `c` |
| nibble de retorno (fila `f`) | `0xFL << (f·16)` | casilla `(f,0)` |
| nibble de retorno (columna `c`) | `0FL << (c·4)` | casilla `(0,c)` |

## 3. Codificación de acciones

5 bits por acción:

```
 bit:   4   3 2   1 0
       signo columna fila
```

- `bit 4` = signo (`1` = `+`, `0` = `-`)
- `bits 3-2` = columna (`0..3`)
- `bits 1-0` = fila (`0..3`)
- `codigoBase = fila | (columna << 2)` (rango 0..15); con signo = `codigoBase | 0b10000`.

`ACCIONES[]` (inicializada en el bloque estático, **no modificar**):

| Índice | Texto | Índice | Texto |
|---|---|---|---|
| 0 | `00+` | 16 | `00-` |
| 1 | `01+` | 17 | `01-` |
| 2 | `02+` | 18 | `02-` |
| … | fila 0, col 2 … | … | … |
| 12–15 | `30+`…`33+` | 28–31 | `30-`…`33-` |

Es decir: `ACCIONES[f·4 + c] = f | (c << 2) | 16` (signo `+`) y
`ACCIONES[16 + f·4 + c] = f | (c << 2)` (signo `-`).

Texto ↔ código: `accionComoTexto(a)` = `fila` + `columna` + signo;
`accionDesde(s)` valida longitud 3, dígitos `0-3` y signo `+/-`, y devuelve el
código. `accionDesde(accionComoTexto(a)) == a` para `a ∈ [0,31]`.

Ejemplos verificados: `accionDesde("21+") == 22` (`2 | 1<<2 | 16`),
`accionComoTexto(22) == "21+"`.

## 4. Aplicar una acción — `aplicar(int)`

```
validarAccion(accion)            →  si no, IllegalArgumentException
fila     = accion & 0b00011
columna  = accion & 0b01100       (bits 2-3)
positivo = (accion & 0b10000) != 0
tmp      = desplazarFila(bitboard, fila, positivo)      ← paso 1
nuevo    = desplazarColumna(tmp, columna, positivo)     ← paso 2
return new Estado(nuevo)                                 (inmutable)
```

### Paso 1 — `desplazarFila` (rotación circular de 4 nibbles)

```
mascaraFila     = MASCARA_FILA_TABLERO << (fila · 16)
mascaraRetorno  = MASCARA_FICHA       << (fila · 16)
extraida        = bitboard & mascaraFila
```

- Derecha (`+`): `rotada = ((extraida << 4) & mascaraFila) | ((extraida >>> 12) & mascaraRetorno)`
- Izquierda (`-`): `mascaraTope = mascaraRetorno << 12`;
  `rotada = ((extraida >>> 4) & mascaraFila) | ((extraida << 12) & mascaraTope)`
- Resultado: `(bitboard & ~mascaraFila) | rotada`

`<< 4` mueve cada nibble una posición hacia bits altos (la pieza avanza de
celda: `(f,c) → (f,c+1)`); el nibble de la casilla más a la derecha sale fuera
de la máscara y entra en el hueco vía `>>> 12 & mascaraRetorno`.

### Paso 2 — `desplazarColumna` (rotación de 4 nibbles cada 16 bits)

```
mascaraColumna  = MASCARA_COLUMNA_TABLERO << (columna · 4)
mascaraRetorno  = MASCARA_FICHA           << (columna · 4)
extraida        = bitboard & mascaraColumna
```

- Abajo (`+`): `rotada = ((extraida << 16) & mascaraColumna) | ((extraida >>> 48) & mascaraRetorno)`
- Arriba (`-`): `mascaraSuperior = 0xF000000000000000L >>> (12 - columna · 4)`;
  `rotada = ((extraida >>> 16) & mascaraColumna) | ((extraida << 48) & mascaraSuperior)`
- Resultado: `(bitboard & ~mascaraColumna) | rotada`

`<< 16` salta exactamente una fila (4 nibbles = 16 bits); `mascaraSuperior`
es el nibble de la fila 3 de la columna elegida (`0xF…` desplazado `12 - c·4`
posiciones hacia la derecha).

`extraida` solo contiene la fila/columna objetivo, por lo que las rotaciones
**nunca arrastran bits de otras filas o columnas**.

## 5. Traza numérica (verificada a mano)

Estado resuelto `R = 00010203040506070809101112131415` aplicando `00+`
(fila 0 a la derecha, luego columna 0 hacia abajo):

```
Antes               Paso 1 (fila 0 ↦)    Paso 2 (columna 0 ↿)
 0  1  2  3           3  0  1  2            12  0  1  2
 4  5  6  7    →      4  5  6  7     →      3  5  6  7
 8  9 10 11           8  9 10 11             4  9 10 11
12 13 14 15          12 13 14 15             8 13 14 15
```

A nivel de bits (fila 0 = word de 16 bits `0x3210`):

```
(0x3210 << 4) & 0xFFFF = 0x2100      (piezas 0,1,2 desplazadas)
(0x3210 >>> 12) & 0xF  = 0x3         (pieza 3 entra por la izquierda)
rotada = 0x2100 | 0x3 = 0x2103       → fila = [3,0,1,2] ✓
```

Columna 0 (word vertical `0xC843` = filas [3,4,8,12] de abajo a arriba):

```
(0xC843 << 16) & máscara  → filas 1,2,3 = 3,4,8   (la fila 3 sale)
(0xC843 >>> 48) & 0xF     = 0xC = 12               (entra en la fila 0)
```

Resultado final: casillas `[12,0,1,2, 3,5,6,7, 4,9,10,11, 8,13,14,15]` →

```
12000102030506070409101108131415
```

Con `-` (`00-`): fila 0 → izquierda `[1,2,3,0]`; columna 0 → arriba →
`04020300080506071209101101131415`.

## 6. `sucesores()`

Itera `ACCIONES[0..31]` en orden y devuelve `new Sucesor(accion, aplicar(accion), 1.0f)`
para cada una (32 elementos, coste unitario). La primera línea de
`verify -s <resuelto>` es:

```
(00+,12000102030506070409101108131415,1.0)
```

y la última: `(33-,<estado>,1.0)`.

## 7. Validaciones y errores

| Método | Condición | Excepción |
|---|---|---|
| `Estado(String)` | longitud ≠ 32 | `IllegalArgumentException` |
| `Estado(String)` | par sin dígito / valor ≥ 16 / duplicado | `IllegalArgumentException` |
| `Estado(int[])` | longitud ≠ 16 | `IllegalArgumentException` |
| `construirBitboard` | ficha fuera de `[0,16)` o duplicada | `IllegalArgumentException` |
| `ficha(int)` | casilla fuera de `[0,16)` | `IllegalArgumentException` (`comprobarCasilla`, ya implementado) |
| `aplicar(int)` | acción fuera de `[0,31]` | `IllegalArgumentException` (`validarAccion`, ya implementado) |
| `accionDesde(String)` | longitud ≠ 3 / fila > 3 / columna > 3 / signo inválido | `IllegalArgumentException` |

Mensajes: descriptivos y con el valor problemático (estilo de los métodos ya
implementados, p. ej. *"Casilla fuera de rango: 16"*).

## 8. Casos de prueba verificados

| # | Entrada | Salida esperada |
|---|---|---|
| T1 | `new Estado("00010203…1415").esResuelto()` | `true` |
| T2 | `ficha(0)`, `ficha(15)`, `ficha(3,3)`, `ficha(2,1)` | `0`, `15`, `15`, `9` |
| T3 | resuelto + `00+` | `12000102030506070409101108131415` |
| T4 | resuelto + `00-` | `04020300080506071209101101131415` |
| T5 | resuelto + `00+,01-` | `00050212030906070413101108011415` |
| T6 | `sucesores().size()` | `32` |
| T7 | misma acción × 7 | vuelve al estado inicial (orden 7, ver `01` §6) |
| T8 | `toString()` del resuelto | `00010203040506070809101112131415` |

## 9. Checklist de implementación (orden sugerido)

1. `Estado(long)` → `this.bitboard = bitboard`.
2. `construirBitboard(int[])` (valida duplicados y rango).
3. `Estado(int[])` → validar longitud + `construirBitboard`.
4. `Estado(String)` → validar longitud, parsear 16 pares, `construirBitboard`.
5. `bitboard()`, `ficha(int)`, `ficha(int,int)`, `esResuelto()`.
6. `desplazarFila` y `desplazarColumna` (§4).
7. `aplicar(int)` (§4) y `sucesores()` (§6).
8. `accionComoTexto`, `accionDesde` (§3).
9. `equals`, `hashCode`, `toString` (`StringBuilder` + cero inicial si `< 10`).
10. Ejecutar los criterios A1–A6 de la spec y los tests T1–T8.

## 10. Quién consume esta clase

| Tarea | Métodos usados |
|---|---|
| 2 — Búsqueda (`Busqueda`, `Nodo`, `Visitados`) | `bitboard()` (clave de `Visitados`), `esResuelto()` (test de solución), `sucesores()`, `aplicar()` |
| 3 — Heurísticas | `ficha(int)` / `ficha(int,int)` (distancias), `bitboard()` (indexado de PBD) |
| CLI | `Estado(String)` con errores controlados, `accionDesde`, `toString` |

## 11. Notas de rendimiento

- **SWAR de nibbles**: las rotaciones son 4 operaciones aritméticas sobre el
  `long` completo (sin bucles, sin arrays); el estado cabe en un `long`, lo que
  hace `equals`/`hashCode` y la tabla de visitados O(1) y sin asignaciones.
- `sucesores()` crea 32 objetos `Estado` + 32 `Sucesor` + 1 `ArrayList` por
  expansión de nodo; aceptable con fastutil/Guava en la frontera (Tarea 2).
  Si se optimiza, cachear la lista no es necesario (los `Estado` son
  inmutables, pero la lista en sí no se reutiliza por seguridad).
- `new Estado(String)` solo se usa en la CLI (una vez por ejecución): la
  validación con máscara de duplicados (p. ej. `long visto` de 16 bits) es O(16)
  y suficiente.
