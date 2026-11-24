---
title: Alfil - Branch and Bound
tags:
  - solucion
  - misc
---

## Técnicas utilizadas

Para ir un poco mas alla en la resolución de este problema, podemos pensar en optimizar las alternativas de casilleros que tiene el alfil como opción para avanzar. Por ejemplo intuitivamente podemos saber que no tiene sentido que el alfil avance en una dirección que es completamente contraria a la ubicación de la posición objetivo.  Siguiendo por esa ida y por medio de la técnica del algoritmo conocida como "Branch and Bound" es como se abordará el problema para optimizar la cantidad de casilleros analizados.
## Idea de la solución

Para resolver este problema vamos a considerar entre todas las alternativas que tiene inicialmente el alfil, cual me acerca mas o menos a la posición objetivo. Cuando el algoritmo BFS analice las casillas a las cual se puede desplazar el alfil se calculará una cota para cada una de estas, y se avanzará con la que en principio resulte mas prometedora. Esta cota se calcula con la distancia entre el casillero y la casilla objetivo por medio de la formula: $d = \sqrt{\text{distancia eje x}^{2} + \text{distancia eje y}^{2}}$ 
Para el calculo de la cota se abstrae de las restricciones propias del problema como los obstáculos y la dirección en la cual se mueve el alfil. Esto es para simplificar el problema y poder analizarlo mejor.

*calculo de cota de casillero:*
`cota` = $\sqrt{(\text{objetivo}_x-\text{casillero}_x)^{2} + (\text{objetivo}_y-\text{casillero}_y)^{2}}$

Las casillas analizadas se insertaran en una cola de prioridad que es un Min-Heap en donde se ordenaran por cantidad de movimientos y en segundo lugar por cota.

El primer casillero obtenido desde la cola de prioridad es el de menos movimientos, en caso de empate se obtiene el de menos cota. si hay un casillero que ofrece una peor cota que la mejor solución es podado y el algoritmo pasa a analizar el siguiente casillero guardado en la cola.

Ejemplo:

![primer-movimiento-alfil](reducido-ajedrez.svg)

| CASILLERO ANALIZADO | CASILLA OBJETIVO | DISTANCIA EN X | DISTANCIA EN Y |               COTA               | MOVIMIENTOS |
| :-----------------: | :--------------: | :------------: | :------------: | :------------------------------: | :---------: |
|        (2,1)        |      (4,1)       |   4 - 2 = 2    |   1 - 1 = 0    |    $\sqrt{2^{2} + 0^{2}} = 2$    |      1      |
|        (2,3)        |      (4,1)       |   4 - 2 = 2    |   1 - 3 = -2   | $\sqrt{2^{2} + (-2)^{2}} = 2,83$ |      1      |

**Cola de prioridad:**

|  #  | CASILLERO | COTA | CANTIDAD DE MOVIMIENTOS |
| :-: | :-------: | :--: | :---------------------: |
|  1  |   (2,1)   |  2   |            1            |
|  2  |   (2,3)   | 2,83 |            1            |
## Código

```text

```

## Complejidad

## Traza de Ejemplo
Simulación de una ejecución del algoritmo para la resolución de un determinado tablero de 8x8 con 6 posiciones bloqueadas. 

### Primer paso
Analizo las posiblidades para el primer movimiento
![traza-ajedrez-1](ajedrez-traza-1.svg)

**Cola de prioridad:**

|  #  | CASILLERO |               COTA               | MOVIMIENTOS |
| :-: | :-------: | :------------------------------: | :---------: |
|  1  |   (3,4)   | $\sqrt{5^{2} + (-1)^{2}} = 5,10$ |      1      |
|  2  |   (2,3)   |    $\sqrt{6^{2} + 0^{2}} = 6$    |      1      |
|  3  |   (2,1)   |  $\sqrt{6^{2} + 2^{2}} = 6,32$   |      1      |

**Mejor Solución:**

La mejor solución hasta ahora es el casillero `(3,4)`



### Segundo paso
Extraigo de la cola de prioridad el casillero `(3,4)` y desde este analizo las posibilidades de movimiento

![traza-ajedrez-2](ajedrez-traza-2.svg)

**Cola de prioridad:**

|  #  | CASILLERO |               COTA               | MOVIMIENTOS |
| :-: | :-------: | :------------------------------: | :---------: |
|  1  |   (2,3)   |    $\sqrt{6^{2} + 0^{2}} = 6$    |      1      |
|  2  |   (2,1)   |  $\sqrt{6^{2} + 2^{2}} = 6,32$   |      1      |
|  3  |   (5,2)   |  $\sqrt{3^{2} + 1^{2}} = 3,16$   |      2      |
|  4  |   (4,3)   |    $\sqrt{4^{2} + 0^{2}} = 4$    |      2      |
|  5  |   (2,5)   | $\sqrt{6^{2} + (-2)^{2}} = 6,32$ |      2      |
|  6  |   (1,6)   | $\sqrt{6^{2} + (-3)^{2}} = 6,71$ |      2      |
**Mejor Solución:**

La mejor solución hasta ahora es el casillero `(5,2)`



### Tercer paso
Se extraen los casilleros `(2,3)` y `(2,1)` pero ambos son podados porque ofrecen una cota peor que la mejor solución encontrada hasta el momento. Por lo tanto se saca de la cola de prioridad una vez mas y se obtiene el casillero `(5,2)`

![traza-ajedrez-3](ajedrez-traza-3.svg)

**Cola de prioridad:**

|  #  | CASILLERO |               COTA               | MOVIMIENTOS |     |
| :-: | :-------: | :------------------------------: | :---------: | --- |
|  1  |   (4,3)   |    $\sqrt{4^{2} + 0^{2}} = 4$    |      2      |     |
|  2  |   (2,5)   | $\sqrt{6^{2} + (-2)^{2}} = 6,32$ |      2      |     |
|  3  |   (1,6)   | $\sqrt{6^{2} + (-3)^{2}} = 6,71$ |      2      |     |
|  4  |   (7,4)   | $\sqrt{1^{2} + (-1)^{2}} = 1,41$ |      3      |     |
|  5  |   (6,3)   |    $\sqrt{2^{2} + 0^{2}} = 4$    |      3      |     |
|  6  |   (8,5)   |  $\sqrt{0^{2} + (-2)^{2}} = 4$   |      3      |     |
|  7  |   (4,1)   |  $\sqrt{4^{2} + 2^{2}} = 4,47$   |      3      |     |
**Mejor Solución:**

La mejor solución hasta ahora es el casillero `(7,4)`


### Cuarto paso
Se extraen los casilleros `(4,3)`, `(2,5)` y `(1,6)` pero todos son podados porque ofrecen una cota peor que la mejor solución encontrada hasta el momento. Por lo tanto se saca de la cola de prioridad una vez mas y se obtiene el casillero `(7,4)`
![traza-ajedrez-4](ajedrez-traza-4.svg)

Analizando las posibilidades dentro del casillero obtenido de la cola de prioridad nos damos cuenta que estamos a un movimiento de finalizar el algoritmo, con lo cual el algoritmo de búsqueda finaliza.

Por ultimo el algoritmo contabiliza un movimiento al contador de movimientos que es el que toma de ir desde el casillero `(7,4)`  al objetivo que es `(8,3)`. Por ultimo se retorna el numero de movimientos junto con la palabra "SI" que indica que es posible llegar desde el punto del partida al objetivo en este tablero en particular.


### Temporal



### Espacial
