
## Sesión 2 
# ESTRUCTURAS DE DATOS EN SQL Y NoSQL

Por CARLOS CÉSAR SÁNCHEZ CORONEL

## Capítulo 1

## Estructuras de Datos en Motores SQL y NoSQL

### 1.1. Semana 2: El Corazón Algorítmico de las Bases de Datos

En la primera sesión comprendimos los fundamentos del dato y la infraestructura física que lo soporta. Ahora vamos a levantar el capó y explorar el motor: los algoritmos y estructuras de datos que hacen posible que una consulta tardé milisegundos en lugar de minutos.

En esta sesión, recorreremos los motores más importantes de la industria y desentrañaremos sus secretos internos: qué estructura de datos utilizan, por qué esa elección y cómo impacta en el rendimiento de las operaciones cotidianas.

#### 1.1.1. El Denominador Común: Los Índices

Antes de sumergirnos en cada motor, comprendamos qué es un índice en términos algorítmicos.

Un índice es una estructura de datos auxiliar que permite acelerar las operaciones de búsqueda a costa de:

- Espacio adicional en memoria/disco
- Sobrecarga en escrituras (INSERT/UPDATE/DELETE deben actualizar el índice)

La complejidad algorítmica de las operaciones sin índice versus con índice es:

| Operación | Sin índice | Con índice |
| :--- | :--- | :--- |
| Búsqueda por clave | O(n) - Table Scan | O(log n) o O(1) |
| Inserción | O(1) (al final) | O(log n) (mantener orden) |
| Eliminación | O(n) (buscar + borrar) | O(log n) |

### 1.2. Motores SQL: El Reino del B-Tree y sus Variantes

Todos los motores SQL tradicionales (Oracle, SQL Server, MySQL, PostgreSQL) comparten una base común: el B-Tree (Árbol-B) como estructura de índice primaria. Sin embargo, cada uno introduce optimizaciones y variantes específicas.

#### 1.2.1. Fundamento: El B-Tree y B+Tree

El B-Tree es un árbol balanceado diseñado específicamente para sistemas de almacenamiento en disco. Sus características fundamentales:

- Balanceo automático: Todas las hojas están a la misma profundidad.
- Alto factor de ramificación (fanout): Cada nodo puede tener cientos de hijos, lo que mantiene el árbol poco profundo (logfanout n).
- Nodos de gran tamaño: Diseñados para coincidir con el tamaño de página del disco (típicamente 4KB, 8KB o 16KB), minimizando accesos a disco.

La variante más común es el B+Tree, donde:

- Los nodos internos solo almacenan claves (no datos), actuando como ”señalizadores”.
- Los nodos hoja almacenan las claves junto con los punteros a los registros (o los datos completos).
- Las hojas suelen estar enlazadas secuencialmente, permitiendo recorridos eficientes por rango.

**Complejidad:**

- Búsqueda por clave: O(log n) acesos a disco.
- Búsqueda por rango: O(log n + k) donde k es el número de resultados.
- Inserción/Eliminación: O(log n) (con posibles splits o merges).

#### 1.2.2. Oracle Database: Índices B-Tree y Bitmap

Oracle, el veterano de los motores empresariales, ofrece múltiples tipos de índices:

- **Índices B-Tree (por defecto):** Implementación clásica de B+Tree. Optimizado para alta cardinalidad (muchos valores distintos).
- **Índices Bitmap:** Utilizan mapas de bits para representar la presencia de valores. Ideales para columnas con baja cardinalidad (ej. género, estado civil). Cada valor distinto tiene un bitmap donde cada bit representa una fila.
    - Ventaja: Operaciones booleanas (AND, OR) extremadamente rápidas mediante operaciones bitwise.
    - Complejidad de búsqueda: O(1) para combinar bitmaps.
    - Desventaja: Problemas de escalabilidad en escrituras concurrentes (bloqueo de filas).
- **Índices Function-Based:** Almacenan el resultado de una función aplicada a la columna, permitiendo búsquedas eficientes como `WHERE UPPER(nombre) = 'JUAN'`.

**Caso de uso típico:** Data Warehousing con índices bitmap para consultas analíticas sobre dimensiones con pocos valores.

#### 1.2.3. SQL Server: B+Tree y Índices Columnstore

SQL Server ofrece dos grandes familias de índices:

