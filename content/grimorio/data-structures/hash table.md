---
title: Hash Table
tags:
  - data-structures
alias:
  - tabla hash
  - hashmap
  - tabla de dispersión
---
## 1. Qué es y cómo funciona

### Intuición
Una tabla hash funciona como un conjunto de cajones numerados donde, mediante una fórmula (función hash), cada elemento se guarda y se busca directamente en una posición específica sin recorrer toda la estructura.

### Definición / propiedades

#### Definición
Una **Tabla Hash** permite almacenar y recuperar elementos de manera eficiente implementando el tipo de dato abstracto **diccionario**, asociando claves (*key*) con valores (*value*).

Permite búsquedas, inserciones y eliminaciones eficientes usando una función hash que transforma la clave en un índice.

#### Propiedades
- **Unicidad de ubicación relativa**:  
  Para una clave `k`, la función hash `h(k)` determina la posición inicial donde debe almacenarse o buscarse.

- **Determinismo del hash**:  
  Para la misma clave, se obtiene siempre el mismo índice

- **Consistencia entre inserción y búsqueda**:  
  Si un elemento fue insertado usando `h(k)`, la búsqueda debe usar la misma función para garantizar su recuperación

- **Manejo de colisiones**:  
  Distintas claves pueden generar el mismo índice (colisión), por lo que la estructura debe implementar un mecanismo de resolución (por ejemplo, encadenamiento o direccionamiento abierto)

- **Factor de carga controlado**:  
  La relación entre la cantidad de elementos almacenados y el tamaño de la tabla (load factor) debe mantenerse dentro de ciertos límites para preservar la eficiencia. Si se supera, se realiza un *rehashing*

### Representación
![Muestra](hashtable.svg)

Se implementa generalmente sobre **vectores unidimensionales**, aunque rara vez sobre matrices (dependiendo de la cantidad de componentes de la clave). La función hash, a partir de la clave en formato String que le demos, en este caso “abel”, nos dará el índice del vector en el que se guardará el valor. En caso de colisión, se usan listas, árboles o direccionamiento abierto.

#### Componentes
- **Vector (Bucket Array)**  
  Un vector de tamaño fijo o dinámico que contiene cuvetas (buckets) o ranuras

- **Función Hash (Hash Function)**  
  Algoritmo que recibe la clave y devuelve un número entero (hash code)

- **Bucket**  
  Posición donde se guarda el elemento (valor), dependiendo del método de resolución de colisiones, este puede almacenar uno o más elementos

#### ¿Qué son las *colisiones*?
Una **colisión** ocurre cuando dos claves distintas generan el mismo índice en la tabla hash. Es inevitable y debe resolverse con técnicas específicas.

Para resolver este problema, existen dos estrategias principales que administran la memoria de forma muy distinta:
1. **Separate Chaining (Encadenamiento):** Utiliza estructuras dinámicas auxiliares (como listas enlazadas) fuera del vector principal.
2. **Open Addressing (Direccionamiento Abierto):** Busca el siguiente espacio libre dentro del mismo vector (ej. *Linear Probing*), lo que ahorra punteros pero puede generar *clustering* (amontonamiento).

A continuación, una comparativa visual de cómo ambas técnicas manejan la inserción y búsqueda frente a una colisión:

![Comparativa entre Separate Chaining (encadenamiento) y Open Addressing (con clustering y uso de tombstones durante el probing)](resolucion-colisiones.svg)

## 2. Operaciones y complejidad

### Operaciones principales
- `hash(clave)` → convierte la clave string en un numero índice del vector  
- `insertar(clave, valor)` → guarda el valor en la posición del vector dado por la función hash
- `buscar(clave)` → busca y devuelve el valor en base a la clave dada calculando su hash
- `eliminar(clave)` → busca y elimina el valor en base a la clave dada calculando su hash

### Complejidad

