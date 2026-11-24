---
title: 'OIA 2006 - Movimientos de un Alfil'
tags:
  - oia
  - bestia
---

## Nombre y enunciado

En un tablero de ajedrez de $n \times n$ casillas, un alfil puede desplazarse desde su posición actual hacia cualquier casilla ubicada en sus diagonales, recorriendo una o más casillas en un solo movimiento. Sin embargo, el tablero contiene obstáculos que impiden su paso.

**Problema:** Determinar si es posible que el alfil llegue desde una casilla inicial $(x, y)$ hasta una casilla destino $(u, v)$ sin atravesar obstáculos y, en caso afirmativo, calcular la menor cantidad de movimientos necesarios.

Enunciado original: Olimpíada Informática Argentina, Certamen Nacional 2006, Nivel 3, Problema 2 ("Movimientos de un Alfil").

---

## Intuición

En un tablero vacío, el problema sería trivial: un alfil siempre llega a cualquier casilla de su mismo color en a lo sumo 2 movimientos (y nunca puede llegar a una casilla de distinto color).

La dificultad aparece al introducir obstáculos: una diagonal que parecía directa puede quedar bloqueada, obligando a dar rodeos de muchos movimientos o dejando la casilla destino completamente inaccesible. Además, a diferencia de un movimiento paso a paso en una grilla común, un único movimiento del alfil puede saltar varias casillas a la vez.

---

## Definición formal

**Entrada:**
- Un entero $n$ ($1 \le n \le 1000$) que indica el tamaño del tablero ($n \times n$).
- Una matriz de $n \times n$ con valores `0` (casilla libre) y `1` (obstáculo).
- Coordenadas $(x, y)$ de la casilla inicial y $(u, v)$ de la casilla destino ($1 \le x, y, u, v \le n$), numeradas desde la esquina superior izquierda.

**Salida:**
- Primera línea: `SI` si es posible llegar al destino, o `NO` en caso contrario.
- Segunda línea: la cantidad mínima de movimientos necesarios (o `0` si la respuesta es `NO`).

---

## Ejemplo concreto

Consideremos un tablero pequeño de $4 \times 4$:

|   | 1 | 2 | 3 | 4 |
|:-:|:-:|:-:|:-:|:-:|
| **1** | 0 | 0 | 0 | 0 |
| **2** | 0 | 0 | 0 | **O** |
| **3** | 0 | 0 | **X** | 0 |
| **4** | 0 | **A** | 0 | 0 |

- **Inicio (A):** `(4, 2)`
- **Destino (O):** `(2, 4)`
- **Obstáculo (X):** `(3, 3)`

Aunque `(4, 2)` y `(2, 4)` están en la misma diagonal, el obstáculo en `(3, 3)` bloquea el camino directo. El alfil debe rodearlo:

<!--
SVG PENDIENTE: content/attachments/bestiario/oia/alfil-ejemplo.svg
(es la misma imagen para alfil.md, alfil-fuerza-bruta.md y alfil-branch-and-bound.md: se crea una sola vez)
Qué mostrar: tablero de 4x4 con índices de fila y columna del 1 al 4 (fila 1 arriba).
- Inicio "A" en (4,2), destino "O" en (2,4), obstáculo "X" en (3,3).
- Recorrido óptimo con flechas numeradas 1, 2 y 3: (4,2) -> (3,1) -> (1,3) -> (2,4).
- Diagonal directa bloqueada (4,2) -> (3,3) en línea punteada.
- Casillas del otro color que el alfil, es decir con fila + columna impar, en gris: "descartadas".
- Leyenda al costado y el resultado: "SI, 3 movimientos".
Estilo: seguir content/attachments/illustration-style.css (SVG autocontenido, con modo claro y oscuro).
Cuando el archivo exista, reemplazar este comentario por esta línea:
![Tablero de 4x4 con el recorrido óptimo del alfil](alfil-ejemplo.svg)
-->

1. `(4, 2)` → `(3, 1)` *(1 movimiento)*
2. `(3, 1)` → `(1, 3)` *(2 movimientos)*
3. `(1, 3)` → `(2, 4)` *(3 movimientos)*

**Resultado:** `SI`, `3` movimientos.

---

## Por dónde empezar

El primer chequeo inmediato surge de las reglas del ajedrez: un alfil siempre se mantiene en casillas del mismo color. Si la casilla inicial $(x, y)$ y el destino $(u, v)$ tienen distinto color —es decir, $(x + y) \bmod 2 \neq (u + v) \bmod 2$—, es imposible llegar y se puede responder `NO` en $O(1)$. Por la misma razón, todas las casillas del color contrario al del alfil quedan automáticamente descartadas del análisis.

Una primera aproximación directa es aplicar **fuerza bruta**, probando todos los caminos simples posibles desde el inicio hasta el destino. Sin embargo, este enfoque se queda corto rápidamente: en un solo movimiento, el alfil puede caer en muchas casillas diferentes a lo largo de sus cuatro diagonales, lo que abre una cantidad enorme de caminos en cada paso y vuelve inviable explorar el espacio completo cuando el tablero es grande.

Para escalar a tableros grandes ($n \le 1000$), resulta necesario descartar ramas inútiles a medida que se explora, aplicando **Branch and Bound** (ramificación y poda) con una cota inferior simple basada en la geometría del tablero.

Conviene pensar el tablero como un [[graph]]: cada casilla libre es un vértice y hay una arista entre dos casillas si el alfil pasa de una a la otra en un solo movimiento. Como todos los movimientos "cuestan" lo mismo, la respuesta es el camino con menos aristas entre el inicio y el destino.

---

## Soluciones disponibles

- [[alfil-fuerza-bruta]]: prueba todos los caminos posibles. Exponencial; sirve para tableros de hasta 5×5.
- [[alfil-branch-and-bound]]: explora primero los estados con menor cota y poda los que no pueden mejorar. Polinomial; resuelve tableros de 1000×1000.
