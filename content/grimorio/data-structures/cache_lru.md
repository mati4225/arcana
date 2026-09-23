---
title: LRU Cache
tags:
  - data-structures
alias:
  - caché LRU
  - least recently used cache
---
## 1. Qué es y cómo funciona
### Intuición
Una **Caché LRU (Least Recently Used)** puede pensarse como un escritorio de trabajo pequeño: si se llena de libros y necesitamos traer uno nuevo de la biblioteca, la decisión más lógica es devolver a la biblioteca el libro que hace más tiempo no tocamos.

La idea central es mantener en memoria rápida los datos más demandados, asumiendo que los datos usados recientemente tienen mayor probabilidad de volver a necesitarse. Así, se evitan consultas repetitivas y costosas a disco, red o bases de datos.

### Definición / propiedades
#### Definición
Un Caché LRU es una estructura de datos de tamaño fijo que mantiene un registro del orden temporal en que sus elementos fueron accedidos. 

#### Propiedades clave:
- **Límite de capacidad:** 
  Nunca excede el tamaño máximo predefinido.

- **Política de desalojo:**
  Al alcanzar su capacidad máxima y recibir un nuevo elemento, expulsa estrictamente aquel que lleva más tiempo sin ser accedido.

- **Complejidad estricta:** 
  `get` y `put` pueden implementarse en $O(1)$ en promedio utilizando una hash table y una lista doblemente enlazada.

### Representación

![[cache_lru.svg]]

Para lograr accesos y actualizaciones inmediatas, la caché LRU orquesta dos estructuras trabajando en conjunto:

1. **Un Diccionario [[hash table]]**
   Asocia cada clave con el nodo correspondiente de la lista, permitiendo localizarlo directamente sin recorrer la lista.

2. **Una Lista Doblemente Enlazada [[doubly linked list]]** 
   Mantiene el orden de prioridad temporal. El "Frente" (Head) guarda el dato usado más recientemente, y el "Final" (Tail) guarda el candidato a ser borrado. Al ser doblemente enlazada, permite arrancar un nodo del medio y moverlo al frente en $O(1)$ sin tener que recorrer toda la estructura.

## 2. Operaciones y complejidad
### Operaciones principales

- **`get(clave)`** → Busca un dato. Si la clave existe en el diccionario (_Cache Hit_), la función retorna el valor y mueve el nodo al frente (marcándolo como el más reciente). Si no existe (_Cache Miss_), retorna vacío.
- **`put(clave, valor)`** → Inserta o actualiza un dato. 
	- Si la clave ya existe, actualiza su valor y mueve el nodo al frente de la lista.
	- Si es un dato nuevo, crea el nodo y lo inserta en el frente.
- **Desalojo:** → Si la caché ya alcanzó su capacidad máxima y se inserta una nueva clave, elimina el último nodo de la lista (el menos recientemente utilizado) y borra su clave del diccionario.

### Complejidad
#### Complejidad temporal

| Métodos                   | Promedio | Peor caso / con colisiones |
| ------------------------- | -------- | -------------------------- |
| `get(clave)`              | $O(1)$   | $O(n)$                     |
| `put(clave, valor)`       | $O(1)$   | $O(n)$                     |
| Mover nodo en la lista    | $O(1)$   | $O(1)$                     |
| Eliminar nodo de la lista | $O(1)$   | $O(1)$                     |

**Nota**: 

El peor caso $O(n)$ corresponde a la implementación interna de la hash table cuando ocurren muchas colisiones.
El costo de recuperar un dato desde la memoria principal, disco o red tras un cache miss no forma parte de la complejidad de la estructura LRU, sino del sistema que la utiliza.
#### Complejidad Espacial
- Espacio total: $O(n)$, donde `n` es la capacidad máxima de la caché.

- La caché mantiene un diccionario (HashMap) y una lista doblemente enlazada, cada uno con hasta `n` elementos, por lo que el espacio total sigue siendo $O(n)$.

### Detalles operativos (Costos ocultos)
- **Sobrecarga de punteros (Memory Overhead):** La lista doblemente enlazada requiere dos punteros adicionales por nodo (hacia el nodo anterior y hacia el siguiente). En entornos con memoria extremadamente restringida, este costo marginal puede ser un factor a considerar.

- **Colisiones en el diccionario:** El rendimiento promedio es $O(1)$, pero si varias claves caen en la misma posición (colisión), el diccionario debe revisarlas, pudiendo degradar la operación hasta $O(n)$ en el peor caso.

## 3. Implementación
### Idea de implementación
Requiere mantener dos estructuras de datos sincronizadas en todo momento:

1. Una **[[hash table]]** que asocia cada clave con el nodo correspondiente de la lista.
2. Una **[[doubly linked list]]** abstracta (con un puntero al `frente` y otro al `final`) que dicta el orden de recencia de uso.

La clave del algoritmo es que el diccionario no guarda el valor crudo, sino el "nodo" entero de la lista. Así, cuando buscamos una clave, el diccionario nos devuelve el nodo exacto, permitiéndonos reubicarlo manipulando sus punteros sin necesidad de recorrer la lista.

### Invariantes
Para no romper la estructura, el código debe garantizar estrictamente dos cosas:

1. **Sincronización absoluta:** Si un nodo se elimina de la lista enlazada (por desalojo), su clave correspondiente debe ser eliminada del diccionario en el mismo paso.
2. **Capacidad:** La longitud del diccionario jamás debe ser mayor a la capacidad máxima definida al inicializar la caché.

