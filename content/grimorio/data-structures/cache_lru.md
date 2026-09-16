## 1. Qué es y cómo funciona

### Intuición
Una **Caché LRU (Least Recently Used)** puede pensarse como un escritorio de trabajo pequeño: si se llena de libros y necesitamos traer uno nuevo de la biblioteca, la decisión más lógica es devolver a la biblioteca el libro que hace más tiempo no tocamos.

La idea central es mantener en una memoria rápida y de acceso inmediato los datos más demandados, asumiendo que la información que no se consultó recientemente tiene baja probabilidad de ser requerida a la brevedad. Resuelve el problema de la latencia al evitar consultas repetitivas y costosas (a disco, red o bases de datos).

### Definición / propiedades
#### Definición
Un Caché LRU es una estructura de datos de tamaño fijo que mantiene un registro del orden temporal en que sus elementos fueron accedidos. 

#### Propiedades clave:

- **Límite de capacidad:** Nunca excede el tamaño máximo predefinido.

- **Política de desalojo:** Al alcanzar su capacidad máxima y recibir un nuevo elemento, expulsa estrictamente aquel que lleva más tiempo sin ser leído o modificado.

- **Complejidad estricta:** Todas sus operaciones elementales deben ejecutarse en un tiempo garantizado de $O(1)$.

### Representación

![Diagrama de arquitectura de un Caché LRU: Hash Map sincronizado con una Lista Doblemente Enlazada](cache_lru.svg)

Para lograr accesos y actualizaciones inmediatas, la caché LRU no puede depender de una sola estructura, orquesta dos trabajando en conjunto:

1. **Un Diccionario [[hash table]]** Almacena las claves apuntando directamente a la ubicación física de los datos. Esto permite saber si un dato existe (y accederlo) de forma instantánea (sin necesidad de recorrer la lista).
2. **Una Lista Doblemente Enlazada [[linked list]]** Mantiene el orden de prioridad temporal. El "Frente" (Head) guarda el dato usado más recientemente, y el "Final" (Tail) guarda el candidato a ser borrado. Al ser doblemente enlazada, permite arrancar un nodo del medio y moverlo al frente en $O(1)$ sin tener que recorrer toda la estructura.


## 2. Operaciones y complejidad

Las operaciones de una Caché LRU se reducen a leer y escribir. Para que el sistema sea eficiente, la estructura garantiza que el tiempo de ejecución no dependa de la cantidad de datos almacenados.

### Operaciones principales

- **`get(clave)`**: Busca un dato. Si la clave existe en el diccionario (_Cache Hit_), la función retorna el valor y, simultáneamente, extrae el nodo de su posición actual en la lista doblemente enlazada para insertarlo en el frente (marcándolo como el más reciente). Si no existe (_Cache Miss_), retorna vacío.
- **`put(clave, valor)`**: Inserta o actualiza un dato.
- Si la clave ya existe, actualiza su valor y mueve el nodo al frente de la lista.
- Si es un dato nuevo, crea el nodo y lo inserta en el frente.
- **Desalojo:** Si la inserción supera la capacidad máxima de la caché, la función elimina el último nodo de la lista (el menos usado) y borra su clave correspondiente del diccionario.

### Complejidad

#### Complejidad temporal

| Métodos                   | Promedio | Peor caso / con colisiones |
| ------------------------- | -------- | -------------------------- |
| `get(clave)`              | $$O(1)$$ | $$O(n)$$                   |
| `put(clave, valor)`       | $$O(1)$$ | $$O(n)$$                   |
| Mover nodo en la lista    | $$O(1)$$ | $$O(1)$$                   |
| Eliminar nodo de la lista | $$O(1)$$ | $$O(1)$$                   |

#### Complejidad Espacial

- **Espacio total:** **O(n)**, donde `n` es la capacidad máxima de la caché.

- La caché utiliza dos estructuras principales: un **diccionario (HashMap)** que almacena hasta `n` entradas y una **lista doblemente enlazada** que almacena hasta `n` nodos.
- Por lo tanto, el espacio utilizado es `O(n) + O(n) = O(n)`.

