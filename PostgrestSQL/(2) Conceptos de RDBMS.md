# Conceptos de RDBMS

Los sistemas de gestión de bases de datos relacionales (RDBMS) son un tipo de sistema de gestión de bases de datos que almacena y organiza los datos en tablas, lo que facilita la manipulación, consulta y gestión de la información. Siguen el modelo relacional definido por E.F. Codd en 1970, lo que significa que los datos se representan como tablas con filas y columnas.

---
## Indice

1. [Modelo de objetos en PostgreSQL](#1-modelo-de-objetos-en-postgresql)
1.1 [Consultas en PostgreSQL](#11-consultas-en-postgresql)
1.1.1 [Consulta de una tabla](#111-consulta-de-una-tabla)
1.1.2 [Tipos de datos en PostgreSQL](#112-tipos-de-datos-en-postgresql)

---

## 1. Modelo de objetos en PostgreSQL

PostgreSQL es un sistema de gestión de bases de datos objeto-relacional (ORDBMS). Esto significa que combina características de bases de datos relacionales (RDBMS) y orientadas a objetos (OODBMS). El modelo de objetos en PostgreSQL proporciona características como tipos de datos definidos por el usuario, herencia y polimorfismo, que mejoran sus capacidades más allá de un típico RDBMS basado en SQL.

### 1.1 Consultas en PostgreSQL

Las consultas son la forma principal de interactuar con una base de datos PostgreSQL y recuperar o manipular datos almacenados en sus tablas. En esta sección, cubriremos los fundamentos de las consultas en PostgreSQL - desde sentencias SELECT básicas hasta técnicas más avanzadas como joins, subconsultas y funciones agregadas.

#### 1.1.1 Consulta de una tabla 

Para recuperar datos de una tabla, se consulta la tabla. Para ello se utiliza una sentencia SQL SELECT. La sentencia se divide en una lista de selección (la parte que enumera las columnas que deben devolverse), una lista de tablas (la parte que enumera las tablas de las que deben recuperarse los datos) y una cualificación opcional (la parte que especifica cualquier restricción). Por ejemplo, para recuperar todas las filas de la tabla **weather**, escriba:

> **`SELECT * FROM weather;`**

Aquí __*__ es una abreviatura de __«todas las columnas»__. Así que el mismo resultado se tendría con:

> **`SELECT city, temp_lo, temp_hi, prcp, date FROM weather;`**

La salida debería ser:

  city         | temp_lo | temp_hi | prcp |    date
---------------|---------|---------|------|------------
 San Francisco |      46 |      50 | 0.25 | 1994-11-27
 San Francisco |      43 |      57 |    0 | 1994-11-29
 Hayward       |      37 |      54 |      | 1994-11-29

__(3 rows)__

Puede escribir expresiones, no sólo simples referencias a columnas, en la lista de selección. Por ejemplo, puede hacer

> **`SELECT city, (temp_hi+temp_lo)/2 AS temp_avg, date FROM weather;`**

Esto debería dar:

  city         | temp_avg |    date
---------------|----------|------------
 San Francisco |       48 | 1994-11-27
 San Francisco |       50 | 1994-11-29
 Hayward       |       45 | 1994-11-29

__(3 rows)__

Observe cómo se utiliza la cláusula AS para reetiquetar la columna de salida. (La cláusula AS es opcional).

Una consulta puede «calificarse» añadiendo una cláusula WHERE que especifique qué filas se desean obtener. La cláusula WHERE contiene una expresión booleana (valor de verdad), y sólo se devuelven las filas para las que la expresión booleana es verdadera. Los operadores booleanos habituales (AND, OR y NOT) están permitidos en la calificación. Por ejemplo, lo siguiente recupera el tiempo de San Francisco en días lluviosos:

> `SELECT * FROM weather`
> **`WHERE city = 'San Francisco' AND prcp > 0.0;`**

resultado:

  city         | temp_lo | temp_hi | prcp |    date
---------------|---------|---------|------|------------
 San Francisco |      46 |      50 | 0.25 | 1994-11-27

__(1 row)__

Puede solicitar que los resultados de una consulta se devuelvan ordenados:

> **`SELECT * FROM weather ORDER BY city;`**

 city          | temp_lo | temp_hi | prcp |    date
---------------|---------|---------|------|------------
 Hayward       |      37 |      54 |      | 1994-11-29
 San Francisco |      43 |      57 |    0 | 1994-11-29
 San Francisco |      46 |      50 | 0.25 | 1994-11-27

En este ejemplo, el orden de clasificación no está totalmente especificado, por lo que podría obtener las filas de San Francisco en cualquier orden. Pero si lo hace, siempre obtendrá los resultados que se muestran arriba:

> **`SELECT * FROM weather ORDER BY city, temp_lo;`**

Puede solicitar que se eliminen las filas duplicadas del resultado de una consulta:

> **`SELECT DISTINCT city FROM weather;`**

 city          |
---------------|
 Hayward       |
 San Francisco |

__(2 rows)__

También en este caso, el orden de las filas puede variar. Puede asegurar resultados consistentes usando DISTINCT y ORDER BY juntos:

> **`SELECT DISTINCT city FROM weather ORDER BY city;`**

Aunque __SELECT *__ es útil para consultas improvisadas, en general se considera de mal estilo en código de producción, ya que añadir una columna a la tabla cambiaría los resultados.

En algunos sistemas de bases de datos, incluyendo versiones antiguas de PostgreSQL, la implementación de DISTINCT ordena automáticamente las filas, por lo que ORDER BY es innecesario. Pero esto no es requerido por el estándar SQL, y PostgreSQL actual no garantiza que DISTINCT cause que las filas sean ordenadas.

#### 1.1.2 Tipos de datos en PostgreSQL

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



