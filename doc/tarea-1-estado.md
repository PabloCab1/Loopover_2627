# Especificación — Tarea 1: `Estado`

| | |
|---|---|
| **Clases** | `Estado`, `Sucesor` (ya implementado) |
| **Depende de** | — |
| **Estado** | ⬜ Pendiente |
| **Fuente de partida** | Javadoc y TODOs de `src/Estado.java` |

## 1. Objetivo

Representar un estado del puzzle Loopover **4x4** como *bitboard* de 64 bits y
producir los 32 sucesores de cada estado aplicando las acciones encadenadas.
`Estado` es la primitiva sobre la que se apoyan la búsqueda (Tarea 2) y las
heurísticas (Tarea 3).

## 2. Modelo e invariantes

- Cada casilla ocupa **4 bits**: la casilla `i` (fila `i / 4`, columna `i % 4`)
  ocupa los bits `[i * 4, i * 4 + 3]`, con la casilla `(0,0)` en los bits más
  bajos. `NUM_CASILLAS = 16`, `BITS_POR_CASILLA = 4`.
- El **bitboard de un estado válido es siempre una permutación** de
  `0..15`: cada ficha aparece exactamente una vez (ficha = valor del nibble).
- **Estado resuelto** = `0xFEDCBA98_76543210L` (nibble `i` = `i`).
  Su representación en texto es `00010203040506070809101112131415`.
- **Acción** (5 bits): `bit 4` = signo (`1` = `+`), `bits 3-2` = columna
  (`0..3`), `bits 1-0` = fila (`0..3`). Rango válido `0..31`
  (`MASCARA_CODIGO_ACCION = 0b11111`).
- La tabla `ACCIONES[32]` **ya está inicializada y no se modifica**:
  `ACCIONES[0..15]` tienen signo `+` (índice `f * 4 + c`),
  `ACCIONES[16..31]` tienen signo `-`.
- **Semántica de una acción** (`fila`, `columna`, `signo`) — *movimiento
  encadenado*, dos desplazamientos en orden fijo:
  1. `desplazarFila(bitboard, fila, positivo)`: con `+` la fila se desplaza
     **a la derecha**; con `-`, **a la izquierda** (retorno circular).
  2. `desplazarColumna(resultado, columna, positivo)`: con `+` la columna se
     desplaza **hacia abajo**; con `-`, **hacia arriba** (retorno circular).
- `Estado` es **inmutable**: `aplicar` devuelve un `Estado` nuevo.

## 3. Contratos de la API pública

### Constructores

| Método | Contrato |
|---|---|
| `Estado(long bitboard)` | Almacena el bitboard tal cual (se asume ya validado por el llamante). |
| `Estado(String representacion)` | Longitud **exactamente 32**; 16 pares de dígitos decimales; cada par en `[0,15]` **sin duplicados**. Cualquier violación → `IllegalArgumentException`. Casilla `i` = par `2i..2i+1`. |
| `Estado(int[] fichas)` | Longitud **exactamente 16**; fichas en `[0,16)` **sin duplicados** → `construirBitboard`. Violación → `IllegalArgumentException`. |

### Consultas

| Método | Contrato |
|---|---|
| `long bitboard()` | Devuelve el bitboard interno (usado por `Visitados`). |
| `int ficha(int casilla)` | Nibble de la casilla: `(bitboard >>> (i * 4)) & 15L`. Fuera de `[0,16)` → `IllegalArgumentException` (vía `comprobarCasilla`). |
| `int ficha(int fila, int columna)` | Delega en `ficha(fila * LADO + columna)`. |
| `boolean esResuelto()` | `bitboard == 0xFEDCBA98_76543210L`. |

### Sucesores y acciones

| Método | Contrato |
|---|---|
| `List<Sucesor> sucesores()` | Lista de **32** sucesores, en el orden de `ACCIONES[]`, cada uno con `costo = 1.0f`. |
| `Estado aplicar(int accion)` | Valida `accion ∈ [0,31]` (vía `validarAccion`, si no → `IllegalArgumentException`) y aplica: extraer fila (bits 0-1), columna (bits 2-3), signo (bit 4) → `desplazarFila` → `desplazarColumna`. Estado receptor intacto. |
| `static String accionComoTexto(int accion)` | Formato `fila + columna + signo` (3 caracteres), p. ej. `01+`, `33-`. |
| `static int accionDesde(String)` | Longitud 3; char 0 = dígito fila `0-3`; char 1 = dígito columna `0-3`; char 2 = `+` o `-`. Viola → `IllegalArgumentException`. `accionDesde(accionComoTexto(a)) == a` para todo `a ∈ [0,31]`. |

### Igualdad y texto

| Método | Contrato |
|---|---|
| `equals` | `instanceof Estado` y mismo `bitboard`. |
| `hashCode` | `Long.hashCode(bitboard)`. |
| `toString()` | 32 dígitos: para cada casilla `i`, `ficha(i)` en **2 dígitos** (cero a la izquierda si `< 10`). Resuelto → `00010203040506070809101112131415`. |

## 4. Utilidades privadas

### `construirBitboard(int[] fichas)`

`resultado |= ((long) fichas[i]) << (i * 4)` para cada casilla. Valida sin
duplicados y rango `[0,16)` → `IllegalArgumentException` si no.