### Detalles operativos (Costos ocultos)

- **Sobrecarga de punteros (Memory Overhead):** Para mantener la lista doblemente enlazada, cada dato almacenado requiere memoria extra para guardar dos punteros (uno hacia el nodo anterior y otro hacia el siguiente). En entornos con memoria extremadamente restringida, este costo marginal puede ser un factor a considerar.
  Imaginá que el diccionario (Hash Map) es un mueble enorme con 100 cajones numerados. Para saber en qué cajón guardar o buscar un dato, se usa una fórmula matemática (la función de _hash_).
- **Colisiones en el diccionario:** Para que la búsqueda sea instantánea ($O(1)$), el diccionario interno debe repartir equitativamente los datos en la memoria. Si varios datos terminan asignados a la misma posición (lo que se conoce como colisión), el sistema tendrá que revisarlos uno por uno dentro de ese casillero, lo que hace que la lectura pierda su velocidad inmediata en esos casos aislados.

## 3. Implementación

### Idea de implementación
La arquitectura de una Caché LRU requiere mantener dos estructuras de datos sincronizadas en todo momento:

1. Una **[[hash table]]** que mapea las claves directamente hacia los nodos físicos.
2. Una **[[linked list]]** abstracta (con un puntero al `frente` y otro al `final`) que dicta el orden de antigüedad.

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

## 4. Uso y criterio

### Casos de uso

- Almacenamiento de cache en sitios web y en consultas de bases de datos.
- Gestión de sesiones
- Gestión de memoria

### Cuando no usarlo
⚠️⚠️⚠️⚠️

### Comparaciones

LRU Cache se directamente con otras estrategias de almacenamiento cache como lo pueden ser LFU (Last frecuently use), FIFO (First In First Out), MRU (Most Recently Use) o Random.

### Ventajas

- Complejidad temporal O(1): Sus dos operaciones (get, put) tienen una complejidad temporal constante.
- Eficiencia espacial: Garantiza que solo los datos más utilizados se almacenen en memoria.

### Desventajas

- Tamaño limitado: La cache se limita por la capacidad especificada por lo que los datos a los que se accede con menos frecuencia serán eliminados.
- Fallos en la cache: Cuando la cache esta llena, cualquiera nuevo acceso provoca un fallo que obliga a obtener los datos de la fuente original.

### Señales de reconocimiento

⚠️⚠️⚠️⚠️


## 5. Relaciones y Extensiones

### Variantes

- LFU: elimina el elemento utilizado con menor frecuencia.
- FIFO: elimina el primer elemento que ingresó.
- MRU: elimina el elemento utilizado mas recientemente.

### Relacion con otras estructuras

- Lista doblemente enlazada: Para mantener el orden de acceso.
- Hash Map: Para permitir un acceso en tiempo constante O(1) a los elementos de la caché.

### Notas avanzadas

En sistemas con múltiples hilos, se deben sincronizar los accesos para evitar inconsistencias. A su vez, se requiere definir un tamaño máximo y una estrategia para expulsar elementos, y además de almacenar los datos, la implementación necesita estructuras auxiliares para mantener el orden de uso.
Por lo tanto, LRU Cache es una estructura compuesta que combina otras para resolver un problema en especifico: El almacenamiento temporal de datos y la decisión eficiente de que datos conservar o eliminar.

## 6. Referencias y recursos

- Silberschatz, A., Galvin, P. B., & Gagne, G. (2018). _Operating System Concepts_ (10ma ed.). Capítulo sobre Memoria Virtual y Políticas de Reemplazo de Páginas.
- Cormen, T. H., Leiserson, C. E., Rivest, L. R., & Stein, C. (2009). _Introduction to Algorithms_ (3ra ed.). MIT Press. (Fundamentos sobre el tiempo amortizado en Tablas Hash y Listas Enlazadas).
- LogicMojo. "LRU Cache". Disponible en: https://logicmojo.com/lru-cache
- Understanding LRU Cache: Efficient Data Storage and Retrieval - https://dev.to/abdullahyasir/understanding-lru-cache-efficient-data-storage-and-retrieval-2jnc