- **Índices Clustered (B+Tree):** Los datos reales se almacenan en las hojas del árbol. Solo uno por tabla. La búsqueda por clave primaria es extremadamente rápida porque una vez localizada la hoja, ya tenemos los datos.
- **Índices Non-Clustered (B+Tree):** Las hojas almacenan punteros a las filas (o a las claves del clustered index). Requieren un paso adicional para recuperar los datos.
- **Índices Columnstore:** Innovación de SQL Server para almacenamiento columnar. Los datos se agrupan por columnas en lugar de por filas, permitiendo compresión masiva y operaciones analíticas rápidas.
    - Complejidad: Las consultas agregadas pueden procesarse escaneando solo las columnas necesarias, con compresión y procesamiento por lotes.
    - Ventaja: Hasta 10x de compresión y 100x de velocidad en consultas analíticas.

**Caso de uso:** Sistemas OLTP con índices clustered para acceso rápido por ID; Data Warehouses con índices columnstore para agregaciones masivas.

#### 1.2.4. MySQL: B+Tree con Motores de Almacenamiento

MySQL es único porque separa el motor de almacenamiento del motor SQL. Los más importantes:

**InnoDB (Motor por defecto)**
- Índices clustered obligatorios: Cada tabla InnoDB tiene un índice clustered (si no hay clave primaria, crea una oculta de 6 bytes).
- Organización por clave primaria: Los datos se almacenan físicamente ordenados por la clave primaria (B+Tree donde las hojas contienen la fila completa).
- Índices secundarios: Almacenan la clave primaria como puntero, no la dirección física directa. Esto significa que una búsqueda por índice secundario requiere dos accesos: índice secundario → clave primaria → datos.

**MyISAM (Motor legacy)**
- Índices no clustered: Los índices y los datos se almacenan por separado. Las hojas de los índices contienen punteros directos a la posición física del registro.
- Más simples, pero sin soporte transaccional ni recuperación ante fallos.

**MEMORY (Motor en RAM)**
- Índices hash: Soporta índices hash para búsquedas por igualdad O(1).
- Limitación: Tablas volátiles (desaparecen al reiniciar), tamaño máximo fijo.

#### 1.2.5. PostgreSQL: La Navaja Suiza de los Índices

PostgreSQL es famoso por su extensibilidad y variedad de tipos de índice:

- **B-Tree (por defecto):** Implementación clásica de B+Tree, similar a los demás.
- **Hash:** Índices basados en tabla hash para búsquedas de igualdad (O(1) promedio). Históricamente no eran transaccionales hasta versiones recientes, pero ahora son confiables.
- **GIN (Generalized Inverted Index):** Índices invertidos para datos compuestos: arrays, documentos JSON, búsqueda de texto completo.
    - Estructura: Índice que mapea elementos (palabras, claves JSON) a las filas que los contienen.
    - Complejidad: Búsqueda de elementos individuales O(log n).
- **GiST (Generalized Search Tree):** Árbol balanceado para datos geométricos y búsquedas por similitud. Soporta operaciones como “contiene”, “intersecta”, “cercano a”.
- **SP-GiST (Space-Partitioned GiST):** Para datos particionales espacialmente (árboles cuádruples, árboles de búsqueda ternaria).
- **BRIN (Block Range Index):** Índice muy ligero que almacena resúmenes de rangos de bloques. Ideal para datos correlacionados físicamente (ej. series temporales). Ocupa miles de veces menos espacio que un B-Tree.
    - Complejidad: Escaneo de rangos de bloques O(número de bloques), pero con bloques mucho más grandes.

**Ejemplo práctico:** Un índice GIN en PostgreSQL permite buscar eficientemente dentro de un campo JSON:
```sql
CREATE INDEX idx_datos ON tabla USING GIN (datos_json);
SELECT * FROM tabla WHERE datos_json @> '{"ciudad": "Madrid"}';
```

### 1.3. Motores NoSQL: Especialización y Rendimiento

Mientras que los motores SQL estandarizan en torno al B-Tree, los NoSQL han evolucionado hacia estructuras altamente especializadas para casos de uso concretos.

#### 1.3.1. Redis: La Colección de Estructuras en Memoria

Redis no es solo una base de datos clave-valor; es un servidor de estructuras de datos. Cada estructura tiene su implementación optimizada.

**Arquitectura de dos niveles**

Redis tiene una arquitectura de dos niveles:

1.  Nivel externo (visto por el usuario): Strings, Lists, Sets, Sorted Sets, Hashes.
2.  Nivel interno (implementación): sds, dict, ziplist, quicklist, skiplist, intset.

