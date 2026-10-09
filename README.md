# Loopover_2627

Solucionador del puzzle **Loopover 4x4** mediante búsqueda en espacio de estado
con movimientos encadenados. Proyecto Java 17 + Maven, con CLI basada en picocli.

> La documentación completa (especificaciones y referencias) vive en [`doc/`](doc/README.md).

## Compilar

```bash
mvn package
```

Genera el jar con shade en `target/loopover.jar` (clase principal `LoopoverApp`).

## Uso

### `verify` — comprobar un estado y aplicar acciones

```bash
java -jar target/loopover.jar verify -s 00010203040506070809101112131415
java -jar target/loopover.jar verify -s 00010203040506070809101112131415 -a 00+,10+
```

- `-s` — estado como cadena de 32 dígitos (16 pares; casilla 0 = par 0..1).
- `-a` — lista de acciones separadas por comas (`fila`, `columna`, `+`/`-`), p. ej. `21+,03-`.
  Sin `-a`, imprime los 32 sucesores del estado.

### `solve` — resolver un estado

```bash
java -jar target/loopover.jar solve -s 01000203040506070809101112131415 -e A_ESTRELLA -v
```

| Opción | Descripción |
|---|---|
| `-s` | Estado de 32 dígitos (obligatorio) |
| `-e` | Estrategia: `PROFUNDIDAD`, `ANCHURA`, `COSTO_UNIFORME`, `VORAZ`, `A_ESTRELLA`, `A_ESTRELLA_ACOTADA` (defecto: `A_ESTRELLA`) |
| `-p` | Profundidad máxima (defecto: 1000) |
| `-c` | Máximo de nodos del árbol en memoria; obligatorio para `A_ESTRELLA_ACOTADA` (SMA*) |
| `-h` | Heurística: `MANHATTAN`, `CERO`, `PARES_8`, `IMPARES_8`, `PBD_8`, `PBD_8_SUMA`, `MANHATTAN_ADMISIBLE`, `PERMUTACIONES` |
| `-m` | Límite de estados visitados; `0` = ilimitado |
| `-v` | Estadísticas de la búsqueda |

## Estructura

```
src/     código fuente (sin paquetes)
doc/     especificaciones (spec-driven development) y referencias
pom.xml  Maven (Java 17, picocli, fastutil, guava)
```

## Estado del proyecto

Desarrollo dirigido por especificación; ver el índice y estado de las tareas en
[`doc/README.md`](doc/README.md).
