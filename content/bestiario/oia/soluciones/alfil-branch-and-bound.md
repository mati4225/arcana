---
title: 'OIA 2006 - Movimientos de un Alfil - Branch and Bound'
tags:
  - oia
  - solucion
---

Volver a la descripción del problema: [[alfil]]

## Técnicas utilizadas

**Branch and Bound (ramificación y poda):** se explora el mismo árbol de caminos que en [[alfil-fuerza-bruta]], pero:

- **Ramificación:** los hijos de una casilla son todas las casillas a las que el alfil llega en un movimiento.
- **Cota inferior:** para cada estado se calcula cuántos movimientos va a necesitar, *como mínimo*, el camino completo.
- **Poda:** se descarta todo estado que no pueda mejorar la mejor solución conocida.
- **Estrategia LC (*least cost*):** siempre se expande primero el estado con menor cota, usando una cola de prioridad (un *heap*, ver [[queue]]).

## Idea de la solución

### La cota inferior

Si el alfil está en la casilla $c$ después de $g$ movimientos, el camino completo va a tener al menos $g + h(c)$ movimientos, donde $h$ mira solo la geometría e **ignora los obstáculos**:

$$
h(c) =
\begin{cases}
0 & \text{si } c \text{ es el destino} \\
1 & \text{si } c \text{ está en una diagonal del destino} \\
2 & \text{en otro caso}
\end{cases}
$$

Es una cota válida porque nunca sobreestima. Si $c$ no está en una diagonal del destino, hacen falta al menos 2 movimientos, y los obstáculos solo pueden agregar más, nunca menos. Además, todas las casillas que se exploran son del color del destino: el chequeo de paridad de la descripción ya descartó el otro caso.

### Las dos podas

1. **Por cota:** si $g + h(c) \ge$ mejor solución encontrada, ese estado y todo su subárbol se descartan sin generarlos.
2. **Por dominancia:** si ya se llegó a la casilla $c$ con $g'$ movimientos y $g' \le g$, el estado actual no puede terminar mejor que aquel. Se guarda en un [[map]] `mejor_g` el menor $g$ con el que se alcanzó cada casilla.

### Por qué alcanza con frenar al primer corte

El *heap* devuelve los estados en orden creciente de $g + h$. Cuando el estado de menor cota ya no puede mejorar la mejor solución, ningún otro puede: se termina la búsqueda entera con `break`, sin vaciar la cola.

> **Relación con BFS.** Si en lugar de un *heap* se usa una cola FIFO (estrategia "en anchura") y se deja solo la poda por dominancia, este mismo algoritmo es exactamente un **BFS**: explora por niveles y la primera vez que llega al destino lo hace con el mínimo. Esa fue la solución que planteamos en el parcial. La estrategia LC con cota agrega el criterio de ir primero hacia donde *parece* estar el destino.

## Código

Usa `libre` y `movidas_desde` de [[alfil-fuerza-bruta]].

```python
import heapq

def cota(casilla, destino):
    (f, c), (u, v) = casilla, destino
    if casilla == destino:
        return 0
    return 1 if abs(f - u) == abs(c - v) else 2

def branch_and_bound(tablero, origen, destino):
    if (sum(origen) - sum(destino)) % 2 != 0 or not libre(tablero, *destino):
        return ("NO", 0)                     # distinto color o destino ocupado
    mejor = float('inf')
    mejor_g = {origen: 0}
    heap = [(cota(origen, destino), 0, origen)]   # (g + h, g, casilla)

    while heap:
        estimado, g, casilla = heapq.heappop(heap)
        if estimado >= mejor:
            break                            # poda por cota: nada en el heap mejora
        if g > mejor_g[casilla]:
            continue                         # poda por dominancia
        if casilla == destino:
            mejor = g
            continue
        for siguiente in movidas_desde(tablero, *casilla):
            if g + 1 < mejor_g.get(siguiente, float('inf')):
                mejor_g[siguiente] = g + 1
                estimado = g + 1 + cota(siguiente, destino)
                if estimado < mejor:
                    heapq.heappush(heap, (estimado, g + 1, siguiente))

    return ("SI", mejor) if mejor < float('inf') else ("NO", 0)
```