Cada clave en Redis es un objeto robj que contiene:
- type: El tipo de dato externo (string, list, hash...)
- encoding: La implementación interna concreta (raw, int, ht, ziplist, skiplist...)
- ptr: Puntero a los datos reales

Esta flexibilidad permite que Redis elija automáticamente la representación más eficiente según los datos.

**Strings (sds - Simple Dynamic String)**
- Estructura: Cadena de bytes con metadatos (longitud, capacidad).
- Optimización:
    - int: Si el string puede representarse como entero, se almacena como long (ahorro masivo de memoria).
    - embstr: Strings cortos (menos de 44 bytes en versiones recientes) se almacenan junto con el objeto redisObject en una misma asignación de memoria.
    - raw: Strings largos con asignación separada.
- Complejidad: Acceso O(1) a longitud, modificación O(n).

**Lists (Listas)**
- Implementación actual: quicklist.
- Estructura: Lista enlazada de nodos, donde cada nodo contiene un ziplist (lista compacta).
- Filosofía: Compromiso entre memoria (ziplist comprime) y rendimiento de inserción/eliminación (lista enlazada permite splits).
- Operaciones: LPUSH, RPOP: O(1) en ambos extremos. Acceso por índice: O(n) en el peor caso, pero optimizado con recorrido desde el extremo más cercano.

**Sets (Conjuntos)**
- Implementación: hashtable (dict) o intset.
- intset: Si todos los elementos son enteros y el conjunto es pequeño, se usa un array ordenado de enteros (búsqueda binaria O(log n), pero con almacenamiento extremadamente compacto).
- hashtable: Cuando el conjunto crece o aparecen strings no numéricos, se convierte a tabla hash (O(1) para inserciones/búsquedas).
- Operaciones: Añadir, eliminar, pertenencia O(1) en hashtable.

**Sorted Sets (ZSET)**
- Implementación dual: skiplist + hashtable.
- Skiplist: Estructura probabilística que mantiene los elementos ordenados por puntuación (score). Permite búsquedas por rango O(log n) e inserciones O(log n).
- Hashtable: Mantiene un mapeo elemento → score para acceso directo O(1).
- Operaciones:
    - ZADD: O(log n) (actualizar ambas estructuras)
    - ZRANGE: O(log n + k) para recuperar rango
    - ZSCORE: O(1) gracias al hashtable

**Streams**
- Implementación: Radix Tree (árbol compacto de prefijos).
- Estructura: Árbol que comprime prefijos comunes, ideal para IDs secuenciales y mensajes con timestamps.
- Operaciones: Inserción y lectura por rangos de tiempo extremadamente eficientes.

#### 1.3.2. Cassandra: LSM-Tree y el Reinado de las Escrituras

Cassandra está diseñado para altísimo rendimiento en escritura, sacrificando simplicidad en lecturas. Su corazón es el LSM-Tree (Log-Structured Merge Tree).

**Arquitectura LSM-Tree**

- Memtable: Estructura en memoria (generalmente un árbol balanceado como SkipList) que recibe las escrituras. Las inserciones son O(log n) en memoria.
- Commit Log (WAL): Escritura secuencial en disco para durabilidad antes de confirmar la operación.
- SSTable (Sorted String Table): Cuando la Memtable se llena, se vuelca a disco como un archivo inmutable ordenado.
- Compaction: Proceso en segundo plano que fusiona múltiples SSTables, eliminando datos obsoletos y manteniendo el rendimiento.

**Ventajas del LSM-Tree**
- Escrituras secuenciales: Todas las escrituras a disco son secuenciales (append-only), eliminando el cuello de botella de los discos mecánicos (seeking).
- Alta compresión: Al ser inmutables y ordenadas, las SSTables pueden comprimirse eficientemente.
- Escalabilidad: Arquitectura distribuida sin maestro (multi-master).

**El Desafío de las Lecturas y las Soluciones de Cassandra**

El problema del LSM-Tree es que una clave puede estar en múltiples SSTables. Para leer, Cassandra debe revisar:
1. La Memtable en memoria
2. Las SSTables en disco (potencialmente decenas o cientos)

Para hacer esto eficiente, Cassandra implementa tres optimizaciones clave:

**Bloom Filters (Filtros de Bloom)**
- ¿Qué son?: Estructura probabilística de memoria que responde ”¿está esta clave en esta SSTable?”.
- Respuestas posibles:
    - ”No” definitivo: Garantizado que la clave NO está.
    - ”Quizás sí”: Puede estar, o puede ser un falso positivo (típicamente 1% de probabilidad configurable).