### Ejemplo de código
El siguiente pseudocódigo en Python ilustra la lógica central de las operaciones, delegando el manejo de punteros a funciones auxiliares para mantener la lectura limpia:

```python
class LRUCache:
    # ... (inicialización de capacidad, diccionario y lista doble) ...

    def get(self, key):
        if key in self.hash_map:
            nodo = self.hash_map[key]
            self.mover_al_frente(nodo) # Desconecta el nodo y lo pone primero: O(1)
            return nodo.valor
        return -1 # Cache Miss

    def put(self, key, value):
        if key in self.hash_map:
            # Si ya existe, actualizamos valor y lo marcamos como reciente
            nodo = self.hash_map[key]
            nodo.valor = value
            self.mover_al_frente(nodo)
        else:
            # Si se llenó, desalojamos el menos usado (el último de la lista)
            if len(self.hash_map) >= self.capacidad:
                nodo_viejo = self.eliminar_ultimo() # O(1)
                del self.hash_map[nodo_viejo.key]   # Mantenemos sincronización

            # Insertamos el nuevo dato al frente
            nuevo_nodo = Nodo(key, value)
            self.insertar_al_frente(nuevo_nodo)
            self.hash_map[key] = nuevo_nodo
```

#### Ejemplo de uso típico
Uso de una caché LRU con capacidad limitada:

```python
cache = LRUCache(2)

cache.put(1, "A")  # Cache: [1]
cache.put(2, "B")  # Cache: [2, 1]

cache.get(1)       # Devuelve "A"
                   # Cache: [1, 2]

cache.put(3, "C")  # Se elimina la clave 2 (menos usada)
                   # Cache: [3, 1]

cache.get(2)       # Devuelve -1 (no está en la caché)

cache.get(3)       # Devuelve "C"
```
## 4. Uso y criterio
### Casos de uso
- Almacenamiento de cache en sitios web y en consultas de bases de datos.
- Gestión de sesiones.
- Gestión de memoria.

### Cuando no usarlo
- Cuando los datos se recorren secuencialmente y rara vez vuelven a consultarse.
- Cuando el criterio de reemplazo debe basarse en la frecuencia de uso y no en la recencia.
- Cuando es necesario conservar todos los elementos sin desalojos automáticos.
- Cuando el costo adicional de mantener una hash table y una lista doblemente enlazada no se justifica.

### Comparaciones
- **vs LFU** → prioriza la frecuencia, no la recencia.
- **vs FIFO** → reemplaza por uso reciente, no por antigüedad.
- **vs MRU** → elimina el menos reciente, no el más reciente.
- **vs Random** → reemplazo basado en uso, no al azar.
### Ventajas / desventajas
Ventajas:

- **Complejidad temporal $O(1)$**: Sus dos operaciones (get, put) tienen una complejidad temporal constante.
- **Eficiencia en accesos repetidos:** mantiene los datos utilizados recientemente, reduciendo la necesidad de recuperarlos nuevamente desde la fuente original.

Desventajas:

- **Tamaño limitado**: La cache se limita por la capacidad especificada por lo que los datos a los que se accede con menos frecuencia serán eliminados.
- **Cache misses:** cuando un dato solicitado no se encuentra en la caché, debe obtenerse nuevamente desde la fuente original.
- **Sobrecarga de memoria**: requiere mantener una hash table y una lista doblemente enlazada sincronizadas, lo que aumenta el uso de memoria.

### Señales de reconocimiento
- Accesos repetidos a los mismos datos.
- Límite fijo de memoria.
- Acceso costoso a la fuente original.
- Reemplazo según uso reciente.

## 5. Relaciones y Extensiones
### Variantes
- **LFU**: elimina el elemento utilizado con menor frecuencia.
- **FIFO**: elimina el primer elemento que ingresó.
- **MRU**: elimina el elemento utilizado más recientemente.

### Relación con otras estructuras
- **Hash Map ([[hash table]])**: Para permitir un acceso en tiempo constante $O(1)$ a los elementos de la caché.
- **Lista doblemente enlazada ([[doubly linked list]])**: Para mantener el orden de acceso.

### Notas avanzadas
En sistemas con múltiples hilos, se deben sincronizar los accesos para evitar inconsistencias. A su vez, se requiere definir un tamaño máximo y una estrategia para expulsar elementos, y además de almacenar los datos, la implementación necesita estructuras auxiliares para mantener el orden de uso.
Por lo tanto, LRU Cache es una estructura compuesta que combina otras para resolver un problema en especifico: El almacenamiento temporal de datos y la decisión eficiente de que datos conservar o eliminar.

## 6. Referencias y recursos
- Silberschatz, A., Galvin, P. B., & Gagne, G. (2018). _Operating System Concepts_ (10ma ed.). Capítulo sobre Memoria Virtual y Políticas de Reemplazo de Páginas.
- Cormen, T. H., Leiserson, C. E., Rivest, L. R., & Stein, C. (2009). _Introduction to Algorithms_ (3ra ed.). MIT Press. (Fundamentos sobre el tiempo amortizado en Tablas Hash y Listas Enlazadas).
- LogicMojo. "LRU Cache". Disponible en: https://logicmojo.com/lru-cache
- Understanding LRU Cache: Efficient Data Storage and Retrieval - https://dev.to/abdullahyasir/understanding-lru-cache-efficient-data-storage-and-retrieval-2jnc