## Traza de ejemplo

Sobre el tablero de 4×4 de la descripción: inicio `(4, 2)`, destino `(2, 4)` y obstáculo en `(3, 3)`.

![Tablero de 4x4 con el recorrido óptimo del alfil](alfil-ejemplo.svg)

![Árbol de búsqueda de Branch and Bound en el tablero de 4x4](alfil-bnb-traza.svg)

Chequeo inicial: $(4 + 2) \bmod 2 = (2 + 4) \bmod 2 = 0$, mismo color. Se sigue.

| Paso | Se extrae (casilla, $g$, $g+h$) | `mejor` | Qué pasa | Heap después |
|---:|---|---:|---|---|
| 1 | (4,2), 0, 1 | ∞ | Expande. Único hijo: (3,1) con $1 + 2 = 3$ | (3,1):3 |
| 2 | (3,1), 1, 3 | ∞ | Expande. (2,2): $2+2=4$. (1,3) está en diagonal del destino: $2+1=3$. (4,2): dominado, ya se llegó con 0 | (1,3):3, (2,2):4 |
| 3 | (1,3), 2, 3 | ∞ | Expande. (2,2) y (3,1): dominados. (2,4) es el destino: $3+0=3$ | (2,4):3, (2,2):4 |
| 4 | (2,4), 3, 3 | ∞ → **3** | Es el destino: `mejor = 3` | (2,2):4 |
| 5 | (2,2), 2, 4 | 3 | $4 \ge 3$: **poda por cota**, se termina | (vacío) |

Resultado: `SI`, `3` movimientos. La rama de (2,2), donde la fuerza bruta encontraba el camino de 4 movimientos, **no se llegó a expandir**: la cota ya sabía que no podía bajar de 4.

## Complejidad

Sea $n$ el lado del tablero.

### Temporal

$O(n^3)$ en el peor caso:

- Gracias a la poda por dominancia y a que la cota es **consistente** ($h(a) \le 1 + h(b)$ para todo movimiento $a \to b$), cada casilla se expande a lo sumo una vez, con su menor $g$.
- Expandir una casilla cuesta $O(n)$, porque desliza en cuatro diagonales de largo hasta $n$. Con $O(n^2)$ casillas, eso da $O(n^3)$.
- Cada casilla entra al *heap* **una sola vez** (ver abajo). Entonces hay $O(n^2)$ inserciones y extracciones de $O(\log n)$ cada una: $O(n^2 \log n)$, que queda dominado por $O(n^3)$.

**Por qué cada casilla entra una sola vez al heap.** Para que una casilla entre dos veces, tendría que mejorar su $g$. Eso pide un padre $p_2$ con $g$ menor que el del padre anterior $p_1$, pero extraído después que él. El *heap* saca en orden no decreciente de $g + h$, y toda casilla que se expande tiene $h \in \{1, 2\}$ (el destino no se expande). Entonces eso solo es posible si $g(p_2) = g(p_1) - 1$ y los dos tienen la misma cota. Pero el padre de $p_2$ tiene una cota menor que $p_2$ (si fuera igual, su $h$ valdría 3), así que salió antes que $p_1$: $p_2$ ya estaba en el *heap* cuando salió $p_1$. Y con cotas iguales el *heap* desempata por $g$, porque la tupla es $(g + h, g, \text{casilla})$, así que habría salido $p_2$ primero. Contradicción. En 1500 tableros aleatorios de hasta 40×40, ninguna casilla entró más de una vez.

### Espacial

$O(n^2)$: `mejor_g` guarda un valor por casilla y el *heap* tiene a lo sumo una entrada por casilla, porque cada casilla entra una sola vez (ver Temporal).

### Mediciones

