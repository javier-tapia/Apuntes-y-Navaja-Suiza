# Conceptos básicos

- [¿Qué es una base de datos?](#qué-es-una-base-de-datos)
- [¿Qué es una tabla?](#qué-es-una-tabla)
- [Algunos tipos de datos](#algunos-tipos-de-datos)
- [Transacciones](#transacciones)
- [Modelo Conceptual, Modelo Lógico y Modelo Físico](#modelo-conceptual-modelo-lógico-y-modelo-físico)
- [Normalización | Formas normales](#normalización--formas-normales)

---

## ¿Qué es una base de datos?
- Conjunto organizado de datos que permite el almacenamiento, gestión y recuperación de la información.

## ¿Qué es una tabla?
- Estructura dentro de una base de datos que organiza los datos en filas y columnas.  
  - Las columnas o campos de un tabla, representan los distintos atributos de los datos almacenados. Tienen un tipo de datos que indica qué datos se pueden guardar en cada una.  
  - Una fila es un registro individual dentro de una tabla que contiene datos. Se representa en una única línea horizontal y contiene valores correspondientes a los diferentes campos de la tabla.

## Algunos tipos de datos
- Numéricos: `Int64`, `numeric`  
- Texto: `string`  
- Fecha y hora: `date`, `time`, `datetime`  
- Booleanos: `boolean`

## Transacciones
- En el contexto de las bases de datos, una **transacción** es un conjunto de operaciones que se ejecutan como una única unidad de trabajo, que puede incluir una o más de las operaciones definidas como **CRUD** (***Create***, ***Read***, ***Update***, ***Delete***). Por ejemplo, se puede tener una transacción que crea un nuevo registro (*Create*) y también actualiza otros registros (*Update*) en la misma operación, y ambas deben completarse exitosamente para que la transacción se considere válida. Si alguna de las operaciones dentro de la transacción falla, se debe hacer un ***rollback*** para que los cambios no se apliquen, asegurando la integridad de los datos.  
- **ACID**: Sigla que identifica las cuatros características de vital importancia de las transacciones.  
  - ***Atomicity*** → En toda transacción, sin importar la cantidad de pasos que involucre, se deben ejecutar o todos los pasos o ninguno. Cada transacción es un solo ente.  
  - ***Consistency*** → Toda transacción debe llevar la Base de Datos de un estado válido a otro estado también válido. De no ser posible, se debe regresar la Base de Datos al estado válido anterior al inicio de la transacción.  
  - ***Isolation*** → Toda transacción que no haya finalizado debe permanecer aislada (e invisible) a cualquier otra transacción.  
  - ***Durability*** → Todo cambio realizado por una transacción ya finalizada debe permanecer en la Base de Datos, incluso en caso de fallas posteriores. Corregida la falla, los cambios realizados deben estar disponibles y en estado válido.  
- Las bases de datos transaccionales utilizan varias técnicas para manejar la **concurrencia** y evitar inconsistencias cuando múltiples transacciones se ejecutan al mismo tiempo. Aquí hay algunas de las más comunes:  
  - **Bloqueos** (***Locks***): Cuando una transacción accede a un recurso (como una fila o una tabla), se puede aplicar un bloqueo para prevenir que otras transacciones accedan al mismo recurso hasta que la primera transacción se complete. Existen diferentes tipos de bloqueos, como:  
    - **Exclusivos**: Impiden que otras transacciones lean o escriban en el recurso.  
    - **Compartidos**: Permiten que otras transacciones lean el recurso, pero no lo escriban.  
  - **Control de Concurrencia Optimista**: Este enfoque permite que las transacciones se ejecuten sin bloqueos, pero verifica si hay conflictos al momento de hacer *commit*. Si se detecta un conflicto (por ejemplo, si dos transacciones han modificado los mismos datos), una o más de las transacciones se revertirán o se volverán a intentar.  
  - **Aislamiento de Transacciones**: Cada base de datos tiene diferentes niveles de aislamiento que controlan cómo se deben comportar las transacciones concurrentes. Algunos niveles incluyen:  
    - ***Read Uncommitted***: Permite a una transacción ver los cambios no confirmados de otras transacciones.  
    - ***Read Committed***: Una transacción solo puede ver cambios que han sido confirmados.  
    - ***Repeatable Read***: Una transacción verá los mismos datos en cada lectura, incluso si otras transacciones modifican esos datos.  
    - ***Serializable***: Este es el nivel más estricto que simula que las transacciones se ejecutan una tras otra, evitando inconsistencias.  
  - **Versionado de Datos**: Algunas bases de datos utilizan un enfoque de versionado donde cada transacción trabaja con una copia de los datos, permitiendo que las transacciones se ejecuten en paralelo sin interferencias. Luego se unen los resultados al momento del *commit*.

## Modelo Conceptual, Modelo Lógico y Modelo Físico
### **Modelo Conceptual**
Es un bosquejo donde se identifica de **qué** se trata todo, **quiénes** intervienen en el negocio y **cómo** se van a hacer las cosas.

### **Modelo Lógico**
Sea como sea que se "pinte", no es suficiente para implementar físicamente la Base de Datos. Sin embargo, _**debe ser suficiente para generar un buen Modelo Físico**_. Y como a esa altura ya se deben conocer todos las Entidades con sus atributos y sus relaciones, ese es el momento de normalizar (ver [Normalización | Formas normales](#normalización--formas-normales)). Tanto el Diagrama de Entidad Relación como el Modelo Relacional son representaciones de un Modelo Lógico.

#### **Modelo Relacional y Diagramas Entidad Relación**
En **Teoría Relacional**, las Relaciones (*Relations* en inglés) se expresan como conjuntos matemáticos del estilo `R={att₁, att₂, ..., attₙ}`. Al ir aplicando operaciones a la primera Relación (que engloba todos los atributos que tenemos), se va descomponiendo en Relaciones más pequeñas tipo `R1={att₁, ... attₓ}`, `R2={att₁, ..attᵧ}`, y así. No hace falta (o no es requerido) dibujar líneas entre ellas, sino que se entiende que se relacionan por los atributos comunes que suelen escribirse al principio (llaves).  
El **Diagrama Entidad Relación** es bastante más visual y tiene distintas Notaciones que hacen "más fácil" entender las Reglas del Negocio. Otra diferencia es que las Relaciones se pasan a llamar Entidades (que luego serán tablas de la BBDD) y las relaciones (*Relationships* en inglés) se pasan a llamar Relaciones.

Algunos conceptos del Modelo Relacional:  
- **Dependencias funcionales** → Son la base del Modelo Relacional. Representan el valor de un conjunto de atributos (determinante) que permite encontrar el valor de otro conjunto de atributos (dependiente). Es decir, A → B (A determina a B).
  - **Triviales** → El dependiente es un subconjunto del determinante
  - **No Triviales** → El dependiente no es un subconjunto del determinante (puede tener elementos en común)
    - **Completas** → El dependiente no es un subconjunto del determinante y no tiene elementos en común
- **Llaves (*keys*)** → Son los determinantes de una dependencia funcional.
  - **Super Llave (*Super Key*)** → Llave que determina todos los atributos de la relación
    - **Llaves Candidatas (*Candidate Keys*)** → Son las super llaves que no tienen ningún subconjunto propio que también sea una super llave
      - **Llave Primaria (*Primary Key*)** → Llave candidata elegida. Su valor debe ser único, no se debe repetir entre las tuplas. Y además, no pueden ser nulas
      - **Llave Alterna** → Llaves candidatas no elegidas
- **Dependencias multivaluadas** → Son dependencias funcionales en las que un atributo en una tabla depende de un conjunto de atributos, pero no de forma única. En otras palabras, si tenemos una tabla en la que un conjunto de atributos determina múltiples valores para otro atributo, estamos ante una dependencia multivaluada.

### **Modelo Físico**
_**Debe ser suficiente para implementar la Base de Datos**_. Es decir, debe definir cómo será almacenada la *data* del sistema. Además, también hay un cambio de terminología:

- Entidades ———> Tablas
- Tuplas ———> Filas o registros
- Atributos ———> Columnas
- Relaciones (entre Entidades) ———> Tablas de Relación / Atributos con igual Dominio / Llaves foráneas (*Foreign Keys*)

El Modelo Físico incluye una serie de características a la hora de trabajar con las Bases de Datos:

- **Índices**
  - ***Cluster* (o primario)** → Determinan el orden físico de los registros en la tabla. Puede haber solo un índice *cluster* por tabla, ya que los datos deben estar en un solo orden físico.
  - ***Non-Cluster*** **(o secundario)** → Son estructuras separadas que apuntan a las ubicaciones físicas de los datos. Pueden existir múltiples índices *non-cluster* en una tabla.
- **Restricciones**
  - **Dominio**
    - **Tipos de datos** → Definición de los tipos de datos específicos que se utilizarán en cada columna (p. ej., entero, *varchar*, fecha, *boolean*). Son una restricción general del dominio de los atributos.
    - **Chequeo de valores** → Se pueden hacer según un Rango de Valores o por la Aplicación de Reglas. También hay que tener en cuenta que no se puede usar un campo de la misma tabla en la restricción de otro campo.
    - **Obligatoriedad** → Definir que un dato no puede ser nulo.
    - **Unicidad** → Definir que un dato debe ser único, que no se puede repetir.
    - **Valores por Defecto** → En caso que un valor sea nulo, definir un valor por defecto para el dato.
  - **Relaciones**
    - **Filas huérfanas** → Aparecen cuando se inserta un registro (fila) nuevo en una tabla con una *Foreign Key* que no existe como *Primary Key* de la tabla padre. Es decir, es una fila hija que no tiene una fila padre. Se le podría indicar al DBMS (*Database Management System*) que aplique una restricción para que no se puedan crear filas huérfanas.
    - **Actualización/Eliminación en Cascada** → Se le puede indicar al DBMS que tome la acción que se realiza en una tabla padre y la aplique también en la tabla hija. Por ejemplo, si se modifica una *Primary Key* en un registro padre, **todos los registros de la tabla hija que estaban asociados mediante una *Foreign Key* a ese registro padre, van a cambiar también el valor de la *Foreign Key*.
    - **Anulación/Valor por Defecto de Llave Foránea** → Se le puede indicar al DBMS que si la PK de la tabla padre es eliminada, anule la FK de la tabla hija en lugar de eliminar los registros asociados a la *key*. Alternativamente, se le puede indicar que le coloque un valor por defecto.
- **Particionamiento**
    - **Vertical** → Dividir una tabla en diversas subtablas que contienen diferentes columnas, manteniendo todas las filas originales. Es útil cuando ciertas columnas son utilizadas con frecuencia por separado, permitiendo que se realicen consultas más eficientes al acceder solo a las columnas necesarias.
    - **Horizontal** → Dividir una tabla en múltiples subtablas que contienen diferentes filas, manteniendo todas las columnas originales. Este tipo de particionamiento es útil para gestionar grandes volúmenes de datos, permitiendo que cada subtabla contenga un subconjunto de datos con características compartidas.

## Normalización | Formas normales
- **Normalización**  
  Es un _**proceso secuencial, cíclico y repetitivo**_ que consiste en aplicar, a todas y cada una de las Entidades del Modelo, unas Reglas bien definidas, con la finalidad de garantizar la integridad de los datos _**evitando la Redundancia, la cual es la causante de las Anomalías de diseño**_ que frecuentemente ocurren en las Bases de Datos mal diseñadas:  
    - **Anomalías de Inserción**: Algunos atributos no pueden ser insertados en una Relación sin que existan otros atributos, entre los cuales no debería haber ninguna dependencia.  
    - **Anomalías de Modificación**: Al modificar datos duplicados, no se modifican todas las instancias.  
    - **Anomalías de Eliminación**: Algunos atributos se pierden por la eliminación de otros atributos diferentes.

      El **Modelo Relacional** de por sí nomás impone una serie de reglas que serán tenidas en cuenta a la hora de iterar a través de las distintas Formas Normales. Estas reglas son:  
    - No debe haber tuplas (filas) repetidas  
    - No debe importar el orden de las tuplas (filas)  
    - Existencia de una Llave Primaria (*Primary Key*)  
    - Atributos atómicos (Ej.: Dirección → Ciudad + Dirección + Código Postal)  
- **1NF (Primera Forma Normal) →** La Relación NO DEBE tener grupos repetitivos. Para eso, se mueve el grupo repetitivo a una nueva tabla y se le agrega la *primary key* original (*foreign key*). Ejemplo 👇
    - Tabla original:

  | Código | Nombre | Apellido | Dormitorio | Cursos                              |
  |--------|--------|----------|------------|-------------------------------------|
  | A-001  | Albert | Einstein | A-305      | Matemática, Modelo de Datos, Física |
  | A-002  | Edgar  | Codd     | B-102      | Modelo de Datos, Química            |
  
    - Tabla con un grupo repetitivo (Curso 1, 2 y 3):

  | Código | Nombre | Apellido | Dormitorio | Curso 1         | Curso 2         | Curso 3 |
  |--------|--------|----------|------------|-----------------|-----------------|---------|
  | A-001  | Albert | Einstein | A-305      | Matemática      | Modelo de Datos | Física  |
  | A-002  | Edgar  | Codd     | B-102      | Modelo de Datos | Química         |         |
  
    - Tabla creando una nueva fila por cada ocurrencia:

  | Código | Nombre | Apellido | Dormitorio | Curso           |
  |--------|--------|----------|------------|-----------------|
  | A-001  | Albert | Einstein | A-305      | Matemática      |
  | A-001  | Albert | Einstein | A-305      | Modelo de Datos |
  | A-001  | Albert | Einstein | A-305      | Física          |
  | A-002  | Edgar  | Codd     | B-102      | Modelo de Datos |
  | A-002  | Edgar  | Codd     | B-102      | Química         |
  
    - Tablas resultantes:
        - Tabla de relación `Alumnos-Materias`:

      | Código Alumno (*Foreign Key*) | Código Curso (*FK*) |
      |-------------------------------|---------------------|
      | A-001                         | 101                 |
      | A-001                         | 300                 |
      | A-001                         | 201                 |
      | A-002                         | 300                 |
      | A-002                         | 401                 |
      
        - Tabla `Alumnos`:

      | Código | Nombre | Apellido | Dormitorio |
      |--------|--------|----------|------------|
      | A-001  | Albert | Einstein | A-305      |
      | A-002  | Edgar  | Codd     | B-102      |
      
        - Tabla `Materias`:

      | Código | Materia         |
      |--------|-----------------|
      | 101    | Matemática      |
      | 102    | Matemática II   |
      | 201    | Física          |
      | 202    | Física II       |
      | 300    | Modelo de Datos |
      | 301    | Diseño de BBDD  |
      | 401    | Química         |

- **2NF (Segunda Forma Normal)** → La Relación DEBE estar en 1NF y todos los atributos DEBEN depender de la *primary key* completa. Es decir, no deben tener una dependencia parcial. Esto significa que si la *primary key* no es una llave compuesta (más de una *key*), la Relación ya está en 2NF. Ejemplo 👇
    - Tabla original con *primary key* compuesta (Alumno y Código):

  | Alumno | Código | Curso            | Calificación |
  |--------|--------|------------------|--------------|
  | A-001  | 101    | Matemática       | A            |
  | A-001  | 300    | Modelos de Datos | C            |
  | A-001  | 201    | Física           | B            |
  | A-002  | 300    | Modelo de Datos  | A            |
  | A-002  | 201    | Física           | B            |
  
    - Tablas resultantes a partir de extraer la dependencia parcial (Curso sólo depende del Código, no de Alumno):

  | Código | Curso           |
  |--------|-----------------|
  | 101    | Matemática      |
  | 300    | Modelo de Datos |
  | 201    | Física          |

  | Alumno | Código | Calificación |
  |--------|--------|--------------|
  | A-001  | 101    | A            |
  | A-001  | 300    | C            |
  | A-001  | 201    | B            |
  | A-002  | 300    | A            |
  | A-002  | 201    | B            |

- **3NF (Tercera Forma Normal)** → La Relación DEBE estar en 2NF y los atributos “no determinantes” NO DEBEN depender transitivamente de la *primary key*. Es decir, si A(*PK*) → B y B → C, se debe crear una nueva tabla tal que B(ahora *PK*) → C. Ejemplo 👇
    - Tabla original:

  | Cliente # | Nombre | Apellido | Saldo   | Vendedor # | Nombre vendedor | Apellido vendedor |
  |-----------|--------|----------|---------|------------|-----------------|-------------------|
  | A-001     | Albert | Einstein | 1500.00 | SD-355     | Salvador        | Dalí              |
  | A-002     | Edgar  | Codd     | 380.00  | PP-121     | Pablo           | Picasso           |
  | A-003     | Isaac  | Newton   | 560.00  | PP-121     | Pablo           | Picasso           |
  | A-004     | Marie  | Curie    | 2000.00 | SD-355     | Salvador        | Dalí              |
  
    - Tablas resultantes a partir de extraer la dependencia transitiva:
        - Tabla `Clientes`:

      | Cliente # | Nombre | Apellido | Saldo   | Vendedor # |
      |-----------|--------|----------|---------|------------|
      | A-001     | Albert | Einstein | 1500.00 | SD-355     |
      | A-002     | Edgar  | Codd     | 380.00  | PP-121     |
      | A-003     | Isaac  | Newton   | 560.00  | PP-121     |
      | A-004     | Marie  | Curie    | 2000.00 | SD-355     |
      
        - Tabla `Vendedores`:

      | Vendedor # | Nombre vendedor | Apellido vendedor |
      |------------|-----------------|-------------------|
      | SD-355     | Salvador        | Dalí              |
      | PP-121     | Pablo           | Picasso           |

- **BCNF (Forma Normal de Boyce y Codd)** → La Relación DEBE estar en 3NF y para cada dependencia funcional no trivial que exista, el determinante DEBE ser una *Super Key*. El problema que puede resolver esta forma normal es la de las dependencias circulares. Es decir, si AB → C y C → B, se debe crear una nueva tabla tal que C(ahora *PK*) → B. Ejemplo 👇
    - Tabla original:

  | Alumno   | Curso           | Profesor |
  |----------|-----------------|----------|
  | Einstein | Matemática      | Pascal   |
  | Einstein | Modelo de Datos | Boyce    |
  | Einstein | Física          | Newton   |
  | Codd     | Modelo de Datos | Boyce    |
  | Codd     | Física          | Maxwell  |
  | Volta    | Matemática      | Pascal   |
  | Volta    | Física          | Newton   |
  
    - Tablas resultantes luego de extraer la dependencia funcional que viola el BCNF:

  | Alumno   | Profesor |
  |----------|----------|
  | Einstein | Pascal   |
  | Einstein | Boyce    |
  | Einstein | Newton   |
  | Codd     | Boyce    |
  | Codd     | Maxwell  |
  | Volta    | Pascal   |
  | Volta    | Newton   |

  | Curso           | Profesor |
  |-----------------|----------|
  | Matemática      | Pascal   |
  | Modelo de Datos | Boyce    |
  | Física          | Newton   |
  | Física          | Maxwell  |

- **4NF (Cuarta Forma Normal)** → La Relación DEBE estar en 3NF (o en BCNF) y para cada dependencia multivaluada no trivial que exista, el determinante DEBE ser una *Super Key*. Ejemplo 👇
    - Tabla original:

  | Alumno  | Idioma   | Destreza |
  |---------|----------|----------|
  | Dalí    | Español  | Pinta    |
  | Dalí    | Español  | Canta    |
  | Dalí    | Catalán  | Pinta    |
  | Dalí    | Catalán  | Canta    |
  | Picasso | Español  | Pinta    |
  | Picasso | Español  | Escribe  |
  | Picasso | Italiano | Pinta    |
  | Picasso | Italiano | Escribe  |
  
    - Tablas resultantes luego de mover la dependencia `Alumno →→ Idioma` a una relación aparte:

  | Alumno  | Idioma   |
  |---------|----------|
  | Dalí    | Español  |
  | Dalí    | Catalán  |
  | Picasso | Español  |
  | Picasso | Italiano |

  | Alumno  | Destreza |
  |---------|----------|
  | Dalí    | Pinta    |
  | Dalí    | Canta    |
  | Picasso | Pinta    |
  | Picasso | Escribe  |

- **5NF (Quinta Forma Normal)** → La Relación DEBE estar en 4NF y NO DEBE poder ser descompuesta no-aditivamente en relaciones más pequeñas (la unión natural no-aditiva -*lossless join*- de las Relaciones descompueastas, da como resultado la Relación original). Es decir, si se puede dividir la Relación de forma tal que luego se puede volver a unir mediante una operación de Unión Natural, entonces NO está en 5FN. Si se realizan las proyecciones de la Relación original, luego se aplican las operaciones de Unión Natural a dichas proyecciones y la Relación resultante no es igual a la Relación original, significa que la Relación original ya está en 5NF. Ejemplo 👇
    - Tabla original:

  | Productora | Canal       | Producto  |
  |------------|-------------|-----------|
  | Inter TV   | Canal Norte | Seriales  |
  | Inter TV   | Canal Norte | Novelas   |
  | Inter TV   | Cine Canal  | Películas |
  | Film Stars | Canal Norte | Películas |
  
    - Tablas luego de descomponer la original:

  | Productora | Canal       |
  |------------|-------------|
  | Inter TV   | Canal Norte |
  | Inter TV   | Cine Canal  |
  | Film Stars | Canal Norte |

  | Productora | Producto  |
  |------------|-----------|
  | Inter TV   | Seriales  |
  | Inter TV   | Novelas   |
  | Inter TV   | Películas |
  | Film Stars | Películas |

  | Canal       | Producto  |
  |-------------|-----------|
  | Canal Norte | Seriales  |
  | Canal Norte | Novelas   |
  | Cine Canal  | Películas |
  | Canal Norte | Películas |
  
    - Tabla resultante luego de realizar las operaciones de Unión Natural de las Relaciones (es diferente a la la Relación original, tiene una fila extra):

  | Productora | Canal       | Producto  |
  |------------|-------------|-----------|
  | Inter TV   | Canal Norte | Seriales  |
  | Inter TV   | Canal Norte | Novelas   |
  | Inter TV   | Canal Norte | Películas |
  | Inter TV   | Cine Canal  | Películas |
  | Film Stars | Canal Norte | Películas |

