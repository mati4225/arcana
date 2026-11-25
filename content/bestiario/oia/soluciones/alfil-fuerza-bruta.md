---
title: 'OIA 2006 - Movimientos de un Alfil - Fuerza Bruta'
tags:
  - oia
  - solucion
---

Volver a la descripción del problema: [[alfil]]

## Técnicas utilizadas

**Fuerza bruta con recursividad:** se generan *todos* los caminos simples (sin repetir casillas) que puede hacer el alfil desde el inicio, y entre los que llegan al destino se queda con el de menos movimientos. No se usa ninguna información para descartar caminos antes de terminarlos: se explora el espacio completo.

## Idea de la solución

El tablero se puede ver como un [[graph]] implícito: cada casilla libre es un vértice, y hay una arista entre dos casillas si el alfil va de una a la otra en **un** movimiento (misma diagonal, sin obstáculos en el medio). No hace falta construir el grafo: los vecinos de una casilla se generan "deslizando" en las cuatro diagonales hasta chocar con un obstáculo o con el borde.

La función recursiva `explorar(casilla, movidas)`:

1. Si `casilla` es el destino, compara `movidas` con la mejor solución encontrada y vuelve.
2. Si no, prueba cada casilla alcanzable en un movimiento que todavía no esté en el camino actual, la marca, se llama recursivamente y la desmarca al volver.

Las casillas del camino actual se guardan en un [[set]] para no repetirlas: sin esa restricción, el alfil podría ir y volver infinitamente entre dos casillas y la recursión no terminaría.

> **Por qué no se corta al encontrar el destino:** el primer camino que llega no tiene por qué ser el más corto. En el ejemplo, la fuerza bruta encuentra primero un camino de 4 movimientos y recién después el de 3. Para asegurar el mínimo hay que mirar todos.

## Código

```python
DIRECCIONES = [(-1, -1), (-1, 1), (1, -1), (1, 1)]

def libre(tablero, f, c):
    n = len(tablero)
    return 1 <= f <= n and 1 <= c <= n and tablero[f - 1][c - 1] == 0

def movidas_desde(tablero, f, c):
    """Todas las casillas a las que el alfil llega en UN movimiento."""
    for df, dc in DIRECCIONES:
        nf, nc = f + df, c + dc
        while libre(tablero, nf, nc):
            yield nf, nc
            nf, nc = nf + df, nc + dc

def fuerza_bruta(tablero, origen, destino):
    mejor = float('inf')
    en_camino = {origen}

    def explorar(casilla, movidas):
        nonlocal mejor
        if casilla == destino:
            mejor = min(mejor, movidas)
            return
        for siguiente in movidas_desde(tablero, *casilla):
            if siguiente not in en_camino:
                en_camino.add(siguiente)
                explorar(siguiente, movidas + 1)
                en_camino.remove(siguiente)

    explorar(origen, 0)
    return ("SI", mejor) if mejor < float('inf') else ("NO", 0)
```

## Traza de ejemplo

Sobre el tablero de 4×4 de la descripción: inicio `(4, 2)`, destino `(2, 4)` y obstáculo en `(3, 3)`.

![Tablero de 4x4 con el recorrido óptimo del alfil](alfil-ejemplo.svg)

![Árbol de llamadas de la fuerza bruta en el tablero de 4x4](alfil-fb-arbol.svg)

Árbol de llamadas completo (cada nivel de sangría es un movimiento más):

```text
(4,2) m=0
└── (3,1) m=1                        ← (3,3) es obstáculo: es el único movimiento posible
    ├── (2,2) m=2
    │   ├── (1,1) m=3                ← sin salidas nuevas, vuelve
    │   └── (1,3) m=3
    │       └── (2,4) m=4  DESTINO   ← mejor = 4
    └── (1,3) m=2
        ├── (2,2) m=3
        │   └── (1,1) m=4            ← sin salidas nuevas, vuelve
        └── (2,4) m=3  DESTINO       ← mejor = 3
```

