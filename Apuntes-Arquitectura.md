<h1>Apuntes de Arquitectura</h1>

***Index***:
<!-- TOC -->
  * [Principios SOLID](#principios-solid)
    * [*Single Responsability*](#single-responsability)
    * [*Open/Closed*](#openclosed)
    * [*Liskov Substitution*](#liskov-substitution)
    * [*Interface Segregation*](#interface-segregation)
    * [*Dependency Inversion*](#dependency-inversion)
  * [*Clean Architecture* vs Guía de Arquitectura de Android vs MVVM](#clean-architecture-vs-guía-de-arquitectura-de-android-vs-mvvm)
    * [Resumen General](#resumen-general)
    * [Desglose de las funciones de cada capa](#desglose-de-las-funciones-de-cada-capa)
      * [MVVM](#mvvm)
      * [Guía de Arquitectura (Google/Android)](#guía-de-arquitectura-googleandroid)
      * [*Clean Architecture*](#clean-architecture)
    * [Diferencia entre Modelos de Datos y Modelos de Dominio](#diferencia-entre-modelos-de-datos-y-modelos-de-dominio)
      * [Modelos de Datos (DTOs - Data Transfer Objects)](#modelos-de-datos-dtos---data-transfer-objects)
      * [Modelos de Dominio](#modelos-de-dominio)
    * [Ejemplo estructura](#ejemplo-estructura)
  * [Patrones de Diseño](#patrones-de-diseño)
<!-- TOC -->

---

## Principios SOLID

### *Single Responsability*
TODO...

### *Open/Closed*
TODO...

### *Liskov Substitution*
TODO...

### *Interface Segregation*
TODO...

### *Dependency Inversion*
TODO...

---

## *Clean Architecture* vs Guía de Arquitectura de Android vs MVVM

### Resumen General
Cada capa en estas arquitecturas tiene un propósito específico y ayuda a mantener un diseño limpio y organizado que facilita la mantenibilidad y escalabilidad de la aplicación.

- **MVVM** se enfoca en la separación de la lógica de presentación y la UI a través de un *ViewModel*.  
- **La arquitectura propuesta por Google** introduce una gestión clara de la UI y la lógica de datos, con una opción para incluir una capa de dominio.  
- ***Clean Architecture*** promueve una estructura altamente desacoplada, donde **_las dependencias fluyen desde las capas exteriores hacia las interiores_**, permitiendo un alto grado de flexibilidad y reutilización. Para evitar que se “crucen los límites” entre las capas, se utiliza el **_Principio de Inversión de Dependencia_** (ver [Dependency Inversion](#dependency-inversion)).

### Desglose de las funciones de cada capa

#### MVVM

| **Capa**      | **Descripción**                                                                                                                                                                   |
|---------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Model**     | Se encarga de la **_lógica de negocio y de la gestión de los datos_**. Puede incluir acceso a bases de datos, servicios web y otros recursos de datos.                            |
| **View**      | Representa la UI. Escucha los cambios en el *ViewModel* y se actualiza en consecuencia. Normalmente consiste en *activities*, *fragments* y *Views*.                              |
| **ViewModel** | Actúa como intermediario entre las capas de Model y de View. Contiene datos que la vista necesita y maneja la lógica de presentación. También gestiona el ciclo de vida de la UI. |

#### Guía de Arquitectura (Google/Android)

| **Capa**               | **Descripción**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |
|------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **UI**                 | La función de la capa de UI (o capa de presentación) es mostrar los datos de la aplicación en la pantalla. **_Se compone de dos cosas: elementos de la UI_** que representan los datos en la pantalla (hechos con *Views* o con Compose) **_y state holders (como los ViewModel)_** que contienen datos, los exponen a la UI y manejan la lógica de presentación.                                                                                                                                                                                                                                      |
| **Data**               | **_Contiene la lógica de negocio_**, la cual está compuesta por reglas que determinan cómo la aplicación crea, almacena y cambia datos. Está formada por **_Repositorios_**, que pueden contener desde cero hasta muchas Fuentes de Datos (***Data Sources***). Se debería crear una clase de Repositorio para cada tipo diferente de dato que se maneja en la aplicación y a su vez, cada clase de Fuente de Datos debe tener la responsabilidad de trabajar con una sola fuente de datos, que puede ser un archivo, una fuente de red (solicitudes a una API en internet) o una Base de Datos local. |
| **Dominio (opcional)** | Se encarga de encapsular la lógica de negocio compleja, o la lógica de negocio simple que reutilizan varios *ViewModels*. Esta capa es opcional, ya que no todas las aplicaciones cumplen estos requisitos. Se encuentra entre la capa de UI y la capa de Data. Las clases de esta capa se denominan comúnmente **_Casos de Uso o interactors_**. Cada Caso de Uso debe ser **_responsable de una única funcionalidad_**.                                                                                                                                                                              |

#### *Clean Architecture*

| **Capa**                                   | **Descripción**                                                                                                                                                                                                                                                                                                                                                              |
|--------------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Enterprise Business Rules / Entities**   | Contiene las **_Reglas de Negocio de alto nivel (lo más abstracto), encapsulando la lógica que es específica del Dominio del Negocio_**. Es independiente de la tecnología y las librerías externas. Las Entidades podrían ser utilizadas por muchas aplicaciones diferentes en la empresa y ningún cambio operativo en ninguna aplicación en particular debería afectarlas. |
| **Application Business Rules / Use Cases** | Define la **_lógica específica de la aplicaciónm encapsulando e implementando todos los Casos de Uso del sistema_**. Se encarga de coordinar el flujo de datos entre las capas superiores e inferiores.                                                                                                                                                                      |
| **Interface Adapters**                     | Esta capa **_convierte datos del formato más conveniente para los Casos de Uso y Entidades, al formato más conveniente para los componentes de la capa más externa (y viceversa)_**. Aquí se encuentran implementaciones de repositorios, APIs y controladores de UI.                                                                                                        |
| **Frameworks & Drivers / Infrastructure**  | Contiene **_elementos externos como Bases de Datos, frameworks de UI, servicios web, etc_**. Esta capa puede incluir las tecnologías que se utilizan para construir la aplicación. Su objetivo es ser **_reemplazable o intercambiable_**.                                                                                                                                   |

<br>

### Diferencia entre Modelos de Datos y Modelos de Dominio

#### Modelos de Datos (DTOs - Data Transfer Objects)

- **Propósito**: **_Representar la estructura de datos tal como viene de fuentes externas (API, base de datos)_**
- **Características**:
    - Suelen ser clases simples con propiedades y sin lógica de negocio
    - Sus nombres y estructura reflejan exactamente lo que devuelve la API o lo que requiere la base de datos
    - Pueden contener anotaciones específicas para serialización/deserialización (como `@SerializedName` en Retrofit)
    - Ejemplo: `MovieDto` con campos exactamente como los devuelve la API

#### Modelos de Dominio

- **Propósito**: **_Representar Entidades de Negocio independientes de la infraestructura externa_**
- **Características**:
    - Contienen la lógica de negocio relevante para la entidad
    - Su estructura está diseñada para las necesidades de la aplicación, no para adaptarse a fuentes externas
    - Son independientes de frameworks y librerías externas
    - No tienen anotaciones específicas de infraestructura
    - Ejemplo: `Movie` con solo los campos relevantes para la lógica de la aplicación

En la práctica, los repositorios suelen transformar los modelos de datos (DTOs) en modelos de dominio al obtener información, y viceversa al guardarla, manteniendo así la capa de dominio aislada de los detalles de implementación de las fuentes de datos.

### Ejemplo estructura
Estructura de paquetes para una supuesta app destinada a mostrar una lista de películas, ver el detalle de una película, agregar una película a favoritos, etc.

```
com.example.movieapp/
├── data/
│   ├── remote/
│   │   ├── api/
│   │   └── models/
│   ├── local/
│   │   ├── dao/
│   │   └── entities/
│   └── repositories/
├── domain/
│   ├── models/
│   ├── repositories/
│   └── usecases/
└── presentation/
    ├── common/
    │   ├── components/
    │   └── theme/
    ├── movielist/
    ├── moviedetail/
    └── favorites/
```

---

## Patrones de Diseño
TODO...