#### Complejidad temporal
| Métodos | Promedio<sup>1</sup> | Peor caso / Con colisiones<br>Resolución: Lista enlazada | Peor caso / Con colisiones<br>Resolución: Árbol binario |
| :--- | :--- | :--- | :--- |
| `hash(clave)` | $O(m)$<sup>2</sup> | - | - |
| `insertar(clave, valor)` | $O(1)$ | $O(n)$<sup>3</sup> | $O(\log n)$ |
| `buscar(clave)` | $O(1)$ | $O(n)$ | $O(\log n)$ |
| `eliminar(clave)` | $O(1)$ | $O(n)$ | $O(\log n)$ |

**Notas:**

<sup>1</sup>El promedio aplica para cada subestructura de resolución.

<sup>2</sup>Siendo *m* la cantidad de caracteres de la clave.

<sup>3</sup>Para cada *n* en las celdas, se hace referencia a la cantidad de elementos de la subestructura de resolución.

#### Complejidad espacial
Esta complejidad puede verse afectada en gran parte por la función de Hash, debido a que según cómo sea de precisa y manejada podemos evitar **colisiones**, variando así las posibilidades de tener un mayor o menor uso de la complejidad espacial, en normas generales podemos deducir que ganamos “velocidad” en cuánto a complejidad computacional, sin embargo perdemos en complejidad espacial, ya que requerimos más estructuras dinámicas o un mayor uso de la memoria para guardar los elementos en caso de tener colisiones, lo que desencadena en que vamos a tener estructuras auxiliares (listas nodos, etc) o también es común que tengamos ciertos espacios vacíos en el array.

- Espacio total: $O(n)$

### Detalles operativos
- **Puede haber colisiones**: Distintas claves pueden mapear al mismo índice; deben resolverse con encadenamiento o direccionamiento abierto.
- **Depende de la función hash**: Una mala función hash degrada el rendimiento hacia $O(n)$.
- **Factor de carga y redimensionamiento**: Cuando la tabla se llena demasiado, se realiza **rehashing**, lo cual es costoso pero poco frecuente (costo amortizado).
- **Claves deben ser hashables**: En implementaciones como Python, deben ser inmutables (ej: `str`, `int`, `tuple`).
- **No mantiene orden (en general)**: Salvo implementaciones específicas (como `dict` en Python moderno), no se garantiza orden de inserción.
- **Posible clustering (direccionamiento abierto)**: La acumulación de elementos contiguos puede degradar el rendimiento.
- **Uso eficiente para búsquedas, no para recorridos ordenados**: Si se necesita orden, otras estructuras (como árboles balanceados) son más adecuadas.

## 3. Implementación

### Idea de implementación
Mantener una colección de pares clave-valor distribuida en posiciones calculadas mediante una función hash.
Cada clave se transforma en un índice del arreglo interno (bucket), permitiendo acceso rápido a los datos.

### Invariantes
- Toda clave válida debe poder ubicarse mediante la misma función hash.
- Cada elemento debe encontrarse en el bucket correspondiente a su hash.
- La estructura debe manejar correctamente las colisiones sin perder información.
- La cantidad de elementos almacenados nunca supera la capacidad lógica definida.

### Ejemplo de código

```python
class HashTable:
    def __init__(self, size):
        self.size = size
        self.table = [None] * size

    def hash_function(self, key):
        h = 5381
        for c in key:
            h = ((h << 5) + h) + ord(c)  # h * 33 + ord(c)
        return h % self.size

    def insert(self, key, value):
        index = self.hash_function(key)
        if self.table[index] is None:
            self.table[index] = [(key, value)]
        else:
            for i, (k, v) in enumerate(self.table[index]):
                if k == key:
                    self.table[index][i] = (key, value)
                    return
            self.table[index].append((key, value))

    def search(self, key):
        index = self.hash_function(key)
        if self.table[index] is not None:
            for k, v in self.table[index]:
                if k == key:
                    return v
        return None

    def delete(self, key):
        index = self.hash_function(key)
        if self.table[index] is not None:
            for i, (k, v) in enumerate(self.table[index]):
                if k == key:
                    del self.table[index][i]
                    return
```


