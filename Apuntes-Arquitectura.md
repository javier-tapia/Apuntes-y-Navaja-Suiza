<h1>Apuntes de Arquitectura</h1>

***Index***:
<!-- TOC -->
  * [Principios SOLID](#principios-solid)
    * [Qué son](#qué-son)
    * [*Single Responsibility*](#single-responsibility)
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
### Qué son
> :warning: Es importante destacar que **_estos principios están interrelacionados y se consideran como un todo_**. Y también **_es posible que en el intento de cumplir uno, se incumpla otro_**.

Los Principios de Diseño de _Software_ en general, son **_guías o pautas para escribir código_** que han demostrado ser eficaces a lo largo del tiempo.  
Su principal característica es que ayudan a escribir un **código limpio** (**_clean code_**). Es decir, permiten lo siguiente:

- Hacer un código entendible, no solo para el autor.
- Mantener y desarrollar una pieza de _software_ rápidamente a largo plazo.
- Crear tests legibles, rápidos, independientes y repetibles.
- Lograr una **alta cohesión** y un **bajo acoplamiento**.

Los SOLID no son los únicos. Ejemplos de otros Principios de Diseño pueden ser **DRY** (**_Don't Repeat Yourself_**), **KISS** (**_Keep It Simple, Stupid_**) o **YAGNI** (**_You Aren't Gonna Need It_**).

### *Single Responsibility*
Una clase debe tener **_una sola razón para cambiar_**. Es decir, debe ocuparse de una **_única parte del comportamiento del sistema_**.  
**Ejemplo**: La clase ``User``, que representa a un usuario, no tiene por qué manejar una base de datos. Esa responsabilidad se debería sacar a otro componente.

```kotlin
// ❌ DON'T
class User {
    private val db = Room.databaseBuilder(
        applicationContext,
        AppDatabase::class.java, "database-name"
    ).build()

    var firstName: String = "Javi"
        set(value) {
            if (value.count() <= 3) println("'$value' no llega ni a 3 letras") else field = value
        }

    fun getUser(): User = db.userDao().getAll()
}

// ✅ DO
class User {
    var firstName: String = "Javi"
        set(value) {
            if (value.count() <= 3) println("'$value' no llega ni a 3 letras") else field = value
        }
}
```

### *Open/Closed*
Una clase debe estar **_abierta a la extensión_** (**_de su comportamiento_**) pero **_cerrada a la modificación_** (**_de su código_**).  
**Ejemplo**: Si la función ``greeting()`` debe modificarse cada vez que se agrega un país, se viola este principio.

```kotlin
// ❌ DON'T
enum class Country {
    ARGENTINA, COLOMBIA, MEXICO
}

class User(val birthPlace: Country) {
    fun greeting() {
        when (birthPlace) {
            Country.ARGENTINA -> println("Qué pasa che")
            Country.COLOMBIA -> println("Qué más parce")
            Country.MEXICO -> println("Qué onda wey")
        }
    }
}

fun main() {
    val userArg = User(Country.ARGENTINA)
    userArg.greeting() // Qué pasa che

    val userCol = User(Country.COLOMBIA)
    userCol.greeting() // Qué más parce
}

// ✅ DO
interface User {
    fun greeting()
}

class UserArgentina : User {
    override fun greeting() = println("Qué pasa che")
}

class UserColombia : User {
    override fun greeting() = println("Qué más parce")
}

class UserMexico : User {
    override fun greeting() = println("Qué onda wey")
}

fun main() {
    val userArg = UserArgentina()
    userArg.greeting() // Qué pasa che

    val userCol = UserColombia()
    userCol.greeting() // Qué más parce
}
```

### *Liskov Substitution*
Si una clase es extendida (heredada), **_se debe poder usar cualquiera de sus clases hijas_** en su lugar **_sin alterar el comportamiento esperado del programa_**.  
**Ejemplo**: Si al usar una clase hija se rompe el comportamiento del programa, no se cumple con este principio. En este ejemplo, ``RegularUser`` no debería redefinir ``retrieveId()`` si no puede garantizar el mismo contrato que su clase base.

```kotlin
// ❌ DON'T
open class User {
    private var firstName: String = "Nombre: Javi"
    private var idClient: Int = 12345
    open fun retrieveName() = println(firstName)

    open fun retrieveId() = println(idClient)
}

class RegularUser : User() {
    override fun retrieveId() = throw Exception("Un usuario regular no puede acceder al ID")
}

fun main() {
    val user = User()
    val regular = RegularUser()

    user.retrieveName() // Nombre: Javi
    regular.retrieveName() // Nombre: Javi
    user.retrieveId() // 12345
    regular.retrieveId() // Exception in thread "main" java.lang.Exception: Un usuario regular no puede acceder al ID
}

// ✅ DO
open class User {
    private var firstName: String = "Nombre: Javi"
    open fun retrieveName() = println(firstName)
}

class RegularUser : User()

open class Admin : User() {
    private var idClient: Int = 12345
    open fun retrieveId() = println(idClient)
}

class Owner : Admin()

fun main() {
    val user = User()
    val regular = RegularUser()
    val admin = Admin()
    val owner = Owner()

    user.retrieveName() // Nombre: Javi
    regular.retrieveName() // Nombre: Javi
    admin.retrieveName() // Nombre: Javi
    owner.retrieveName() // Nombre: Javi

    // user.retrieveId() -> No tiene acceso a este método
    // regular.retrieveId() -> No tiene acceso a este método
    admin.retrieveId() // 12345
    owner.retrieveId() // 12345
}
```

### *Interface Segregation*
Una clase **_nunca debería depender de métodos que no usa_**. Es mejor tener más interfaces pequeñas que pocas interfaces grandes. Esto ayuda a **_reducir el acoplamiento_** innecesario y **_facilitar el mantenimiento_**.  
**Ejemplo**: Crear una nueva interfaz para evitar el comportamiento por defecto de una función que no se necesita. Luego, que cada clase implemente la interfaz que necesite.

```kotlin
// ❌ DON'T
interface User {
    val name: String
    val address: String
    val isFullAge: Boolean
}

class RegularUser : User {
    override val name: String
        get() = "Beto A Saber"
    override val address: String
        get() = "Calle Falsa 123"
    override val isFullAge: Boolean
        get() = true
}

class Admin : User {
    override val name: String
        get() = "Javi"
    override val address: String
        get() = "Av Siempre Viva"
    override val isFullAge: Boolean
        get() = throw UnsupportedOperationException()
}

// ✅ DO
interface User {
    val name: String
    val address: String
}

interface ValidateAge {
    val isFullAge: Boolean
}

class RegularUser : User, ValidateAge {
    override val name: String
        get() = "Beto A Saber"
    override val address: String
        get() = "Calle Falsa 123"
    override val isFullAge: Boolean
        get() = true
}

class Admin : User {
    override val name: String
        get() = "Javi"
    override val address: String
        get() = "Dr Moreno"
}
```

### *Dependency Inversion*
Una clase debe **_depender de abstracciones y no de implementaciones concretas_**. Este principio promueve un diseño desacoplado y facilita la prueba unitaria (_mocking_/_stubbing_).  

> ℹ️ La **_inyección de dependencias_** es una técnica que permite implementar este principio

**Ejemplo**: En lugar de depender de implementaciones, se deberían crear abstracciones (interfaces) que definan el contrato, y luego inyectar las implementaciones concretas según la necesidad.

```kotlin
// ❌ DON'T
class Greeting {
    fun greet() = print("Hola. ")
}

class Claim {
    fun claim() = println("Estoy haciendo un reclamo.")
}

class Whatever(
    private val greeting: Greeting,
    private val claim: Claim
) {
    fun communicate() {
        greeting.greet()
        claim.claim()
    }
}

fun main() {
    val greeting = Greeting()
    val claim = Claim()
    val whatever = Whatever(greeting, claim)

    whatever.communicate() // Hola. Estoy haciendo un reclamo.
}

// ✅ DO
interface MessageType {
    fun giveMessage()
}

interface ReasonType {
    fun giveReason()
}

class Greeting : MessageType {
    override fun giveMessage() {
        print("Hola. ")
    }
}

class SayBye : MessageType {
    override fun giveMessage() {
        print("Hasta nunca. ")
    }
}

class Claim : ReasonType {
    override fun giveReason() {
        println("Estoy haciendo un reclamo.")
    }
}

class Threat : ReasonType {
    override fun giveReason() {
        println("¡Volveré con mi abogado!")
    }
}

class Whatever(
    private val messageType: MessageType,
    private val reasonType: ReasonType
) {
    fun communicate() {
        messageType.giveMessage()
        reasonType.giveReason()
    }
}

fun main() {
    val messageTypeGreeting: MessageType = Greeting()
    val reasonTypeClaim: ReasonType = Claim()
    var whatever = Whatever(messageTypeGreeting, reasonTypeClaim)
    whatever.communicate() // Hola. Estoy haciendo un reclamo.

    val messageTypeBye: MessageType = SayBye()
    val reasonTypeThreat = Threat()
    whatever = Whatever(messageTypeBye, reasonTypeThreat)
    whatever.communicate() // Hasta nunca. ¡Volveré con mi abogado!
}
```

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
