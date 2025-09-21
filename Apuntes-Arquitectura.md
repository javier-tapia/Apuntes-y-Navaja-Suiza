# Apuntes de Arquitectura

- [Principios SOLID](#principios-solid)
   - [_Single Responsability_](#single-responsability)
   - [_Open/Closed_](#openclosed)
   - [_Liskov Substitution_](#liskov-substitution)
   - [_Interface Segregation_](#interface-segregation)
   - [_Dependency Inversion_](#dependency-inversion)
- [*Clean Architecture* vs Guía de Arquitectura de Android vs MVVM](#clean-architecture-vs-guía-de-arquitectura-de-android-vs-mvvm)
   - [Resumen General](#resumen-general)
   - [Desglose de las funciones de cada capa](#desglose-de-las-funciones-de-cada-capa)
     - [MVVM](#mvvm)
     - [Guía de Arquitectura (Google/Android)](#guía-de-arquitectura-googleandroid)
     - [*Clean Architecture*](#clean-architecture)
   - [Diferencia entre Modelos de Datos y Modelos de Dominio](#diferencia-entre-modelos-de-datos-y-modelos-de-dominio)
     - [Modelos de Datos (DTOs - Data Transfer Objects)](#modelos-de-datos-dtos---data-transfer-objects)
     - [Modelos de Dominio](#modelos-de-dominio)
- [Ejemplo: Aplicación de películas](#ejemplo-aplicación-de-películas)
   - [1. Arquitectura General: Clean Architecture + MVVM](#1-arquitectura-general-clean-architecture--mvvm)
     - [Capas principales](#capas-principales)
   - [2. Componentes Clave](#2-componentes-clave)
     - [Capa de Datos](#capa-de-datos)
     - [Capa de Dominio](#capa-de-dominio)
     - [Capa de Presentación](#capa-de-presentación)
   - [3. Flujo de Datos](#3-flujo-de-datos)
   - [4. Tecnologías Recomendadas](#4-tecnologías-recomendadas)
   - [5. Estructura de Paquetes sugerida](#5-estructura-de-paquetes-sugerida)

---
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

## Ejemplo: Aplicación de películas

### 1. Arquitectura General: Clean Architecture + MVVM

La aplicación seguirá una arquitectura por capas siguiendo los principios de Clean Architecture combinada con el patrón MVVM (Model-View-ViewModel) recomendado por Google:

#### Capas principales

1. **Capa de Presentación (UI)**
  - Componentes *Compose*
  - *ViewModels*
  - Estados de UI
2. **Capa de Dominio**
  - Casos de uso (*Use Cases*)
  - Modelos de dominio
  - Interfaces de repositorios
3. **Capa de Datos**
  - Implementaciones de repositorios
  - Fuentes de datos (remota y local)
  - Modelos de datos (DTOs)

### 2. Componentes Clave

#### Capa de Datos

**Fuente de Datos Remota:**

- `MovieApiService`: Interfaz para definir los *endpoints* de la API de películas
- `MovieRemoteDataSource`: Clase que utiliza el servicio API para obtener datos

**Fuente de Datos Local:**

- `MovieDatabase`: Base de datos *Room* para almacenar películas favoritas
- `MovieDao`: Interfaz para acceder a la base de datos
- `MovieLocalDataSource`: Clase que utiliza el DAO para operaciones locales

**Repositorios:**

- `MovieRepository`: Implementación que coordina fuentes de datos remotas y locales

#### Capa de Dominio

**Modelos:**

- `Movie`: Modelo de dominio que representa una película

**Casos de Uso:**

- `GetMoviesUseCase`: Obtener lista de películas
- `GetMovieDetailsUseCase`: Obtener detalles de una película
- `ToggleFavoriteUseCase`: Marcar/desmarcar película como favorita
- `GetFavoriteMoviesUseCase`: Obtener películas favoritas

#### Capa de Presentación

**ViewModels:**

- `MovieListViewModel`: Gestiona la lista de películas
- `MovieDetailViewModel`: Gestiona los detalles de una película
- `FavoritesViewModel`: Gestiona la lista de favoritos

**Estados de UI:**

- `MovieListUiState`: Estado para la pantalla de lista de películas
- `MovieDetailUiState`: Estado para la pantalla de detalles
- `FavoritesUiState`: Estado para la pantalla de favoritos

**Pantallas Compose:**

- `MovieListScreen`: Muestra la lista de películas
- `MovieDetailScreen`: Muestra los detalles de una película
- `FavoritesScreen`: Muestra las películas favoritas

### 3. Flujo de Datos

1. **Obtener lista de películas:**
  - UI solicita datos → *ViewModel* → *UseCase* → *Repository* → API
  - Los datos fluyen de vuelta como: API → *Repository* → *UseCase* → *ViewModel* → UI (como estado)
2. **Marcar como favorito:**
  - UI envía acción → *ViewModel* → *UseCase* → *Repository* → Base de datos local
  - Confirmación: Base de datos → *Repository* → *UseCase* → *ViewModel* → UI (actualización de estado)
3. **Ver detalles:**
  - UI solicita detalles → *ViewModel* → *UseCase* → *Repository* → API/Local
  - Datos fluyen de vuelta como: Fuente de datos → *Repository* → *UseCase* → *ViewModel* → UI

### 4. Tecnologías Recomendadas

- **Jetpack Compose**: Para toda la UI
- ***ViewModel* + *StateFlow***: Para gestionar y exponer estados de UI
- **Kotlin Coroutines + Flow**: Para operaciones asíncronas y flujos de datos reactivos
- ***Retrofit***: Para comunicación con la API
- ***Room***: Para persistencia local de favoritos
- ***Hilt/Dagger***: Para inyección de dependencias
- ***Navigation Compose***: Para la navegación entre pantallas

### 5. Estructura de Paquetes sugerida

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
