### 1. Qué es y cómo funciona

**Intuición**
Imaginá la memoria caché de un microprocesador: el espacio es diminuto, pero inmensamente rápido. Cuando el espacio se llena y necesitamos traer un dato nuevo, debemos decidir qué borrar. La estrategia **LRU (Least Recently Used - Menos Usado Recientemente)** asume que los datos que hace más tiempo no se consultan son los menos propensos a necesitarse en el futuro inmediato. El problema central que resuelve es la **latencia**: evita accesos costosos (a disco o red) manteniendo en memoria rápida únicamente la información más demandada por el sistema.

**Definición y propiedades**
Un Caché LRU es una estructura de datos de tamaño fijo que mantiene un registro del orden temporal en que sus elementos fueron accedidos. Sus propiedades invariantes son:

* **Límite de capacidad:** Nunca excede el tamaño máximo predefinido.
* **Política de desalojo:** Al alcanzar su capacidad máxima y recibir un nuevo elemento, expulsa estrictamente aquel que lleva más tiempo sin ser leído o modificado.
* **Complejidad estricta:** Todas sus operaciones elementales deben ejecutarse en un tiempo garantizado de $O(1)$.

**Representación**
Para lograr accesos y actualizaciones inmediatas, la caché LRU no puede depender de una sola estructura; orquesta dos trabajando en conjunto:

1. **Un Diccionario ([Hash Map](https://programacion-avanzada.github.io/arcana/grimorio/data-structures/hash-table)):** Almacena las claves apuntando directamente a la ubicación física de los datos. Esto permite saber si un dato existe (y accederlo) de forma instantánea (sin necesidad de recorrer la lista).
2. **Una Lista Doblemente Enlazada ([Doubly Linked List](https://programacion-avanzada.github.io/arcana/grimorio/data-structures/doubly-linked-list)):** Mantiene el orden de prioridad temporal. El "Frente" (Head) guarda el dato usado más recientemente, y el "Final" (Tail) guarda el candidato a ser borrado. Al ser doblemente enlazada, permite arrancar un nodo del medio y moverlo al frente en $O(1)$ sin tener que recorrer toda la estructura.

**(Esta imagen tiene que ser un SVG que nose que es)**

![alt text](image.png)


### 2. Operaciones y complejidad

Las operaciones de una Caché LRU se reducen a leer y escribir. Para que el sistema sea eficiente, la estructura garantiza que el tiempo de ejecución no dependa de la cantidad de datos almacenados.

**Operaciones principales**

* **`get(clave)`:** Busca un dato. Si la clave existe en el diccionario (*Cache Hit*), la función retorna el valor y, simultáneamente, extrae el nodo de su posición actual en la lista doblemente enlazada para insertarlo en el frente (marcándolo como el más reciente). Si no existe (*Cache Miss*), retorna vacío.
* **`put(clave, valor)`:** Inserta o actualiza un dato.
* Si la clave ya existe, actualiza su valor y mueve el nodo al frente de la lista.
* Si es un dato nuevo, crea el nodo y lo inserta en el frente.
* **Desalojo:** Si la inserción supera la capacidad máxima de la caché, la función elimina el último nodo de la lista (el menos usado) y borra su clave correspondiente del diccionario.



**Complejidad**

* **Tiempo:** Ambas operaciones, `get` y `put`, operan en **$O(1)$** (tiempo constante). El diccionario provee el acceso instantáneo y la lista doblemente enlazada permite reubicar el nodo alterando únicamente las referencias (punteros) de sus nodos vecinos.
* **Espacio:** **$O(n)$**, donde $n$ es la capacidad máxima de la caché. El espacio total es la suma de los $n$ elementos en el diccionario y los $n$ nodos en la lista enlazada.

**Detalles operativos (Costos ocultos)**

* **Sobrecarga de punteros (Memory Overhead):** Para mantener la lista doblemente enlazada, cada dato almacenado requiere memoria extra para guardar dos punteros (uno hacia el nodo anterior y otro hacia el siguiente). En entornos con memoria extremadamente restringida, este costo marginal puede ser un factor a considerar.
Imaginá que el diccionario (Hash Map) es un mueble enorme con 100 cajones numerados. Para saber en qué cajón guardar o buscar un dato, se usa una fórmula matemática (la función de *hash*).
* **Colisiones en el diccionario:** Para que la búsqueda sea instantánea ($O(1)$), el diccionario interno debe repartir equitativamente los datos en la memoria. Si varios datos terminan asignados a la misma posición (lo que se conoce como colisión), el sistema tendrá que revisarlos uno por uno dentro de ese casillero, lo que hace que la lectura pierda su velocidad inmediata en esos casos aislados. 



### 3. Implementación

**Idea de implementación**
La arquitectura de una Caché LRU requiere mantener dos estructuras de datos sincronizadas en todo momento:

1. Una **[Tabla Hash / Diccionario](https://programacion-avanzada.github.io/arcana/grimorio/data-structures/hash-table)** que mapea las claves directamente hacia los nodos físicos.
2. Una **[Lista Doblemente Enlazada](https://programacion-avanzada.github.io/arcana/grimorio/data-structures/doubly-linked-list)** abstracta (con un puntero al `frente` y otro al `final`) que dicta el orden de antigüedad.

La clave del algoritmo es que el diccionario no guarda el valor crudo, sino el "nodo" entero de la lista. Así, cuando buscamos una clave, el diccionario nos devuelve el nodo exacto, permitiéndonos reubicarlo manipulando sus punteros sin necesidad de recorrer la lista.

**Invariantes**
Para no romper la estructura, el código debe garantizar estrictamente dos cosas:

1. **Sincronización absoluta:** Si un nodo se elimina de la lista enlazada (por desalojo), su clave correspondiente debe ser eliminada del diccionario en el mismo paso.
2. **Capacidad:** La longitud del diccionario jamás debe ser mayor a la capacidad máxima definida al inicializar la caché.

**Ejemplo de código**
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


referencias: logicmojo.com