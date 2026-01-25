<h1>Apuntes de Arquitectura</h1>

***Index***:
<!-- TOC -->
  * [Principios SOLID](#principios-solid)
    * [Qué son los Principios de Diseño](#qué-son-los-principios-de-diseño)
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
      * [Modelos de Datos](#modelos-de-datos)
      * [Modelos de Dominio](#modelos-de-dominio)
      * [Transformación entre capas](#transformación-entre-capas)
    * [Ejemplo estructura](#ejemplo-estructura)
  * [Técnicas para analizar código "desconocido"](#técnicas-para-analizar-código-desconocido)
    * [Estrategias generales](#estrategias-generales)
  * [Patrones de Diseño](#patrones-de-diseño)
    * [Qué es un Patrón de Diseño](#qué-es-un-patrón-de-diseño)
    * [Clasificación de los Patrones](#clasificación-de-los-patrones)
    * [*Singleton*](#singleton)
    * [*Builder*](#builder)
    * [*Abstract Factory*](#abstract-factory)
    * [*Factory Method*](#factory-method)
    * [*Adapter*](#adapter)
    * [*Facade*](#facade)
    * [*Decorator*](#decorator)
    * [*Strategy*](#strategy)
    * [*Observer*](#observer)
    * [*State*](#state)
    * [Referencias](#referencias)
<!-- TOC -->

---

## Principios SOLID
### Qué son los Principios de Diseño
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
#### Modelos de Datos
- **Propósito**: **_Representar la estructura de datos tal como viene de fuentes externas (APIs) o como se almacena internamente (Base de Datos)_** :arrow_right: **_Son modelos dependientes de la infraestructura, porque reflejan la estructura de la API o de la base de datos_**
- **Características**:
    - Son clases simples **_orientadas a transporte/almacenamiento, sin lógica de negocio_**
    - Sus nombres y estructura reflejan exactamente lo que devuelve la API o lo que necesita la base de datos
    - Suelen tener anotaciones externas (Retrofit/Moshi/Gson/Room)
    - Pueden variar cuando cambia la API o el esquema de la BBDD
    - 📌 **Ejemplos**: `MovieDto`, que representa la respuesta JSON del servidor; `MovieEntity`, que representa el formato en el que la base de datos (Room/SQLDelight/ObjectBox) necesita almacenar los datos

#### Modelos de Dominio
- **Propósito**: **_Representar Entidades de Negocio independientes de la infraestructura_** :arrow_right: **_Son el “corazón” de la aplicación, independientes de frameworks y librerías externas_**
- **Características**:
    - Pueden contener reglas de negocio, aunque en muchos proyectos se mantienen como **_modelos simples sin lógica compleja_**
    - Su estructura responde a las **necesidades de la aplicación**, no a las limitaciones externas
    - No tienen anotaciones de librerías externas
    - 📌 **Ejemplo**: `Movie`, que solo tiene los campos relevantes para la lógica de la aplicación

#### Transformación entre capas
En la práctica, los Repositorios se apoyan en **_mappers_** para transformar los Modelos de Datos en Modelos de Dominio al obtener información, y viceversa al guardarla, manteniendo así la **_capa de Dominio aislada de los detalles de implementación de las fuentes de datos (capa de Data)_**.

- API → ``MovieDto`` → **_mapper_** → ``Movie`` 
- DB → ``MovieEntity`` → **_mapper_** → ``Movie`` 
- Dominio → **_mapper_** → ``MovieEntity`` (si se guarda localmente)

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
│   └── repositories/ (implementaciones concretas)
├── domain/
│   ├── models/
│   ├── repositories/ (interfaces)
│   └── usecases/
└── presentation/
    ├── common/
    │   ├── components/
    │   └── theme/
    ├── movielist/
    ├── moviedetail/
    └── favorites/
```

## Técnicas para analizar código "desconocido"
El objetivo es construir un **modelo mental del flujo**. Es decir, un **mapa simplificado** en el que no se necesita entender cada línea, sino las **conexiones principales entre componentes**.

```
Entrada → Transformación → Salida
```

### Estrategias generales
- ***Top-down*** :arrow_right: Empezar desde el punto de entrada (*Activity*/*Fragment*) y seguir las llamadas.
- ***Bottom-up*** :arrow_right: Empezar desde donde se necesita entender y rastrear hacia atrás quién lo llama.
- **Buscar patrones conocidos** :arrow_right: MVP, MVVM, MVI, etc. Una vez identificado, se puede tener una idea de dónde buscar cada cosa.

---

## Patrones de Diseño
### Qué es un Patrón de Diseño
Es una **solución reutilizable y comprobada** que muestra **cómo estructurar el código** para solucionar **problemas de diseño comunes y habituales**.  
A menudo los patrones se confunden con algoritmos porque ambos conceptos describen soluciones típicas a problemas conocidos. Mientras que un algoritmo siempre define un grupo claro de acciones para lograr un objetivo, un patrón es una descripción de más alto nivel de una solución. El código del mismo patrón aplicado a dos programas distintos puede ser diferente.  
Una analogía de **un algoritmo sería una receta de cocina**: ambos cuentan con pasos claros para alcanzar una meta. Por su parte, **un patrón es más similar a un plano**, ya que se puede observar cómo son su resultado y sus funciones, pero el orden exacto de la implementación depende del desarrollador.

### Clasificación de los Patrones
> :warning: Los ejemplos de cada patrón presentados más abajo representan implementaciones genéricas extrapolables a cualquier lenguaje orientado a objetos, no necesariamente son una versión idiomática en Kotlin, que para algunos casos podría resolverlo más fácilmente debido a las facilidades del lenguaje (ver [Design Patterns with Kotlin](Code%20Snippets%20with%20Kotlin/Design%20Patterns.md))

Los Patrones de Diseño se suelen clasificar en tres grandes grupos:

- **Patrones Creacionales** :arrow_right: Proporcionan mecanismos de creación de objetos que incrementan la flexibilidad y la reutilización de código existente. Agrupa a: _Factory Method_, _Abstract Factory_, _Builder_, _Singleton_ y _Prototype_.
- **Patrones Estructurales** :arrow_right: Explican cómo ensamblar objetos y clases en estructuras más grandes a la vez que se mantiene la flexibilidad y eficiencia de la estructura. Agrupa a: _Adapter_, _Bridge_, _Composite_, _Decorator_, _Facade_, _Flyweight_ y _Proxy_.
- **Patrones de Comportamiento** :arrow_right: Se encargan de una comunicación efectiva y la asignación de responsabilidades entre objetos. Agrupa a: _Chain of Responsibility_, _Command_, _Iterator_, _Mediator_, _Memento_, _Observer_, _State_, _Strategy_, _Template Method_ y _Visitor_.

### *Singleton*
Patrón Creacional que permite asegurarse de que **una clase tenga una única instancia**, a la vez que **proporciona un punto de acceso global a dicha instancia y evita que otro código la sobreescriba**.

El motivo más habitual es **controlar el acceso a algún recurso compartido**; por ejemplo, una base de datos o un archivo.

<br>
<p align="center">
    <img src="images/singleton.png" width="1000" alt="">
</p>
<br>

📌 **Ejemplo**:  
> 🧠 **Idea clave**: el constructor es privado y el acceso se hace a través de un método estático.

```kotlin
// ================================
// Clase Singleton
// ================================
class Logger private constructor() {

    fun log(message: String) {
        println("Log: $message")
    }

    companion object {
        private var instance: Logger? = null

        fun getInstance(): Logger {
            if (instance == null) {
                // Nota: si la app soporta multihilo, se debería sincronizar este bloque
                instance = Logger()
            }
            return instance!!
        }
    }
}

// ================================
// Cliente
// ================================
fun main() {
    val logger1 = Logger.getInstance()
    val logger2 = Logger.getInstance()

    println(logger1 == logger2) // true
    logger1.log("Aplicación iniciada.") // Log: Aplicación iniciada.
}
```

### *Builder*
Patrón Creacional que permite **construir objetos complejos paso a paso** y producir distintos tipos y representaciones de un objeto empleando el mismo código de construcción. A su vez, **no permite a otros objetos acceder al producto mientras se construye**. Para eso, se extrae el código de construcción del objeto de su propia clase a **objetos independientes llamados _builders_**. Cada uno de esos _builders_ representa un "paso" de la construcción del objeto. Y lo importante, es que no es necesario llamarlos a todos: se pueden invocar solo aquellos que sean necesarios para producir una configuración particular del objeto.

Opcionalmente, también se puede utilizar una clase **director**, la cual puede **definir el orden en el que se deben ejecutar los pasos** para la construcción de objetos "habituales" o más usados, mientras que el _builder_ proporciona la implementación de dichos pasos.

Este patrón es util para **evitar que los constructores de las clases tengan una enorme cantidad de parámetros** (incluso cuando no se necesitan todos en todo momento) o la necesidad de crear **múltiples subclases** que cubran todas las combinaciones posibles de los parámetros.

<br>
<p align="center">
    <img src="images/builder.png" width="1000" alt="">
</p>
<br>

📌 **Ejemplo**:  
> 🧠 **Idea clave**: el ``Director`` orquesta el proceso; el ``Builder`` encapsula los pasos concretos.

```kotlin
// ================================
// Producto final
// ================================
class Product {
    private val parts = mutableListOf<String>()

    fun addPart(part: String) = parts.add(part)

    fun show() {
        println("Partes del producto: ${parts.joinToString()}")
    }
}

// ================================
// Interfaz Builder
// ================================
interface Builder {
    fun reset()
    fun buildPartA()
    fun buildPartB()
    fun buildPartZ()
    fun getResult(): Product
}

// ================================
// Builder concreto
// ================================
class ConcreteBuilder : Builder {
    private var product = Product()

    override fun reset() {
        product = Product()
    }

    override fun buildPartA() {
        product.addPart("Parte A")
    }

    override fun buildPartB() {
        product.addPart("Parte B")
    }

    override fun buildPartZ() {
        product.addPart("Parte Z")
    }

    override fun getResult(): Product = product
}

// ================================
// Director opcional: define el orden de construcción
// ================================
class Director(private var builder: Builder) {

    fun changeBuilder(builder: Builder) {
        this.builder = builder
    }

    fun make(type: String) {
        builder.reset()
        if (type == "simple") {
            builder.buildPartB()
        } else {
            builder.buildPartA()
            builder.buildPartB()
            builder.buildPartZ()
        }
    }
}

// ================================
// Cliente
// ================================
fun main() {
    val builder = ConcreteBuilder()
    val director = Director(builder)

    director.make("complejo")
    val product = builder.getResult()
    product.show() // Partes del producto: Parte A, Parte B, Parte Z
}
```

### *Abstract Factory*
Patrón Creacional que permite producir **familias de objetos relacionados (y sus variantes)** sin especificar sus clases concretas, ya sea porque no se conozcan de antemano o sencillamente porque se quiere permitir una futura extensibilidad.

El patrón sugiere declarar de forma explícita **interfaces para cada producto diferente de la familia de productos** y hacer que todas **las variantes de los productos sigan esas interfaces**.  
Luego, declarar la *Abstract Factory*: una **interfaz con una lista de métodos de creación para todos los productos que son parte de la familia de productos**, los cuales deben devolver productos abstractos representados por las interfaces que extraídas previamente.  
A su vez, para cada variante de una familia de productos, se crea una **clase de fábrica independiente** basada en la interfaz *Abstract Factory*, la cual devuelve productos de un tipo particular (variantes específicas de los productos).

En resumen, se crean dos niveles de abstracción: uno para los objetos creados y otro para las fábricas.

<br>
<p align="center">
    <img src="images/abstract-factory.png" width="1000" alt="">
</p>
<br>

📌 **Ejemplo**:  
> 🧠 **Idea clave**: se crean familias completas de productos relacionados (botones y _checkboxes_) sin conocer sus clases concretas.

```kotlin
// ================================
// Interfaces abstractas para los productos
// ================================
interface Button {
    fun paint()
}

interface Checkbox {
    fun paint()
}

// ================================
// Implementaciones concretas
// ================================
class WinButton : Button {
    override fun paint() = println("Renderizando botón estilo Windows")
}

class MacButton : Button {
    override fun paint() = println("Renderizando botón estilo macOS")
}

class WinCheckbox : Checkbox {
    override fun paint() = println("Renderizando checkbox estilo Windows")
}

class MacCheckbox : Checkbox {
    override fun paint() = println("Renderizando checkbox estilo macOS")
}

// ================================
// Fábrica abstracta
// ================================
interface GUIFactory {
    fun createButton(): Button
    fun createCheckbox(): Checkbox
}

// ================================
// Fábricas concretas
// ================================
class WinFactory : GUIFactory {
    override fun createButton(): Button = WinButton()
    override fun createCheckbox(): Checkbox = WinCheckbox()
}

class MacFactory : GUIFactory {
    override fun createButton(): Button = MacButton()
    override fun createCheckbox(): Checkbox = MacCheckbox()
}

// ================================
// Cliente
// ================================
class Application(private val factory: GUIFactory) {

    private val button = factory.createButton()
    private val checkbox = factory.createCheckbox()

    fun render() {
        button.paint() // Renderizando botón estilo macOS
        checkbox.paint() // Renderizando checkbox estilo macOS
    }
}

fun main() {
    val factory: GUIFactory = MacFactory()
    val app = Application(factory)
    app.render()
}
```

### *Factory Method*
Patrón Creacional que delega la responsabilidad de crear el objeto a un único método. Normalmente, esto termina siendo una clase ``Factory`` (o `Creator`) cuya única función es crear objetos del tipo de la interfaz del producto. Las subclases que heredan de dicha clase ``Factory``, van a sobreescribir su método de creación, los cuales retornarán los productos concretos (las diferentes implementaciones de la interfaz del producto).

Es util para **evitar duplicar la lógica** de creación de objetos, **ocultar la complejidad** de la creación de objetos y crear **funcionalidades más genéricas**.

<br>
<p align="center">
    <img src="images/factory-method.png" width="1000" alt="">
</p>
<br>

📌 **Ejemplo**:  
> 🧠 **Idea clave**: El *Factory Method* define un punto de extensión en una clase base para delegar la creación de objetos a sus 
> subclases. Cada subclase decide qué producto concreto instanciar, pero el código cliente solo conoce la interfaz común.

```kotlin
// ================================
// Interfaz del producto
// ================================
interface Product {
    fun doStuff()
}

// ================================
// Productos concretos
// ================================
class ConcreteProductA : Product {
    override fun doStuff() = println("Haciendo algo con el Producto A")
}

class ConcreteProductB : Product {
    override fun doStuff() = println("Haciendo algo con el Producto B")
}

// ================================
// Clase base (creador)
// ================================
abstract class Factory {
    fun someOperation() {
        val product = createProduct()
        product.doStuff()
    }

    // Factory Method a implementar por las subclases
    protected abstract fun createProduct(): Product
}

// ================================
// Fábricas concretas
// ================================
class ConcreteFactoryA : Factory() {
    override fun createProduct(): Product = ConcreteProductA()
}

class ConcreteFactoryB : Factory() {
    override fun createProduct(): Product = ConcreteProductB()
}

// ================================
// Cliente
// ================================
fun main() {
    val factoryA: Factory = ConcreteFactoryA()
    factoryA.someOperation()   // Haciendo algo con el Producto A

    val factoryB: Factory = ConcreteFactoryB()
    factoryB.someOperation()   // Haciendo algo con el Producto B
}
```

### *Adapter*
Patrón Estructural que permite la colaboración entre objetos con **interfaces incompatibles**. Un adaptador o _adapter_ es un objeto especial que **convierte la interfaz de un objeto, de forma que otro objeto pueda comprenderla**.  
Es decir: un Objeto A (**_Target_**) puede invocar con seguridad los métodos del Adaptador, ya que tiene su misma interfaz; y a su vez, al recibir una llamada, el Adaptador pasa la solicitud a un Objeto B (**_Adaptee_**), pero en un formato y orden que ese Objeto B espera. En ocasiones se puede incluso crear un adaptador de dos direcciones que pueda convertir las llamadas en ambos sentidos.

<br>
<p align="center">
    <img src="images/adapter.png" width="1000" alt="">
</p>
<br>

📌 **Ejemplo**:  
> 🧠 **Idea clave**: El _Adapter_ implementa la interfaz que el cliente espera (**_Target_**) y traduce las llamadas hacia una clase existente con interfaz incompatible (**_Adaptee_**), convirtiendo datos y formato.

```kotlin
// =======================
// Target (lo que el cliente conoce)
// =======================
interface ClientInterface {
    fun method(data: String): String
}

// =======================
// Adaptee (servicio existente / incompatible)
// =======================
class Service {
    fun serviceMethod(specialData: String): String {
        return "Service procesó: $specialData"
    }
}

// =======================
// Adapter
// =======================
class Adapter(private val service: Service) : ClientInterface {
    override fun method(data: String): String {
        val specialData = convertToServiceFormat(data)
        return service.serviceMethod(specialData)
    }

    private fun convertToServiceFormat(data: String): String {
        return "[Adaptado] $data"
    }
}

// =======================
// Cliente
// =======================
fun main() {
    val client: ClientInterface = Adapter(Service())
    println(client.method("Solicitud del cliente")) // Service procesó: [Adaptado] Solicitud del cliente
}
```

### *Facade*
Patrón Estructural que proporciona una interfaz simplificada a **Subsistema Complejo** (una librería, un _framework_ o cualquier otro grupo complejo de clases). Resulta útil cuando la aplicación se tiene que integrar con una librería sofisticada con decenas de funciones, de la cual sólo se necesita una pequeña parte.  
También puede crearse una clase **Fachada Adicional** para evitar contaminar una única fachada con funciones no relacionadas que podrían convertirla en otra estructura compleja. Las fachadas adicionales pueden utilizarse por clientes y por otras fachadas.

<br>
<p align="center">
    <img src="images/facade.png" width="1000" alt="">
</p>
<br>

📌 **Ejemplo**:  
> 🧠 **Idea clave**: El **Cliente** no interactúa directamente con múltiples clases del **Subsistema**. En su lugar, se comunica con una **Fachada**, que orquesta y delega las llamadas necesarias. Opcionalmente, puede existir una **Fachada Adicional** para agrupar otra parte del subsistema y evitar que la fachada principal crezca demasiado.

```kotlin
// ===============================
// Subsistema (complejo)
// ===============================
class AudioDecoder {
    fun decode(file: String) = println("Decodificando audio de $file")
}

class VideoDecoder {
    fun decode(file: String) = println("Decodificando video de $file")
}

class Renderer {
    fun render() = println("Renderizando contenido en pantalla")
}

class NetworkStreamer {
    fun stream() = println("Transmitiendo datos por red")
}

// ===============================
// Fachada Adicional
// ===============================
class StreamingFacade(
    private val networkStreamer: NetworkStreamer,
    private val renderer: Renderer
) {
    fun anotherOperation() {
        networkStreamer.stream()
        renderer.render()
    }
}

// ===============================
// Fachada Principal
// ===============================
class MediaFacade(
    // linksToSubsystemObjects
    private val audioDecoder: AudioDecoder,
    private val videoDecoder: VideoDecoder,
    private val renderer: Renderer,
    // optionalAdditionalFacade
    private val streamingFacade: StreamingFacade? = null
) {

    fun subsystemOperation(file: String) {
        println("MediaFacade: iniciando reproducción simplificada...")
        audioDecoder.decode(file)
        videoDecoder.decode(file)
        renderer.render()
    }

    fun playStreaming() {
        streamingFacade?.anotherOperation()
    }
}

// ===============================
// Cliente
// ===============================
fun main() {
    val renderer = Renderer()
    val streaming = StreamingFacade(NetworkStreamer(), renderer)

    val facade = MediaFacade(
        audioDecoder = AudioDecoder(),
        videoDecoder = VideoDecoder(),
        renderer = renderer,
        streamingFacade = streaming
    )

    // El cliente usa una API simple
    facade.subsystemOperation("movie.mp4")
    facade.playStreaming()
}

// MediaFacade: iniciando reproducción simplificada...
// Decodificando audio de movie.mp4
// Decodificando video de movie.mp4
// Renderizando contenido en pantalla
// Transmitiendo datos por red
// Renderizando contenido en pantalla
```

### *Decorator*
Patrón Estructural que permite **añadir funcionalidades a objetos** sin la necesidad de crear una nueva subclase, colocándolos dentro de **objetos encapsuladores (_wrappers_)** que contienen estas funcionalidades. Los objetos resultantes **se estructurarán como una pila**, siendo el último decorador de la pila el objeto con el que el cliente trabaja.  
Sirve para estructurar la lógica de negocio en capas, crear un decorador para cada capa y componer objetos con varias combinaciones de esta lógica, en tiempo de ejecución. El código cliente puede tratar a todos estos objetos de la misma forma, ya que todos siguen una interfaz común.

<br>
<p align="center">
    <img src="images/decorator.png" width="1000" alt="">
</p>
<br>

📌 **Ejemplo**:  
> 🧠 **Idea clave**: Los decoradores **implementan la misma interfaz que el componente**, contienen una referencia al objeto envuelto (_wrappee_) y delegan la operación principal, agregando comportamiento antes o después. Se pueden **apilar dinámicamente**.

```kotlin
// ================================
// Component
// ================================
interface Component {
    fun execute()
}

// ================================
// Concrete Component
// ================================
class ConcreteComponent : Component {
    override fun execute() {
        println("ConcreteComponent: ejecución base")
    }
}

// ================================
// Base Decorator
// ================================
open class BaseDecorator(
    protected val wrappee: Component
) : Component {

    override fun execute() {
        wrappee.execute() // Delegación al componente envuelto
    }
}

// ================================
// Concrete Decorator A
// ================================
class LoggingDecorator(component: Component) : BaseDecorator(component) {

    override fun execute() {
        println("LoggingDecorator: antes de ejecutar")
        super.execute()
        println("LoggingDecorator: después de ejecutar")
    }
}

// ================================
// Concrete Decorator B
// ================================
class SecurityDecorator(component: Component) : BaseDecorator(component) {

    override fun execute() {
        println("SecurityDecorator: chequeo de seguridad")
        super.execute()
    }
}

// ================================
// Client
// ================================
fun main() {
    val component = ConcreteComponent()

    val decorated =
        SecurityDecorator(
            LoggingDecorator(component)
        )

    decorated.execute()
}

// SecurityDecorator: chequeo de seguridad
// LoggingDecorator: antes de ejecutar
// ConcreteComponent: ejecución base
// LoggingDecorator: después de ejecutar
```

### *Strategy*
Patrón de Comportamiento que permite extraer una familia de algoritmos de una única clase, colocar cada uno de ellos en una clase separada y hacer sus **objetos intercambiables**. Cada una de esas clases separadas se llama **Estrategia**. Y la clase original, llamada **Contexto**, debe tener un campo para almacenar una referencia a una de las estrategias, ya que no es responsable de seleccionar un algoritmo adecuado para la tarea, sino que es el cliente quien le pasa la estrategia deseada.

<br>
<p align="center">
    <img src="images/strategy.png" width="1000" alt="">
</p>
<br>

📌 **Ejemplo**:  
> 🧠 **Idea clave**: El **Contexto delega el comportamiento** a un objeto Estrategia que implementa una interfaz común. La estrategia puede **cambiarse en tiempo de ejecución**, alterando el algoritmo sin modificar el contexto.

```kotlin
// ================================
// Strategy
// ================================
interface Strategy {
    fun execute(data: String): String
}

// ================================
// Concrete Strategies
// ================================
class UpperCaseStrategy : Strategy {
    override fun execute(data: String): String {
        return data.uppercase()
    }
}

class ReverseStrategy : Strategy {
    override fun execute(data: String): String {
        return data.reversed()
    }
}

// ================================
// Context
// ================================
class Context(
    private var strategy: Strategy
) {

    fun setStrategy(strategy: Strategy) {
        this.strategy = strategy
    }

    fun doSomething(data: String) {
        val result = strategy.execute(data)
        println("Resultado: $result")
    }
}

// ================================
// Client
// ================================
fun main() {
    val context = Context(UpperCaseStrategy())

    context.doSomething("kotlin") // Resultado: KOTLIN

    context.setStrategy(ReverseStrategy())

    context.doSomething("kotlin") // Resultado: niltok
}
```

### *Observer*
> ℹ️ **Nota:**  
> También llamado Observador, Publicación-Suscripción, Modelo-patrón, _Event-Subscriber_, _Listener_

Patrón de Comportamiento que permite definir un **mecanismo de suscripción para notificar a varios objetos (_subscribers_)** sobre cualquier **evento que le suceda al objeto que están observando (_publisher_)**.  
Sirve para cuando los cambios en el estado de un objeto necesitan cambiar otros objetos y el grupo de esos objetos es desconocido de antemano o cambia dinámicamente.

<br>
<p align="center">
    <img src="images/observer.png" width="1000" alt="">
</p>
<br>

📌 **Ejemplo**:  
> 🧠 **Idea clave**: El **_Publisher_ mantiene una lista de suscriptores** y los notifica cuando cambia su estado. Los suscriptores implementan una interfaz común y reaccionan al evento mediante ``update()``.

```kotlin
// ================================
// Subscriber
// ================================
interface Subscriber {
    fun update(context: String)
}

// ================================
// Concrete Subscribers
// ================================
class EmailSubscriber(private val name: String) : Subscriber {
    override fun update(context: String) {
        println("Email a $name: nuevo estado = $context")
    }
}

class SmsSubscriber(private val name: String) : Subscriber {
    override fun update(context: String) {
        println("SMS a $name: nuevo estado = $context")
    }
}

// ================================
// Publisher
// ================================
class Publisher {
    private val subscribers = mutableListOf<Subscriber>()
    private var mainState: String = "Inicial"

    fun subscribe(s: Subscriber) {
        subscribers.add(s)
    }

    fun unsubscribe(s: Subscriber) {
        subscribers.remove(s)
    }

    fun notifySubscribers() {
        for (s in subscribers) {
            s.update(mainState)
        }
    }

    fun mainBusinessLogic(newState: String) {
        println("Publisher: cambiando estado a $newState")
        mainState = newState
        notifySubscribers()
    }
}

// ================================
// Client
// ================================
fun main() {
    val publisher = Publisher()

    val email = EmailSubscriber("Ana")
    val sms = SmsSubscriber("Luis")

    publisher.subscribe(email)
    publisher.subscribe(sms)

    publisher.mainBusinessLogic("ACTIVO")
}

// Publisher: cambiando estado a ACTIVO
// Email a Ana: nuevo estado = ACTIVO
// SMS a Luis: nuevo estado = ACTIVO
```

### *State*
TODO...

📌 **Ejemplo**:  
> 🧠 **Idea clave**:

```kotlin

```

### Referencias
- [Design Patterns In Kotlin](https://medium.com/@michalankiersztajn/list/design-patterns-in-kotlin-12e52466affe)
- [Refactoring Guru - Patrones de diseño](https://refactoring.guru/es/design-patterns/catalog)
