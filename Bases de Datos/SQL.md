<h1>SQL</h1>

***Index***:
<!-- TOC -->
  * [¿Qué es SQL?](#qué-es-sql)
  * [DDL, DML y DCL](#ddl-dml-y-dcl)
  * [`SELECT`, `FROM` y `WHERE`](#select-from-y-where)
  * [`ALIAS`](#alias)
  * [*Store Procedures*](#store-procedures)
  * [Vistas](#vistas)
  * [Operadores de Combinación - `JOIN’s`](#operadores-de-combinación---joins)
  * [Operadores de Conjunto](#operadores-de-conjunto)
  * [Operadores de Comparación](#operadores-de-comparación)
  * [Operadores Lógicos](#operadores-lógicos)
  * [Funciones de Agregación (*Grouping Functions*)](#funciones-de-agregación-grouping-functions)
  * [Funciones Escalares](#funciones-escalares)
  * [Funciones de Fecha y Hora](#funciones-de-fecha-y-hora)
  * [Funciones de Ventana](#funciones-de-ventana)
<!-- TOC -->

---

## ¿Qué es SQL?
- ***Structured Query Language*** es un lenguaje para interactuar con las bases de datos.
- Una sentencia SQL es un conjunto de instrucciones que consta de comandos, cláusulas, operadores, funciones y palabras reservadas.
- _**CRUD - Create, Read, Update, Delete**_. Permite insertar, consultar, actualizar y eliminar datos en una BBDD.

## DDL, DML y DCL
Hay tres grupos de instrucciones que forman SQL:
  - **DDL (Lenguaje de Definición de Datos)** → Son las instrucciones que permiten trabajar con la *metadata*, con el esquema de la Base de Datos. Se incluyen comandos como los siguientes:  
    - `CREATE`: Crea objetos  
    - `ALTER`: Modifica objetos  
    - `DROP`: Elimina objetos  
  - **DML (Lenguaje de Manipulación de Datos)** → Son las instrucciones que permiten accesar y manipular los datos. Se incluyen comandos como los siguientes:  
    - `SELECT`: Consulta datos  
    - `INSERT`: Agrega datos  
    - `UPDATE`: Modifica datos  
    - `DELETE`: Elimina datos  
    - `TRUNCATE`: Elimina todos los datos de una tabla ☠️  
  - **DCL (Lenguaje de Control de Datos)** → Son las instrucciones que permiten definir la permisología que los usuarios van a tener sobre los objetos de la Base de Datos. e incluyen comandos como los siguientes:  
    - `GRANT`: Otorga permisos  
    - `REVOKE`: Quita permisos

## `SELECT`, `FROM` y `WHERE`
- `SELECT` → ¿Qué lista de atributos (columnas) quiero recuperar? **OBLIGATORIA**. Define qué datos se van a mostrar.
- `FROM` → ¿De qué tabla o tablas se deben extraer los datos? **OBLIGATORIA**. Anteponer siempre al nombre de la tabla el *dataset* `WHOWNER`
- `WHERE` → ¿Deben cumplir alguna condición? **OPCIONAL**. Permite filtrar los resultados (filas).

## `ALIAS`
- Es un nombre temporal que se asigna a una tabla o a una columna en una consulta.
- Utiliza la palabra reservada `as` → Ej.: `Select USERNAME as Nombre_Usuario`

## *Store Procedures*
- Grupo de una o más instrucciones SQL que se almacenan como un objeto en la metadata de la Base de Datos.  
- Puede invocar a otro SP o puede ser invocado desde otros SP’s.  
- Desde una Función, no es posible invocar SP’s, ya que los SP’s pueden afectar el estado de la Base de Datos (con instrucciones DML, configurando opciones, etc.) y esta es una restricción de las Funciones.  
- Siempre debe devolver un valor. Si no se le especifica ningún valor de retorno, el DBSM va a asignar un número entero para indicar el estado final de la ejecución del SP.

## Vistas
Es una tabla virtual que se define a partir de una o más tablas reales mediante una consulta (generalmente una sentencia `SELECT`). No almacena datos por sí misma, sino que presenta los datos de las tablas base en una forma específica.

**Vista común**:  
- Ideal para mostrar datos actualizados y ocultar complejidad o restringir acceso.  
- Puede o no ser actualizable. Es decir, modificar los datos subyacentes de una única tabla base mediante operaciones DML (`INSERT`, `UPDATE`, `DELETE`) directamente sobre la vista.  
- **No almacena datos** → Se recalcula siempre.

**Vista materializada**:  
- Para materializar una vista, se le debe crear un índice *cluster* (primario). Para eso, la vista debe ser determinista, de lo contrario no puede ser indexada (y por lo tanto, materializada).  
- Ideal cuando hay consultas pesadas que se repiten y no se necesitan datos actualizados al instante.  
- Mejora rendimiento, pero hay que refrescarla.  
- No se actualiza automáticamente con las tablas base.

## Operadores de Combinación - `JOIN’s`
- Permiten combinar filas de dos o más tablas, basándose en una relación lógica entre ellas.  
- Utilizan la palabra reservada `ON` para especificar las condiciones bajo las cuales se deben combinar las filas de las tablas. Dicho de otra forma, define en base a qué se debe establecer una coincidencia.  
- Existen cuatro tipos principales de `JOIN’s`:  
   - `INNER JOIN` (o simplemente, `JOIN`) → Es la intersección entre dos conjuntos. Devuelve solo las filas que tienen coincidencias en ambas tablas.  
   - `LEFT JOIN` → Devuelve todas las filas de la tabla de la izquierda y las filas coincidentes de la tabla de la derecha. Si no hay coincidencias, se devuelven `null's` para las columnas de la tabla de la derecha.  
   - `RIGHT JOIN` → Devuelve todas las filas de la tabla de la derecha y las filas coincidentes de la tabla de la izquierda. Si no hay coincidencias, se devuelven `null's` para las columnas de la tabla de la izquierda.  
   - `FULL JOIN` → Devuelve todas las filas de ambas tablas, con coincidencias donde sea posible. Si no hay coincidencias, se devuelven `null's`.

## Operadores de Conjunto
- Combinan resultados de múltiples consultas.  
    - `UNION`: Combina resultados y elimina duplicados.  
    - `UNION ALL`: Combina resultados y conserva duplicados.  
    - `INTERSECT`: Devuelve filas comunes en ambas consultas.  
    - `EXCEPT` (o `MINUS`): Devuelve filas de la primera consulta que no están en la segunda.

## Operadores de Comparación
- Permiten comparar valores en una consulta. Se utilizan normalmente en la cláusula `WHERE` para filtrar registros en función de ciertas condiciones.  
    - `=`: Igual a.  
    - `!=` o `<>`: No igual a.  
    - `>`: Mayor que.  
    - `<`: Menor que.  
    - `>=`: Mayor o igual que.  
    - `<=`: Menor o igual que.

## Operadores Lógicos
- Se utilizan para combinar múltiples condiciones en una sentencia `WHERE`, permitiendo construir consultas más complejas y precisas.  
    - `AND`: Se utiliza para combinar condiciones.  
    - `OR`: Se utiliza para establecer condiciones alternativas.  
    - `NOT`: Invierte el valor de una condición.

## Funciones de Agregación (*Grouping Functions*)
- Realizan un cálculo sobre un conjunto de valores y devuelven un único valor resumido.  
    - `COUNT`: Calcula la cantidad de filas en un conjunto de resultados.  
    - `MIN`: Devuelve el valor mínimo de un conjunto de valores. Puede aplicarse a columnas numéricas, *strings* o fechas.  
    - `MAX`: Devuelve el valor máximo de un conjunto de valores. Puede aplicarse a columnas numéricas, *strings* o fechas.  
    - `SUM`: Devuelve la suma total de los valores en una columna numérica.  
    - `AVG`: Devuelve el promedio (media aritmética) de los valores en una columna numérica.  
- Las funciones de agregación se suelen utilizar en conjunto con algunas cláusulas que permiten visualizar mejor los resultados.  
    - `GROUP BY`: Organiza los resultados de una consulta en grupos según los valores de una o más columnas específicas.  
    - `ORDER BY`: Organiza los resultados de una consulta por los valores de una o más columnas. Puede ser ascendente (`ASC`) o descendente (`DESC`). → Ej.: `order by USERNAME DESC`  
    - `HAVING`: Permite filtrar los resultados después de que se ha realizado la agregación. → Ej.: `Having Cantidad > 100`

## Funciones Escalares
- Operan sobre un único valor y devuelven un único valor. Algunas son específicas del tipo de datos.  
    - `UPPER()`: Convierte texto a mayúsculas.  
    - `LOWER()`: Convierte texto a minúsculas.  
    - `LENGTH()`: Devuelve la longitud de una cadena.  
    - `ROUND()`: Redondea un número a una cierta cantidad de decimales.  
    - `CONCAT()`: Combina varias cadenas de texto en una.

## Funciones de Fecha y Hora
- Están diseñadas para trabajar con datos de fecha y hora.  
    - `NOW()`: Devuelve la fecha y hora actuales.  
    - `DATE()`: Extrae la parte de fecha de un valor de fecha y hora.  
    - `DATEDIFF()`: Calcula la diferencia en días entre dos fechas.  
    - `CURRENT_DATE()`: Devuelve la fecha actual.

## Funciones de Ventana
- Permiten realizar cálculos sobre un conjunto de filas relacionadas, a menudo dentro de una consulta que incluye **`PARTITION BY`**.  
    - `ROW_NUMBER()`: Asigna un número secuencial a las filas dentro de una partición.  
    - `RANK()`: Asigna un rango a las filas dentro de una partición.  
    - `SUM() OVER()`: Suma un conjunto de valores sobre una ventana.