Implementación en Python de este archivo, comparada con un BFS con barrido (ver abajo), en tableros de 1000×1000:

| Tablero de 1000×1000 | Respuesta | Branch and Bound | BFS con barrido |
|---|---|---:|---:|
| Sin obstáculos | SI, 2 | 0,01 s | 0,19 s |
| 30 % de obstáculos al azar | SI, 309 | 1,7 s | 1,1 s |
| Destino encerrado (del mismo color, inalcanzable) | NO | 12 min (737 s) | 5,5 s |

Cuando el destino está cerca, la cota lleva directo hacia él y Branch and Bound es el más rápido. Cuando no hay solución, la cota no poda nada y se paga entero el $O(n^3)$.

### Mejora posible: barrido por dirección

Con la estrategia FIFO (el BFS del parcial) se puede bajar a $O(n^2)$. Al deslizar en una dirección, se corta si se llega a una casilla que ya fue "barrida" en esa misma dirección, porque lo que sigue ya tiene su mínimo. Ese argumento depende de que los estados salgan en orden creciente de $g$, que la cola FIFO garantiza y el *heap* por $g+h$ no. Por eso no se combina con la estrategia LC.

## Cuándo usar esta técnica

### Favorable cuando

- Se busca un **óptimo** (el mínimo de movimientos) y existe una cota inferior barata y válida.
- El destino está "cerca en línea recta": la cota guía la búsqueda directo hacia él y la poda corta el resto apenas aparece una solución.
- Tableros grandes con solución: con $n = 1000$ resuelve en segundos o menos (ver Mediciones), mientras que la fuerza bruta ya no termina con $n = 7$.

### Limitaciones

- La cota es **débil**: solo puede valer 0, 1 o 2. En tableros con muchos obstáculos, donde el camino real tiene decenas o cientos de movimientos, casi todos los estados empatan y la búsqueda se comporta como un BFS con el costo extra del *heap*.
- Si la respuesta es `NO` pero las casillas son del mismo color, no hay nada que podar: recorre todo lo alcanzable antes de concluir. En un tablero de 1000×1000 eso tardó 12 minutos.
- Para este problema en particular, un BFS con barrido ($O(n^2)$, sin *heap*) es asintóticamente mejor. Branch and Bound gana cuando la cota es informativa, y acá lo es poco.

## Comparación con la solución por fuerza bruta

| | [[alfil-fuerza-bruta]] | Branch and Bound |
|---|---|---|
| Qué explora | Todos los caminos simples | Cada casilla, una vez, en orden de cota |
| Peor caso temporal | Exponencial, $O(n \cdot b^{L})$ | Polinomial, $O(n^3)$ |
| Espacio | $O(n^2)$ (pila de recursión) | $O(n^2)$ |
| Ejemplo 4×4 | 10 llamadas | 3 expansiones |
| Tablero vacío de 6×6 | Unos 40 s (27 millones de llamadas) | Instantáneo |
| Tablero vacío de 7×7 | No terminó en 9 minutos | Instantáneo |
| 1000×1000 con 30 % de obstáculos | Impensable | 1,7 s |
| Correctitud | Evidente: prueba todo | Depende de que la cota sea válida |

La fuerza bruta solo conviene como verificación en tableros mínimos: es tan simple que es difícil que esté mal, y por eso se usó para probar esta solución en 3000 tableros aleatorios de hasta 5×5, con resultados idénticos. Para cualquier tablero real, Branch and Bound es la elección: transforma un problema exponencial en uno polinomial gracias a las dos podas.

## Referencias

- Cátedra de Programación Avanzada (UNLaM), clase de Ramificación y poda: estrategias de exploración FIFO, LIFO y LC.
- [[COR2011]], capítulo de búsqueda en anchura (BFS) en grafos.
- Hart, Nilsson y Raphael (1968), *A Formal Basis for the Heuristic Determination of Minimum Cost Paths*: con una cota consistente, la estrategia LC equivale al algoritmo A*.