- Complejidad: O(k) donde k es número de funciones hash (constante).
- Impacto: Con 100 SSTables, los bloom filters eliminan el 99 % de las comprobaciones de disco.

**Partition Index (Índice de Partición)**
- Estructura: Índice disperso que mapea rangos de claves a posiciones en el archivo de datos.
- Búsqueda: Una vez que el bloom filter dice ”quizás sí”, se consulta el índice para localizar el bloque exacto donde podría estar la clave.
- Implementación: Árbol B+ o, en versiones modernas (Cassandra 5.0+), BTI (Big Trie Index) con índices fuera del heap.

**Compression Metadata**
- Mapa de offsets comprimidos a posiciones en disco, permitir leer solo los bloques necesarios.

**Índices Secundarios en Cassandra**

Cassandra ofrece varias opciones para indexar columnas no clave:
- Secondary Index (2i): Índice local por nodo, con estructura de SSTable.
- SASI (SSTable-Attached Secondary Index): Índice más eficiente que puede usar B-Tree o Trie para búsquedas por rango y prefijo.

#### 1.3.3. MongoDB: B-Tree con Documentos

MongoDB, aunque NoSQL, toma prestada la estructura clásica de los índices SQL:
- Índice por defecto: B-Tree (variante WiredTiger).
- Índice compuesto: B-Tree con claves múltiples.
- Índice multikey: Para indexar campos que son arrays. MongoDB crea entradas de índice para cada elemento del array.
- Índice geoespacial: Implementado con geohashing o árboles cuádruples (quadtrees).
- Índice hashed: Para sharding, utiliza una función hash para distribuir claves uniformemente.

#### 1.3.4. Neo4j: Grafos y Recorridos

Neo4j es una base de datos de grafos, y sus estructuras están optimizadas para recorridos de relaciones:
- Almacenamiento nativo de grafos: Los nodos y relaciones se almacenan con punteros físicos directos (no tablas de unión).
- Estructura: Cada nodo mantiene una lista enlazada de relaciones entrantes y salientes.
- Índices: Para búsquedas por propiedades, Neo4j utiliza Índices de lucene (basados en FST - Finite State Transducers) o B-Tree (según la versión).
- Recorridos: La verdadera potencia está en los recorridos (traversals), que saltan de nodo en nodo siguiendo relaciones en tiempo constante (sin JOINs costosos).

#### 1.3.5. Elasticsearch: Índices Invertidos y FST

Elasticsearch, construido sobre Lucene, está especializado en búsqueda de texto completo:
- Índice invertido: Mapa de términos a listas de documentos que los contienen.
- Estructura: Para cada término, una lista de IDs de documentos (postings list) que suele estar comprimida con codificaciones de diferencias.
- FST (Finite State Transducer): Para la búsqueda por prefijo y autocompletado, utiliza un autómata que permite recorrer el diccionario de términos eficientemente.
- BKD Trees: Para campos numéricos y geoespaciales, utiliza árboles de bloques KD (árboles k-dimensionales) que permiten búsquedas por rango eficientes.

### 1.4. Tabla Comparativa: Complejidades Algorítmicas

| Motor | Estructura primaria | Búsqueda por clave | Búsqueda por rango |
| :--- | :--- | :--- | :--- |
| Oracle (B-Tree) | B+Tree | O(log n) | O(log n + k) |
| Oracle (Bitmap) | Bitmap | O(1) combinación | Limitado |
| SQL Server | B+Tree | O(log n) | O(log n + k) |
| MySQL InnoDB | B+Tree (clustered) | O(log n) | O(log n + k) |
| PostgreSQL B-Tree | B+Tree | O(log n) | O(log n + k) |
| PostgresQL GIN | Índice invertido | O(log n) | O(log n + k) |
| Redis Strings | sds | O(1) | No aplica |
| Redis Lists | quicklist | O(n) acceso índice | No aplica |
| Redis Sorted Sets | skiplist + hashtable | O(log n) (por score) | O(log n + k) |
| Cassandra | LSM-Tree | O(log n) por SSTable* | Limitado por clave de partición |
| MongoDB | B-Tree | O(log n) | O(log n + k) |
| Elasticsearch | Índice invertido | O(1) término a posting | O(k) |
| Neo4j | Grafos nativos | O(1) por nodo (ID) | Recorrido dependiente |

*Nota: En Cassandra, con m SSTables y bloom filters, la lectura efectiva es O(m<sub>false positives</sub> × log n) donde m<sub>false positives</sub> suele ser pequeño gracias a los filtros.