### `desplazarFila(long bitboard, int fila, boolean positivo)`

La fila ocupa 16 bits a partir de `fila * 16`:
`mascaraFila = 0xFFFFL << (fila * 16)`,
`mascaraRetorno = MASCARA_FICHA << (fila * 16)`, `extraida = bitboard & mascaraFila`.

- Derecha (`positivo`): `rotada = ((extraida << 4) & mascaraFila) | ((extraida >>> 12) & mascaraRetorno)`
- Izquierda: `mascaraTope = mascaraRetorno << 12`;
  `rotada = ((extraida >>> 4) & mascaraFila) | ((extraida << 12) & mascaraTope)`

Resultado = `(bitboard & ~mascaraFila) | rotada` (las otras 3 filas intactas).

### `desplazarColumna(long bitboard, int columna, boolean positivo)`

La columna son 4 nibbles separados 16 bits:
`mascaraColumna = 0x000F000F000F000FL << (columna * 4)`,
`mascaraRetorno = MASCARA_FICHA << (columna * 4)`, `extraida = bitboard & mascaraColumna`.

- Abajo (`positivo`): `rotada = ((extraida << 16) & mascaraColumna) | ((extraida >>> 48) & mascaraRetorno)`
- Arriba: `mascaraSuperior = 0xF000000000000000L >>> (12 - columna * 4)`;
  `rotada = ((extraida >>> 16) & mascaraColumna) | ((extraida << 48) & mascaraSuperior)`

Resultado = `(bitboard & ~mascaraColumna) | rotada`.

> `extraida` solo contiene la fila/columna, por lo que las rotaciones no
> arrastran bits de otras filas/columnas.

## 5. Casos de prueba

**Resuelto** `R = 00010203040506070809101112131415`:

| Acción | Resultado esperado (`toString`) | Verificación |
|---|---|---|
| — | `00010203040506070809101112131415` | `esResuelto() == true` |
| `00+` | `12000102030506070409101108131415` | fila 0 → derecha `[3,0,1,2]`; columna 0 → abajo |
| `00-` | `04020300080506071209101101131415` | fila 0 → izquierda `[1,2,3,0]`; columna 0 → arriba |

Detalle del cálculo de `R.aplicar("00+")`:

```
antes:            después de fila 0 ↦:     después de columna 0 ↿:
 0  1  2  3         3  0  1  2              12  0  1  2
 4  5  6  7   →     4  5  6  7       →      3  5  6  7
 8  9 10 11         8  9 10 11               4  9 10 11
12 13 14 15        12 13 14 15               8 13 14 15
```

Otros:
- `ficha(0) == 0`, `ficha(15) == 15`, `ficha(3,3) == 15`, `ficha(2,1) == 9`.
- `R.sucesores().size() == 32`; el sucesor por `ACCIONES[0]` (`00+`) es
  `(00+,12000102030506070409101108131415,1.0)`.
- `accionDesde("21+") == 2 | (1 << 2) | 16 == 22`; `accionComoTexto(22) == "21+"`.
- Errores: `new Estado("")` / `new Estado("0".repeat(31))` /
  `new Estado("x".repeat(32))` / ficha duplicada / ficha `16` →
  `IllegalArgumentException`; `ficha(16)`, `aplicar(32)`, `accionDesde("00")`,
  `accionDesde("40+")`, `accionDesde("00*")` → `IllegalArgumentException`.

## 6. Criterios de aceptación

```bash
mvn -q package

# A1: estado resuelto
java -jar target/loopover.jar verify -s 00010203040506070809101112131415 -a 00+
# → 12000102030506070409101108131415

# A2: inverso
java -jar target/loopover.jar verify -s 00010203040506070809101112131415 -a 00-
# → 04020300080506071209101101131415

# A3: composición de acciones (el estado de partida no se modifica)
java -jar target/loopover.jar verify -s 00010203040506070809101112131415 -a 00+,01-
# → 00050212030906070413101108011415

# A4: los 32 sucesores, en orden de ACCIONES[] y coste 1.0
java -jar target/loopover.jar verify -s 00010203040506070809101112131415
# → 32 líneas "(00+,...,1.0)" … "(33-,...,1.0)"

# A5: errores controlados (exit code 1, mensaje en stderr)
java -jar target/loopover.jar verify -s 000102030405060708091011121314
java -jar target/loopover.jar verify -s 00010203040506070809101112131415 -a 99+
```

## 7. Decisiones abiertas

1. **Orden de `sucesores()`**: se especifica el orden de `ACCIONES[]` (fila,
   columna, `+` primero por bloque de 16). Afecta al desempate de la búsqueda
   (Tarea 2), no a la corrección.
2. **Iguales con mismo bitboard y distinta identidad**: `equals` por bitboard
   implica que `Visitados` puede deduplicar por valor. Confirmar en Tarea 2.
3. **Sin comprobación de paridad/solvabilidad en el constructor**: cualquier
   permutación se acepta. Si el grupo generado por las 32 acciones no cubre
   `16!`, habrá estados sin solución (la Tarea 2 los resolverá con fallo por
   profundidad/límites). Posible mejora futura: chequeo de paridad en
   `Estado`.