#### Ejemplo de uso típico
Memoización con tabla hash (Ackermann)

```python
def ackermann(m, n, hashtable):
    if m < 0 or n < 0:
        return -1

    if m == 0:
        return n + 1

    if n == 0:
        return ackermann(m - 1, 1, hashtable)

    clave = (m, n)
    print(f"Proceso la clave -> {clave}")

    if clave in hashtable:
        print(f"Reutilizo el resultado de la clave -> {clave}")
        return hashtable.get(clave)
    else:
        valor = ackermann(
            m - 1,
            ackermann(m, n - 1, hashtable),
            hashtable
        )
        hashtable[clave] = valor
        return valor
```

---

## 4. Uso y criterio

### Casos de uso
- Diccionarios  
- Cachés  
- Conteo de frecuencias  
- Verificación de pertenencia  
- Eliminación de duplicados  

### Cuándo NO usarlo
- Cuando se necesita orden  
- Recorridos ordenados  
- Datasets pequeños  
- Muchas colisiones  

### Comparaciones
- **vs listas** → mejor búsqueda  
- **vs arrays** → mejor acceso por clave  
- **vs árboles** → más rápida en promedio, pero sin orden  

### Ventajas / desventajas

Ventajas:

- Acceso promedio $O(1)$
- Alta eficiencia
- Flexibilidad de claves

Desventajas:

- No mantiene orden
- Depende de la función hash
- Puede degradar a $O(n)$
- Mayor uso de memoria

### Señales de reconocimiento
- Búsqueda frecuente por clave
- Necesidad de rapidez
- Necesidad de conteo
- Necesidad de verificación de existencia

### ¿Cuándo conviene usarlo?
- Cuando el problema requiere acceso rápido por clave y el orden no es importante

**Trade off**: Una tabla de hash permite realizar operaciones fundamentales en tiempo constante promedio ($O(1)$), siendo altamente eficiente para búsquedas y actualizaciones. Sin embargo, su rendimiento depende críticamente de la calidad de la función hash, el manejo de colisiones y el control del factor de carga.


## 5. Relaciones y extensiones

### Variantes

#### Open Addressing (direccionamiento abierto)
En lugar de usar listas, todo se guarda en el mismo arreglo.
- Linear Probing: busca la siguiente posición libre secuencialmente. Simple, pero sufre de clustering
- Quadratic Probing: salta en intervalos cuadráticos. Reduce clustering, pero puede no cubrir toda la tabla
- Double Hashing: usa una segunda función hash para el salto. Mucho mejor distribución, menos colisiones

#### Separate Chaining
Cada índice apunta a una lista (o estructura).
- Listas enlazadas (clásico)
- Árboles balanceados (como en Java desde Java 8)

Más flexible, pero usa más memoria

### Mejoras modernas optimizadas

#### Cuckoo Hashing
- Usa **dos funciones hash** y permite reubicar elementos  
- Búsqueda $O(1)$ garantizada  
- Inserciones pueden ser costosas por reubicaciones

#### Robin Hood Hashing
- Idea: “robar” slots si reduces la desigualdad
- Minimiza la variación en tiempos de búsqueda
- Muy eficiente en práctica (usado en sistemas reales)

### Versiones orientadas a memoria caché

#### Cache-conscious Hash Table
- Diseñadas para aprovechar mejor la caché del CPU  
- Menos saltos de memoria → más rápidas en práctica

### Optimizaciones comunes
- Resize dinámico (rehashing)
- Funciones hash universales
- Load factor tuning
- Evitar clustering primario/secundario

