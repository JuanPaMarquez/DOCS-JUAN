# Conceptos de RDBMS

Los sistemas de gestión de bases de datos relacionales (RDBMS) son un tipo de sistema de gestión de bases de datos que almacena y organiza los datos en tablas, lo que facilita la manipulación, consulta y gestión de la información. Siguen el modelo relacional definido por E.F. Codd en 1970, lo que significa que los datos se representan como tablas con filas y columnas.

---
## Indice

1. [Modelo de objetos en PostgreSQL](#1-modelo-de-objetos-en-postgresql)
    1. [Consultas en PostgreSQL](#11-consultas-en-postgresql)
        1. [Consulta de una tabla](#111-consulta-de-una-tabla)
    2. [Tipos de datos en PostgreSQL](#12-tipos-de-datos-en-postgresql)
    3. [Filas en PostgreSQL](#13-filas-en-postgresql)
        1. [Concepto](#131-concepto)
        2. [Comparación de filas y matrices](#132-comparación-de-filas-y-matrices)
            1. [In](#1321-in)
            2. [NOT IN](#1322-not-in)
            3. [ANY/SOME (array)](#1323-anysome-array)
            4. [ALL (array)](#1324-all-array)
            5. [Comparación de constructores de filas](#1325-comparación-de-constructores-de-filas)
            6. [Comparación de tipos de compuestos](#1326-comparación-de-tipos-de-compuestos)
    4. [Columnas en PostgreSQL](#14-columnas-en-postgresql)
        1. [Columnas](#141-columnas)
        2. [PostgreSQL ADD COLUMN](#142-postgresql-add-column)
            1. [ADD COLUMN](#1421-add-column)
    5. [Tablas en PostgreSQL](#15-tablas-en-postgresql)
        1. [Fundamentos de la tabla](#151-fundamentos-de-la-tabla)
    6. [Schemas](#16-schemas)

---

## 1. Modelo de objetos en PostgreSQL

PostgreSQL es un sistema de gestión de bases de datos objeto-relacional (ORDBMS). Esto significa que combina características de bases de datos relacionales (RDBMS) y orientadas a objetos (OODBMS). El modelo de objetos en PostgreSQL proporciona características como tipos de datos definidos por el usuario, herencia y polimorfismo, que mejoran sus capacidades más allá de un típico RDBMS basado en SQL.

### 1.1 Consultas en PostgreSQL

Las consultas son la forma principal de interactuar con una base de datos PostgreSQL y recuperar o manipular datos almacenados en sus tablas. En esta sección, cubriremos los fundamentos de las consultas en PostgreSQL - desde sentencias SELECT básicas hasta técnicas más avanzadas como joins, subconsultas y funciones agregadas.

#### 1.1.1 Consulta de una tabla 

Para recuperar datos de una tabla, se consulta la tabla. Para ello se utiliza una sentencia SQL SELECT. La sentencia se divide en una lista de selección (la parte que enumera las columnas que deben devolverse), una lista de tablas (la parte que enumera las tablas de las que deben recuperarse los datos) y una cualificación opcional (la parte que especifica cualquier restricción). Por ejemplo, para recuperar todas las filas de la tabla **weather**, escriba:

```
SELECT * FROM weather;
```

Aquí __*__ es una abreviatura de __«todas las columnas»__. Así que el mismo resultado se tendría con:

```
SELECT city, temp_lo, temp_hi, prcp, date FROM weather;
```

La salida debería ser:

  city         | temp_lo | temp_hi | prcp |    date
---------------|---------|---------|------|------------
 San Francisco |      46 |      50 | 0.25 | 1994-11-27
 San Francisco |      43 |      57 |    0 | 1994-11-29
 Hayward       |      37 |      54 |      | 1994-11-29

__(3 rows)__

Puede escribir expresiones, no sólo simples referencias a columnas, en la lista de selección. Por ejemplo, puede hacer

```
SELECT city, (temp_hi+temp_lo)/2 AS temp_avg, date FROM weather;
```

Esto debería dar:

  city         | temp_avg |    date
---------------|----------|------------
 San Francisco |       48 | 1994-11-27
 San Francisco |       50 | 1994-11-29
 Hayward       |       45 | 1994-11-29

__(3 rows)__

Observe cómo se utiliza la cláusula AS para reetiquetar la columna de salida. (La cláusula AS es opcional).

Una consulta puede «calificarse» añadiendo una cláusula WHERE que especifique qué filas se desean obtener. La cláusula WHERE contiene una expresión booleana (valor de verdad), y sólo se devuelven las filas para las que la expresión booleana es verdadera. Los operadores booleanos habituales (AND, OR y NOT) están permitidos en la calificación. Por ejemplo, lo siguiente recupera el tiempo de San Francisco en días lluviosos:

```
SELECT * FROM weather`
WHERE city = 'San Francisco' AND prcp > 0.0;
```

resultado:

  city         | temp_lo | temp_hi | prcp |    date
---------------|---------|---------|------|------------
 San Francisco |      46 |      50 | 0.25 | 1994-11-27

__(1 row)__

Puede solicitar que los resultados de una consulta se devuelvan ordenados:

```
SELECT * FROM weather ORDER BY city;
```

 city          | temp_lo | temp_hi | prcp |    date
---------------|---------|---------|------|------------
 Hayward       |      37 |      54 |      | 1994-11-29
 San Francisco |      43 |      57 |    0 | 1994-11-29
 San Francisco |      46 |      50 | 0.25 | 1994-11-27

En este ejemplo, el orden de clasificación no está totalmente especificado, por lo que podría obtener las filas de San Francisco en cualquier orden. Pero si lo hace, siempre obtendrá los resultados que se muestran arriba:

```
SELECT * FROM weather ORDER BY city, temp_lo;
```

Puede solicitar que se eliminen las filas duplicadas del resultado de una consulta:

```
SELECT DISTINCT city FROM weather;
```

 city          |
---------------|
 Hayward       |
 San Francisco |

__(2 rows)__

También en este caso, el orden de las filas puede variar. Puede asegurar resultados consistentes usando DISTINCT y ORDER BY juntos:

```
SELECT DISTINCT city FROM weather ORDER BY city;
```

Aunque __SELECT *__ es útil para consultas improvisadas, en general se considera de mal estilo en código de producción, ya que añadir una columna a la tabla cambiaría los resultados.

En algunos sistemas de bases de datos, incluyendo versiones antiguas de PostgreSQL, la implementación de DISTINCT ordena automáticamente las filas, por lo que ORDER BY es innecesario. Pero esto no es requerido por el estándar SQL, y PostgreSQL actual no garantiza que DISTINCT cause que las filas sean ordenadas.

### 1.2 Tipos de datos en PostgreSQL

PostgreSQL ofrece un conjunto rico y diverso de tipos de datos, que se adaptan a una amplia gama de aplicaciones y garantizan la integridad de los datos y el rendimiento. Estos incluyen tipos numéricos estándar como enteros, números de punto flotante, y tipos seriales para campos auto-incrementados. Los tipos de caracteres como VARCHAR y TEXT manejan diferentes longitudes de texto, mientras que DATE, TIME y TIMESTAMP soportan una variedad de requisitos de datos temporales. PostgreSQL también soporta un amplio conjunto de tipos booleanos, enumerados (ENUM) y compuestos, que permiten estructuras de datos más complejas. Además, destaca por su compatibilidad con los tipos de datos JSON y JSONB, que permiten un almacenamiento y consulta eficientes de datos semiestructurados. La inclusión de tipos de array, tipos de datos geométricos y la extensión PostGIS para datos geográficos amplía aún más la versatilidad de PostgreSQL, convirtiéndolo en una potente herramienta para un amplio espectro de necesidades de gestión de datos.

| **Nombre**                                | **Alias**             | **Descripción**                                                              |
|-------------------------------------------|-----------------------|------------------------------------------------------------------------------|
| `bigint`                                  | `int8`                | Entero con signo de 8 bytes.                                                |
| `bigserial`                               | `serial8`             | Entero autoincremental de 8 bytes.                                          |
| `bit [ (n) ]`                             |                       | Cadena de bits de longitud fija.                                            |
| `bit varying [ (n) ]`                     | `varbit [ (n) ]`      | Cadena de bits de longitud variable.                                        |
| `boolean`                                 | `bool`                | Valor lógico booleano (verdadero/falso).                                    |
| `box`                                     |                       | Caja rectangular en un plano.                                               |
| `bytea`                                   |                       | Datos binarios (“matriz de bytes”).                                         |
| `character [ (n) ]`                       | `char [ (n) ]`        | Cadena de caracteres de longitud fija.                                      |
| `character varying [ (n) ]`               | `varchar [ (n) ]`     | Cadena de caracteres de longitud variable.                                  |
| `cidr`                                    |                       | Dirección de red IPv4 o IPv6.                                               |
| `circle`                                  |                       | Círculo en un plano.                                                        |
| `date`                                    |                       | Fecha de calendario (año, mes, día).                                        |
| `double precision`                        | `float8`              | Número de coma flotante de doble precisión (8 bytes).                       |
| `inet`                                    |                       | Dirección de host IPv4 o IPv6.                                              |
| `integer`                                 | `int, int4`           | Entero con signo de 4 bytes.                                                |
| `interval [ fields ] [ (p) ]`             |                       | Intervalo de tiempo.                                                        |
| `json`                                    |                       | Datos JSON textuales.                                                       |
| `jsonb`                                   |                       | Datos JSON binarios descompuestos.                                          |
| `line`                                    |                       | Línea infinita en un plano.                                                 |
| `lseg`                                    |                       | Segmento de línea en un plano.                                              |
| `macaddr`                                 |                       | Dirección MAC (Control de Acceso a Medios).                                 |
| `macaddr8`                                |                       | Dirección MAC en formato EUI-64.                                            |
| `money`                                   |                       | Monto en moneda.                                                            |
| `numeric [ (p, s) ]`                      | `decimal [ (p, s) ]`  | Número exacto con precisión seleccionable.                                  |
| `path`                                    |                       | Camino geométrico en un plano.                                              |
| `pg_lsn`                                  |                       | Número de Secuencia de Registro de PostgreSQL.                              |
| `pg_snapshot`                             |                       | Instantánea de ID de transacción a nivel de usuario.                        |
| `point`                                   |                       | Punto geométrico en un plano.                                               |
| `polygon`                                 |                       | Camino geométrico cerrado en un plano.                                      |
| `real`                                    | `float4`              | Número de coma flotante de precisión simple (4 bytes).                      |
| `smallint`                                | `int2`                | Entero con signo de 2 bytes.                                                |
| `smallserial`                             | `serial2`             | Entero autoincremental de 2 bytes.                                          |
| `serial`                                  | `serial4`             | Entero autoincremental de 4 bytes.                                          |
| `text`                                    |                       | Cadena de caracteres de longitud variable.                                  |
| `time [ (p) ] [ without time zone ]`      |                       | Hora del día sin zona horaria.                                              |
| `time [ (p) ] with time zone`             | `timetz`              | Hora del día con zona horaria.                                              |
| `timestamp [ (p) ] [ without time zone ]` |                       | Fecha y hora sin zona horaria.                                              |
| `timestamp [ (p) ] with time zone`        | `timestamptz`         | Fecha y hora con zona horaria.                                              |
| `tsquery`                                 |                       | Consulta de búsqueda de texto.                                              |
| `tsvector`                                |                       | Documento de búsqueda de texto.                                             |
| `txid_snapshot`                           |                       | Instantánea de ID de transacción a nivel de usuario (obsoleta; ver pg_snapshot). |
| `uuid`                                    |                       | Identificador único universal.                                              |
| `xml`                                     |                       | Datos XML.                                                                  |

> **Compatibilidad**: SQL especifica los siguientes tipos (o su ortografía): bigint, bit, bit variable, boolean, char, character variable, character, varchar, date, double precision, integer, interval, numeric, decimal, real, smallint, time (with or without time zone), timestamp (with or without time zone), xml.

Cada tipo de datos tiene una representación externa determinada por sus funciones de entrada y salida. Muchos de los tipos incorporados tienen formatos externos obvios. Sin embargo, varios tipos son exclusivos de PostgreSQL, como las rutas geométricas, o tienen varios formatos posibles, como los tipos de fecha y hora. Algunas de las funciones de entrada y salida no son invertibles, es decir, el resultado de una función de salida puede perder precisión cuando se compara con la entrada original.

### 1.3 Filas en PostgreSQL

Una fila en PostgreSQL representa un registro único e identificable con un conjunto específico de campos en una tabla. Cada fila de una tabla se compone de una o más columnas, donde cada columna puede almacenar un tipo específico de datos (por ejemplo, entero, carácter, fecha, etc.). La estructura de una tabla determina el esquema de sus filas, y cada fila de una tabla debe adherirse a este esquema.

#### 1.3.1 Concepto

La noción fundamental en Postgres es la de tabla, que es una colección de filas con nombre. Cada fila tiene el mismo conjunto de columnas con nombre, y cada columna es de un tipo específico. Además, cada fila tiene un identificador de objeto permanente (OID) que es único en todo el clúster de la base de datos. Históricamente, las tablas se han denominado clases en Postgres, las filas son instancias de objetos y las columnas son atributos. Esto tiene sentido si se consideran los aspectos objeto-relacionales del sistema de base de datos, pero en este manual utilizaremos la terminología SQL habitual. Como se ha comentado anteriormente, las tablas se agrupan en bases de datos, y un conjunto de bases de datos gestionadas por un único proceso postmaster constituye un cluster de bases de datos.

#### 1.3.2 Comparación de filas y matrices

Esta sección describe varias construcciones especializadas para realizar comparaciones múltiples entre grupos de valores. Estas formas están relacionadas sintácticamente con las formas de subconsulta de la sección anterior, pero no implican subconsultas. Las formas que involucran subexpresiones de matrices son extensiones de PostgreSQL; el resto son compatibles con SQL. Todas las formas de expresión documentadas en esta sección devuelven resultados booleanos (verdadero/falso).

##### 1.3.2.1 IN

```
expression IN (value [, ...])
```

El lado derecho es una lista de expresiones entre paréntesis. El resultado es __«verdadero»__ si el resultado de la expresión de la izquierda es igual a cualquiera de las expresiones de la derecha. Se trata de una notación abreviada para

```
expression = value1
OR
expression = value2
OR
...
```

##### 1.3.2.2 NOT IN

```
expression NOT IN (value [, ...])
```

El lado derecho es una lista de expresiones entre paréntesis. El resultado es «verdadero» si el resultado de la expresión de la izquierda es desigual a todas las expresiones de la derecha. Se trata de una notación abreviada para

```
expression <> value1
AND
expression <> value2
AND
...
```
Tenga en cuenta que si la expresión de la izquierda da nulo, o si no hay valores iguales en la derecha y al menos una expresión de la derecha da nulo, el resultado de la construcción NOT IN será nulo, no verdadero como cabría esperar ingenuamente. Esto se ajusta a las reglas normales de SQL para combinaciones booleanas de valores nulos.

> __Consejo:__
> __x NOT IN y es equivalente a NOT (x IN y) en todos los casos. Sin embargo, es mucho más probable que los valores nulos hagan tropezar al principiante cuando trabaja con NOT IN que cuando trabaja con IN. Si es posible, es mejor expresar la condición de forma positiva.__

##### 1.3.2.3 ANY/SOME (array)

```
expression operator ANY (array expression)
expression operator SOME (array expression)
```

El lado derecho es una expresión entre paréntesis, que debe producir un valor de matriz. La expresión de la izquierda se evalúa y compara con cada elemento de la matriz utilizando el __operador__ dado, que debe dar un resultado booleano. El resultado de ANY es «true» si se obtiene algún resultado verdadero. El resultado es «false» si no se obtiene ningún resultado verdadero (incluido el caso en que la matriz tenga cero elementos).

Si la expresión de la matriz produce una matriz nula, el resultado de ANY será nulo. Si la expresión de la izquierda produce null, el resultado de ANY es normalmente null (aunque un operador de comparación no estricto podría producir un resultado diferente). Además, si la matriz de la derecha contiene elementos nulos y no se obtiene un resultado de comparación verdadero, el resultado de ANY será nulo, no falso (de nuevo, suponiendo un operador de comparación estricto). Esto está de acuerdo con las reglas normales de SQL para combinaciones booleanas de valores nulos.

SOME es un sinónimo de ANY.

##### 1.3.2.4 ALL (array)

```
expression operator ALL (array expression)
```

El lado derecho es una expresión entre paréntesis, que debe producir un valor de matriz. La expresión de la izquierda se evalúa y compara con cada elemento de la matriz utilizando el operador dado, que debe dar un resultado booleano. El resultado de ALL es «true» si todas las comparaciones son verdaderas (incluido el caso en que la matriz tenga cero elementos). El resultado es «false» si se encuentra algún resultado falso.

Si la expresión de la matriz da como resultado una matriz nula, el resultado de ALL será nulo. Si la expresión de la izquierda produce null, el resultado de ALL es normalmente null (aunque un operador de comparación no estricto podría producir un resultado diferente). Además, si la matriz de la derecha contiene algún elemento nulo y no se obtiene un resultado de comparación falso, el resultado de ALL será nulo, no verdadero (de nuevo, suponiendo un operador de comparación estricto). Esto está de acuerdo con las reglas normales de SQL para combinaciones booleanas de valores nulos.

##### 1.3.2.5 Comparación de constructores de filas

```
row_constructor operator row_constructor
```

Cada lado es un constructor de fila. Los dos constructores de fila deben tener el mismo número de campos. El operador dado se aplica a cada par de campos correspondientes. (Dado que los campos pueden ser de tipos diferentes, esto significa que se puede seleccionar un operador específico diferente para cada par). Todos los operadores seleccionados deben ser miembros de alguna clase de operadores B-tree, o ser el negador de un miembro = de una clase de operadores B-tree, lo que significa que la comparación de constructores de fila sólo es posible cuando el operador es =, <>, <, <=, >, o >=, o tiene una semántica similar a uno de ellos.

Los casos = y <> funcionan de forma ligeramente distinta a los demás. Dos filas se consideran iguales si todos sus miembros correspondientes no son nulos y son iguales; las filas son desiguales si alguno de sus miembros correspondientes no es nulo y es desigual; en caso contrario, el resultado de la comparación de filas es desconocido (nulo).

Para los casos <, <=, > y >=, los elementos de las filas se comparan de izquierda a derecha, deteniéndose en cuanto se encuentra un par de elementos desigual o nulo. Si cualquiera de este par de elementos es nulo, el resultado de la comparación de filas es desconocido (nulo); en caso contrario, la comparación de este par de elementos determina el resultado. Por ejemplo, ROW(1,2,NULL) < ROW(1,3,0) da verdadero, no nulo, porque no se considera el tercer par de elementos.

```
row_constructor IS DISTINCT FROM row_constructor
```

Esta construcción es similar a una comparación de filas <>, pero no produce nulos para entradas nulas. En su lugar, cualquier valor nulo se considera desigual a (distinto de) cualquier valor no nulo, y dos nulos cualesquiera se consideran iguales (no distintos). Por lo tanto, el resultado será verdadero o falso, nunca nulo.

```
row_constructor IS NOT DISTINCT FROM row_constructor
```

Esta construcción es similar a una comparación de filas =, pero no produce nulos para entradas nulas. En su lugar, cualquier valor nulo se considera desigual a (distinto de) cualquier valor no nulo, y dos nulos cualesquiera se consideran iguales (no distintos). Por lo tanto, el resultado siempre será verdadero o falso, nunca nulo.

##### 1.3.2.6 Comparación de tipos de compuestos

```
record operator record
```

La especificación SQL requiere que la comparación por filas devuelva NULL si el resultado depende de comparar dos valores NULL o un NULL y un no-NULL. PostgreSQL hace esto sólo cuando compara los resultados de dos constructores de fila o cuando compara un constructor de fila con la salida de una subconsulta. En otros contextos en los que se comparan dos valores de tipo compuesto, dos valores de campo NULL se consideran iguales, y un NULL se considera mayor que un no-NULL. Esto es necesario para tener un comportamiento de ordenación e indexación coherente para los tipos compuestos.

Se evalúa cada lado y se comparan por filas. Las comparaciones de tipos compuestos están permitidas cuando el operador es =, <>, <, <=, > o >=, o tiene una semántica similar a uno de ellos. (En concreto, un operador puede ser un operador de comparación de filas si es miembro de una clase de operadores de árbol B, o es el negador del miembro = de una clase de operadores de árbol B). El comportamiento por defecto de los operadores anteriores es el mismo que el de __IS [ NOT ] DISTINCT FROM__ para los constructores de filas (véase el apartado 9.25.5).

Para soportar la comparación de filas que incluyen elementos sin una clase de operador B-tree por defecto, se definen los siguientes operadores para la comparación de tipos compuestos: *=, *<>, *<, *<=, *> y *>=. Estos operadores comparan la representación binaria interna de las dos filas. Dos filas pueden tener una representación binaria diferente aunque la comparación de las dos filas con el operador de igualdad sea verdadera. El orden de las filas con estos operadores de comparación es determinista, pero no tiene ningún otro significado. Estos operadores se utilizan internamente para las vistas materializadas y pueden ser útiles para otros fines especializados, como la replicación y la deduplicación del árbol B. Sin embargo, no están pensados para ser útiles en general a la hora de escribir consultas.

### 1.4 Columnas en PostgreSQL

Las columnas son un componente fundamental del modelo de objetos de PostgreSQL. Se utilizan para almacenar los datos reales dentro de una tabla y definir sus atributos, como el tipo de datos, restricciones y otras propiedades.

#### 1.4.1 Columnas

Las columnas de vista contienen información sobre todas las columnas de tabla (o columnas de vista) de la base de datos. No se incluyen las columnas del sistema (ctid, etc.). Sólo se muestran las columnas a las que tiene acceso el usuario actual (por ser el propietario o tener algún privilegio).

| **Columna**              | **Tipo**              | **Descripción**                                                                                                                                                                             |
|--------------------------|-----------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `table_catalog`          | `sql_identifier`     | Nombre de la base de datos que contiene la tabla (siempre la base de datos actual).                                                                                                         |
| `table_schema`           | `sql_identifier`     | Nombre del esquema que contiene la tabla.                                                                                                                                                   |
| `table_name`             | `sql_identifier`     | Nombre de la tabla.                                                                                                                                                                         |
| `column_name`            | `sql_identifier`     | Nombre de la columna.                                                                                                                                                                       |
| `ordinal_position`       | `cardinal_number`    | Posición ordinal de la columna dentro de la tabla (la cuenta comienza en 1).                                                                                                                |
| `column_default`         | `character_data`     | Expresión por defecto de la columna.                                                                                                                                                        |
| `is_nullable`            | `yes_or_no`          | `YES` si la columna puede ser nula, `NO` si se sabe que no puede ser nula.                                                                                                                  |
| `data_type`              | `character_data`     | Tipo de dato de la columna. Puede ser un tipo incorporado, `ARRAY`, o `USER-DEFINED`. Si se basa en un dominio, refiere al tipo subyacente.                                                 |
| `character_maximum_length` | `cardinal_number` | Longitud máxima declarada si el tipo de dato es una cadena de caracteres o bits; nulo para otros tipos o si no se declaró longitud máxima.                                                  |
| `character_octet_length` | `cardinal_number`    | Longitud máxima posible en octetos (bytes) si es un tipo de caracteres; nulo para otros tipos.                                                                                              |
| `numeric_precision`      | `cardinal_number`    | Precisión (número de dígitos significativos) si es un tipo numérico; nulo para otros tipos.                                                                                                 |
| `numeric_precision_radix`| `cardinal_number`    | Base de la precisión (`2` o `10`) para tipos numéricos; nulo para otros tipos.                                                                                                              |
| `numeric_scale`          | `cardinal_number`    | Escala (dígitos a la derecha del punto decimal) para tipos numéricos exactos; nulo para otros tipos.                                                                                        |
| `datetime_precision`     | `cardinal_number`    | Precisión en segundos fraccionales para tipos de fecha, hora, timestamp o intervalo; nulo para otros tipos.                                                                                 |
| `interval_type`          | `character_data`     | Especificación de los campos incluidos en un intervalo (ej., `YEAR TO MONTH`); nulo para otros tipos.                                                                                       |
| `interval_precision`     | `cardinal_number`    | Aplica a una característica no disponible en PostgreSQL (ver `datetime_precision`).                                                                                                         |
| `character_set_catalog`  | `sql_identifier`     | Aplica a una característica no disponible en PostgreSQL.                                                                                                                                    |
| `character_set_schema`   | `sql_identifier`     | Aplica a una característica no disponible en PostgreSQL.                                                                                                                                    |
| `character_set_name`     | `sql_identifier`     | Aplica a una característica no disponible en PostgreSQL.                                                                                                                                    |
| `collation_catalog`      | `sql_identifier`     | Nombre de la base de datos que contiene la colación de la columna; nulo si es predeterminado o no es colacionable.                                                                           |
| `collation_schema`       | `sql_identifier`     | Nombre del esquema que contiene la colación de la columna; nulo si es predeterminado o no es colacionable.                                                                                   |
| `collation_name`         | `sql_identifier`     | Nombre de la colación de la columna; nulo si es predeterminado o no es colacionable.                                                                                                        |
| `domain_catalog`         | `sql_identifier`     | Nombre de la base de datos donde está definido el dominio del tipo de columna; nulo si no aplica.                                                                                           |
| `domain_schema`          | `sql_identifier`     | Nombre del esquema donde está definido el dominio del tipo de columna; nulo si no aplica.                                                                                                   |
| `domain_name`            | `sql_identifier`     | Nombre del dominio del tipo de columna; nulo si no aplica.                                                                                                                                  |
| `udt_catalog`            | `sql_identifier`     | Nombre de la base de datos donde se define el tipo de dato de la columna; nulo si no aplica.                                                                                                |
| `udt_schema`             | `sql_identifier`     | Nombre del esquema donde se define el tipo de dato de la columna; nulo si no aplica.                                                                                                        |
| `udt_name`               | `sql_identifier`     | Nombre del tipo de dato de la columna; nulo si no aplica.                                                                                                                                   |
| `scope_catalog`          | `sql_identifier`     | Aplica a una característica no disponible en PostgreSQL.                                                                                                                                    |
| `scope_schema`           | `sql_identifier`     | Aplica a una característica no disponible en PostgreSQL.                                                                                                                                    |
| `scope_name`             | `sql_identifier`     | Aplica a una característica no disponible en PostgreSQL.                                                                                                                                    |
| `maximum_cardinality`    | `cardinal_number`    | Siempre nulo, ya que los arreglos tienen cardinalidad ilimitada en PostgreSQL.                                                                                                              |
| `dtd_identifier`         | `sql_identifier`     | Identificador único del descriptor de tipo de datos de la columna, útil para unir con otros identificadores similares.                                                                       |
| `is_self_referencing`    | `yes_or_no`          | Aplica a una característica no disponible en PostgreSQL.                                                                                                                                    |
| `is_identity`            | `yes_or_no`          | `YES` si la columna es una columna de identidad; de lo contrario, `NO`.                                                                                                                     |
| `identity_generation`    | `character_data`     | Si la columna es de identidad, indica `ALWAYS` o `BY DEFAULT`, según la definición.                                                                                                         |
| `identity_start`         | `character_data`     | Valor inicial de la secuencia interna de una columna de identidad; nulo si no aplica.                                                                                                       |
| `identity_increment`     | `character_data`     | Incremento de la secuencia interna de una columna de identidad; nulo si no aplica.                                                                                                          |
| `identity_maximum`       | `character_data`     | Valor máximo de la secuencia interna de una columna de identidad; nulo si no aplica.                                                                                                        |
| `identity_minimum`       | `character_data`     | Valor mínimo de la secuencia interna de una columna de identidad; nulo si no aplica.                                                                                                        |
| `identity_cycle`         | `yes_or_no`          | `YES` si la secuencia interna de una columna de identidad tiene ciclo; de lo contrario, `NO`.                                                                                               |
| `is_generated`           | `character_data`     | `ALWAYS` si la columna es generada, de lo contrario, `NEVER`.                                                                                                                               |
| `generation_expression`  | `character_data`     | Expresión de generación si la columna es generada; nulo si no aplica.                                                                                                                       |
| `is_updatable`           | `yes_or_no`          | `YES` si la columna es actualizable; de lo contrario, `NO`.                                                                                                                                 |

Dado que los tipos de datos se pueden definir de varias maneras en SQL, y PostgreSQL contiene formas adicionales de definir tipos de datos, su representación en el esquema de información puede ser algo difícil. La columna data_type se supone que identifica el tipo incorporado subyacente de la columna. En PostgreSQL, esto significa que el tipo está definido en el esquema del catálogo del sistema pg_catalog. Esta columna puede ser útil si la aplicación puede manejar los tipos incorporados conocidos de forma especial (por ejemplo, formatear los tipos numéricos de forma diferente o utilizar los datos en las columnas de precisión). Las columnas udt_name, udt_schema y udt_catalog siempre identifican el tipo de datos subyacente de la columna, incluso si la columna está basada en un dominio. (Dado que PostgreSQL trata los tipos incorporados como tipos definidos por el usuario, los tipos incorporados también aparecen aquí. Esto es una extensión del estándar SQL). Estas columnas deberían usarse si una aplicación quiere procesar los datos de forma diferente según el tipo, porque en ese caso no importaría si la columna está realmente basada en un dominio. Si la columna se basa en un dominio, la identidad del dominio se almacena en las columnas nombre_dominio, esquema_dominio y catálogo_dominio. Si quieres emparejar columnas con sus tipos de datos asociados y tratar los dominios como tipos separados, podrías escribir coalesce(nombre_dominio, nombre_udt), etc.

#### 1.4.2 PostgreSQL ADD COLUMN

Para añadir una columna a una tabla existente, tenemos que utilizar la sentencia ALTER TABLE.

* La sentencia ALTER TABLE se utiliza para añadir, eliminar o modificar columnas de una tabla existente.
* La sentencia ALTER TABLE también se utiliza para añadir y eliminar varias restricciones en una tabla existente.

##### 1.4.2.1 ADD COLUMN

Queremos añadir una columna llamada __color__ a nuestra tabla de __cars__.

Al añadir columnas también debemos especificar el tipo de datos de la columna. Nuestra columna color será un __string__, y especificamos tipos de __string__ con la palabra clave __VARCHAR__. también queremos restringir el número de caracteres a 255:

__Ejemplo:__
Añade una columna llamada color:
```
ALTER TABLE cars
ADD color VARCHAR(255);
```

### 1.5 Tablas en PostgreSQL

Una tabla es uno de los objetos primarios de almacenamiento de datos en PostgreSQL. En términos simples, una tabla es una colección de filas o registros, organizados en columnas. Cada columna tiene un nombre único y contiene datos de un tipo específico.

#### 1.5.1 Fundamentos de la tabla

Una tabla en una base de datos relacional es muy parecida a una tabla en papel: Consta de filas y columnas. El número y orden de las columnas es fijo, y cada columna tiene un nombre. El número de filas es variable: refleja la cantidad de datos almacenados en un momento dado. SQL no garantiza el orden de las filas de una tabla. Cuando se lee una tabla, las filas aparecerán en un orden no especificado, a menos que se solicite explícitamente su ordenación Además, SQL no asigna identificadores únicos a las filas, por lo que es posible tener varias filas completamente idénticas en una tabla. Esto es una consecuencia del modelo matemático que subyace en SQL, pero normalmente no es deseable. Más adelante veremos cómo resolver este problema.

Cada columna tiene un tipo de datos. El tipo de datos restringe el conjunto de valores posibles que pueden asignarse a una columna y asigna semántica a los datos almacenados en la columna para que puedan utilizarse para cálculos. Por ejemplo, una columna declarada de tipo numérico no aceptará cadenas de texto arbitrarias, y los datos almacenados en ella podrán utilizarse para cálculos matemáticos. Por el contrario, una columna declarada de tipo cadena de caracteres aceptará casi cualquier tipo de datos, pero no se presta a cálculos matemáticos, aunque están disponibles otras operaciones como la concatenación de cadenas.

PostgreSQL incluye un conjunto considerable de tipos de datos incorporados que se adaptan a muchas aplicaciones. Los usuarios también pueden definir sus propios tipos de datos. Algunos de los tipos de datos más utilizados son __integer__ para números enteros, __numeric__ para números posiblemente fraccionarios, __text__ para cadenas de caracteres, __date__ para fechas, __time__ para valores de hora del día y __timestamp__ para valores que contienen fecha y hora.

Para crear una tabla, se utiliza el comando __CREATE TABLE__. En este comando se especifica al menos un nombre para la nueva tabla, los nombres de las columnas y el tipo de datos de cada columna. Por ejemplo:

```
CREATE TABLE my_first_table (
    first_column text,
    second_column integer
);
```

Esto crea una tabla llamada __my_first_table__ con dos columnas. La primera columna se denomina __first_column__ y tiene un tipo de datos de texto; la segunda columna tiene el nombre __second_column__ y el tipo entero. Los nombres de tabla y columna siguen la sintaxis de identificadores explicada en el apartado 4.1.1. Los nombres de los tipos suelen ser también identificadores, pero hay algunas excepciones. Observe que la lista de columnas está separada por comas y rodeada de paréntesis.

Por supuesto, el ejemplo anterior es muy rebuscado. Normalmente, se asignan nombres a las tablas y columnas que indican el tipo de datos que almacenan. Veamos un ejemplo más realista:

```
CREATE TABLE products (
    product_no integer,
    name text,
    price numeric
);
```

El tipo "numeric" puede almacenar componentes fraccionarios, como sería típico de las cantidades monetarias.

> __Consejo: Cuando se crean muchas tablas interrelacionadas, es aconsejable elegir un patrón de nombres coherente para las tablas y las columnas. Por ejemplo, existe la opción de utilizar nombres singulares o plurales para los nombres de las tablas, ambos preferidos por algunos teóricos.__

Existe un límite en el número de columnas que puede contener una tabla. Dependiendo del tipo de columna, oscila entre 250 y 1600. Sin embargo, definir una tabla con un número de columnas tan elevado es poco habitual y, a menudo, un diseño cuestionable.

Si ya no necesita una tabla, puede eliminarla mediante el comando DROP TABLE. Por ejemplo

```
DROP TABLE my_first_table;
DROP TABLE products;
```

Intentar eliminar una tabla que no existe es un error. No obstante, es habitual en los archivos de script SQL intentar eliminar incondicionalmente cada tabla antes de crearla, ignorando cualquier mensaje de error, de forma que el script funcione tanto si la tabla existe como si no. (Si lo desea, puede utilizar la variante DROP TABLE IF EXISTS para evitar los mensajes de error, pero esto no es SQL estándar).

### 1.6 Schemas

Los esquemas son una parte esencial del modelo de objetos de PostgreSQL, y ayudan a proporcionar estructura, organización y espacio de nombres para los objetos de la base de datos. Un esquema es una colección de objetos de base de datos, como tablas, vistas, índices y funciones, que se organizan dentro de un espacio de nombres específico.