| Llamada | Casilla | Movidas | Camino actual | Resultado |
|---:|---|---:|---|---|
| 1 | (4,2) | 0 | (4,2) | genera (3,1) |
| 2 | (3,1) | 1 | (4,2) (3,1) | genera (2,2) y (1,3); (4,2) ya está en el camino |
| 3 | (2,2) | 2 | … (2,2) | genera (1,1) y (1,3) |
| 4 | (1,1) | 3 | … (2,2) (1,1) | sin vecinos nuevos |
| 5 | (1,3) | 3 | … (2,2) (1,3) | genera (2,4) |
| 6 | (2,4) | 4 | … (1,3) (2,4) | **destino**: `mejor = 4` |
| 7 | (1,3) | 2 | (4,2) (3,1) (1,3) | genera (2,2) y (2,4) |
| 8 | (2,2) | 3 | … (1,3) (2,2) | genera (1,1) |
| 9 | (1,1) | 4 | … (2,2) (1,1) | sin vecinos nuevos |
| 10 | (2,4) | 3 | … (1,3) (2,4) | **destino**: `mejor = 3` |

Resultado: `SI`, `3` movimientos, después de 10 llamadas. Hay que recorrer el árbol entero para saber que no existe un camino de 2.

## Complejidad

Sean $n$ el lado del tablero, $L \le \lceil n^2/2 \rceil$ la cantidad de casillas libres del color del alfil (las únicas que puede pisar) y $b \le 2(n-1)$ la cantidad máxima de casillas alcanzables en un movimiento.

### Temporal

$O(n \cdot b^{L})$ en el peor caso, es decir, **exponencial**. La recursión enumera todos los caminos simples: en cada nivel hay hasta $b$ opciones y un camino puede tener hasta $L$ movimientos, así que hay a lo sumo $b^L$ caminos, y generar los vecinos de cada casilla cuesta $O(n)$ (deslizar en cuatro diagonales).

Lo que se mide al correrlo, de esquina a esquina en un tablero **sin obstáculos**:

| Tablero | Llamadas recursivas |
|---|---:|
| 3×3 | 8 |
| 4×4 | 110 |
| 5×5 | 27.408 |
| 6×6 | 27.341.862 (unos 40 s) |
| 7×7 | no terminó en 9 minutos |

Paradójicamente, el tablero vacío es el peor caso: todo camino simple de un tablero con obstáculos también existe en el tablero vacío, así que sacar obstáculos solo agrega caminos para enumerar, aunque la respuesta sea siempre 1 o 2.

![Crecimiento de las llamadas de la fuerza bruta según el tamaño del tablero](alfil-fb-crecimiento.svg)

### Espacial

$O(L)$, que es $O(n^2)$: la profundidad máxima de la recursión (la pila de llamadas) y el [[set]] con las casillas del camino actual. Sin contar el tablero de entrada.

## Cuándo usar esta técnica

### Favorable cuando

- El tablero es muy chico (hasta 5×5; con 6×6 ya tarda unos 40 segundos) y se quiere una respuesta que sea **trivialmente correcta**, por ejemplo para verificar otra solución: así se probó la de [[alfil-branch-and-bound]] contra esta en miles de tableros aleatorios.
- Se necesitan **todos** los caminos (por ejemplo, contarlos o listarlos), no solo el mínimo.

### Limitaciones

- Inviable desde 7×7: el enunciado permite $n \le 1000$.
- Hace muchísimo trabajo repetido: llega a la misma casilla por caminos distintos y vuelve a explorar todo lo que sigue desde ahí, aunque ya haya llegado antes con menos movimientos.
- No aprovecha que se busca un **mínimo**: sigue explorando caminos que ya son más largos que la mejor solución conocida.

## Comparación con la solución por Branch and Bound

[[alfil-branch-and-bound]] ataca justo las dos últimas limitaciones:

| | Fuerza bruta | Branch and Bound |
|---|---|---|
| Orden de exploración | En profundidad, en el orden en que aparecen los vecinos | Primero los estados más prometedores (menor cota) |
| Caminos más largos que el mejor | Los explora igual | Los poda con la cota inferior |
| Llegar a una casilla ya visitada con más movimientos | La explora de nuevo | La descarta (poda por dominancia) |
| Ejemplo 4×4 | 10 llamadas, encuentra 4 antes que 3 | 3 casillas expandidas; el primer camino que encuentra ya es el óptimo |
| Peor caso | Exponencial | Polinomial: $O(n^3)$ |

Agregarle a esta solución una sola línea, `if movidas >= mejor: return`, ya la convierte en un *backtracking* con poda. Branch and Bound va más allá: además de cortar, elige **qué** explorar primero.

## Referencias

- Cátedra de Programación Avanzada (UNLaM), clase de Fuerza bruta y Backtracking.
- Enunciado: Olimpíada Informática Argentina, Certamen Nacional 2006, Nivel 3, Problema 2.