### Relación con otras estructuras
**Base del tipo abstracto “diccionario” ([[map]])**: Generaliza la idea de asociar claves con valores, similar a estructuras como mapas ordenados, pero optimizada para acceso directo en lugar de orden.

**Alternativa a árboles de búsqueda (como BST o árboles balanceados)**: Mientras los árboles mantienen los datos ordenados con complejidad $O(\log n)$, la hash table prioriza velocidad $O(1)$ promedio sacrificando el orden.

**Usada en el [[set]]**: Implementa estructuras donde solo interesa la pertenencia de elementos (sin valores asociados), reutilizando el mecanismo de hashing.

**Base de índices y cachés**: Se utiliza para acceso rápido en sistemas como cachés, tablas de símbolos en compiladores y bases de datos (indexación por clave).

**Puede combinarse con otras estructuras**: Por ejemplo:
 - Hash table + [[linked list]] (encadenamiento)
 - Hash table + [[dynamic array]] (implementaciones modernas como en Python)

**No adecuada para datos ordenados o recorridos secuenciales eficientes**: Si se requiere mantener orden o hacer búsquedas por rango, estructuras como árboles balanceados son más apropiadas.

### Notas avanzadas

#### Estructuras inmutables
Una hash table puede implementarse de forma **persistente** (inmutable), donde cada operación (insertar, borrar, actualizar) devuelve una nueva versión sin modificar la anterior. Esto se logra mediante **sharing estructural**: en lugar de copiar toda la tabla, solo se duplican las partes afectadas.
Un enfoque común en lenguajes funcionales es usar estructuras como **Hash Array Mapped Trie (HAMT)**.

##### Implicancias:
- Las operaciones siguen siendo cercanas a $O(1)$ promedio (a veces $O(\log n)$ por la estructura tipo trie).
- Se comparten grandes porciones de memoria entre versiones → eficiencia espacial.
- Permite funcionalidades como **undo/redo**, backtracking o concurrencia sin locks.
- Trade-off: mayor complejidad de implementación y overhead constante mayor que una tabla mutable.

#### Concurrencia (thread-safety)
Una hash table compartida entre múltiples hilos puede sufrir **race conditions** si no se sincroniza correctamente.
Existen varias estrategias:
- Bloqueo global (mutex): Toda la tabla se protege con un lock.
- Bloqueo por segmentos (lock striping): Se divide la tabla en regiones, cada una con su propio lock.
- Implementaciones lock-free / concurrentes: Usan operaciones atómicas (como compare-and-swap) para evitar locks.
- Tablas thread-local: Cada hilo mantiene su propia tabla para evitar compartir estado.

##### Implicancias:
- Con locks globales: simple pero puede haber contención.
- Con granularidad fina: mejor paralelismo, más complejidad.
- Lock-free: alta escalabilidad, pero difícil de implementar correctamente.
- Rehashing es especialmente delicado en concurrencia (puede requerir detener o coordinar hilos).

#### Aleatoriedad (hashing y seguridad)
El comportamiento de una hash table depende fuertemente de la **calidad de la función hash**.
- Una buena función hash distribuye uniformemente las claves.
- Una mala distribución produce muchas colisiones → degradación a $O(n)$.

Para mitigar problemas:
- Hashing aleatorizado (randomized hashing): Se introduce una semilla aleatoria para evitar patrones predecibles.
- Protección contra ataques (hash flooding): En sistemas expuestos (como servidores web), entradas maliciosas pueden forzar colisiones.
- Ejemplo: lenguajes como Python usan hash salteado (hash randomization) para strings.

##### Implicancias:
- Mejora la distribución promedio y evita clustering.
- Aumenta la seguridad frente a ataques de denegación de servicio.
- Puede romper reproducibilidad exacta entre ejecuciones (el orden interno cambia).

## 6. Referencias y recursos
- [[COR2011]] - Chapter 11: Hash Tables
- [[DRO1995]] - Chapter 10: Hashing
