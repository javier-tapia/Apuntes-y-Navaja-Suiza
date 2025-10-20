<h1>Apuntes de Kotlin</h1>  

***Index***:
<!-- TOC -->
  * [0. Modificadores de visibilidad: *private*, *protected*, *internal* y *public* (por defecto)](#0-modificadores-de-visibilidad-private-protected-internal-y-public-por-defecto)
  * [1. Clases y Objetos en Kotlin](#1-clases-y-objetos-en-kotlin)
    * [1.1. Declaración](#11-declaración)
    * [1.2. Clases de enumeración (*enum class*)](#12-clases-de-enumeración-enum-class)
    * [1.3. Clases selladas (*sealed class*)](#13-clases-selladas-sealed-class)
    * [1.4. Clases abstractas (*abstract class*)](#14-clases-abstractas-abstract-class)
    * [1.5. Interfaces](#15-interfaces)
    * [1.6. Tipos genéricos](#16-tipos-genéricos)
      * [:clipboard: A modo de resumen:](#clipboard-a-modo-de-resumen)
      * [1.6.1. ``invariant``](#161-invariant)
      * [1.6.2. ``covariant``](#162-covariant)
      * [1.6.3. ``contravariant``](#163-contravariant)
    * [1.7. Clases anidadas e internas (*inner class*)](#17-clases-anidadas-e-internas-inner-class)
    * [1.8. *Data classes*](#18-data-classes)
    * [1.9. *Object expressions* y *Object declarations*](#19-object-expressions-y-object-declarations)
      * [1.9.1. *Object expression*](#191-object-expression)
      * [1.9.2. *Object declaration*](#192-object-declaration)
  * [2. Propiedades y fields](#2-propiedades-y-fields)
    * [2.1. Declaración](#21-declaración)
    * [2.2. *Default arguments* y *Named arguments*](#22-default-arguments-y-named-arguments)
    * [2.3. *Backing fields* (*getters* y *setters*)](#23-backing-fields-getters-y-setters)
    * [2.4. *Class Delegation*](#24-class-delegation)
    * [2.5. *Delegated properties*](#25-delegated-properties)
      * [2.5.1. *Lazy properties*](#251-lazy-properties)
      * [2.5.2. *Observable properties*](#252-observable-properties)
      * [2.5.3. Almacenar propiedades en un *map*](#253-almacenar-propiedades-en-un-map)
  * [3. Funciones](#3-funciones)
    * [3.1. Declaración](#31-declaración)
    * [3.2. *Infix functions*](#32-infix-functions)
    * [3.3. *Varargs* (número variable de argumentos)](#33-varargs-número-variable-de-argumentos)
    * [3.4. *Expresiones Lambda* y *Funciones Anónimas*](#34-expresiones-lambda-y-funciones-anónimas)
      * [3.4.1. *Trailing lambda*](#341-trailing-lambda)
      * [3.4.2. Lambda con un solo parámetro](#342-lambda-con-un-solo-parámetro)
      * [3.4.3. *Function Type*](#343-function-type)
      * [3.4.4. *Non-local returns*](#344-non-local-returns)
    * [3.5. Funciones de extensión](#35-funciones-de-extensión)
    * [3.6. Funciones de Orden Superior (*Higher-Order Functions*)](#36-funciones-de-orden-superior-higher-order-functions)
    * [3.7. *Function type with receiver* y *Function literal with receiver*](#37-function-type-with-receiver-y-function-literal-with-receiver)
    * [3.8. *Closures*](#38-closures)
    * [3.9. *Inline Functions*](#39-inline-functions)
      * [3.9.1. Parámetros de tipo *reified*](#391-parámetros-de-tipo-reified)
    * [3.10. Tipos genéricos](#310-tipos-genéricos)
  * [4. Colecciones](#4-colecciones)
    * [4.1. Descripción](#41-descripción)
    * [4.2. ``Collection<T>``](#42-collectiont)
    * [4.3. ``List<T>``](#43-listt)
    * [4.4. ``Set<T>``](#44-sett)
    * [4.5. *Map* (o *Dictionary*)](#45-map-o-dictionary)
    * [4.6. Construcción de colecciones](#46-construcción-de-colecciones)
      * [4.6.1. Colecciones vacías](#461-colecciones-vacías)
      * [4.6.2. Inicializar listas](#462-inicializar-listas)
    * [4.7. Copia de colecciones](#47-copia-de-colecciones)
    * [4.8. *Iterators*](#48-iterators)
      * [4.8.1. *ListIterator*](#481-listiterator)
      * [4.8.2. *MutableIterator*](#482-mutableiterator)
    * [4.9. Rangos y progresiones](#49-rangos-y-progresiones)
      * [4.9.1. Rangos](#491-rangos)
      * [4.9.2. Progresiones](#492-progresiones)
    * [4.10. ``Sequence<T>``](#410-sequencet)
      * [4.10.1. Construcción](#4101-construcción)
      * [4.10.2. *Sequence operations*](#4102-sequence-operations)
    * [4.11. Operaciones](#411-operaciones)
      * [4.11.1. Operaciones comunes](#4111-operaciones-comunes)
      * [4.11.2. Operaciones de escritura](#4112-operaciones-de-escritura)
  * [5. Hilos, Corrutinas (*Coroutines*) y *Flows*](#5-hilos-corrutinas-coroutines-y-flows)
  * [6. Otros constructos del lenguaje](#6-otros-constructos-del-lenguaje)
    * [6.1. *Operator overloading*](#61-operator-overloading)
    * [6.2. *Destructuring declarations*](#62-destructuring-declarations)
    * [6.3. *Type checks* y *Casts*](#63-type-checks-y-casts)
    * [6.4. *This expression*](#64-this-expression)
    * [6.5. *Null safety*](#65-null-safety)
    * [6.6. *SAM* (*Single Abstract Method*) *conversions*](#66-sam-single-abstract-method-conversions)
    * [6.7. *Reflection*](#67-reflection)
    * [6.8. *Scope functions*](#68-scope-functions)
  * [Referencias](#referencias)
<!-- TOC -->

---

## 0. Modificadores de visibilidad: *private*, *protected*, *internal* y *public* (por defecto)

Las funciones, las propiedades, las clases, los *objects* y las *interfaces* se pueden declarar en “***top-level***” (directamente dentro del paquete). Aún así, para usar una declaración en *top-level* **de otro paquete, se debe importar**.
-	***public*** (por defecto): la declaración será **visible desde cualquier lugar**.
-	***private***: la declaración será **visible sólo dentro del archivo** que la contiene.
-	***internal***: la declaración será **visible en cualquier lugar dentro del mismo módulo** (conjunto de archivos .kt ―Kotlin― que se compilan juntos).
-	***protected***: para declaraciones *top-level* **no está disponible**.

Para ***miembros*** (funciones y propiedades) declaradas **dentro de una clase**:
-	***public*** (por defecto): la declaración será **visible desde cualquier lugar** que pueda acceder a la clase que la contiene.
-	***private***: la declaración será **visible sólo dentro de la clase** que la contiene.
-	***internal***: la declaración será **visible en cualquier lugar dentro del mismo módulo** que pueda acceder a la clase que la contiene.
-	***protected***: la declaración será **visible dentro de la clase** que la contiene **y en las subclases** también.

Los constructores primarios son públicos por defecto, por lo cual podrá ser visto por cualquiera que pueda ver la clase. Para especificar una visibilidad al constructor, se agrega el modificador seguido de la palabra clave ***constructor***:  

```kotlin
    class Example private constructor(arg: Int) {
        // Do something
    }
```

Las **variables locales**, las **funciones locales** y las **clases locales** (las declaradas dentro de una función), **no pueden tener modificadores de visibilidad**.

```kotlin
     var topLevel = "Top-level publica"
     private var topLevelPrivate = "Top-level privada"
     internal var topLevelInterna = "Top-level interna"
     
     open class Ejemplo {
          var publica = "Publica"
          private var privada = "Privada"
          internal var interna = "Interna"
          protected var protegida = "Protegida"
          fun imprimirPrivada() {
               println(privada)
          }
     }
     
     class HeredaEjemplo : Ejemplo() {
          fun imprimirProtected() {
               println(protegida)
          }
     }
     
     fun main() {
          var ejemplo = Ejemplo()
          var ejemploHeredada = HeredaEjemplo()
          println(topLevel)
          println(topLevelPrivate)
          println(topLevelInterna)
          println(ejemplo.publica)
          println(ejemplo.interna)
          ejemplo.imprimirPrivada()
          ejemploHeredada.imprimirProtected()
     }
```

---

## 1. Clases y Objetos en Kotlin
### 1.1. Declaración

```kotlin
    class Example (name: String): AnOpenClass(), Interface {
        // Do something
    }
```

Los ***dos puntos*** indican que hereda de otra clase (solo una) o *interfaces* (pueden ser más de una), separadas por comas.

Entre paréntesis, se pueden requerir los parámetros o las propiedades del ***constructor primario***. Además del constructor primario, también puede tener un **bloques** ***``init``*** (utilizado para poner código de inicialización; **se ejectuta inmediatamente después del constructor primario**) y uno o más ***constructores secundarios*** (deben llevar la palabra reservada ***``constructor``***; **se ejecutan después del constructor primario y el bloque** ***init*** si lo hubiera; pueden delegar parámetros al constructor primario con ***``this``***). Los constructores secundarios suelen usarse cuando se necesita extender una clase que proporcione múltiples constructores que inicialicen la clase de diferentes maneras.
   
```kotlin
    class Sample(private var s: String) {
        constructor(t: String, u: String) : this(t) {
            this.s += u
    
            println(s)  // Imprime: TBU
        }
    
        init {
            s += "B"
        }
    }
    
    fun main() {
        val sample = Sample("T", "U")
    }
```

Para crear una instancia de una clase (un ***objeto***), no se utiliza la palabra reservada *new* utilizada en Java:

```kotlin
    val customer = Customer("Joe Smith")
```

Para poder ***heredar*** una clase y ***sobreecribir*** (***override***) uno de sus miembros, tanto la clase como el miembro deben tener el modificador ``open``.

```kotlin
    open class Ejemplo {
        open fun saludar() = println("Hola mundo")
    }
    
    class EjemploDos : Ejemplo() {
        var obj = Ejemplo()
        var saludo = obj.saludar()
        override fun saludar() = println("Chau")
    }
    
    fun main() {
        var objDos = EjemploDos()
        objDos.saludo // Imprime 'Hola mundo'
        objDos.saludar() // Imprime 'Chau'
    }
```

### 1.2. Clases de enumeración (*enum class*)
El uso más básico de las clases *enum* es implementar enumeraciones con ***seguridad de tipo*** (***type-safe***). Cada constante *enum* es una **instancia** de la clase *enum*, y se declaran separadas con comas. Dado que cada elemento es una instancia de la clase *enum*, también se pueden inicializar haciendo referencia a una propiedad definida en el constructor (un **valor**). Y además de valor, pueden tener **comportamiento**, de dos maneras: definiendo sus propios **métodos** (abstractos o no) o implementando ***interfaces***.

```kotlin
    enum class DiaSemana(val dia: Int) {
        LUNES(1) {
            override fun esFinDeSemana() = false
        },
        MARTES(2) {
            override fun esFinDeSemana() = false
        },
        MIERCOLES(3) {
            override fun esFinDeSemana() = false
        },
        JUEVES(4) {
            override fun esFinDeSemana() = false
        },
        VIERNES(5) {
            override fun esFinDeSemana() = false
        },
        SABADO(6) {
            override fun esFinDeSemana() = true
        },
        DOMINGO(7) {
            override fun esFinDeSemana() = true
        };
    
        abstract fun esFinDeSemana(): Boolean
    
        fun anteriorA(diaSemana: DiaSemana): Boolean = dia < diaSemana.dia
    }
    
    fun main() {
        val dia1 = DiaSemana.JUEVES
        val dia2 = DiaSemana.DOMINGO
        println(dia1.esFinDeSemana()) // false
        println(dia2.esFinDeSemana()) // true
        println(dia1.anteriorA(dia2)) // true
        println(dia2.anteriorA(dia1)) // false
    }
```

### 1.3. Clases selladas (*sealed class*)
Permiten tener varias **clases heredadas** (se convierten en un tipo de “*enum* de clases”) y representar jerarquías de **clases restringidas** en las que un valor debe tener uno de los tipos de un conjunto limitado, pero no puede tener ningún otro tipo. Cada clase tiene sus propiedades, además de las heredadas, es decir, una subclase de una clase sellada puede tener múltiples instancias que pueden contener estado. Se utilizan mucho para realizar **comprobaciones de tipo** (***type-checks***) en expresiones y sentencias con ``when``:

```kotlin
    sealed class Operacion {
        class Suma(val valor: Int) : Operacion()
        class Resta(val valor: Int) : Operacion()
        class Multiplicacion(val valor: Int) : Operacion()
        class Division(val valor: Int) : Operacion()
    }
    
    fun main() {
        val numero = 5
        var operacion: Operacion
        operacion = Operacion.Suma(3)
        val numero2 = calcular(numero, operacion)
        println(numero2)  // 8
        operacion = Operacion.Multiplicacion(6)
        println(calcular(numero2, operacion))  // 48
    }
    
    fun calcular(x: Int, op: Operacion) = when (op) {
        is Operacion.Suma -> x + op.valor
        is Operacion.Resta -> x - op.valor
        is Operacion.Multiplicacion -> x * op.valor
        is Operacion.Division -> x / op.valor
    }
```

### 1.4. Clases abstractas (*abstract class*)
Una clase y algunos de sus miembros pueden ser declarados como ***abstract***. Un miembro abstracto **no tiene una implementación en su clase**. Tampoco se necesita anotar una clase o función abstracta con ***open*** (se sobreentiende que es así). Además, **no se puede crear una instancia** de una clase abstracta.

```kotlin
    abstract class ClaseAbstracta {
        var nombre: String = "Nati"
        abstract fun saludar()
    }
    
    class Nombre() : ClaseAbstracta() {
        override fun saludar() {
            println("Hola $nombre")
        }
    }
    
    fun main() {
        var obj = Nombre()
        obj.saludar() // Imprime 'Hola Nati'
    }
```

### 1.5. Interfaces
Pueden contener **tanto declaraciones de métodos abstractos como implementaciones de métodos por defecto** (se diferencian porque unos no tienen la implementación y los otros sí; no hace falta usar la palabra reservada *default* como lo hace Java). La o las clases que implementen la interfaz, **deben implementar todo** lo que la interfaz tenga declarado. Lo que las hace diferentes de las clases abstractas es que **las interfaces no pueden contener estado**. Pueden tener propiedades, pero deben ser abstractas o proporcionar implementaciones de acceso (*getters* y *setters*).

```kotlin
    interface EjemploInterface {
        fun saludar()
    }
    
    class ClaseA : EjemploInterface {
        override fun saludar() = println("Hola")
    }
    
    class ClaseC : EjemploInterface {
        override fun saludar() = println("Chau")
    }
    
    class ClaseB(var tipo: EjemploInterface) {
        var saludo = when (tipo) {
            is ClaseA -> tipo.saludar()
            is ClaseC -> tipo.saludar()
            else -> println("Error")
        }
    }
    
    fun main() {
        var tipo: EjemploInterface = ClaseC()
        var obj = ClaseB(tipo)
        obj.saludo // Imprime ‘Chau’
    }
```

Si una clase implementa dos *interfaces* que tienen un método no abstracto con el mismo nombre, al llamar a ese método desde la clase existe un **conflicto de nombres** y el compilador generará un error. Este problema se puede resolver proporcionando una nueva implementación del método:

```kotlin
    interface A {
        fun llamar() {
            println("Desde interface A")
        }
    }
    
    interface B {
        fun llamar() {
            println("Desde interface B")
        }
    }
    
    class ClaseImp : A, B {
        // Implementación explícita del método en la clase
        override fun llamar() {
            super<A>.llamar() // llama al método de la interface A
        }
    }
    
    fun main(args: Array<String>) {
        val obj = ClaseImp()
        obj.llamar()
    }
```

### 1.6. Tipos genéricos
Permiten definir clases, métodos (ver el apartado de genéricos de *Funciones*) y propiedades de manera que se puede **acceder a ellos utilizando diferentes tipos**. Es común utilizarlos en combinación con las **colecciones** para crear estructuras cuyos elementos no están limitados a un tipo determinado.  
Una clase o un método de tipo genérico se declara con **parámetros de tipo** (o **tipo parametrizado**) entre corchetes angulares, y al crear una instancia de esa clase se debe proporcionar el tipo real del argumento (salvo que pueda ser inferido), por ejemplo:

```kotlin
    class Caja<T>(t: T) {
        var valor = t
    }
    
    fun main() {
        val caja1 = Caja<Int>(1) // val caja1: Caja<Int> = Caja<Int>(1)
        val caja2 = Caja(1) // Inferencia del tipo
        val caja3 =
            Caja<Int>("Hola") // ERROR. Type mismatch: inferred type is String but Int was expected
        val caja3 = Caja("Hola")
        val caja4 = Caja(listOf('a', 'b', 'c', 'd'))
    
        println(caja1.valor) // 1
        println(caja2.valor) // 1
        println(caja3.valor) // Hola
        println(caja4.valor) // [a, b, c, d]
    }
```

El uso de genéricos conlleva una serie de ventajas, como son la **seguridad de tipo** (la comprobación se realiza en tiempo de compilación para evitar problemas durante la ejecución) y que **no requiere la conversión entre tipos** ni encasillar el objeto en un tipo determinado.  
Al trabajar con genéricos, los tipos no mantienen la herencia de sus clases. Y en función de si esas clases actúan como **consumidores o productores**, tendremos que usar **covarianza o contravarianza**.

#### :clipboard: A modo de resumen:

- **_Consumir_** implica tener **_funciones que devuelven un valor del tipo genérico_**.
- **_Producir_** implica tener **_funciones que reciben por argumento un objeto del tipo genérico_**.

Por tanto:

- Para **_devolver valores de un tipo más genérico que el tipo original_**, se necesita usar **_covarianza_** (se usa ***``out``*** en el tipo).
- Para **_pasar valores de un tipo más genérico que el original_**, se necesita usar **_contravarianza_** (se usa ***``in``*** en el tipo).
- Para **_trabajar con el tipo original_**, el tipo será **_invariante_** (no se usa ``in`` ni ``out``).

#### 1.6.1. ``invariant``
Cuando el parámetro de tipo no tiene modificadores de varianza (***``out``*** o ***`in`***), por defecto es invariante con respecto a su tipo genérico (no se pueden ni consumir ni producir valores con un tipo más genérico). Significa que **no hay relación entre dos tipos** generados por una clase genérica. Por ejemplo, no hay relación entre *``Cup<Int>``* y *``Cup<Number>``*, *``Cup<Any>``* o *``Cup<Nothing>``*.

```kotlin
     class Cup<T>
     
     fun main(args: Array<String>) {
          val anys: Cup<Any> = Cup<Int>() // Error: Type mismatch
          val nothings: Cup<Nothing> = Cup<Int>() // Error: Type mismatch
     }
```

#### 1.6.2. ``covariant``
Para hacer que el parámetro de tipo sea covariante con respecto a su tipo genérico (se pueden consumir sus valores con un tipo más genérico), se utiliza el **modificador** ***``out``***. Significa que **cuando** ***A*** **es subtipo de** ***B*** y ***Cup*** **es covariante**, el tipo ***``Cup<A>``*** **es subtipo de** ***``Cup<B>``***.

```kotlin
     class Cup<out T>
     
     open class B
     
     class A : B()
     
     fun main() {
          val b: Cup<B> = Cup<A>() // OK
          val a: Cup<A> = Cup<B>() // Error: Type mismatch
     
          val anys: Cup<Any> = Cup<Int>() // OK
          val nothings: Cup<Nothing> = Cup<Int>() // Error: Type mismatch
     }
```

Para asegurar la seguridad de tipo, Kotlin prohíbe el uso de parámetros de tipo ***covariante*** en posiciones *in* (argumentos de métodos públicos) o *invariantes* (propiedades públicas), debiéndolo usar **solamente en posiciones** ***out*** (tipos de retorno de métodos públicos), de ahí viene el uso del modificador *out* para los tipos covariantes.

```kotlin
     interface Source<out T> {
          fun nextT(): T
     }
     
     fun demo(strs: Source<String>) {
          val objects: Source<Any> = strs // Esto está OK, ya que T es un parámetro out
     // ...
     }
```

#### 1.6.3. ``contravariant``
Para hacer que el parámetro de tipo sea contravariante, se utiliza el **modificador** ***``in``***. Significa que cuando ***A*** **es subtipo de** ***B*** y ***Cup*** **es contravariante**, el tipo ***``Cup<A>``*** **es supertipo de** ***``Cup<B>``***.

```kotlin
     class Cup<in T>
     
     open class B
     
     class A : B()
     
     fun main() {
          val b: Cup<B> = Cup<A>() // Error: Type mismatch
          val a: Cup<A> = Cup<B>() // OK
     
          val anys: Cup<Any> = Cup<Int>() // Error: Type mismatch
          val nothings: Cup<Nothing> = Cup<Int>() // OK
     }
```

Los parámetros de tipo ***contravariante***, que se realizan con el modificador ***in*** en Kotlin, **sólo se permiten en posiciones** ***in*** (argumentos de métodos públicos), estando prohibidos en posiciones *out* (tipos de retorno de métodos públicos) e *invariantes* (propiedades públicas).

```kotlin
     interface Comparable<in T> {
          operator fun compareTo(other: T): Int
     }
     
     fun demo(x: Comparable<Number>) {
          x.compareTo(1.0) // 1.0 es de tipo Double, que es un subtipo de Number
     // Así, se puede asignar x a una variable de tipo Comparable<Double>
          val y: Comparable<Double> = x // OK!
     }
```

### 1.7. Clases anidadas e internas (*inner class*)
Una clase anidada marcada como *``inner``* puede acceder a los miembros de su clase externa (*outer class*). **Las clases internas llevan una referencia a un objeto de una clase externa**.

```kotlin
     class Outer {
         private val bar: Int = 1
     
         inner class Inner {
             fun foo() = bar
         }
     }
     
     val demo = Outer().Inner().foo() // == 1
```

### 1.8. *Data classes*
Se declaran anteponiendo la palabra reservada *``data``*. Tienen como objetivo principal **almacenar datos**, pero a su vez el compilador genera automáticamente funciones para todas las propiedades declaradas en el constructor primario: ***``equals()``*** / ***``hashCode()``***; ***``toString()``***; **funciones** ***``componentN()``***; función ***``copy()``***. Por esta razón, y porque en Kotlin no es necesario declarar los *setters* y *getters* de las propiedades (ver [*Properties y Fields*](#2-propiedades-y-fields)), se considera a las *data classes* una mejor versión de los *POJO* usados en Java, ya que no requieren de tanto código. **Requerimientos**: el constructor primario debe tener **al menos un parámetro**; todos los parámetros del constructor primario **deben marcarse con** ***val*** **o** ***var***; **no pueden ser** *abstract*, *open*, *sealed* o *inner*.

```kotlin
    data class User(val name: String, val age: Int)
```

### 1.9. *Object expressions* y *Object declarations*
Se utilizan para crear un objeto de alguna clase con alguna modificación sin declarar explícitimente una nueva subclase. **No pueden tener constructores** y pueden extender otras clases o implementar interfaces:

#### 1.9.1. *Object expression*
Juegan el mismo papel en Kotlin que las *clases anónimas* en Java (declaran e instancian una clase al mismo tiempo). Se **inicializan de inmediato** cuando son utilizados.

```kotlin
     open class A(x: Int) {
          open val y: Int = x
     }
     
     interface B { /*...*/ }
     
     fun main() {
     // Si un supertipo tiene un constructor, se le deben pasar los parámetros. 
     // Muchos supertipos se pueden especificar como una lista separada por comas.
          val ab: A = object : A(1), B {
               override val y = 15
          }
     
          println(ab.y) // Imprime: 15
     }
```

#### 1.9.2. *Object declaration*
Es una **implementación del patrón** ***singleton***. Siempre tienen nombre y **se inicializan** ***lazily***, cuando son accedidos **por primera vez**. No pueden ser locales (anidados dentro de una función), pero se pueden anidar en otros *objects declarations* o en clases no internas (*non-inner classes*). Además, como no es una expresión, no se puede utilizar en una asignación de variable como un *object expression*.

```kotlin
    // Crea un object declaration
     object DoAuth {
          // Define el método del objeto
          fun takeParams(username: String, password: String) {
               println("input Auth parameters = $username:$password")
          }
     }
     
     fun main() {
     // Llama al método. Acá es cuando realmente el objeto es creado.
          DoAuth.takeParams("foo", "qwerty")
     }
```

Un **``companion object``** es un *object declaration* dentro de una clase. Los miembros del mismo pueden ser llamados usando el nombre de la clase (es como invocar un *método estático* en Java).

```kotlin
     class BigBen { // Define una clase
          companion object Bonger { // Define un companion object. Se puede omitir el nombre.
               fun getBongs(nTimes: Int) { // Define un método dentro del companion object
                    for (i in 1..nTimes) {
                         print("BONG ")
                    }
               }
          }
     }
     
     fun main() {
          BigBen.getBongs(12) // Llama al método vía el nombre de la Clase.
     }
     
     // Imprime: BONG BONG BONG BONG BONG BONG BONG BONG BONG BONG BONG BONG
```

---

## 2. Propiedades y fields

### 2.1. Declaración

```kotlin
    val ejemplo: String = “Hola mundo”
```

Las variables pueden ser de tipo ***``var``*** (mutables), ***``val``*** (inmutables) o ***``const``*** (constante en tiempo de compilación, es decir, **no se puede asignar un valor a la variable en tiempo de ejecución** como en las *``val``*). Las propiedades *``const``* deben estar declaradas en *top-level* o ser miembro de un *object* o de un *companion object* (ver apartado 1.9), deben ser inicializadas con un *String* o con un tipo primitivo, y no pueden tener *custom getters* (ver apartado 2.3).

En relación a su visibilidad e inicialización:
- Una propiedad ***top-level*** (dentro del paquete) deber ser inicializada.
- Una propiedad ***miembro*** (dentro de la clase) debe ser inicializada **o** ser abstracta. También puede inicializarse más tarde con el modificador ***``lateinit``*** (sólo en *var properties* y para tipos no primitivos; *lateinit* retrasa la inicialización de la variable sin peligro de devolver una referencia nula) o con la función ***``lazy()``*** (ver *Propiedades delegadas*).
- Una variable ***local*** (dentro de una función) debe tener anotado el tipo **o** ser inicializada, y no pueden ser sobreescritas.
- Un ***parámetro*** debe tener anotado el tipo. Dentro del *header* de una clase, si se agrega *var* o *val* al nombre del parámetro, se vuelve una *property* de la clase (**se declaran e inicializan** en el **constructor primario**). Por otro lado, los ***parámetros*** **de las funciones siempre son** ***val*** (de sólo lectura).

### 2.2. *Default arguments* y *Named arguments*
A los **parámetros de una función**, pueden asignárseles **valores por defecto**, que se usan cuando el argumento correspondiente **se omite** al invocar la función. O también se les puede **cambiar el valor** por defecto **al nombrarlos** y darles **otro valor** al momento de invocar la función. Esto evita *overloads* (definir múltiples métodos con el mismo nombre pero diferentes parámetros) y permite llamadas más claras, para evitar errores por un orden incorrecto de los argumentos (no es lo mismo *``"nombre@mail.com"``* que *``"mail.com@nombre"``*).  
   
```kotlin
    fun foo(name: String, number: Int = 42, uppercase: Boolean = false) =
        (if (uppercase) name.uppercase() else name) + number
    
    fun useFoo() = listOf(
        foo("a"),  // Imprime: a42
        foo("b", number = 1),  // Imprime: b1
        foo("c", uppercase = true),  // Imprime: C42
        foo(name = "d", number = 2, uppercase = true)  // Imprime: D2
    )
    
    fun main() {
        println(useFoo())
    }
```

### 2.3. *Backing fields* (*getters* y *setters*)
El *backing field* es un campo (*field*) **autogenerado** para cualquier propiedad que solo se puede usar dentro de los métodos *accesores* (*getter* o *setter*) y **estará presente solamente si la propiedad usa la implementación por defecto de al menos uno de los accesores, o si un accesor personalizado lo referencia a través de la palabra reservada** ***field***. Este *backing field* **se usa para evitar la llamada recursiva** a la misma propiedad desde el *getter* o el *setter*, lo que finalmente evita el *StackOverflowError*. Las propiedades con accesores *custom*, **no pueden ser abstractos**.

```kotlin
    class User {
        var firstName: String = "Hola"
            get() = field
            set(value) {
                if (value.count() <= 3) println("'$value' no llega ni a 3 letras") else field = value
            }
    }
    
    fun main() {
        var user = User()
    
        println(user.firstName) // Imprime el valor actual del field de la propiedad firstName ("Hola")
        user.firstName = "Chau" // Llama al custom setter para asignarle un nuevo valor al field
        println(user.firstName)   // Imprime el nuevo valor del field ("Chau")
        user.firstName = "Hi" // Imprime: 'Hi' no llega ni a 3 letras
    }
```

### 2.4. *Class Delegation*
El patrón *Delegation* ha probado ser una buena **alternativa a la implementación por herencia** y Kotlin lo soporta de forma nativa, sin requerir de código *boilerplate*. Una clase puede **implementar una** ***interface*** delegando todos su miembros públicos a un objeto especificado. Para esto, se utiliza la palabra reservada ***``by``***.

```kotlin
    interface Programador {
        fun programar()
    }
    
    class Empleado(val programador: Programador) : Programador by programador
    
    class MejorProgramador : Programador {
        override fun programar() {
            println("Soy un crack codeando")
        }
    }
    
    class ProgramadorLento : Programador {
        override fun programar() {
            println("Programo lento pero seguro")
        }
    }
    
    fun main() {
        val empleado = Empleado(ProgramadorLento())
        empleado.programar() // Programo lento pero seguro
    }
```

### 2.5. *Delegated properties*
La expresión después de la palabra reservada ***``by``***, indica que será la **expresión delegada** o ***property delegate***. Es decir, que los **métodos accesores** ***getter*** (**y** ***setter*** **si es** ***var***) **de la** ***property***, serán delegados a los métodos ***getValue()*** (y ***setValue()*** **para** ***vars***) del delegado, que no tiene que implementar ninguna *interface*, pero debe proveer esas ***operator functions***:

```kotlin
    import kotlin.reflect.KProperty
    
    class Example {
        var p: String by Delegate()
    }
    
    class Delegate {
        operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
            return "$thisRef, thank you for delegating '${property.name}' to me!"
        }
    
        operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
            println("$value has been assigned to '${property.name}' in $thisRef.")
        }
    }
    
    fun main() {
        val e = Example()
        println(e.p)  // Example@4c873330, thank you for delegating 'p' to me!
    
        e.p = "NEW"   // NEW has been assigned to 'p' in Example@4c873330.
    }
```

***Standard delegates***:  La librería standard de Kotlin provee *factory methods* para varios tipos útiles de delegados: *Lazy*, *Observable* y almacenamiento de propiedades en un *map*.

#### 2.5.1. *Lazy properties*
El valor no se calcula hasta el primer acceso. ***``lazy()``*** es una función que **toma un** ***lambda*** **y retorna una instancia de** ***``Lazy<T>``***, que puede servir como un **delegado** para implementar una propiedad *lazy*: el primer llamado a *get()* ejecuta el *lambda* pasado a *lazy()* y recuerda el resultado; las llamadas subsecuentes a *get()* simplemente retornan ese resultado. Es decir, utilizando la expresión ***``by lazy``***, lo que está entre corchetes (el ***lambda initializer***) recién se ejecuta cuando la variable se utiliza por primera vez, y en posteriores invocaciones retornará el valor computado inicialmente.

```kotlin
     val lazyValue: String by lazy {
          println("computed!")
          "Hello"
     }
     
     fun main() {
          println(lazyValue) // computed! \n Hello
          println(lazyValue) // Hello
     }
```

#### 2.5.2. *Observable properties*
Los *listeners* son notificados sobre cambios en la propiedad. ***Delegates.observable()*** toma dos argumentos: el valor inicial y una función para manipular las modificaciones. Esa función, que se llama cada vez que se modifica el valor de la propiedad, tiene tres parámetros: una propiedad a asignar, el valor anterior y el valor nuevo.

```kotlin
     import kotlin.properties.Delegates
     
     class User {
          var name: String by Delegates.observable("<no name>") { prop, old, new ->
               println("$old -> $new")
          }
     }
     
     fun main() {
          val user = User()
          user.name = "first"  // <no name> -> first
          user.name = "second" // first -> second
     }
```

#### 2.5.3. Almacenar propiedades en un *map*
En vez de un *field* separado para cada propiedad, se puede usar un mapa para almacenar propiedades. A menudo aparece en aplicaciones como, por ejemplo, parsear un JSON. En estos casos, **se usa la instancia misma del** ***Map*** **como delegado**, y las propiedades delegadas toman los valores de dicha instancia, a través de los *keys*.

```kotlin
     class User(val map: Map<String, Any?>) {
          val name: String by map
          val age: Int by map
     }
     
     fun main() {
          val user = User(
               mapOf(
                    "name" to "John Doe",
                    "age" to 25
               )
          )
          println(user.name) // John Doe
          println(user.age)  // 25
     }
```

Esto también funciona para propiedades *var*, usando ***MutableMap*** en vez de *Map*.

```kotlin
     class MutableUser(val map: MutableMap<String, Any?>) {
          var name: String by map
          var age: Int by map
     }
```

---

## 3. Funciones

### 3.1. Declaración

```kotlin
    fun completeName(name: String, lastname: String): String {
        // Do something
        return "$name $lastname"
    }
```

Si se especifica un tipo de retorno, la función debe retornar un valor (expresión ***``return``***). Si no, por defecto retorna ***Unit*** (*void* en Java).

También puede ser de ***single-expression***, en cuyo caso el ***return*** **se omite** y si el tipo de retorno puede ser inferido, también se puede omitir:

```kotlin
    fun foo(x: Int, y: Int): Int = x * y
```

### 3.2. *Infix functions*
La palabra reservada ***``infix``*** sirve para llamar a una función **omitiendo el punto y los paréntesis**. Sólo se puede aplicar a **funciones miembro de una clase** y a **funciones de extensión**, y siempre a funciones con **un solo parámetro** (además este parámetro no debe aceptar un número variable de argumentos ni tener un valor predeterminado).

```kotlin
    infix fun Int.multiplicarPor(factor: Int) = this * factor
    
    fun main() {
        val producto = 20 multiplicarPor 5 // es igual a 20.multiplicarPor(5)
        println("20 x 5 = $producto")
    }
```

### 3.3. *Varargs* (número variable de argumentos)
Un parámetro de una función (normalmente el último) se puede marcar con el **modificador** ***``vararg``***, permitiendo que se pase un número variable de argumentos a la función.

```kotlin
    fun <T> asList(vararg ts: T): List<T> {
        val result = ArrayList<T>()
        for (t in ts) // ts is an Array
            result.add(t)
        return result
    }
    
    fun main() {
        val list = asList(1, 2, 3)
    }
```

Dentro de una función, un parámetro *vararg* de **tipo** ***T*** es visible como un ***array*** **de** ***T***, es decir, la variable ***ts*** en el ejemplo anterior tiene el tipo ***Array\<out T>***. Solo un parámetro puede marcarse como *vararg*. Si un parámetro *vararg* no es el último en la lista, los valores para los siguientes parámetros se pueden pasar usando la sintaxis del argumento con nombre (*named arguments*) o, si el parámetro tiene un tipo función (*function type*), pasando un *lambda* fuera del paréntesis.
Cuando se llama a una función *vararg*, se pueden pasar argumentos uno por uno, por ejemplo ***asList (1, 2, 3)***; o, si hay un *array* y se quiere pasar su contenido a la función, se usa el **operador de extensión** (prefijando el array con el **operador ``*``**):

```kotlin
    val a = arrayOf(1, 2, 3)
    val list = asList(-1, 0, *a, 4)
```

### 3.4. *Expresiones Lambda* y *Funciones Anónimas*
Son **literales funciones** (***function literals***), es decir, funciones que **no se declaran**, sino que **se pasan inmediatamente como una expresión**. La sintaxis de una expresión *lambda* es:

```kotlin
    { x: Int, y: Int -> x + y }
```

La sintaxis de una función anónima es similar a la de una expresión *lambda*, pero a diferencia de ésta, **explicita el tipo de retorno** de la función. La declaración es similar a una función común, excepto porque su nombre es omitido:

```kotlin
    fun(x: Int, y: Int): Int = x + y
```

#### 3.4.1. *Trailing lambda*
Si el último parámetro de una función es una función, entonces una expresión *lambda* pasada como el argumento correspondiente puede ser colocado **fuera del paréntesis**:

```kotlin
    val product = items.fold(1) { acc, e -> acc * e }
```

#### 3.4.2. Lambda con un solo parámetro
Si la expresión *lambda* tiene **un único parámetro**, está permitido no declararlo y omitir la ``->`` :

```kotlin
    ints.filter { it > 0 } // this literal is of type '(it: Int) -> Boolean'
```

#### 3.4.3. *Function Type*
Una variable también puede **tiparse con una función** (***function type***) y asignarle una función ***lambda***:

```kotlin
    val isEven: (Int) -> Boolean = { it % 2 == 0 }
```

En este caso, la función toma un argumento de tipo *Int* y devuelve un *Boolean*. O también, puede tomar un argumento de tipo *Int* y devolver otro *Int*:

```kotlin
    val square: (Int)->Int = { x -> x * x }
```

#### 3.4.4. *Non-local returns*
Solo se puede usar un retorno normal no calificado para salir de una función nombrada o una función anónima. Esto significa que para salir de un *lambda*, hay que usar una **etiqueta** (***label***), y un retorno simple está prohibido dentro de un *lambda*, ya que no puede hacer que la función que lo encierra regrese.

```kotlin
     fun ordinaryFunction(block: () -> Unit) {
          println("hi!")
     }
     
     fun foo() {
          ordinaryFunction volver@{
               return // ERROR: no se puede hacer que foo() retorne acá
               return@volver // OK, calificado con el label. Imprime: hi!
          }
     }
     
     fun main() {
          foo()
     }
```

Pero si la función a la que se pasa el *lambda* está marcada como ***``inline``*** (ver *Inline Functions* en el apartado 3.9), el ***``return``*** también puede estar *inlined*, por lo que lo siguiente sí está permitido:

```kotlin
     inline fun inlined(block: () -> Unit) {
          println("hi!")
     }
     
     fun foo() {
          inlined {
               return // OK: el lambda está inlined
          }
     }
     
     fun main() {
          foo()
     }
```

Dichos retornos (ubicados en un *lambda*, pero que salen de la función que los encierra) se denominan **retornos no locales** (***non-local returns***). Es habitual este tipo de construcción en bucles, que las funciones en línea suelen incluir:

```kotlin
     fun hasZeros(ints: List<Int>): Boolean {
          ints.forEach {
               if (it == 0) return true // Retorna de hasZeros
          }
          return false
     }
```

### 3.5. Funciones de extensión
Una función de extensión se declara agregando como prefijo **el tipo que está siendo extendido** (***receiver type***). La palabra reservada ***``this``*** (ver [*This expression*](#64-this-expression)) dentro de una función de extensión corresponde al ***receiver object*** (objeto *receptor* o *destinatario*, el que se pasa antes del punto):

```kotlin
    fun MutableList<Int>.swap(index1: Int, index2: Int) {
        val tmp = this[index1] // 'this' corresponds to the list
        this[index1] = this[index2]
        this[index2] = tmp
    }
    
    val list = mutableListOf(1, 2, 3)
    list.swap(0, 2) // 'this' inside 'swap()' will hold the value of 'list'
```

### 3.6. Funciones de Orden Superior (*Higher-Order Functions*)
Una función de orden superior toma otra función (o expresión *lambda*) como **parámetro**, **devuelve una función**, **o hace ambos**. Para pasar una función como parámetro de una función de orden superior, se la debe referenciar con el ***operador ``::``*** (ver [*Reflection*](Kotlin/Reflection.md))

```kotlin
    fun calCircumference(radius: Double) = (2 * Math.PI) * radius
    
    fun calArea(radius: Double): Double = (Math.PI) * Math.pow(radius, 2.0)
    
    fun circleOperation(radius: Double, op: (Double) -> Double): Double {
        val result = op(radius)
        return result
    }
    
    print(circleOperation(3.0, ::calArea)) // 28.274333882308138
    print(circleOperation(3.0, calArea)) // No va a compilar
    print(circleOperation(3.0, calArea())) // No va a compilar
    print(circleOperation(6.7, ::calCircumference)) // 42.09734155810323
```

Otro ejemplo, en el que la función de orden superior devuelve una función:

```kotlin
     fun multiplier(factor: Double): (Double) -> Double = { number -> number * factor }
     
     fun main() {
         val doubler = multiplier(2.0)
         print(doubler(5.6)) // 11.2 
     }
```

### 3.7. *Function type with receiver* y *Function literal with receiver*
Partiendo de una función de extensión simple como ejemplo, si se necesita asociar esa función con una propiedad, entonces se puede usar una **referencia de función** (***function reference***) (ver [*Reflection*](Kotlin/Reflection.md)).

```kotlin
    fun Int.square() = this * this // Extension function
    
    val squareFunType: Int.() -> Int =
        Int::square // Function type with receiver, usando function reference
    
    fun main() {
        println(2.square())            // Imprime: 4. En la función square(), this refiere al receiver object (el 2)
        //println(square(2))    	   // Error: Unresolved reference. No se puede invocar una función de extensión pasando el receptor como primer argumento
        println(squareFunType(2))      // Imprime: 4. Se puede invocar una función de extensión pasando el receptor como primer argumento
        println(2.squareFunType())     // Imprime: 4
    }
```

Se ha utilizado ***``Int.() -> Int``*** que es un **tipo función con receptor** (***function type with receiver***). En lugar de usar la referencia, se puede definir la función usando una de las ***function literals with receiver***. Dentro del cuerpo de la *function literal*, el objeto receptor pasado a una llamada se convierte en un ***this*** **implícito**, de modo que se puede acceder a los miembros de ese objeto receptor **sin ningún calificador adicional**, **o** acceder al objeto receptor **utilizando una expresión** ***this***. Este comportamiento es **similar a las funciones de extensión**, que también permiten acceder a los miembros del objeto receptor dentro del cuerpo de la función. A continuación, se muestra un ejemplo de una ***function literal with receiver*** junto con su tipo, donde se llama a la función ***``plus``*** en el objeto receptor:

```kotlin
    val suma: Int.(Int) -> Int =
        { other -> plus(other) } // plus() se llama en el receiver object (el 4). Es decir, this.plus(other)
    println(suma(4, 2))   // Imprime: 6
    println(4.suma(2))  // Imprime: 6
```

La **sintaxis de función anónima** permite especificar el tipo de receptor de una *function literal* directamente. Esto puede ser útil si se necesita declarar una variable de un *function type with receiver* y usarla más tarde.

```kotlin
    val squareFunLiteral = fun Int.(other: Int): Int = this * other // Sintaxis de función anónima
    println(squareFunLiteral(2, 2))   // Imprime: 4
    println(2.squareFunLiteral(2))  // Imprime: 4
```

### 3.8. *Closures*
Una expresión *lambda* o una función anónima (así como una función local y un *object expression*) pueden acceder a su *closure*, es decir, a las **variables declaradas en un ámbito** (***scope***) **externo**. Las variables capturadas en el *closure* pueden ser modificadas en el *lambda*.

```kotlin
    var sum = 0
    ints.filter { it > 0 }.forEach {
        sum += it
    }
    print(sum)
```

### 3.9. *Inline Functions*
Cuando se crean funciones que reciben funciones como argumento, el compilador necesita crear clases anónimas, **consumiendo recursos** por la asignación de memoria extra que precisa. Una forma de evitar esto es marcar la función como ***``inline``***. Al hacer esto, **el compilador internamente copia esa función** por su código en los sitios donde se llame, evitando crear una nueva referencia a la función. Sin embargo, hay que tener en cuenta que al marcar una función como *inline*, el *byte-code* también se hace más grande, por lo que es **altamente recomendable** solo alinear funciones de orden superior **pequeñas que acepten** ***lambdas*** **como parámetros**.

```kotlin
    import android.R.attr.radius
    
    inline fun circleOperation(radius: Double, op: (Double) -> Double) {
        println("Radius is $radius")
        val result = op(radius)
        println("The result is $result")
    }
    
    fun main(args: Array<String>) {
        circleOperation(5.3, { (2 * Math.PI) * it })
    }
    
    public static final void circleOperation(double radius, @NotNull Function1 op) {
        Intrinsics.checkParameterIsNotNull(op, "op");
        String var4 = "Radius is "+radius;
        System.out.println(var4);
        double result =((Number) op . invoke (radius)).doubleValue();
        String var6 = "The result is "+result;
        System.out.println(var6);
    }
    
    // En vez de llamar a la función circleOperation, el compilador copia el código.
    public static final void main(@NotNull String[] args) {
        Intrinsics.checkParameterIsNotNull(args, "args");
        double radius $iv = 5.3 D;
        String var3 = "Radius is "+radius$iv;
        System.out.println(var3);
        double result $iv = 6.283185307179586 D * radius $iv;
        String var9 = "The result is "+result$iv;
        System.out.println(var9);
    }
```

En caso de que haya más de dos parámetros *lambda* en una función, se puede decidir cuál *lambda* **no alinear** usando el modificador ***``noinline``*** sobre el parámetro. Esto es útil especialmente para un parámetro *lambda* que tomará mucho código.

```kotlin
    inline fun myFunction(op: (Double) -> Double, noinline op2: (Int) -> Int) {
        // Do something
    }
```

#### 3.9.1. Parámetros de tipo *reified*
A veces, se necesita acceder a un tipo pasado como parámetro de una función. Normalmente, esto se solventa pasando la clase como parámetro de la función, haciendo el **código más complejo y poco estético**:

```kotlin
     import kotlin.jvm.java
     
     fun <T> TreeNode.findParentOfType(clazz: Class<T>): T? {
          var p = parent
          while (p != null && !clazz.isInstance(p)) {
               p = p.parent
          }
          @Suppress("UNCHECKED_CAST")
          return p as T?
     }
     
     // Utiliza reflexión para comprobar que un nodo es de cierto tipo.
     treeNode.findParentOfType(MyTreeNode::class.java)
```

Marcando un tipo como ***``reified``***, se tendrá la capacidad de **utilizar ese tipo dentro de la función**. Es importante que la función que lo utilice sea *inline*, ya que el código necesita ser sustituido en el lugar desde el que se ejecuta para poder funcionar:

```kotlin
     inline fun <reified T> TreeNode.findParentOfType(): T? {
          var p = parent
          while (p != null && p !is T) {
               p = p.parent
          }
          return p as T?
     }
     
     // Se puede llamar a la función pasándole como parámetro directamente el tipo.
     treeNode.findParentOfType<MyTreeNode>()
```

### 3.10. Tipos genéricos
Como las clases, las funciones también pueden tener **parámetros de tipo** con la sintaxis ***``fun <T> nombreFuncion(parametro: tipo<T>)``***:

```kotlin
    fun <T> mostrarValor(lista: ArrayList<T>) {
        for (elemento in lista) {
            println(elemento)
        }
    }
    
    fun main() {
        val stringLista: ArrayList<String> = arrayListOf<String>("Hola", "Mundo")
        mostrarValor(stringLista)
    
        val floatLista: ArrayList<Float> = arrayListOf<Float>(10.5f, 5.0f, 25.5f)
        mostrarValor(floatLista)
    }
```

Y también se pueden aplicar a **funciones de extensión** de tipo genérico:

```kotlin
    fun <T> ArrayList<T>.mostrarValor() {
        for (elemento in this) {
            println(elemento)
        }
    }
    
    fun main() {
        val stringLista: ArrayList<String> = arrayListOf<String>("Hola", "Mundo")
        stringLista.mostrarValor()
    
        val floatLista: ArrayList<Float> = arrayListOf<Float>(10.5f, 5.0f, 25.5f)
        floatLista.mostrarValor()
    }
```

---

## 4. Colecciones

### 4.1. Descripción
Las colecciones son grupos de un **número variable de elementos** (posiblemente cero) que comparten la importancia del problema que se está resolviendo y se operan de forma común.  
Los siguientes tipos de colección son relevantes para Kotlin: ***List***, ***Set*** y ***Map*** (o ***Dictionary***). Kotlin permite manipular colecciones independientemente del tipo exacto de objetos almacenados en ellas, ya que la librería estándar de Kotlin ofrece *interfaces*, clases y funciones genéricas para crear, completar y administrar colecciones de cualquier tipo. Un par de *interfaces* representan cada tipo de colección: una ***read-only*** y una ***mutable***.

La interfaz de **solo lectura** (***read-only***) proporciona operaciones para **acceder a los elementos** de la colección.  
La interfaz ***mutable*** extiende la correspondiente interfaz de solo lectura con **operaciones de escritura: agregar, eliminar y actualizar** sus elementos. Alterar una colección mutable **no requiere que sea una** ***var***: las operaciones de escritura modifican el mismo objeto de colección mutable, por lo que la referencia no cambia. Sin embargo, si se intenta **reasignar una colección** ***val***, se obtendrá un **error de compilación**.

```kotlin
     val numbers = mutableListOf("one", "two", "three", "four")
     numbers.add("five")   // this is OK    
     //numbers = mutableListOf("six", "seven")      // compilation error
```

### 4.2. ``Collection<T>``
Es la raíz de la jerarquía de colecciones, que hereda de la interfaz ***``Iterable<T>``***. Se puede utilizar *Collection* como un parámetro de una función que se aplica a diferentes tipos de *Collection* (***List*** **o** ***Set***, ya que ***Map*** **no hereda de esta interfaz**).  
***``MutableCollection<E>``***: es una *Collection* **con operaciones de escritura**, como agregar y quitar.

### 4.3. ``List<T>``
Es una **colección ordenada** con acceso a los elementos a través de **índices**. Los elementos pueden aparecer **más de una vez** en una lista.  
***``MutableList<T>``***: es una *List* **con operaciones de escritura específicas** de *List*, por ejemplo, agregar o eliminar un elemento en una posición específica. A pesar de la similitud con los *arrays* de Java, hay una **diferencia importante: el tamaño de un** ***array*** **se define en la inicialización y nunca se cambia**. Y a su vez, un ***array*** **es mutable** (se puede cambiar a través de cualquier referencia).

La **implementación predeterminada** de *List* es ***ArrayList***, que se puede considerar como un *array* de tamaño variable.

```kotlin
     val array = arrayOf(1, 2, 3) // Array de Java (tamaño fijo; mutable)
     array[0] = array[1]
     println(array[0]) // 2
     
     val lista = listOf(1, 2, 3) // Lista (tamaño fijo; solo lectura)
     lista[0] = lista[1] // No compila porque List es de solo lectura 
     
     val arrayList = arrayListOf(1, 2, 3) // ArrayList (tamaño variable; mutable)
     arrayList[0] = arrayList[1]
     println(arrayList[0]) // 2
     
     val mutableList = mutableListOf(1, 2, 3) // MutableList (tamaño variable; mutable)
     mutableList[0] = mutableList[1]
     println(mutableList[0]) // 2
```

### 4.4. ``Set<T>``
Es una **colección de elementos únicos**. Generalmente, **el orden** de los elementos **no tiene importancia**. Asimismo, los elementos nulos también son únicos (no puede haber dos elementos *null*).  
***``MutableSet<E>``***: es un *Set* con operaciones de escritura de *MutableCollection*. La **implementación predeterminada** de *Set* (***LinkedHashSet***) conserva el orden de inserción de los elementos. Por lo tanto, las funciones que dependen del orden, como ***``first()``*** o ***``last()``***, devuelven resultados predecibles en dichos *sets*. Una **implementación alternativa**, ***HashSet***, no dice nada sobre el orden de los elementos, por lo que llamar a dichas funciones devuelve resultados impredecibles. Sin embargo, ***HashSet*** **requiere menos memoria** para almacenar la misma cantidad de elementos.

```kotlin
     val numbers = setOf(1, 2, 3, 4)  // LinkedHashSet is the default implementation
     val numbersBackwards = setOf(4, 3, 2, 1)
     
     println("The sets are equal: ${numbers == numbersBackwards}") // The sets are equal: true
     println(numbers.first() == numbersBackwards.first()) // false
     println(numbers.first() == numbersBackwards.last())  // true
```

### 4.5. *Map* (o *Dictionary*)
``Map <K, V>`` no hereda de la interfaz ``Collection``; sin embargo, también es un tipo de colección de Kotlin. Un *map* almacena ***pares clave-valor*** o **entradas** (***key-value pairs*** o ***entries***); las **claves son únicas**, pero se pueden emparejar **diferentes claves con valores iguales** (repetidos). La interfaz *Map* proporciona funciones específicas, como el acceso al valor por clave, la búsqueda de claves y valores, etc. Además, como se accede a los valores a través de sus claves, el orden de los pares no importa, por lo que dos *maps* que contienen pares iguales son iguales independientemente del orden de los pares.

```kotlin
    val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key4" to 1)
    val anotherMap = mapOf("key2" to 2, "key1" to 1, "key4" to 1, "key3" to 3)
    
    println("The maps are equal: ${numbersMap == anotherMap}")
    println("All keys: ${numbersMap.keys}")
    println("All values: ${numbersMap.values}")
    if ("key2" in numbersMap) println("Value by key \"key2\": ${numbersMap["key2"]}")
    if (1 in numbersMap.values) println("The value 1 is in the map")
    if (numbersMap.containsValue(1)) println("The value 1 is in the map") // same as previous
    
    // The maps are equal: true
    // All keys: [key1, key2, key3, key4]
    // All values: [1, 2, 3, 1]
    // Value by key "key2": 2
    // The value 1 is in the map
    // The value 1 is in the map
```

***``MutableMap<K, V>``*** es un *Map* con operaciones de escritura de *maps*, por ejemplo, se puede agregar un nuevo par clave-valor o actualizar el valor asociado con la clave dada. La **implementación predeterminada** de *Map* (***LinkedHashMap***) conserva el orden de inserción de los elementos al iterar el *map*. A su vez, una **implementación alternativa**, ***HashMap***, no dice nada sobre el orden de los elementos.

```kotlin
     val numbersMap = mutableMapOf("one" to 1, "two" to 2)
     numbersMap.put("three", 3)
     numbersMap["one"] = 11
     
     println(numbersMap) // {one=11, two=2, three=3}
```

### 4.6. Construcción de colecciones
La forma más común de crear una colección es con las funciones de librería estándar ***``listOf<T>()``***, ***``setOf<T>()``***, ***``mutableListOf<T>()``***, ***``mutableSetOf<T>()``***. Lo mismo está disponible para *maps* con las funciones ***``mapOf()``*** y ***``mutableMapOf()``***. Las claves y los valores del *map* se pasan como objetos de tipo ***``Pair``*** (generalmente creados con la función ***infix*** ***``to``***).

```kotlin
    val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key4" to 1)
```

Para evitar el uso excesivo de memoria, se pueden utilizar formas alternativas. Por ejemplo, se puede crear un *mutable map* y poblarlo usando las operaciones de escritura. La función ***``apply()``*** puede ayudar a mantener la inicialización fluida.

```kotlin
    val numbersMap = mutableMapOf<String, String>().apply { this["one"] = "1"; this["two"] = "2" }
```

#### 4.6.1. Colecciones vacías
También hay funciones para crear colecciones sin ningún elemento: ***``emptyList()``***, ***``emptySet()``*** y ***``emptyMap()``***. Al crear colecciones vacías, se debe **especificar el tipo de elementos** que contendrá la colección.

```kotlin
    val empty = emptyList<String>()
```

#### 4.6.2. Inicializar listas
Para las **listas**, hay un constructor que toma el **tamaño de la lista** y la **función inicializadora** que define el valor del elemento en función de su índice.

```kotlin
     val doubled = List(3, { it * 2 })  // or MutableList 
     println(doubled) // [0, 2, 4]
```

### 4.7. Copia de colecciones
Las funciones de copia de colecciones, como ***``toList()``***, ***``toMutableList()``***, ***``toSet()``*** y otras, crean una nueva colección con los mismos elementos. **Si se agregan o eliminan elementos a la colección original, esto no afectará a las copias**. Por su parte, las copias también pueden cambiarse **independientemente de la colección original**.

```kotlin
    val sourceList = mutableListOf(1, 2, 3)
    val copyList = sourceList.toMutableList()
    val readOnlyCopyList = sourceList.toList()
    sourceList.add(4)
    println("Source size: ${sourceList.size}") // Source size: 4
    println("Copy size: ${copyList.size}")     // Copy size: 3
    
    //readOnlyCopyList.add(4)  // compilation error
    println("Read-only copy size: ${readOnlyCopyList.size}") // Read-only copy size: 3
```

Alternativamente, se pueden crear **nuevas referencias a la misma instancia de la colección**. Se crean nuevas referencias cuando se inicializa una variable de una colección con una colección existente. Entonces, **cuando la instancia de la colección se modifica** a través de una referencia, **los cambios se reflejan en todas sus referencias**.

```kotlin
    val sourceList = mutableListOf(1, 2, 3)
    val referenceList = sourceList
    referenceList.add(4)
    println("Source size: ${sourceList.size}") // Source size: 4
```

### 4.8. *Iterators*
Los iteradores (objetos que brindan acceso a los elementos secuencialmente sin exponer la estructura subyacente de la colección), se pueden obtener si se **hereda a la interfaz** ***``Iterable<T>``***, incluidos *Set* y *List*, llamando a la función ***``iterator()``***. Una vez obtenido, apunta al primer elemento de la colección; llamando a la función ***``next()``*** se retorna este elemento y se mueve el *iterator* al próximo elemento, si es que existe. Una vez que el *iterator* recorre el último elemento, **ya no se puede usar** para recuperar más elementos o resetearlo a posiciones previas. Si se quiere iterar la colección otra vez, **se debe crear un nuevo** ***iterator***. El **ciclo** ***for*** obtiene un *iterator* **implícitamente**. Al igual que con la función ***``forEach()``***, que permite iterar una colección automáticamente y ejecutar un código dado para cada elemento.

```kotlin
    val numbers = listOf("one", "two", "three", "four")
    numbers.forEach {
     println(it)
    }
```

#### 4.8.1. *ListIterator*
Es una implementación especial para las listas, que le **permite iterar hacia adelante y atrás**, por lo que puede usarse luego de que llega al último elemento. Para la itereación hacia atrás se usan las **fuciones** ***``hasPrevious()``*** y ***``previous()``***. Además, proporciona información sobre los índices de elementos con las **funciones** ***``nextIndex()``*** y ***``previousIndex()``***.

#### 4.8.2. *MutableIterator*
Al iterar **colecciones mutables**, se puede utilizar la función ***``remove()``*** para borrar elementos mientras se itera la colección. Además, con ***MutableListIterator*** se puede remplazar (***``set()``***) o insertar (***``add()``***) elementos mientras se itera la lista.

```kotlin
     val numbers = mutableListOf("one", "four", "four") 
     val mutableListIterator = numbers.listIterator()
     
     mutableListIterator.next()
     mutableListIterator.add("two")
     mutableListIterator.next()
     mutableListIterator.set("three")   
     println(numbers) // [one, two, three, four]
```

### 4.9. Rangos y progresiones
Se pueden crear rangos usando la función ***``rangeTo()``*** (o en su forma de operador **``..``** ). Dichos rangos se utilizan para iterar, generalmente en ciclos *for*. Para iterar números en **orden inverso**, se usa la función ***``downTo``***. También se puede especificar el paso con la función ***``step``***. Para iterar un rango de números que **no incluya el último elemento**, se utiliza la función ***``until``***.

#### 4.9.1. Rangos
Un rango define un intervalo cerrado en el sentido matemático: se define por sus dos valores extremos que están incluidos en el rango. La operación principal en rangos es ***``contains``***, que generalmente se usa en forma de **operadores** ***``in``*** y ***``!in``***.

#### 4.9.2. Progresiones
Rangos de tipo integral (***IntRange***, ***LongRange***, ***CharRange***) pueden tratarse como progresiones aritméticas. En Kotlin, estas progresiones se definen mediante tipos especiales: ***IntProgression***, ***LongProgression*** y ***CharProgression***. Las progresiones tienen tres propiedades esenciales: el primer elemento (***``first``***), el último elemento (***``last``***) y un paso distinto de cero (***``step``***). Cuando se crea una progresión implícitamente al iterar un rango, el primer y último elemento de esta progresión son los extremos del rango, y el paso por defecto es 1.

### 4.10. ``Sequence<T>``
Las **secuencias** (paquete *kotlin.sequences*) ofrecen las mismas funciones que un *Iterable* pero implementan otro enfoque para el procesamiento de colecciones de varios pasos. Cuando el procesamiento de un *Iterable* incluye varios pasos, se ejecutan de manera ***eagerly***: cada paso de procesamiento se completa y devuelve su resultado (una **colección intermedia**). Y el siguiente paso se ejecuta en esta colección intermedia. A su vez, el procesamiento de múltiples pasos de secuencias se ejecuta de manera ***lazy*** cuando es posible: la computación real ocurre **solo cuando se solicita el resultado de toda la cadena** de procesamiento. El orden de ejecución de las operaciones también es diferente: ***Sequence*** realiza **todos los pasos de procesamiento** uno por uno **para cada elemento**, mejorando el rendimiento. En cambio, ***Iterable*** completa **cada paso de procesamiento para toda la colección** y luego **procede al siguiente paso**. Hay que tener en cuenta que al procesar colecciones más pequeñas o al realizar cálculos más simples, no es conveniente usar *Sequences* por el gasto que produce su ejecución *lazy*.

#### 4.10.1. Construcción
Para crear una secuencia, se puede llamar a la función ***``sequenceOf()``*** que enumera los elementos como sus argumentos.

```kotlin
    val numbersSequence = sequenceOf("four", "three", "two", "one")
```

Si ya se tiene un objeto *Iterable* (como un *List* o un *Set*), se puede crear una secuencia a partir de él llamando a ***``asSequence()``***.

```kotlin
     val numbers = listOf("one", "two", "three", "four")
     val numbersSequence = numbers.asSequence()
```

Una forma más de crear una secuencia es construyéndola ***con una función que calcula sus elementos***. Para construir una secuencia basada en una función, se llama a ***``generateSequence()``*** con esta función como argumento. Opcionalmente, se puede especificar el primer elemento como un valor explícito o como resultado de una llamada de función. La generación de la secuencia se detiene cuando la función proporcionada devuelve un valor nulo.

```kotlin
     val oddNumbers = generateSequence(1) { it + 2 } // `it` is the previous element
     println(oddNumbers.take(5).toList()) // [1, 3, 5, 7, 9]
     //println(oddNumbers.count())     // error: the sequence is infinite
     
     val oddNumbersLessThan10 = generateSequence(1) { if (it < 10) it + 2 else null }
     println(oddNumbersLessThan10.count()) // 6
```

Finalmente, hay una función que permite producir elementos de secuencia uno por uno o por trozos de tamaños arbitrarios: la función ***``sequence()``***, que toma una expresión *lambda* que contiene llamadas a las **funciones** ***``yield()``*** y ***``yieldAll()``***. Estas funciones devuelven un elemento al consumidor de la secuencia y **suspenden la ejecución de** ***sequence()*** hasta que el consumidor solicite el siguiente elemento. ***yield()*** toma un solo elemento como argumento; ***yieldAll()*** puede tomar un objeto *Iterable*, un *Iterador* u otro *Sequence*. Un argumento *Sequence* de *yieldAll()* puede ser infinito. En ese caso, dicha llamada debe ser la última: todas las llamadas posteriores nunca se ejecutarán.

```kotlin
     val oddNumbers = sequence {
     yield(1)
     yieldAll(listOf(3, 5))
     yieldAll(generateSequence(7) { it + 2 })
     }
     println(oddNumbers.take(5).toList()) // [1, 3, 5, 7, 9]
```

#### 4.10.2. *Sequence operations*
Las operaciones de secuencia se pueden clasificar en los siguientes grupos con respecto a sus requisitos de estado: ***stateless*** y ***stateful***.  
- ***Stateless***: no requieren ningún estado y procesan cada elemento de forma independiente, por ejemplo, ***``map()``*** o ***``filter()``***. Las operaciones sin estado también pueden requerir una pequeña cantidad constante de estado para procesar un elemento, por ejemplo, ***``take()``*** o ***``drop()``***.  
- ***Stateful***: requieren una cantidad significativa de estado, generalmente proporcional al número de elementos en una secuencia.

Si **una operación de secuencia devuelve otra secuencia**, que se produce de forma *lazy*, se llama ***intermedia***. **De lo contrario, la operación es** ***terminal***. Ejemplos de operaciones terminales son ***``toList()``*** o ***``sum()``***. Los elementos de secuencia se pueden recuperar solo con operaciones terminales.

### 4.11. Operaciones
Las operaciones de colección se declaran en la librería estándar de dos formas: **funciones miembro** de interfaces de colección y **funciones de extensión**. Las funciones miembro definen operaciones que son esenciales para un tipo de colección. Por ejemplo, *Collection* contiene la función ***``isEmpty()``*** para verificar si está vacío; *List* contiene ***``get()``*** para el acceso por índice a los elementos, y así sucesivamente. Otras operaciones de colección se declaran como funciones de extensión. Se trata de funciones de filtrado, transformación, ordenación y otras funciones de procesamiento de colecciones.  
A su vez, cada tipo de colección tiene su propio conjunto de **operaciones específicas** o que se ejecutan de una manera específica según el tipo de colección.  
Así, *List* tiene ***``get()``*** (o su forma corta con los **``[ ]``**), ***``getOrElse()``***, ***``getOrNull()``***, ***``subList()``***, **``indexOf()``**, ***``lastIndexOf()``***, ***``binarySearch()``***, ***``add()``***, ***``addAll()``***, ***``set()``*** (o su forma corta con los **``[ ]``**), ***``fill()``*** (reemplaza todos los elementos de la colección con el valor especificado), ***``removeAt()``***, entre otros;  
*Set* tiene ***``union()``*** (puede ser usada en su forma *infix*: ***`a union b`***), ***``intersect()``*** (puede ser usada en su forma *infix*: ***``a intersect b``***), ***``substract()``*** (puede ser usada en su forma *infix*: ***``a substract b``***);  
y *Map* tiene ***``get()``*** (o su forma corta con los **``[ ]``**), ***``getValue()``***, ***``getOrElse()``***, ***``getOrDefault()``***, ***``filter()``*** (al que se le pasa un predicado con un *Pair* como argumento), ***``filterKeys()``***, ***``filterValues()``***, ***``plus``*** (**``+``**) y  ***``minus``*** (**``-``**) (trabajan diferente que en otras colecciones), ***``put()``*** (agrega un nuevo par clave-valor a un mapa mutable, sobreescribiendo los valores si las claves dadas ya existen en el mapa), ***``putAll()``*** (añade múltiples entradas a la vez, sobreescribiendo los valores si las claves dadas ya existen en el mapa), ***``remove()``*** (si se especifica tanto la clave como el valor, el elemento con esta clave se eliminará solo si su valor coincide con el segundo argumento).

#### 4.11.1. Operaciones comunes
Las operaciones comunes están disponibles **tanto para colecciones de solo lectura como mutables**. Estas operaciones devuelven sus resultados **sin afectar la colección original**. Los resultados de tales operaciones deben almacenarse en variables o usarse de alguna otra manera, por ejemplo, pasarse a otras funciones. Para ciertas operaciones de colección, existe una opción para especificar el **objeto de destino**. El destino es una colección mutable a la que la función agrega sus elementos resultantes en lugar de devolverlos en un nuevo objeto. Para realizar operaciones con destinos, hay funciones separadas con el **sufijo** ***To*** en sus nombres, por ejemplo, ***``filterTo()``*** en lugar de ***``filter()``*** o ***``associateTo()``*** en lugar de ***``associate()``***. Estas funciones toman la **colección de destino como parámetro adicional**.

```kotlin
     val numbers = listOf("one", "two", "three", "four")
     val filterResults = mutableListOf<String>()  //destination object
     numbers.filterTo(filterResults) { it.length > 3 }
     numbers.filterIndexedTo(filterResults) { index, _ -> index == 0 }
     println(filterResults) // contains results of both operations
```

- Ver [***Transformations***](Kotlin/Colecciones/Transformations.md)
- Ver [***Filtering***](Kotlin/Colecciones/Filtering.md)
- Ver [***plus and minus operators***](Kotlin/Colecciones/plus%20and%20minus%20operators.md)
- Ver [***Grouping***](Kotlin/Colecciones/Grouping.md)
- Ver [***Retrieving collection parts***](Kotlin/Colecciones/Retrieving%20collection%20parts.md)
- Ver [***Retrieving single elements***](Kotlin/Colecciones/Retrieving%20single%20elements.md)
- Ver [***Ordering***](Kotlin/Colecciones/Ordering.md)
- Ver [***Aggregate operations***](Kotlin/Colecciones/Aggregate%20operations.md)

#### 4.11.2. Operaciones de escritura
Para colecciones mutables, hay operaciones de escritura que **cambian el estado de la colección**. Dichas operaciones incluyen agregar, eliminar y actualizar elementos. Para ciertas operaciones, existen pares de funciones para realizar **la misma operación**: una aplica la operación *in situ* y la otra devuelve el resultado como una colección separada. Por ejemplo, ***``sort()``*** ordena una colección mutable en el lugar, por lo que su estado cambia; ***``sorted()``*** crea una nueva colección que contiene los mismos elementos de forma ordenada.

- Ver [***Adding elements***](Kotlin/Colecciones/Adding%20elements.md)
- Ver [***Removing elements***](Kotlin/Colecciones/Removing%20elements.md)

---

## 5. Hilos, Corrutinas (*Coroutines*) y *Flows*

- Ver [***Hilos, Coroutines & Flows***](Kotlin/Hilos,%20Coroutines%20&%20Flows.md)

---

## 6. Otros constructos del lenguaje

### 6.1. *Operator overloading*
La sobrecarga de operadores permite asignar comportamientos a los operadores **para que actúen sobre tipos creados por el programador**. Para implementar un operador, se proporciona una **función miembro** o una **función de extensión** con un nombre fijo, para el tipo correspondiente. Las funciones que **sobrecargan** operadores necesitan ser marcadas con el **modificador** ***``operator``***.  
El siguiente ejemplo, muestra una clase *Counter* que comienza con un valor dado y puede ser incrementado usando el operador sobrecargado ***``plus``*** (**``+``**):

```kotlin
    data class Counter(val dayIndex: Int) {
        operator fun plus(increment: Int): Counter {
            return Counter(dayIndex + increment)
        }
    }
    
    fun main() {
        val counter = Counter(2)
    
        println(counter + 3)  // Counter(dayIndex=5)
    }
```

### 6.2. *Destructuring declarations*
A veces es conveniente **desestructurar un objeto** en varias variables. Una declaración de desestructuración **crea múltiples variables a la vez** que luego pueden usarse de forma **independiente**. Se compilan utilizando **funciones** ***``componentN()``***, que algunas estructuras implementan por defecto, como las ***data classes*** y las **colecciones** (estas últimas, como **funciones de extensión**). 
Si una variable no se necesita, se puede colocar un guión bajo (**``_``**) en vez del nombre.

```kotlin
    data class Data(val name: String, val age: Int)
    
    // Una función que retorna un objeto Data (dos valores)
    fun sendData(): Data {
        return Data("Jack", 30)
    }
    
    fun main() {
        val obj = sendData()
    // Usando la instancia para acceder a las propiedades
        println("Name is ${obj.name}")
        println("Age is ${obj.age}")
    
    // Creando dos variables usando destructuring declaration
        val (name, age) = sendData()
        println("Name is $name")
        println("Age is $age")
    }
    
    // Internamente, se compila:
    val name = obj.component1()
    val age = obj.component2()
```

### 6.3. *Type checks* y *Casts*
Los **operadores** ***``is``*** y ***``!is``*** sirven para **comprobar, en tiempo de ejecución, si un objeto es de un tipo determinado o no**. Esto, que en Kotlin se conoce como ***Smart Cast***, permite que el compilador inserte un *casting* seguro cuando se necesite:

```kotlin
     if (obj is String) {
         println(obj.length) // obj se castea automáticamente a String
     }
```

*Smart cast* no funciona si el compilador **no puede garantizar que la variable no cambie entre la comprobación y su uso**. Específicamente, se aplica a los siguientes casos:  
• **Variables locales** **``val``**: siempre, excepto para propiedades delegadas locales.  
• **Propiedades** **``val``**: si la propiedad es privada, o interna o si la comprobación se realiza en el mismo módulo en el que la propiedad se declara. No se aplica a propiedades ***``open``*** o a las que tienen *getters* personalizados.  
• **Variables locales** **``var``**: si la variable no se modifica entre la comprobación y el uso, si no es capturada en un *lambda* que la modifique y si no es una propiedad delegada local.  
• **Propiedades** **``var``**: Nunca (la variable se puede modificar en cualquier momento por otro código.)

Para realizar *castings* **explícitos**, Kotlin utiliza los **operadores** ***``as``*** y ***``as?``***. La diferencia entre ambos es que ***as*** es “inseguro” (arroja una ***ClassCastException*** si el *cast* no es posible), mientras que ***as?*** es “seguro” o anulable (en caso de una falla, retorna *null* en lugar de una excepción).

```kotlin
    val x: String = y as String  // Si y es nulo, arroja una excepción
    val x: String = y as? String  // Si y es nulo, retorna null
```

### 6.4. *This expression*
Para denotar el receptor (*receiver*) actual, se usan **expresiones con** ***``this``***: en un miembro de una clase, *this* refiere al objeto actual de esa clase; en una función de extensión o una *function literal* con receptor, *this* denota el parámetro receptor (*receiver*) que se pasa en el lado izquierdo del punto.  
Si *this* no tiene calificadores, se refiere al ámbito de inclusión más interno. Para hacer referencia a *this* en otros ámbitos, se utilizan calificadores de etiqueta (*labels*).

```kotlin
    class A { // implicit label @A
        inner class B { // implicit label @B
            fun Int.foo() { // implicit label @foo
                val a = this@A // A's this
                val b = this@B // B's this
    
                val c = this // foo()'s receiver, an Int
                val c1 = this@foo // foo()'s receiver, an Int
    
                val funLit = lambda@ fun String.() {
                    val d = this // funLit's receiver
                }
    
                val funLit2 = { s: String ->
                    // foo()'s receiver, since enclosing lambda expression
                    // doesn't have any receiver
                    val d1 = this
                }
            }
        }
    }
```

Cuando se llama a una función miembro, se puede omitir el **``this.``**. Si se tiene una función no miembro con el mismo nombre, se debe utilizar con precaución, porque en algunos casos se puede llamar la equivocada.

```kotlin
    fun printLine() {
        println("Top-level function")
    }
    
    class A {
        fun printLine() {
            println("Member function")
        }
    
        fun invokePrintLine(omitThis: Boolean = false) {
            if (omitThis) printLine()
            else this.printLine()
        }
    }
    
    A().invokePrintLine() // Member function
    A().invokePrintLine(omitThis = true) // Top-level function
```

### 6.5. *Null safety*
En Kotlin, el sistema de tipos distingue entre referencias que pueden contener nulos (***nullable references***) y aquellas que no pueden (***non-null references***). Para permitir valores nulos, se puede declarar una variable como anulable con el **operador de llamada seguro** **``?``** (***safe call operator***). Se puede, por ejemplo, colocar una llamada segura en el lado izquierdo de una asignación. Luego, si uno de los receptores en la cadena de llamadas seguras es nulo, la asignación se omite y la expresión de la derecha no se evalúa en absoluto:

```kotlin
    // If either 'person' or 'person.department' is null, the function is not called:
    person?.department?.head = managersPool.getManager()
```

Cuando se tiene una referencia *b* que acepta valores nulos, se puede decir "si *b* no es nulo, utilícelo; de lo contrario, use algún valor no nulo". En lugar de usar una **expresión** ***if*** completa, esto se puede expresar con el operador ***Elvis*** **``?:``**

```kotlin
    // Cumplen el mismo objetivo
    val l: Int = if (b != null) b.length else -1
    val l = b?.length ?: -1
```

Si la expresión a la izquierda de **``?:``** no es nula, el operador *Elvis* la devuelve; de lo contrario, devuelve la expresión de la derecha. Es decir, la expresión del lado derecho se evalúa solo si el lado izquierdo es nulo. Y también, dado que ***``throw``*** y ***``return``*** son expresiones en Kotlin, también se pueden usar en el lado derecho del operador *Elvis*. Esto puede ser muy útil, por ejemplo, para verificar los argumentos de la función:

```kotlin
    fun foo(node: Node): String? {
        val parent = node.getParent() ?: return null
        val name = node.getName() ?: throw IllegalArgumentException("name expected")
    // Whatever...
    }
```

El **operador de aserción no nulo** **``!!``** convierte cualquier valor en un tipo no nulo y lanza una excepción si el valor es nulo.

```kotlin
    val l = b!!.length // Arrojará un NPE si b es nulo
```

Las **conversiones regulares** (***cast***) pueden resultar en una ***ClassCastException*** **si el objeto no es del tipo de destino**. Otra opción es usar **conversiones seguras** (***safe casts***) que devuelvan un valor nulo si el intento no tuvo éxito (ver *Type Checks y Casts* en apartado 6.3):

```kotlin
    val aInt: Int? = a as? Int
```

Si se tiene una colección de elementos de un tipo que acepta valores nulos y se desea filtrar elementos no nulos, se puede utilizar la función ***``filterNotNull``***:

```kotlin
    val nullableList: List<Int?> = listOf(1, 2, null, 4)
    val intList: List<Int> = nullableList.filterNotNull()
    println(intList.joinToString()) // Imprime: 1, 2, 4
```

### 6.6. *SAM* (*Single Abstract Method*) *conversions*
Esta característica sólo funciona para **interoperar con Java**. Las ***functions literals*** (*lambdas* y funciones anónimas) de Kotlin pueden ser **convertidas** automáticamente en **implementaciones de ***interfaces*** funcionales** (*interfaces* con un único método abstracto) de Java. Y como tiene un único método abstracto, **se puede omitir el nombre al implementarlo**:

```kotlin
    import java.util.*
    
    fun getList(): List<Int> {
        val arrayList = arrayListOf(1, 5, 2)
        // El método sort pide como parámetros un objeto de tipo List<T> (el array)
        // y un objeto de tipo Comparator<T>, interface funcional que pide implementar 
        // el método compare(obj1: T, obj2: T).
        // Al convertir la función compare() a expresión lambda, el nombre se omite.
        Collections.sort(arrayList, { x, y -> y - x })
        return arrayList
    }
```

### 6.7. *Reflection*

- Ver [*Reflection*](Kotlin/Reflection.md)

### 6.8. *Scope functions*

- Ver [*Scope functions*](Kotlin/Scope%20Functions.md)

---

## Referencias

- [Kotlin Documentation](https://kotlinlang.org/docs/reference/)
- [Learn Kotlin by Example](https://play.kotlinlang.org/byExample/overview)
- [Kotlin Koans](https://play.kotlinlang.org/koans/overview)
- [DevExperto](https://devexperto.com/)
- [Kotlin From Scratch](https://code.tutsplus.com/series/kotlin-from-scratch--cms-1209)
- [Kotlin programmer dictionary](https://blog.kotlin-academy.com/kotlin-programmer-dictionary-2cb67fff1fe2)
- [Kotlin Doc Blogspot - Kotlin y Android](https://kotlindoc.blogspot.com/)
