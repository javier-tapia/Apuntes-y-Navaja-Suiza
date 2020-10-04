# Apuntes de Kotlin  

### Contenido general:  
#### 0. Modificadores de visibilidad
#### 1. Clases y Objetos
#### 2. Propiedades y *fields*
#### 3. Funciones
#### 4. Corrutinas (*Coroutines*)
#### 5. Colecciones
#### 6. Otros constructos del lenguaje
#### Referencias

---
---



### 0. Modificadores de visibilidad: *private* , *protected* , *internal* y *public* (por defecto)

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
class Algo private constructor(arg: Int) { ... }
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
    fun imprimirPrivada(){
        println(privada)
    }
}

class HeredaEjemplo: Ejemplo() {
    fun imprimirProtected(){
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



### 1. Clases y Objetos  
   - 1.1. ***Declaración***:

     ```kotlin
     class Ejemplo (nombre: String): ClaseAbierta(), Interface {}
     ```

       - 1.1.1. Los ***dos puntos*** indican que hereda de otra clase (solo una) o *interfaces* (pueden ser más de una), separadas por comas.

       - 1.1.2. Entre paréntesis, se pueden requerir los parámetros o las propiedades del ***constructor primario***.

       - 1.1.3. Para crear una instancia de una clase (un ***objeto***), no se utiliza la palabra reservada *new* utilizada en Java:

         ```kotlin
         val customer = Customer("Joe Smith")
         ```

       - 1.1.4. Para poder ***heredar*** una clase y ***sobreecribir*** (***override***) uno de sus miembros, tanto la clase como el miembro deben tener el modificador ``open``.

         ```kotlin
         open class Ejemplo {
             open fun saludar() = println("Hola mundo")
         }

         class EjemploDos: Ejemplo() {
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

   - 1.2. **Clases de enumeración** (***enum class***): El uso más básico de las clases *enum* es implementar enumeraciones con ***seguridad de tipo*** (***type-safe***). Cada constante *enum* es una **instancia** de la clase *enum*, y se declaran separadas con comas. Dado que cada elemento es una instancia de la clase *enum*, también se pueden inicializar haciendo referencia a una propiedad definida en el constructor (un **valor**). Y además de valor, pueden tener **comportamiento**, de dos maneras: definiendo sus propios **métodos** (abstractos o no) o implementando ***interfaces***.

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

   - 1.3. **Clases selladas** (***sealed class***): Permiten tener varias **clases heredadas** (se convierten en un tipo de “*enum* de clases”) y representar jerarquías de **clases restringidas** en las que un valor debe tener uno de los tipos de un conjunto limitado, pero no puede tener ningún otro tipo. Cada clase tiene sus propiedades, además de las heredadas, es decir, una subclase de una clase sellada puede tener múltiples instancias que pueden contener estado. Se utilizan mucho para realizar **comprobaciones de tipo** (***type-checks***) en expresiones y sentencias con ``when``:

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

   - 1.4. **Clases abstractas**: Una clase y algunos de sus miembros pueden ser declarados como ***abstract***. Un miembro abstracto **no tiene una implementación en su clase**. Tampoco se necesita anotar una clase o función abstracta con ***open*** (se sobreentiende que es así). Además, **no se puede crear una instancia** de una clase abstracta.

     ```kotlin
     abstract class ClaseAbstracta {
         var nombre: String = "Nati"
         abstract fun saludar()
     }

     class Nombre(): ClaseAbstracta() {
         override fun saludar(){
             println("Hola $nombre")
         }
     }

     fun main() {
         var obj = Nombre()
         obj.saludar() // Imprime 'Hola Nati'
     }
     ```

   - 1.5. ***Interfaces***: Pueden contener **tanto declaraciones de métodos abstractos como implementaciones de métodos por defecto** (se diferencian porque unos no tienen la implementación y los otros sí; no hace falta usar la palabra reservada *default* como lo hace Java). La o las clases que implementen la interfaz, **deben implementar todo** lo que la interfaz tenga declarado. Lo que las hace diferentes de las clases abstractas es que **las interfaces no pueden contener estado**. Pueden tener propiedades, pero deben ser abstractas o proporcionar implementaciones de acceso (*getters* y *setters*).

     ```kotlin
     interface EjemploInterface {
         fun saludar()
     }

     class ClaseA: EjemploInterface {
         override fun saludar() = println("Hola")
     }

     class ClaseC: EjemploInterface {
         override fun saludar() = println("Chau")
     }

     class ClaseB(var tipo: EjemploInterface) {
         var saludo = when(tipo) {
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

       - 1.5.1. Si una clase implementa dos *interfaces* que tienen un método no abstracto con el mismo nombre, al llamar a ese método desde la clase existe un **conflicto de nombres** y el compilador generará un error. Este problema se puede resolver proporcionando una nueva implementación del método:

         ```kotlin
         interface A {
             fun llamar() {
                 println("Desde interface A")
             }
         }

         interface B  {
             fun llamar() {
                 println("Desde interface B")
             }
         }

         class ClaseImp: A, B {
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

   - 1.6. **Tipos genéricos**: permiten definir clases, métodos (ver el apartado de genéricos de *Funciones*) y propiedades de manera que se puede **acceder a ellos utilizando diferentes tipos**. Es común utilizarlos en combinación con las **colecciones** para crear estructuras cuyos elementos no están limitados a un tipo determinado. 
     Una clase o un método de tipo genérico se declara con **parámetros de tipo** (o **tipo parametrizado**) entre corchetes angulares, y al crear una instancia de esa clase se debe proporcionar el tipo real del argumento (salvo que pueda ser inferido), por ejemplo:

     ```kotlin
     class Caja<T>(t: T) {
         var valor = t
     }

     fun main() {
         val caja1 = Caja<Int>(1) // val caja1: Caja<Int> = Caja<Int>(1)
         val caja2 = Caja(1) // Inferencia del tipo
         val caja3 = Caja<Int>("Hola") // ERROR. Type mismatch: inferred type is String but Int was expected
         val caja3 = Caja("Hola")
         val caja4 = Caja(listOf('a', 'b', 'c', 'd'))
         
         println(caja1.valor) // 1
         println(caja2.valor) // 1
         println(caja3.valor) // Hola
         println(caja4.valor) // [a, b, c, d]
     }
     ```

     El uso de genéricos conlleva una serie de ventajas, como son la **seguridad de tipo** (la comprobación se realiza en tiempo de compilación para evitar problemas durante la ejecución) y que **no requiere la conversión entre tipos** ni encasillar el objeto en un tipo determinado.

       - 1.6.1. ***``invariant``***: cuando el parámetro de tipo no tiene modificadores de varianza (***``out``*** o ***`in`***), por defecto es invariante. Significa que **no hay relación entre dos tipos** generados por una clase genérica. Por ejemplo, no hay relación entre *``Cup<Int>``* y *``Cup<Number>``*, *``Cup<Any>``* o *``Cup<Nothing>``*.

         ```kotlin
         class Cup<T>

         fun main(args: Array<String>) {
             val anys: Cup<Any> = Cup<Int>() // Error: Type mismatch
             val nothings: Cup<Nothing> = Cup<Int>() // Error: Type mismatch
         }
         ```

       - 1.6.2. ***``covariant``***: para hacer que el parámetro de tipo sea covariante, se utiliza el **modificador** ***``out``***. Significa que **cuando** ***A*** **es subtipo de** ***B*** y ***Cup*** **es covariante**, el tipo ***``Cup<A>``*** **es subtipo de** ***``Cup<B>``***.

         ```kotlin
         class Cup<out T>

         open class B

         class A: B()
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

       - 1.6.3. ***``contravariant``***: para hacer que el parámetro de tipo sea contravariante, se utiliza el **modificador** ***``in``***. Significa que cuando ***A*** **es subtipo de** ***B*** y ***Cup*** **es contravariante**, el tipo ***``Cup<A>``*** **es supertipo de** ***``Cup<B>``***.

         ```kotlin
         class Cup<in T>

         open class B

         class A: B()

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

   - 1.7. **Clases anidadas e internas** (***inner class***): Una clase anidada marcada como *``inner``* puede acceder a los miembros de su clase externa (*outer class*). **Las clases internas llevan una referencia a un objeto de una clase externa**.

   - 1.8. ***Data classes***: se declaran anteponiendo la palabra reservada *``data``*. Tienen como objetivo principal **almacenar datos**, pero a su vez el compilador genera automáticamente funciones para todas las propiedades declaradas en el constructor primario: ***``equals()``*** / ***``hashCode()``***; ***``toString()``***; **funciones** ***``componentN()``***; **función** ***``copy()``***. Por esta razón, y porque en Kotlin no es necesario declarar los *setters* y *getters* de las propiedades (ver *Properties y Fields*), se considera a las *data classes* una mejor versión de los *POJO* usados en Java, ya que no requieren de tanto código. **Requerimientos**: el constructor primario debe tener **al menos un parámetro**; todos los parámetros del constructor primario **deben marcarse con** ***val*** **o** ***var***; **no pueden ser** *abstract*, *open*, *sealed* o *inner*.

   - 1.9. ***Object expressions*** **y** ***Object declarations***: Se utilizan para crear un objeto de alguna clase con alguna modificación sin declarar explícitimente una nueva subclase. **No pueden tener constructores** y pueden extender otras clases o implementar interfaces:

       - 1.9.1. ***Object expression***: juegan el mismo papel en Kotlin que las *clases anónimas* en Java (declaran e instancian una clase al mismo tiempo). Se **inicializan de inmediato** cuando son utilizados.

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

       - 1.9.2. ***Object declaration***: es una **implementación del patrón** ***singleton***. Siempre tienen nombre y **se inicializan** ***lazily***, cuando son accedidos **por primera vez**. No pueden ser locales (anidados dentro de una función), pero se pueden anidar en otros *objects declarations* o en clases no internas (*non-inner classes*). Además, como no es una expresión, no se puede utilizar en una asignación de variable como un *object expression*.

         ```kotlin
         // Crea un object declaration
         object DoAuth {
             // Define el método del objeto
             fun takeParams(username: String, password: String){         
                 println("input Auth parameters = $username:$password")
             }
         }

         fun main(){
             // Llama al método. Acá es cuando realmente el objeto es creado.
             DoAuth.takeParams("foo", "qwerty")
         }
         ```

           - 1.9.2.1. ***Companion Object***: es un *object declaration* dentro de una clase. Los miembros del mismo pueden ser llamados usando el nombre de la clase (es como invocar un *método estático* en Java).

             ```kotlin
             class BigBen { // Define una clase
                 companion object Bonger { // Define un companion object. Se puede omitir el nombre.
                     fun getBongs(nTimes: Int) { // Define un método dentro del companion object
                         for (i in 1 .. nTimes) {
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



### 2. Propiedades y fields

   - 2.1. ***Declaración***:

     ```kotlin
     val ejemplo: String = “Hola mundo”
     ```

       - 2.1.1. Una propiedad ***top-level*** (dentro del paquete) deber ser inicializada.

       - 2.1.2. Una propiedad ***miembro*** (dentro de la clase) debe ser inicializada **o** ser abstracta. También puede inicializarse más tarde con el modificador ***``lateinit``*** (sólo en *var properties*; *lateinit* retrasa la inicialización de la variable sin peligro de devolver una referencia nula) o con la **función** ***``lazy()``*** (ver *Propiedades delegadas*).

       - 2.1.3. Una variable ***local*** (dentro de una función) debe tener anotado el tipo **o** ser inicializada, y no pueden ser sobreescritas.

       - 2.1.4. Un ***parámetro*** debe tener anotado el tipo. Dentro del *header* de una clase, si se agrega *var* o *val* al nombre del parámetro, se vuelve una *property* del **constructor primario** de la clase. Por otro lado, los ***parámetros*** **de las funciones siempre son** ***val*** (de sólo lectura).

   - 2.2. ***Default arguments*** **y** ***Named arguments***: a los **parámetros de una función**, pueden asignárseles **valores por defecto**, que se usan cuando el argumento correspondiente **se omite** al invocar la función. O también se les puede **cambiar el valor** por defecto **al nombrarlos** y darles **otro valor** al momento de invocar la función. Esto evita *overloads* (definir múltiples métodos con el mismo nombre pero diferentes parámetros) y permite llamadas más claras, para evitar errores por un orden incorrecto de los argumentos (no es lo mismo *``"nombre@mail.com"``* que *``"mail.com@nombre"``*).  
   
     ```kotlin
     fun foo(name: String, number: Int = 42, toUpperCase: Boolean = false) =
        (if (toUpperCase) name.toUpperCase() else name) + number

     fun useFoo() = listOf(
             foo("a"),  // Imprime: a42
             foo("b", number = 1),  // Imprime: b1
             foo("c", toUpperCase = true),  // Imprime: C42
             foo(name = "d", number = 2, toUpperCase = true)  // Imprime: D2
     )

     fun main() {
         println(useFoo())
     }
     ```

   - 2.3. ***Backing fields*** (***getters*** y ***setters***): El *backing field* es un campo (*field*) **autogenerado** para cualquier propiedad que solo se puede usar dentro de los métodos *accesores* (*getter* o *setter*) y **estará presente solamente si la propiedad usa la implementación por defecto de al menos uno de los accesores, o si un accesor personalizado lo referencia a través de la palabra reservada** ***field***. Este *backing field* **se usa para evitar la llamada recursiva** a la misma propiedad desde el *getter* o el *setter*, lo que finalmente evita el *StackOverflowError*. Las propiedades con accesores *custom*, **no pueden ser abstractos**.

     ```kotlin
     class User{
         var firstName : String = "Hola"  
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

   - 2.4. ***Class Delegation***: el patrón *Delegation* ha probado ser una buena **alternativa a la implementación por herencia** y Kotlin lo soporta de forma nativa, sin requerir de código *boilerplate*. Una clase puede **implementar una** ***interface*** delegando todos su miembros públicos a un objeto especificado. Para esto, se utiliza la **palabra reservada** ***``by``***.

     ```kotlin
     interface Programador {
         fun programar()
     }

     class Empleado(val programador: Programador) : Programador by programador

     class MejorProgramador : Programador {
         override fun programar(){
             println("Soy un crack codeando")
         }
     }

     class ProgramadorLento : Programador {
         override fun programar(){
           println("Programo lento pero seguro")
         }
     }

     fun main() {
         val empleado = Empleado(ProgramadorLento())
         empleado.programar() // Programo lento pero seguro
     }
     ```

   - 2.5. ***Delegated properties***: La expresión después de la palabra reservada ***``by``***, indica que será la **expresión delegada** o ***property delegate***. Es decir, que los **métodos accesores** ***getter*** (**y** ***setter*** **si es** ***var***) **de la** ***property***, serán delegados a los métodos ***getValue()*** (y ***setValue()*** **para** ***vars***) del delegado, que no tiene que implementar ninguna *interface*, pero debe proveer esas ***operator functions***:

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

     fun main(){
         val e = Example()
       println(e.p)  // Example@4c873330, thank you for delegating 'p' to me!
         
         e.p = "NEW"   // NEW has been assigned to 'p' in Example@4c873330.
     }
     ```

     ***Standard delegates***:  La librería standard de Kotlin provee *factory methods* para varios tipos útiles de delegados: *Lazy*, *Observable* y almacenamiento de propiedades en un *map*.

       - 2.5.1. ***Lazy properties***: El valor no se calcula hasta el primer acceso. ***``lazy()``*** es una función que **toma un** ***lambda*** **y retorna una instancia de** ***``Lazy<T>``***, que puede servir como un **delegado** para implementar una propiedad *lazy*: el primer llamado a *get()* ejecuta el *lambda* pasado a *lazy()* y recuerda el resultado; las llamadas subsecuentes a *get()* simplemente retornan ese resultado. Es decir, utilizando la expresión ***``by lazy``***, lo que está entre corchetes (el ***lambda initializer***) recién se ejecuta cuando la variable se utiliza por primera vez, y en posteriores invocaciones retornará el valor computado inicialmente.

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

       - 2.5.2. ***Observable properties***: los *listeners* son notificados sobre cambios en la propiedad. ***Delegates.observable()*** toma dos argumentos: el valor inicial y una función para manipular las modificaciones. Esa función, que se llama cada vez que se modifica el valor de la propiedad, tiene tres parámetros: una propiedad a asignar, el valor anterior y el valor nuevo.

         ```kotlin
         import kotlin.properties.Delegates

         class User {
             var name: String by Delegates.observable("<no name>") {
                 prop, old, new ->
                 println("$old -> $new")
             }
         }

         fun main() {
             val user = User()
             user.name = "first"  // <no name> -> first
             user.name = "second" // first -> second
         }
         ```

       - 2.5.3. **Almacenar propiedades en un** ***map***, en vez de un *field* separado para cada propiedad. A menudo aparece en aplicaciones como, por ejemplo, parsear un JSON. En estos casos, **se usa la instancia misma del** ***Map*** **como delegado**, y las propiedades delegadas toman los valores de dicha instancia, a través de los *keys*.

         ```kotlin
         class User(val map: Map<String, Any?>) {
             val name: String by map
             val age: Int     by map
         }

         fun main() {
             val user = User(mapOf(
                 "name" to "John Doe",
                 "age"  to 25
             ))
             println(user.name) // John Doe
             println(user.age)  // 25
         }
         ```

         Esto también funciona para propiedades *var*, usando ***MutableMap*** en vez de *Map*.

         ```kotlin
         class MutableUser(val map: MutableMap<String, Any?>) {
             var name: String by map
             var age: Int     by map
         }
         ```



---



### 3. Funciones

   - 3.1. ***Declaración***:

     ```kotlin
     fun nombreCompleto(name: String, lastname: String): String {}
     ```

       - 3.1.1. Si se especifica un tipo de retorno, la función debe retornar un valor (expresión ***``return``***). Si no, por defecto retorna ***Unit*** (*void* en Java).

       - 3.1.2. También puede ser de ***single-expression***, en cuyo caso el ***return*** **se omite** y si el tipo de retorno puede ser inferido, también se puede omitir:

         ```kotlin
         fun foo(x: Int, y: Int): Int = x * y
         ```

   - 3.2. ***Infix functions***: la palabra reservada ***``infix``*** sirve para llamar a una función **omitiendo el punto y los paréntesis**. Sólo se puede aplicar a **funciones miembro de una clase** y a **funciones de extensión**, y siempre a funciones con **un solo parámetro** (además este parámetro no debe aceptar un número variable de argumentos ni tener un valor predeterminado).

     ```kotlin
     infix fun Int.multiplicarPor(factor: Int) = this * factor

     fun main() {
         val producto = 20 multiplicarPor 5 // es igual a 20.multiplicarPor(5)
         println("20 x 5 = $producto")
     }
     ```

   - 3.3. ***Varargs*** (Número variable de argumentos): Un parámetro de una función (normalmente el último) se puede marcar con el **modificador** ***``vararg``***, permitiendo que se pase un número variable de argumentos a la función.

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

   - 3.4. ***Expresiones Lambda*** y ***Funciones Anónimas***: son **literales funciones** (***function literals***), es decir, funciones que **no se declaran**, sino que **se pasan inmediatamente como una expresión**. La sintaxis de una expresión *lambda* es:

     ```kotlin
     { x: Int, y: Int -> x + y }
     ```

     La sintaxis de una función anónima es similar a la de una expresión *lambda*, pero a diferencia de ésta, **explicita el tipo de retorno** de la función. La declaración es similar a una función común, excepto porque su nombre es omitido:

     ```kotlin
     fun(x: Int, y: Int): Int = x + y
     ```

       - 3.4.1. ***Trailing lambda***: Si el último parámetro de una función es una función, entonces una expresión *lambda* pasada como el argumento correspondiente puede ser colocado **fuera del paréntesis**:

         ```kotlin
         val product = items.fold(1) { acc, e -> acc * e }
         ```

       - 3.4.2. Si la expresión *lambda* tiene **un único parámetro**, está permitido no declararlo y omitir la -> :

         ```kotlin
         ints.filter { it > 0 } // this literal is of type '(it: Int) -> Boolean'
         ```

       - 3.4.3. Una variable también puede **tiparse con una función** (***function type***) y asignarle una **función** ***lambda***:

         ```kotlin
         val isEven: (Int) -> Boolean = { it % 2 == 0 }
         ```

         En este caso, la función toma un argumento de tipo *Int* y devuelve un *Boolean*. O también, puede tomar un argumento de tipo *Int* y devolver otro *Int*:

         ```kotlin
         val square: (Int)->Int = { x -> x * x }
         ```

       - 3.4.4. ***Non-local returns***: solo se puede usar un retorno normal no calificado para salir de una función nombrada o una función anónima. Esto significa que para salir de un *lambda*, hay que usar una **etiqueta** (***label***), y un retorno simple está prohibido dentro de un *lambda*, ya que no puede hacer que la función que lo encierra regrese.

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

   - 3.5. ***Funciones de extensión***: Una función de extensión se declara agregando como prefijo **el tipo que está siendo extendido** (***receiver type***). La palabra reservada ***``this``*** (ver *This expressions* en *Otros*) dentro de una función de extensión corresponde al ***receiver object*** (objeto *receptor* o *destinatario*, el que se pasa antes del punto):

     ```kotlin
     fun MutableList<Int>.swap(index1: Int, index2: Int) {
         val tmp = this[index1] // 'this' corresponds to the list
         this[index1] = this[index2]
         this[index2] = tmp
     }

     val list = mutableListOf(1, 2, 3)
     list.swap(0, 2) // 'this' inside 'swap()' will hold the value of 'list'
     ```

   - 3.6. **Funciones de Orden Superior** (***Higher-Order Functions***): una función de orden superior toma otra función (o expresión *lambda*) como **parámetro**, **devuelve una función**, **o hace ambos**. Para pasar una función como parámetro de una función de orden superior, se la debe referenciar con el ***operador ``::``*** (ver *Reflection* en *Otros*)

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
     fun multiplier(factor: Double): (Double) -> Double = { number -> number*factor }

     fun main() { 
         val doubler = multiplier(2.0)
         print(doubler(5.6)) // 11.2 
     }
     ```

   - 3.7. ***Function type with receiver*** y ***Function literal with receiver***: partiendo de una función de extensión simple como ejemplo, si se necesita asociar esa función con una propiedad, entonces se puede usar una **referencia de función** (***function reference***) (ver *Reflection* en *Otros*).

     ```kotlin
     fun Int.square() = this * this // Extension function
     println(2.square())   // Imprime: 4. En la función square(), this refiere al receiver object (el 2)
     println(square(2))    // Error: Unresolved reference

     val squareFunType: Int.()->Int = Int::square // Function type with receiver, usando function reference
     println(squareFunType(2))      // Imprime: 4
     println(2.squareFunType())     // Imprime: 4
     ```

     Se ha utilizado ***``Int.() -> Int``*** que es un **tipo función con receptor** (***function type with receiver***). En lugar de usar la referencia, se puede definir la función usando una de las ***function literals with receiver***. Dentro del cuerpo de la *function literal*, el objeto receptor pasado a una llamada se convierte en un ***this*** **implícito**, de modo que se puede acceder a los miembros de ese objeto receptor **sin ningún calificador adicional**, **o** acceder al objeto receptor **utilizando una expresión** ***this***. Este comportamiento es **similar a las funciones de extensión**, que también permiten acceder a los miembros del objeto receptor dentro del cuerpo de la función. A continuación, se muestra un ejemplo de una ***function literal with receiver*** junto con su tipo, donde se llama a la **función** ***``plus``*** en el objeto receptor:

     ```kotlin
     val suma: Int.(Int)->Int = { other -> plus(other) } // plus() se llama en el receiver object (el 2)
     println(suma(2, 2))   // Imprime: 4
     println(2.suma(2))  // Imprime: 4
     ```

     La **sintaxis de función anónima** permite especificar el tipo de receptor de una *function literal* directamente. Esto puede ser útil si se necesita declarar una variable de un *function type with receiver* y usarla más tarde.

     ```kotlin
     val squareFunLiteral = fun Int.(other: Int): Int = this * other // Sintaxis de función anónima
     println(squareFunLiteral(2, 2))   // Imprime: 4
     println(2.squareFunLiteral(2))  // Imprime: 4
     ```

   - 3.8. ***Closures***: Una expresión *lambda* o una función anónima pueden acceder a su *closure*, es decir, a las **variables declaradas en un ámbito** (***scope***) **externo**. Las variables capturadas en el *closure* pueden ser modificadas en el *lambda*.

     ```kotlin
     var sum = 0
     ints.filter { it > 0 }.forEach {
         sum += it
     }
     print(sum)
     ```

   - 3.9. ***Inline Functions***: cuando se crean funciones que reciben funciones como argumento, el compilador necesita crear clases anónimas, **consumiendo recursos** por la asignación de memoria extra que precisa. Una forma de evitar esto es marcar la función como ***``inline``***. Al hacer esto, **el compilador internamente copia esa función** por su código en los sitios donde se llame, evitando crear una nueva referencia a la función. Sin embargo, hay que tener en cuenta que al marcar una función como *inline*, el *byte-code* también se hace más grande, por lo que es **altamente recomendable** solo alinear funciones de orden superior **pequeñas que acepten** ***lambdas*** **como parámetros**.

     ```kotlin
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
           String var4 = "Radius is " + radius;
           System.out.println(var4);
           double result = ((Number)op.invoke(radius)).doubleValue();
           String var6 = "The result is " + result;
           System.out.println(var6);
     }

     // En vez de llamar a la función circleOperation, el compilador copia el código.
     public static final void main(@NotNull String[] args) {
           Intrinsics.checkParameterIsNotNull(args, "args");
           double radius$iv = 5.3D;
           String var3 = "Radius is " + radius$iv;
           System.out.println(var3);
           double result$iv = 6.283185307179586D * radius$iv;
           String var9 = "The result is " + result$iv;
           System.out.println(var9);
     }
     ```

     En caso de que haya más de dos parámetros *lambda* en una función, se puede decidir cuál *lambda* **no alinear** usando el modificador ***``noinline``*** sobre el parámetro. Esto es útil especialmente para un parámetro *lambda* que tomará mucho código.

     ```kotlin
     inline fun myFunc(op: (Double) -> Double, noinline op2: (Int) -> Int) {
        // Realizar operaciones
     }
     ```

       - 3.9.1. **Parámetros de tipo** ***reified***: a veces, se necesita acceder a un tipo pasado como parámetro de una función. Normalmente, esto se solventa pasando la clase como parámetro de la función, haciendo el **código más complejo y poco estético**:

         ```kotlin
         fun <T> TreeNode.findParentOfType(clazz: Class<T>): T? {
         var p = parent
         while (p != null && !clazz.isInstance(p)) {
         p = p.parent
         }
         @Suppress("UNCHECKED_CAST")
         return p as T?
         }

         // Utiliza reflexión para comprobar que un nodo
         // es de cierto tipo.
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

         // Se puede llamar a la función pasándole como parámetro
         // directamente el tipo.
         treeNode.findParentOfType<MyTreeNode>()
         ```

   - 3.10. ***Tipos genéricos***: Como las clases, las funciones también pueden tener **parámetros de tipo** con la sintaxis ***``fun <T> nombreFuncion(parametro: tipo<T>)``***:

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



### 4. Corrutinas (*Coroutines*)

   - 4.1. Las corrutinas permiten ejectuar **tareas en segundo plano** de una forma sencilla para, por ejemplo, **no bloquear el hilo principal**. Para eso, se utilizan **funciones de suspensión** (***suspension functions***), que son ***``delay()``***, ***``await()``*** (que se utiliza junto con el *builder async{}*) y ***``withContext()``*** (una práctica recomendada consiste en usar *withContext()* a fin de garantizar que todas las funciones sean seguras para el subproceso principal (*main-safe*), lo cual significa que se puede llamar a la función desde el subproceso principal).

       - 4.1.1. También se puede indicar que una función personalizada es de suspensión anteponiéndole la **palabra reservada** ***``suspend``*** (pausa la ejecución de la corrutina actual y guarda todas las variables locales) o ***``resume``*** (continúa la ejecución de una corrutina suspendida desde donde se detuvo).

       - 4.1.2. A las funciones de suspensión sólo se las puede llamar **desde una corrutina** o **desde otra función de suspensión**.

   - 4.2. ***Scope***: el **ámbito donde la corrutina tendrá efecto**. Se puede crear un *scope* propio (un objeto ***CoroutineScope***) o utilizar el ***GlobalScope***, que es un *scope* general que provee la biblioteca de corrutinas de Kotlin, el cual estará vivo durante todo el tiempo de vida de la aplicación.

       - 4.2.1. Para ejectuar una corrutina en Android, primero se debe importar la biblioteca ***kotlinx-coroutines-android*** y luego definirle un *scope*. Implementando la librería de ***ViewModel***, también se puede utilizar ***viewModelScope***, que es un *CoroutineScope* predefinido.

   - 4.3. ***Builders***: permiten **comenzar una corrutina**, definiendo **bloques donde se ejecturá el contenido** de la misma. Estos *builders* son ***``runBlocking{}``*** (más que nada se utiliza para los *tests*), ***``launch{}``*** (el más utilizado; inicia una corrutina nueva y no le muestra el resultado a quien la llama) y ***``async{}``*** (inicia una corrutina nueva y permite mostrar un resultado con la función de suspensión ***``await``***; permite ejecutar **varias tareas en segundo plano en paralelo**).

   - 4.4. ***Dispatchers***: Tanto a los *builders* como a las funciones de suspensión, se les puede pasar un *dispatcher*, que es para **determinar qué subproceso** (***thread***) **se utiliza para la ejecución de la corrutina**. Existen cuatro tipos: ***Unconfined*** (se utiliza más que nada para *testing*), ***Main*** (define el hilo principal; sirve por ejemplo para llamar a funciones *suspend*, ejecutar operaciones del *framework* de la UI de Android y actualizar objetos *LiveData*), ***Default*** (no se define; sirve para ejecutar operaciones que requieren de un uso intensivo de la CPU, como por ejemplo clasificar una lista o analizar un JSON) y ***IO*** (para operaciones bloqueantes de entrada/salida, por ejemplo, peticiones a un servidor, a una base de datos, a un fichero, o usar el componente *Room* de Android).

   - 4.5. ***Jobs***: son controladores del ciclo de vida de las corrutinas. Cada corrutina que se crea con *launch* o *async* muestra una instancia de *Job* que **identifica de forma única la corrutina y administra su ciclo de vida**. También se puede pasar un elemento *Job* a *CoroutineScope* para administrar más aspectos de su ciclo de vida, como por ejemplo cancelar una corrutina según una condición.

   - 4.6. En Android, se puede implementar la biblioteca ***lifecycle-runtime-ktx*** para definir un ***lifecyleContext*** y evitar escribir código de más para los casos en que la corrutina no debe finalizarse si finaliza el ciclo de vida de una *activity*, por ejemplo, en cuyo caso se lanzaría una excepción.



---



### 5. Colecciones

   - 5.1. Las colecciones son grupos de un **número variable de elementos** (posiblemente cero) que comparten la importancia del problema que se está resolviendo y se operan de forma común. 
     Los siguientes tipos de colección son relevantes para Kotlin: ***List***, ***Set*** y ***Map*** (o ***Dictionary***). Kotlin permite manipular colecciones independientemente del tipo exacto de objetos almacenados en ellas, ya que la biblioteca estándar de Kotlin ofrece *interfaces*, clases y funciones genéricas para crear, completar y administrar colecciones de cualquier tipo. Un par de *interfaces* representan cada tipo de colección: una ***read-only*** y una ***mutable***.

       - 5.1.1. La interfaz de **solo lectura** (***read-only***) proporciona operaciones para **acceder a los elementos** de la colección.

       - 5.1.2. La interfaz ***mutable*** extiende la correspondiente interfaz de solo lectura con **operaciones de escritura: agregar, eliminar y actualizar** sus elementos. Alterar una colección mutable **no requiere que sea una** ***var***: las operaciones de escritura modifican el mismo objeto de colección mutable, por lo que la referencia no cambia. Sin embargo, si se intenta **reasignar una colección** ***val***, se obtendrá un **error de compilación**.

         ```kotlin
         val numbers = mutableListOf("one", "two", "three", "four")
         numbers.add("five")   // this is OK    
         //numbers = mutableListOf("six", "seven")      // compilation error
         ```

   - 5.2. ***``Collection<T>``***: es la raíz de la jerarquía de colecciones, que hereda de la interfaz ***``Iterable<T>``***. Se puede utilizar *Collection* como un parámetro de una función que se aplica a diferentes tipos de *Collection* (***List*** **o** ***Set***, ya que ***Map*** **no hereda de esta interfaz**).

       - 5.2.1. ***``MutableCollection<E>``***: es una *Collection* **con operaciones de escritura**, como agregar y quitar.

   - 5.3. ***``List<T>``***: es una **colección ordenada** con acceso a los elementos a través de **índices**. Los elementos pueden aparecer **más de una vez** en una lista.

       - 5.3.1. ***``MutableList<T>``***: es una *List* **con operaciones de escritura específicas** de *List*, por ejemplo, agregar o eliminar un elemento en una posición específica. A pesar de la similitud con los *arrays* de Java, hay una **diferencia importante: el tamaño de un** ***array*** **se define en la inicialización y nunca se cambia**. Y a su vez, un ***array*** **es mutable** (se puede cambiar a través de cualquier referencia).  
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

   - 5.4. ***``Set<T>``***: es una **colección de elementos únicos**. Generalmente, **el orden** de los elementos **no tiene importancia**. Asimismo, los elementos nulos también son únicos (no puede haber dos elementos *null*).

       - 5.4.1. ***``MutableSet<E>``***: es un *Set* con operaciones de escritura de *MutableCollection*. La **implementación predeterminada** de *Set* (***LinkedHashSet***) conserva el orden de inserción de los elementos. Por lo tanto, las funciones que dependen del orden, como ***``first()``*** o ***``last()``***, devuelven resultados predecibles en dichos *sets*. Una **implementación alternativa**, ***HashSet***, no dice nada sobre el orden de los elementos, por lo que llamar a dichas funciones devuelve resultados impredecibles. Sin embargo, ***HashSet*** **requiere menos memoria** para almacenar la misma cantidad de elementos.

         ```kotlin
         val numbers = setOf(1, 2, 3, 4)  // LinkedHashSet is the default implementation
         val numbersBackwards = setOf(4, 3, 2, 1)

         println("The sets are equal: ${numbers == numbersBackwards}") // The sets are equal: true
         println(numbers.first() == numbersBackwards.first()) // false
         println(numbers.first() == numbersBackwards.last())  // true
         ```

   - 5.5. ***Map*** (o ***Dictionary***): ***``Map <K, V>``*** no es un heredero de la interfaz *Collection*; sin embargo, también es un tipo de colección de Kotlin. Un *map* almacena ***pares clave-valor*** o **entradas** (***key-value pairs*** o ***entries***); las **claves son únicas**, pero se pueden emparejar **diferentes claves con valores iguales** (repetidos). La interfaz *Map* proporciona funciones específicas, como el acceso al valor por clave, la búsqueda de claves y valores, etc. Además, como se accede a los valores a través de sus claves, el orden de los pares no importa, por lo que dos *maps* que contienen pares iguales son iguales independientemente del orden de los pares.

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

       - 5.5.1. ***``MutableMap<K, V>``***: es un *Map* con operaciones de escritura de *maps*, por ejemplo, se puede agregar un nuevo par clave-valor o actualizar el valor asociado con la clave dada. La **implementación predeterminada** de *Map* (***LinkedHashMap***) conserva el orden de inserción de los elementos al iterar el *map*. A su vez, una **implementación alternativa**, ***HashMap***, no dice nada sobre el orden de los elementos.

         ```kotlin
         val numbersMap = mutableMapOf("one" to 1, "two" to 2)
         numbersMap.put("three", 3)
         numbersMap["one"] = 11

         println(numbersMap) // {one=11, two=2, three=3}
         ```

   - 5.6. ***Construcción de colecciones***: La forma más común de crear una colección es con las funciones de biblioteca estándar ***``listOf<T>()``***, ***``setOf<T>()``***, ***``mutableListOf<T>()``***, ***``mutableSetOf<T>()``***. Lo mismo está disponible para *maps* con las funciones ***``mapOf()``*** y ***``mutableMapOf()``***. Las claves y los valores del *map* se pasan como objetos de tipo ***``Pair``*** (generalmente creados con la **función** ***infix*** ***``to``***).

     ```kotlin
     val numbersMap = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key4" to 1)
     ```

     Para evitar el uso excesivo de memoria, se pueden utilizar formas alternativas. Por ejemplo, se puede crear un *mutable map* y poblarlo usando las operaciones de escritura. La función ***``apply()``*** puede ayudar a mantener la inicialización fluida.

     ```kotlin
     val numbersMap = mutableMapOf<String, String>().apply { this["one"] = "1"; this["two"] = "2" }
     ```

       - 5.6.1. ***Colecciones vacías***: También hay funciones para crear colecciones sin ningún elemento: ***``emptyList()``***, ***``emptySet()``*** y ***``emptyMap()``***. Al crear colecciones vacías, se debe **especificar el tipo de elementos** que contendrá la colección.

         ```kotlin
         val empty = emptyList<String>()
         ```

       - 5.6.2. ***Funciones de inicialización para listas***: Para las **listas**, hay un constructor que toma el **tamaño de la lista** y la **función inicializadora** que define el valor del elemento en función de su índice.

         ```kotlin
         val doubled = List(3, { it * 2 })  // or MutableList 
         println(doubled) // [0, 2, 4]
         ```

   - 5.7. ***Copia de colecciones***: Las funciones de copia de colecciones, como ***``toList()``***, ***``toMutableList()``***, ***``toSet()``*** y otras, crean una nueva colección con los mismos elementos. **Si se agregan o eliminan elementos a la colección original, esto no afectará a las copias**. Por su parte, las copias también pueden cambiarse **independientemente de la colección original**.

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

   - 5.8. ***Iterators***: Los iteradores (objetos que brindan acceso a los elementos secuencialmente sin exponer la estructura subyacente de la colección), se pueden obtener si se **hereda a la interfaz** ***``Iterable<T>``***, incluidos *Set* y *List*, llamando a la **función** ***``iterator()``***. Una vez obtenido, apunta al primer elemento de la colección; llamando a la **función** ***``next()``*** se retorna este elemento y se mueve el *iterator* al próximo elemento, si es que existe. Una vez que el *iterator* recorre el último elemento, **ya no se puede usar** para recuperar más elementos o resetearlo a posiciones previas. Si se quiere iterar la colección otra vez, **se debe crear un nuevo** ***iterator***. El **ciclo** ***for*** obtiene un *iterator* **implícitamente**. Al igual que con la **función** ***``forEach()``***, que permite iterar una colección automáticamente y ejecutar un código dado para cada elemento.

     ```kotlin
     val numbers = listOf("one", "two", "three", "four")
     numbers.forEach {
         println(it)
     }
     ```

       - 5.8.1. ***ListIterator***: es una implementación especial para las listas, que le **permite iterar hacia adelante y atrás**, por lo que puede usarse luego de que llega al último elemento. Para la itereación hacia atrás se usan las **fuciones** ***``hasPrevious()``*** y ***``previous()``***. Además, proporciona información sobre los índices de elementos con las **funciones** ***``nextIndex()``*** y ***``previousIndex()``***.

       - 5.8.2. ***MutableIterator***: al iterar **colecciones mutables**, se puede utilizar la función ***``remove()``*** para borrar elementos mientras se itera la colección. Además, con ***MutableListIterator*** se puede remplazar (***``set()``***) o insertar (***``add()``***) elementos mientras se itera la lista.

         ```kotlin
         val numbers = mutableListOf("one", "four", "four") 
         val mutableListIterator = numbers.listIterator()

         mutableListIterator.next()
         mutableListIterator.add("two")
         mutableListIterator.next()
         mutableListIterator.set("three")   
         println(numbers) // [one, two, three, four]
         ```

   - 5.9. ***Rangos y progresiones***: se pueden crear rangos usando la función ***``rangeTo()``*** (o en su forma de **operador** **``..``** ). Dichos rangos se utilizan para iterar, generalmente en ciclos *for*. Para iterar números en **orden inverso**, se usa la **función** ***``downTo``***. También se puede especificar el paso con la **función** ***``step``***. Para iterar un rango de números que **no incluya el último elemento**, se utiliza la **función** ***``until``***.

       - 5.9.1. ***Rangos***: Un rango define un intervalo cerrado en el sentido matemático: se define por sus dos valores extremos que están incluidos en el rango. La operación principal en rangos es ***``contains``***, que generalmente se usa en forma de **operadores** ***``in``*** y ***``!in``***.

       - 5.9.2. ***Progresiones***: Rangos de tipo integral (***IntRange***, ***LongRange***, ***CharRange***) pueden tratarse como progresiones aritméticas. En Kotlin, estas progresiones se definen mediante tipos especiales: ***IntProgression***, ***LongProgression*** y ***CharProgression***. Las progresiones tienen tres propiedades esenciales: el primer elemento (***``first``***), el último elemento (***``last``***) y un paso distinto de cero (***``step``***). Cuando se crea una progresión implícitamente al iterar un rango, el primer y último elemento de esta progresión son los extremos del rango, y el paso por defecto es 1.

   - 5.10. ***``Sequence<T>``***: Las **secuencias** (paquete *kotlin.sequences*) ofrecen las mismas funciones que un *Iterable* pero implementan otro enfoque para el procesamiento de colecciones de varios pasos. Cuando el procesamiento de un *Iterable* incluye varios pasos, se ejecutan de manera ***eagerly***: cada paso de procesamiento se completa y devuelve su resultado (una **colección intermedia**). Y el siguiente paso se ejecuta en esta colección intermedia. A su vez, el procesamiento de múltiples pasos de secuencias se ejecuta de manera ***lazy*** cuando es posible: la computación real ocurre **solo cuando se solicita el resultado de toda la cadena** de procesamiento. El orden de ejecución de las operaciones también es diferente: ***Sequence*** realiza **todos los pasos de procesamiento** uno por uno **para cada elemento**, mejorando el rendimiento. En cambio, ***Iterable*** completa **cada paso de procesamiento para toda la colección** y luego **procede al siguiente paso**. Hay que tener en cuenta que al procesar colecciones más pequeñas o al realizar cálculos más simples, no es conveniente usar *Sequences* por el gasto que produce su ejecución *lazy*.

       - 5.10.1. ***Construcción***: Para crear una secuencia, se puede llamar a la función ***``sequenceOf()``*** que enumera los elementos como sus argumentos.

         ```kotlin
         val numbersSequence = sequenceOf("four", "three", "two", "one")
         ```

         Si ya se tiene un objeto *Iterable* (como un *List* o un *Set*), se puede crear una secuencia a partir de él llamando a ***``asSequence()``***.

         ```kotlin
         val numbers = listOf("one", "two", "three", "four")
         val numbersSequence = numbers.asSequence()
         ```

         Una forma más de crear una secuencia es construyéndola con una función que calcula sus elementos. Para construir una secuencia basada en una función, se llama a ***``generateSequence()``*** con esta función como argumento. Opcionalmente, se puede especificar el primer elemento como un valor explícito o como resultado de una llamada de función. La generación de la secuencia se detiene cuando la función proporcionada devuelve un valor nulo.

         ```kotlin
         val oddNumbers = generateSequence(1) { it + 2 } // `it` is the previous element
         println(oddNumbers.take(5).toList()) // [1, 3, 5, 7, 9]
         //println(oddNumbers.count())     // error: the sequence is infinite

         val oddNumbersLessThan10 = generateSequence(1) { if (it < 10) it + 2 else null }
         println(oddNumbersLessThan10.count()) // 6
         ```

         Finalmente, hay una función que permite producir elementos de secuencia uno por uno o por trozos de tamaños arbitrarios: la **función** ***``sequence()``***, que toma una expresión *lambda* que contiene llamadas a las **funciones** ***``yield()``*** y ***``yieldAll()``***. Estas funciones devuelven un elemento al consumidor de la secuencia y **suspenden la ejecución de** ***sequence()*** hasta que el consumidor solicite el siguiente elemento. ***yield()*** toma un solo elemento como argumento; ***yieldAll()*** puede tomar un objeto *Iterable*, un *Iterador* u otro *Sequence*. Un argumento *Sequence* de *yieldAll()* puede ser infinito. En ese caso, dicha llamada debe ser la última: todas las llamadas posteriores nunca se ejecutarán.

         ```kotlin
         val oddNumbers = sequence {
             yield(1)
             yieldAll(listOf(3, 5))
             yieldAll(generateSequence(7) { it + 2 })
         }
         println(oddNumbers.take(5).toList()) // [1, 3, 5, 7, 9]
         ```

       - 5.10.2. ***Sequence operations***: Las operaciones de secuencia se pueden clasificar en los siguientes grupos con respecto a sus requisitos de estado: ***stateless*** y ***stateful***.  
         ***Stateless***: no requieren ningún estado y procesan cada elemento de forma independiente, por ejemplo, ***``map()``*** o ***``filter()``***. Las operaciones sin estado también pueden requerir una pequeña cantidad constante de estado para procesar un elemento, por ejemplo, ***``take()``*** o ***``drop()``***.  
         ***Stateful***: requieren una cantidad significativa de estado, generalmente proporcional al número de elementos en una secuencia.
         Si **una operación de secuencia devuelve otra secuencia**, que se produce de forma *lazy*, se llama ***intermedia***. **De lo contrario, la operación es** ***terminal***. Ejemplos de operaciones terminales son ***``toList()``*** o ***``sum()``***. Los elementos de secuencia se pueden recuperar solo con operaciones terminales.

   - 5.11. ***Operaciones***: Las operaciones de colección se declaran en la biblioteca estándar de dos formas: **funciones miembro** de interfaces de colección y **funciones de extensión**. Las funciones miembro definen operaciones que son esenciales para un tipo de colección. Por ejemplo, *Collection* contiene la función ***``isEmpty()``*** para verificar si está vacío; *List* contiene ***``get()``*** para el acceso por índice a los elementos, y así sucesivamente. Otras operaciones de colección se declaran como funciones de extensión. Se trata de funciones de filtrado, transformación, ordenación y otras funciones de procesamiento de colecciones.  
     A su vez, cada tipo de colección tiene su propio conjunto de **operaciones específicas** o que se ejecutan de una manera específica según el tipo de colección.  
     Así, *List* tiene ***``get()``*** (o su forma corta con los **``[ ]``**), ***``getOrElse()``***, ***``getOrNull()``***, ***``subList()``***, **``indexOf()``**, ***``lastIndexOf()``***, ***``binarySearch()``***, ***``add()``***, ***``addAll()``***, ***``set()``*** (o su forma corta con los **``[ ]``**), ***``fill()``*** (reemplaza todos los elementos de la colección con el valor especificado), ***``removeAt()``***, entre otros;  
     *Set* tiene ***``union()``*** (puede ser usada en su forma *infix*: ***`a union b`***), ***``intersect()``*** (puede ser usada en su forma *infix*: ***``a intersect b``***), ***``substract()``*** (puede ser usada en su forma *infix*: ***``a substract b``***);  
     y *Map* tiene ***``get()``*** (o su forma corta con los **``[ ]``**), ***``getValue()``***, ***``getOrElse()``***, ***``getOrDefault()``***, ***``filter()``*** (al que se le pasa un predicado con un *Pair* como argumento), ***``filterKeys()``***, ***``filterValues()``***, ***``plus``*** (**``+``**) y  ***``minus``*** (**``-``**) (trabajan diferente que en otras colecciones), ***``put()``*** (agrega un nuevo par clave-valor a un mapa mutable, sobreescribiendo los valores si las claves dadas ya existen en el mapa), ***``putAll()``*** (añade múltiples entradas a la vez, sobreescribiendo los valores si las claves dadas ya existen en el mapa), ***``remove()``*** (si se especifica tanto la clave como el valor, el elemento con esta clave se eliminará solo si su valor coincide con el segundo argumento).

       - 5.11.1. ***Operaciones comunes***: Las operaciones comunes están disponibles **tanto para colecciones de solo lectura como mutables**. Estas operaciones devuelven sus resultados **sin afectar la colección original**. Los resultados de tales operaciones deben almacenarse en variables o usarse de alguna otra manera, por ejemplo, pasarse a otras funciones. Para ciertas operaciones de colección, existe una opción para especificar el **objeto de destino**. El destino es una colección mutable a la que la función agrega sus elementos resultantes en lugar de devolverlos en un nuevo objeto. Para realizar operaciones con destinos, hay funciones separadas con el **sufijo** ***To*** en sus nombres, por ejemplo, ***``filterTo()``*** en lugar de ***``filter()``*** o ***``associateTo()``*** en lugar de ***``associate()``***. Estas funciones toman la **colección de destino como parámetro adicional**.

         ```kotlin
         val numbers = listOf("one", "two", "three", "four")
         val filterResults = mutableListOf<String>()  //destination object
         numbers.filterTo(filterResults) { it.length > 3 }
         numbers.filterIndexedTo(filterResults) { index, _ -> index == 0 }
         println(filterResults) // contains results of both operations
         ```

           - 5.11.1.1. ***Transformations***: Estas funciones crean nuevas colecciones a partir de las existentes según las reglas de transformación proporcionadas.

               - 5.11.1.1.1. ***Mapping***: crea una nueva colección a partir de los resultados de una función sobre los elementos de una colección. La función básica de *mapping* es ***``map()``***, que aplica una determinada función *lambda* a cada elemento y devuelve una lista de los resultados respetando el orden original de los elementos en la colección original. Si además se quiere usar los índices de los elementos como argumento, se usa ***``mapIndexed()``***.Si la transformación produce *null* en ciertos elementos, se pueden filtrar los *null* de la colección de resultados llamando a la función ***``mapNotNull()``*** en lugar de ***``map()``*** y ***``mapIndexedNotNull()``*** en lugar de ***``mapIndexed()``***.

                 ```kotlin
                 val numeros = setOf(1, 2, 3, 4, 5)
                 println(numeros.map { it * 2 })
                 // [2, 4, 6, 8, 10]
                 println(numeros.mapIndexed { idx, value -> value * idx + 1 })
                 // [1, 3, 7, 13, 21]
                 println(numeros.mapNotNull { if (it == 2) null else it * 3 })
                 // [3, 9, 12, 15]
                 println(numeros.mapIndexedNotNull { idx, value -> if (idx == 0) null else value * idx })
                 // [2, 6, 12, 20]
                 ```

                 Al transformar diccionarios se tienen dos opciones: transformar claves sin cambiar los valores o viceversa. Para aplicar una transformación determinada a las claves se usa ***``mapKeys()``***, y ***``mapValues()``*** para transformar los valores, aunque ambas pueden utilizar tanto las claves como los valores.

                 ```kotlin
                 val numerosDic = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key11" to 11)
                 println(numerosDic.mapKeys { it.key.toUpperCase() + it.value})
                 // {KEY11=1, KEY22=2, KEY33=3, KEY1111=11}
                 println(numerosDic.mapValues { it.value + it.key.length })
                 // {key1=5, key2=6, key3=7, key11=16}
                 ```

               - 5.11.1.1.2. ***Zipping***: consiste en **construir pares a partir de elementos con las mismas posiciones en dos colecciones**. En la biblioteca estándar de Kotlin esto se realiza mediante la **función de extensión** ***``zip()``*** (comprimir), que cuando se utiliza sobre una colección con otra colección como argumento, **devuelve una lista de objetos** ***Pair*** en la que los elementos de la primera colección (receptor) son los primeros elementos en estos pares. Si las colecciones tienen diferentes tamaños, el resultado de *zip()* es el tamaño más pequeño y los últimos elementos de la colección más grande no se incluyen en el resultado. La función *zip()* también puede ser llamada en forma *infix*: ***``a zip b``***. También se puede hacer la transformación inversa (descomprimir) con la **función** ***``unzip()``*** que genera dos listas a partir de pares de elementos, y obtiene así las listas originales.

                 ```kotlin
                 val capitales = listOf("Madrid", "Berlín", "París")
                 val paises = listOf("España", "Alemania", "Francia")
                 println(capitales zip paises)  
                 // [(Madrid, España), (Berlín, Alemania), (París, Francia)]
                 println(capitales.zip(paises)) 
                 // [(Madrid, España), (Berlín, Alemania), (París, Francia)]

                 val dosPaises = listOf("España", "Alemania")
                 println(capitales.zip(dosPaises)) 
                 // [(Madrid, España), (Berlín, Alemania)]
                 ```

                 También se puede utilizar *zip()* con una función de transformación que **toma dos parámetros**: el elemento receptor y el elemento argumento, y en este caso la lista que devuelve **contiene los valores resultantes** de esa función.

                 ```kotlin
                 val capitales = listOf("Madrid", "Berlín", "París")
                 val paises = listOf("españa", "alemania", "francia")

                 val capitalPais = capitales.zip(paises) { capital, pais -> "La capital de ${pais.capitalize()} es $capital" }
                 for (item in capitalPais) println(item)
                 // La capital de España es Madrid
                 // La capital de Alemania es Berlín
                 // La capital de Francia es París
                 ```

               - 5.11.1.1.3. ***Association***: permiten **construir diccionarios** (***Map***) a partir de los elementos de una colección y ciertos valores asociados con ellos y, según el tipo de asociación, los elementos pueden ser claves o valores en el diccionario. La función básica ***``associateWith()``*** crea un *Map* en el que los elementos de la colección original son claves y los valores se generan a partir de ellos mediante una función de transformación determinada (si dos elementos son iguales, solo el último permanece).

                 ```kotlin
                 val numeros = listOf("uno", "dos", "tres", "cuatro")
                 println(numeros.associateWith { it.capitalize() })
                 // {uno=Uno, dos=Dos, tres=Tres, cuatro=Cuatro}
                 println(numeros.associateWith { it.length })
                 // {uno=3, dos=3, tres=4, cuatro=6}
                 ```

                 La función ***`associateBy()`*** permite crear diccionarios en los que los elementos de la colección original son valores y las claves se generan a partir de ellos (si dos elementos son iguales, solo el último permanece en el mapa). Esta función también se puede llamar con una función de transformación de valor.

                 ```kotlin
                 val numeros = listOf("uno", "dos", "tres", "cuatro", "cinco")

                 println(numeros.associateBy { it.first().toUpperCase() })
                 // {U=uno, D=dos, T=tres, C=cinco} // C=cuatro se omite

                 println(numeros.associateBy(keySelector = { it.first().toUpperCase() }, valueTransform = { it.length }))
                 // {U=3, D=3, T=4, C=5} // C=6 se omite
                 ```

                 Otra forma de crear diccionarios en los que tanto las claves como los valores se producen a partir de los elementos de una colección es la **función** ***``associate()``***, que utiliza una función *lambda* que devuelve un *Pair* con la clave y el valor correspondientes. Si cualquiera de los pares tuviera la misma clave, solo la última se agregará al mapa.

                 ```kotlin
                 val lista = listOf("a", "b", "c", "d")
                 val dict = lista.associate { it.capitalize() to it }
                 println(dict) // {A=a, B=b, C=c, D=d}

                 val intList = listOf(1, 2, 3)
                 println(intList.associate { Pair(it.toString(), it) })
                 // {1=1, 2=2, 3=3}
                 ```

               - 5.11.1.1.4. ***Flattening***: cuando se utilizan **colecciones anidadas** (por ejemplo una lista de *sets*) pueden ser útiles algunas funciones de la biblioteca estándar que proporcionan acceso a sus elementos, como la **función** ***``flatten()``*** que **devuelve una lista única de todos los elementos** de las colecciones anidadas.

                 ```kotlin
                 // List<Set<Int>>
                 val listaSet = listOf(setOf(1, 2, 3), setOf(4, 5, 6), setOf(1, 2))
                 println(listaSet.flatten()) // [1, 2, 3, 4, 5, 6, 1, 2]
                 ```

                 Por su parte, la **función** ***``flatMap()``*** toma el resultado que devuelve una función que opera sobre cada uno de los elementos de las colecciones anidadas para obtener una **lista única de todos los valores** de los elementos de las colecciones anidadas.

                 ```kotlin
                 data class ContenedorListas(val values: List<String>)

                 fun main() {
                     // List<ContenedorListas>
                     val listas = listOf(
                             ContenedorListas(listOf("uno", "dos", "tres")),
                             ContenedorListas(listOf("cuatro", "cinco", "seis")),
                             ContenedorListas(listOf("siete", "ocho"))
                     )
                     println(listas.flatMap { it.values })
                     // [uno, dos, tres, cuatro, cinco, seis, siete, ocho]
                 }
                 ```

               - 5.11.1.1.5. ***String representation***: Cuando se necesita recuperar el contenido de una colección en un formato legible, se pueden usar **funciones** que transforman las colecciones en *String*, como ***``joinToString()``*** y ***``joinTo()``***. Mientras que *joinToString()* construye un único String a partir de los elementos de la colección en función de los argumentos proporcionados, la función *joinTo()* hace lo mismo pero agrega el resultado a un objeto determinado. Cuando se utilizan con los argumentos por defecto, estas funciones devuelven un resultado similar a la función *toString()* sobre una colección.

                 ```kotlin
                 val numeros = listOf("uno", "dos", "tres", "cuatro")

                 println(numeros)            // [uno, dos, tres, cuatro]
                 println(numeros.toString()) // [uno, dos, tres, cuatro]
                 println("$numeros")         // [uno, dos, tres, cuatro]

                 println(numeros.joinToString()) // uno, dos, tres, cuatro

                 val listString = StringBuffer("Lista de números: ")
                 numeros.joinTo(listString)
                 println(listString) // Lista de números: uno, dos, tres, cuatro
                 ```

                 Para crear una representación personalizada, se pueden especificar los **parámetros** ***``separator``***, ***``prefix``*** y ***``postfix``*** de la función (ver *Default arguments y Named arguments* en el apartado 2.2).

                 ```kotlin
                 val numeros = listOf("uno", "dos", "tres", "cuatro")
                 println(numeros.joinToString(separator = " | ", prefix = "Inicio: ", postfix = "... Fin"))
                 // Inicio: uno | dos | tres | cuatro... Fin
                 ```

                 Para colecciones más grandes, es práctico especificar el **parámetro** ***``limit``*** que indica una serie de elementos que se incluirán en el resultado, mientras que si el tamaño de la colección lo excede, todos los demás elementos se reemplazarán por un solo valor determinado por el **parámetro** ***``truncated``***.

                 ```kotlin
                 val numeros = (1..100).toList()
                 println(numeros.joinToString(limit = 10, truncated = "... hasta ${numeros.last()}"))
                 // 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, ... hasta 100
                 ```

                 También se puede personalizar la representación de los elementos de la colección utilizando una función de transformación sobre cada uno de ellos.

                 ```kotlin
                 val numeros = listOf("uno", "dos", "tres", "cuatro")
                 println(numeros.joinToString { "${it.toUpperCase()}: ${it.length} letras" })
                 // UNO: 3 letras, DOS: 3 letras, TRES: 4 letras, CUATRO: 6 letras
                 ```

           - 5.11.1.2. ***Filtering***: las condiciones de filtrado se definen mediante **predicados: funciones** ***lambda*** **que toman un elemento de la colección y devuelven un valor booleano** (***true*** significa que el elemento dado coincide con el predicado, ***false*** significa lo contrario).

               - 5.11.1.2.1. ***Filtering by predicate***: La **función** básica de filtrado es ***``filter()``***. Cuando se llama con un predicado, **devuelve los elementos de la colección que coinciden**. Tanto para las listas y los conjuntos (*List* y *Set*) la colección resultante es una lista (*List*), mientras que para los diccionarios (*Map*) también es un *Map*.

                 ```kotlin
                 val numeros = listOf("uno", "dos", "tres", "cuatro")
                 val letrasMas3 = numeros.filter { it.length > 3 }
                 println(letrasMas3) // [tres, cuatro]

                 val numerosDict = mapOf("key1" to 1, "key2" to 2, "key3" to 3, "key11" to 11)
                 val filtroMap = numerosDict.filter { (key, value) -> key.endsWith("1") && value > 10}
                 println(filtroMap) // {key11=11}
                 ```

                 Los predicados en *filter()* solo pueden verificar los valores de los elementos, por lo que si se desea usar posiciones de elementos en el filtro, se debe usar ***``filterIndexed()``*** que toma un predicado con **dos argumentos: el índice y el valor de un elemento**. Para filtrar colecciones por **condiciones negativas**, se usa ***`filterNot()`*** que **devuelve una lista de elementos para los que el predicado produce** ***false***.

                 ```kotlin
                 val numeros = listOf("uno", "dos", "tres", "cuatro")
                 val filteredIdx = numeros.filterIndexed { index, s -> (index != 0) && (s.length < 5) }
                 val filteredNot = numeros.filterNot { it.length <= 3 }

                 println(filteredIdx) // [dos, tres]
                 println(filteredNot) // [tres, cuatro]
                 ```

                 También hay funciones que filtran elementos de una colección **en base a su tipo**, como ***``filterIsInstance()``*** (devuelve elementos de una colección de un tipo determinado) y ***``filterNotNull()``*** (devuelve todos los elementos que no son nulos).

                 ```kotlin
                 val num = listOf(null, 1, "dos", 3.0, "cuatro")
                 num.filterIsInstance<String>().forEach {
                     print("${it.toUpperCase()} ") // DOS CUATRO
                 }

                 val num2 = listOf(null, "uno", "dos", null)
                 num2.filterNotNull().forEach {
                     print("$it: ${it.length} ") // uno: 3 dos: 3
                 }
                 ```

               - 5.11.1.2.2. ***Partitioning***: otra **función** de filtrado es ***``partition()``***, que filtra una colección por un predicado y mantiene los elementos que no coinciden en una lista separada. Por lo tanto, **tiene un** ***Pair*** **de listas como valor de retorno**: la primera lista que contiene los elementos que coinciden con el predicado y la segunda que contiene todo lo demás de la colección original.

                 ```kotlin
                 val numeros = listOf("uno", "dos", "tres", "cuatro")
                 val (match, noMatch) = numeros.partition { it.length > 3 }

                 println(match)   // [tres, cuatro]
                 println(noMatch) // [uno, dos]
                 ```

               - 5.11.1.2.3. ***Testing predicates***: hay funciones que simplemente **prueban** (***test***) **un predicado en los elementos de una colección**. A saber, ***``any()``*** (devuelve *true* si **al menos un elemento coincide con el predicado** dado); ***``none()``*** (devuelve *true* si **ninguno de los elementos coincide con el predicado** dado); ***``all()``*** (devuelve *true* si **todos los elementos coinciden con el predicado** dado). 
                 Hay que tener en cuenta que *all()* devuelve *true* cuando se le llama con cualquier predicado válido sobre una colección vacía. Tal comportamiento se conoce en lógica como **verdad vacía** (***vacuous truth***): un enunciado condicional o universal que **solo es verdadero porque el antecedente no se puede satisfacer**.

                 ```kotlin
                 val numeros = listOf("uno", "dos", "tres", "cuatro")

                 println(numeros.any { it.endsWith("s") })  // true
                 println(numeros.none { it.endsWith("a") }) // true
                 println(numeros.all { it.endsWith("s") })  // false
                 println(emptyList<Int>().all { it > 5 })   // true
                 ```

                 *any()* y *none()* también **se pueden usar sin un predicado** y, en este caso, simplemente comprueban si la colección está vacía: *any()* devuelve *true* si hay elementos y *false* si no los hay, y *none()* hace lo contrario.

                 ```kotlin
                 val numeros = listOf("uno", "dos", "tres", "cuatro")
                 val vacia = emptyList<String>()

                 println(numeros.any()) // true
                 println(vacia.any())   // false

                 println(numeros.none()) // false
                 println(vacia.none())   // true
                 ```

           - 5.11.1.3. ***plus and minus operators***: En Kotlin, los operadores ***``plus``*** (**``+``**) y ***``minus``*** (**``-``**) se definen para las colecciones. Toman una colección como primer operando; el segundo operando puede ser un elemento u otra colección. El valor de retorno es una **nueva colección de solo lectura**.

             ```kotlin
             val numbers = listOf("one", "two", "three", "four")

             val plusList = numbers + "five"
             val minusList = numbers - listOf("three", "four")
             println(plusList)  // [one, two, three, four, five]
             println(minusList) // [one, two]
             ```

           - 5.11.1.4. ***Grouping***: La **función** básica ***``groupBy()``*** **toma una función** ***lambda*** **y devuelve un** ***Map***. En este *map*, cada clave es el resultado del *lambda* y el valor correspondiente es la lista de elementos en los que se devuelve este resultado. Esta función se puede utilizar, por ejemplo, para agrupar una lista de *strings* por su primera letra. También se puede llamar a *groupBy()* con un **segundo argumento** ***lambda***: **una función de transformación** de valor. En el *map* resultante de *groupBy()* con dos *lambdas*, las claves producidas por la **función** ***keySelector*** se mapean con los resultados de la **función de transformación** de valor en lugar de los elementos originales.

             ```kotlin
             val numbers = listOf("one", "two", "three", "four", "five")

             println(numbers.groupBy { it.first().toUpperCase() })
             println(numbers.groupBy(keySelector = { it.first() }, valueTransform = { it.toUpperCase() }))

             // {O=[one], T=[two, three], F=[four, five]}
             // {o=[ONE], t=[TWO, THREE], f=[FOUR, FIVE]}
             ```

             Si se desea agrupar elementos y luego **aplicar una operación a todos los grupos a la vez**, se usa la **función** ***``groupingBy()``***, que devuelve una instancia de tipo ***Grouping***. La instancia de *Grouping* permite aplicar operaciones a todos los grupos de manera *lazy*: los grupos se crean **antes de la ejecución de la operación**.  
             *Grouping* admite las siguientes operaciones: ***``eachCount()``*** cuenta los elementos de cada grupo; ***``fold()``*** y ***``reduce()``*** realizan operaciones de plegado (*fold*) y reducción (*reduce*) en cada grupo como una colección separada y devuelven los resultados; ***``aggregate()``*** aplica una operación dada después de todos los elementos de cada grupo y devuelve el resultado. Esta es la forma genérica de realizar cualquier operación en un *Grouping*. Se usa para implementar operaciones personalizadas cuando *fold* o *reduce* no es suficiente.

             ```kotlin
             val numbers = listOf("one", "two", "three", "four", "five", "six")
             println(numbers.groupingBy { it.first() }.eachCount()) // {o=1, t=2, f=2, s=1}
             ```

           - 5.11.1.5. ***Retrieving collection parts***: Estas funciones utilizan varias formas, como listar sus posiciones explícitamente o especificar el tamaño del resultado, para obtener una colección resultante que **selecciona y toma ciertos elementos** de la colección a la que se aplican.

               - 5.11.1.5.1. ***Slice***: la **función** ***``slice()``*** devuelve una lista de elementos de la colección con unos índices determinados que se pasan como argumento, ya sea como un rango o como una colección de valores enteros.

                 ```kotlin
                 val numeros = listOf("uno", "dos", "tres", "cuatro", "cinco", "seis")
                 println(numeros.slice(1..3))            // [dos, tres, cuatro]
                 println(numeros.slice(0..4 step 2))     // [uno, tres, cinco]
                 println(numeros.slice(setOf(3, 5, 0)))  // [cuatro, seis, uno]
                 ```

               - 5.11.1.5.2. ***Take and drop***: para obtener un número específico de elementos se usan las **funciones** ***``take()``*** y ***``takeLast()``***, con la diferencia de que *take()* cuenta a partir del primer elemento y *takeLast()* a partir del último, hacia atrás. Cuando se llama a estas funciones con un número mayor que el tamaño de la colección, ambas devuelven la colección completa. Y para tomar todos los elementos excepto un número dado de elementos, ya sea desde el primer o último elemento, se usan las **funciones** ***``drop()``*** y ***``dropLast()``*** respectivamente.

                 ```kotlin
                 val numeros = listOf("uno", "dos", "tres", "cuatro", "cinco", "seis")
                 println(numeros.take(3))     
                 // [uno, dos, tres] // los tres primeros elementos
                 println(numeros.takeLast(3)) 
                 // [cuatro, cinco, seis] // los tres últimos elementos
                 println(numeros.drop(1))     
                 // [dos, tres, cuatro, cinco, seis] // todos excepto el primer elemento
                 println(numeros.dropLast(5)) 
                 // [uno] // todos exceptos los últimos cinco elementos
                 ```

                 También se pueden utilizar predicados para definir el número de elementos que se deben tomar o eliminar. Hay cuatro funciones similares a las descritas anteriormente: ***``takeWhile()``***, toma los elementos hasta que uno (al cual excluye) no coincide con el predicado (si el primer elemento de la colección no coincide con el predicado, el resultado está vacío); ***``takeLastWhile()``***, selecciona el rango de elementos que coinciden con el predicado desde el final de la colección (o si el último elemento de la colección no coincide con el predicado, el resultado está vacío); ***``dropWhile()``***, con el mismo predicado es lo opuesto a *takeWhile()*, y devuelve los elementos desde el primero que no coincide con el predicado hasta el final; ***``dropLastWhile()``***, con el mismo predicado es lo opuesto a *takeLastWhile()*, y devuelve los elementos que no coinciden con el predicado empezando desde el último.

                 ```kotlin
                 val numeros = listOf("uno", "dos", "tres", "cuatro", "cinco", "seis")
                 println(numeros.takeWhile { !it.startsWith('c') })
                 // [uno, dos, tres]
                 println(numeros.takeLastWhile { it != "tres" })
                 // cuatro, cinco, seis]
                 println(numeros.dropWhile { it.length == 3 })
                 // [tres, cuatro, cinco, seis]
                 println(numeros.dropLastWhile { it.contains('i') })
                 // [uno, dos, tres, cuatro]
                 ```

               - 5.11.1.5.3. ***Chunked***: para dividir una colección en partes de un tamaño determinado se usa la **función** ***``chunked()``***, que toma un solo argumento (el tamaño del fragmento) y devuelve una lista de listas del tamaño dado. La primera sublista contiene los primeros *n* elementos, la segunda los *n* siguientes y así hasta el final de la colección, por lo que la última sublista puede ser más pequeña si no hay suficientes elementos para completarla.

                 ```kotlin
                 val numeros = (0..13).toList()
                 println(numeros.chunked(3))
                 // [[0, 1, 2], [3, 4, 5], [6, 7, 8], [9, 10, 11], [12, 13]]
                 ```

                 También se puede aplicar una transformación a cada fragmento o sublista utilizando una función *lambda* con *chunked()*.

                 ```kotlin
                 val numeros = (0..13).toList()
                 println(numeros.chunked(3) { it.sum() })
                 // [3, 12, 21, 30, 25]
                 ```

               - 5.11.1.5.4. ***Windowed***: la **función** ***``windowed()``*** también devuelve rangos de elementos de un tamaño determinado, pero a diferencia de *chunked()*, cada uno de los fragmentos o ventanas (*windows*) empieza a partir de cada elemento de la colección.

                 ```kotlin
                 val numeros = listOf("uno", "dos", "tres", "cuatro", "cinco")
                 println(numeros.windowed(3))
                 // [[uno, dos, tres], [dos, tres, cuatro], [tres, cuatro, cinco]]
                 ```

                 *windowed()* también admite **parámetros** opcionales: 

                 * ***``step``*** define la distancia entre los primeros elementos de dos ventanas adyacentes. Por defecto el valor es 1, por lo que el resultado contiene ventanas que comienzan con todos los elementos. Si incrementa el paso a 2, solo recibirá ventanas a partir de elementos impares: primero, tercero, y así sucesivamente.
                 * ***``partialWindows``***, por defecto *true*, incluye la ventana final de tamaño más pequeño, y si es *false* la excluye.

                 Con *windowed()* también se puede aplicar una transformación a los rangos devueltos con una función *lambda*.

                 ```kotlin
                 val numeros = (1..10).toList()
                 println(numeros.windowed(3, step = 2, partialWindows = true))
                 // [[1, 2, 3], [3, 4, 5], [5, 6, 7], [7, 8, 9], [9, 10]]
                 println(numeros.windowed(3) { it.sum() })
                 // [6, 9, 12, 15, 18, 21, 24, 27]
                 ```

                 Por último, para construir ventanas de dos elementos hay una **función** específica: ***``zipWithNext()``***, que para cada elemento crea pares de elementos adyacentes. También se puede llamar con una función de transformación tomando dos elementos de la colección como argumentos.

                 ```kotlin
                 val numeros = listOf("uno", "dos", "tres", "cuatro", "cinco")
                 println(numeros.zipWithNext())
                 // [(uno, dos), (dos, tres), (tres, cuatro), (cuatro, cinco)]
                 println(numeros.zipWithNext() { s1, s2 -> s1.length > s2.length })
                 // [false, false, false, true]
                 ```

           - 5.11.1.6. ***Retrieving single elements***: una lista es una colección ordenada, por lo que cada elemento tiene una posición que se puede usar para referir al elemento. En cambio, un *set* no es una colección ordenada, aunque aún así los elementos se almacenan en cierto orden (orden de inserción en *LinkedHashSet*, orden natural en *SortedSet*, etc.). Por lo tanto, los resultados que se obtienen de una posición son impredecibles, a menos que se conozca la implementación específica de *Set* utilizada.

               - 5.11.1.6.1. ***Retrieving by position***: para recuperar un elemento en una posición específica, existe la **función** ***``elementAt()``*** que se llama con un número entero como argumento, y devuelve el elemento de la colección en esa posición (recordando que el primer elemento está en la posición 0 y el último en la posición tamaño - 1). Es útil para colecciones que no proporcionan (o que no se sabe que proporcionen) acceso indexado, si bien en el caso de *List* se suele utilizar el operador de acceso indexado *``get()``* o el índice entre ``[]``.También hay herramientas útiles para recuperar el primer y último elemento de la colección: ***``first()``*** y ***``last()``***.

                 ```kotlin
                 val numeros = linkedSetOf("uno", "dos", "tres", "cuatro", "cinco")
                 println(numeros.elementAt(3)) // cuatro

                 val numerosOrdenados = sortedSetOf("uno", "dos", "tres", "cuatro")
                 println(numerosOrdenados.elementAt(0)) // cuatro

                 val numerosLista = listOf(1, 2, 3, 4, 5)
                 println(numerosLista.elementAt(2)) // 3
                 println(numerosLista.get(2))       // 3
                 println(numerosLista[2])           // 3

                 println(numerosLista.first()) // 1
                 println(numerosLista.last())  // 5
                 ```

                 Para evitar excepciones al recuperar elementos con posiciones no existentes se deben usar variaciones seguras de *elementAt()* como ***``elementAtOrNull()``*** (devuelve *null* cuando la posición especificada está fuera de los límites de la colección); o ***``elementAtOrElse()``*** (toma como argumento, además del índice de posición, una función *lambda* con ese mismo índice, y cuando se llama con una posición fuera de los límites devuelve el resultado de esa *lambda*).

                 ```kotlin
                 val numeros = listOf("uno", "dos", "tres", "cuatro", "cinco")
                 println(numeros.elementAtOrNull(5)) // null
                 println(numeros.elementAtOrElse(5) { index -> "Valor no definido para el índice $index" })
                 ```

               - 5.11.1.6.2. ***Retrieving by condition***: las **funciones** *``first()``* y *``last()``* también permiten buscar elementos en una colección que coincidan con un predicado determinado. Así, cuando se llama a *first()* con un predicado que comprueba un elemento de la colección, devuelve el primer elemento en el que el predicado se traduce en verdadero. A su vez, *last()* con un predicado devuelve el último elemento que coincide con él. Si ningún elemento coincide con el predicado, ambas funciones lanzan excepciones, y para evitarlo se usan ***``firstOrNull()``*** y ***``lastOrNull()``***, que devuelven *null* si no encuentran elementos coincidentes. Otra alternativa para evitar estas excepciones es usar ***``find()``*** en lugar de *``firstOrNull()``* y ***``findLast()``*** en lugar de *``lastOrNull()``*.

                 ```kotlin
                 val numeros = listOf("uno", "dos", "tres", "cuatro", "cinco", "seis")
                 println(numeros.first { it.length > 3 }) // tres
                 println(numeros.last { it.startsWith("c") }) // cinco

                 println(numeros.firstOrNull { it.length > 6 }) // null

                 println(numeros.find { it.length > 6 }) // null
                 ```

               - 5.11.1.6.3. ***Random element***: si se necesita recuperar un elemento arbitrario de una colección, se puede usar la **función** ***``random()``***, que se puede llamar sin argumentos o con un objeto tipo *Random* como fuente de la aleatoriedad.

                 ```kotlin
                 import kotlin.random.Random
                 import kotlin.random.Random.Default.nextInt

                 fun main() {
                     val numeros = listOf("uno", "dos", "tres", "cuatro", "cinco", "seis")
                     println(numeros.random())
                     println(numeros.random(Random(nextInt(0, numeros.size))))
                 }
                 ```

               - 5.11.1.6.4. ***Checking existence***: para verificar si un elemento está presente en la colección, se puede usar la **función** ***``contains()``***, que devuelve *true* si existe un elemento en la colección igual al argumento utilizado en la llamada a la función. También se puede llamar a *contains()* con la **palabra reservada** ***``in``***. Además, para comprobar la presencia de varios argumentos, se puede utilizar ***``containsAll()``*** con una colección de elementos que queremos verificar como argumento. También se puede verificar si una colección contiene algún elemento o está vacía con ***``isNotEmpty()``*** y con ***``isEmpty()``***.

                 ```kotlin
                 val numeros = listOf("uno", "dos", "tres", "cuatro", "cinco", "seis")
                 println(numeros.contains("cuatro")) // true
                 println("cero" in numeros)          // false

                 println(numeros.containsAll(listOf("cuatro", "dos"))) // true
                 println(numeros.containsAll(listOf("uno", "cero")))   // false

                 println(numeros.isEmpty())    // false
                 println(numeros.isNotEmpty()) // true
                 ```

           - 5.11.1.7. ***Ordering***: El orden de los elementos es un aspecto importante en ciertos tipos de colecciones. Por ejemplo, dos listas con los mismos elementos no son iguales si sus elementos están ordenados de manera diferente. A continuación se describen las funciones de ordenación que **se aplican a las colecciones de solo lectura**. Estas funciones devuelven su resultado como una nueva colección que contiene los elementos de la colección original en el orden solicitado.  

               - 5.11.1.7.1. En Kotlin, los objetos se pueden ordenar siguiendo distintos criterios. Primero, hay un orden natural para los objetos que heredan la **interfaz** ***``Comparable``*** y que se utiliza para clasificar los objetos del mismo tipo cuando no se especifica ningún otro criterio. Por ejemplo, los tipos numéricos utilizan el orden numérico tradicional (1 es mayor que 0 y -3.4f es mayor que -5f) y *Char* y *String* usan el orden lexicográfico ('b' es mayor que 'a' y 'mundo' es mayor que 'hola').
                 Además, el usuario puede definir un orden natural para otro tipo haciendo que herede de la interfaz *Comparable*, lo que requiere implementar la **función** ***``compareTo()``***. Esta función debe tomar otro objeto del mismo tipo como argumento y devuelve un valor entero que muestra qué objeto es mayor: **si es positivo el objeto receptor es mayor, si es negativo el argumento es mayor y si es cero ambos objetos son iguales**. Un ejemplo de una clase que se puede usar para ordenar versiones:

                 ```kotlin
                 class Version(private val primero: Int, private val segundo: Int) : Comparable<Version> {
                     override fun compareTo(other: Version): Int {
                         return when {
                             this.primero != other.primero -> this.primero - other.primero
                             this.segundo != other.segundo -> this.segundo - other.segundo
                             else -> 0
                         }
                     }
                 }

                 fun main() {
                     println(Version(1, 2) > Version(1, 3)) // false
                     println(Version(2, 0) > Version(1, 5)) // true
                 }
                 ```

                 Se pueden ordenar instancias de cualquier tipo de la forma que interese, pudiendo definir un orden para objetos no comparables o bien definir un orden no natural para un tipo comparable. Para definir un orden determinado para un tipo, se debe crear un ***Comparator*** para él. *Comparator* contiene la **función** ***``compare()``*** que toma dos instancias de una clase y devuelve un resultado entero que se interpreta como se vio antes en la función *compareTo()*.

                 ```kotlin
                 val ordenLongitud = Comparator { str1: String, str2: String -> str1.length - str2.length }
                 println(listOf("aaa", "bb", "c").sortedWith(ordenLongitud))
                 // [c, bb, aaa]
                 ```

                 En el ejemplo anterior se organizan las palabras por su longitud en lugar del orden lexicográfico natural. Se puede definir un **comparador** de manera más concisa con la **función** ***``compareBy()``***, que toma una función *lambda* que produce un valor comparable de una instancia y define el orden personalizado como el orden natural de los valores producidos, por lo que se puede escribir el código anterior así:

                 ```kotlin
                 println(listOf("aaa", "bb", "c").sortedWith(compareBy { it.length }))
                 ```

                 El paquete de colecciones de Kotlin proporciona funciones específicas para ordenar los elementos de las colecciones por su **orden natural**, por **criterios personalizados** o **de manera aleatoria**. Estas funciones **se aplican a las colecciones de solo lectura** y devuelven como resultado una nueva colección que contiene los mismos elementos que la colección original pero presentados en el orden requerido.  

               - 5.11.1.7.2. ***Natural order***: las **funciones** básicas ***``sorted()``*** y ***``sortedDescending()``*** devuelven los elementos de una colección ordenados en secuencia ascendente y descendente **según su orden natural**. Estas funciones se aplican a las colecciones de elementos comparables.

                 ```kotlin
                 val numeros = listOf("uno", "dos", "tres", "cuatro")
                 println("En orden ascendente: ${numeros.sorted()}")
                 println("En orden descendente: ${numeros.sortedDescending()}")
                 ```

               - 5.11.1.7.3. ***Custom order***: Para utilizar **criterios personalizados** u **ordenar objetos no comparables** podemos usar las **funciones** ***``sortedBy()``*** y ***``sortedByDescending()``***, que toman una función que asigna los elementos de la colección a valores de *Comparable* y ordena la colección en el orden natural de esos valores. Para definir un criterio personalizado para ordenar una colección, se puede proporcionar un *Comparator* propio, llamando a la **función** ***``sortedWith()``***.

                 ```kotlin
                 val numeros = listOf("uno", "dos", "tres", "cuatro", "cinco")

                 val ordenLongitud = numeros.sortedBy { it.length }
                 println("Orden ascendente por longitud: $ordenLongitud")

                 val ultimaLetra = numeros.sortedByDescending { it.last() }
                 println("Orden descendente por última letra: $ultimaLetra")

                 println("Orden ascendente por longitud: ${numeros.sortedWith(compareBy { it.length })}")
                 ```

               - 5.11.1.7.4. ***Reverse order***: se pueden recuperar la colección en orden inverso utilizando la **función** ***``reversed()``***. Hay que tener en cuenta que *reversed()* devuelve una nueva colección con copias de los elementos y, por tanto, si más adelante cambia la colección original, esto no afectará a los resultados obtenidos previamente con *reversed()*. En cambio, la **función** ***``asReversed()``*** devuelve una vista invertida de la misma instancia de la colección, por lo que es preferible frente a *reversed()* al requerir menos recursos siempre que la lista original no vaya a cambiar o sea inmutable.

                 ```kotlin
                 val numeros = mutableListOf("uno", "dos", "tres", "cuatro")

                 val ordenInverso = numeros.reversed()
                 val ordenInversoAs = numeros.asReversed()
                 println(ordenInverso)   // [cuatro, tres, dos, uno]
                 println(ordenInversoAs) // [cuatro, tres, dos, uno]

                 numeros.add("cinco")
                 println(ordenInverso)   // [cuatro, tres, dos, uno]
                 println(ordenInversoAs) // [cinco, cuatro, tres, dos, uno]
                 ```

               - 5.11.1.7.5. ***Random order***: la **función** ***``shuffled()``*** devuelve una nueva lista que contiene los elementos de la colección mezclados en un **orden aleatorio**, pudiéndose llamar sin argumentos o con un objeto tipo ***``Random``***.

                 ```kotlin
                 val numeros = mutableListOf("uno", "dos", "tres", "cuatro")
                 println(numeros.shuffled())
                 // [tres, uno, cuatro, dos]
                 ```

           - 5.11.1.8. ***Aggregate operations***: las colecciones de Kotlin contienen funciones para operaciones de agrupación (o agregación) de uso común: operaciones que **devuelven un valor único en función del contenido** de la colección, como por ejemplo, los valores mínimo y máximo (***``min()``*** y ***``max()``***), el número de elementos (***``count()``***), la suma de todos los elementos (***``sum()``***) o el valor promedio (***``average()``***). Un ejemplo de operación de agrupamiento es el cálculo del promedio de temperatura a partir de los valores de temperatura diaria durante un mes.

             ```kotlin
             fun main() {
                 val gradosC = listOf(25.5, 26.1, 24.3, 25.2)

              println("Datos: ${gradosC.count()}")                // Datos: 4
              println("Máximo: %.1f".format(gradosC.max()))       // Máximo: 26.1
              println("Mínimo: %.1f".format(gradosC.min()))       // Mínimo: 24.3
              println("Media: %.1f".format(gradosC.average()))    // Media: 25.3
              println("Suma: %.1f".format(gradosC.sum()))         // Suma: 101.1
             }
             ```

             También hay otras funciones que sirven para recuperar los elementos más pequeños y más grandes mediante funciones de selección o mediante un comparador personalizado, como son:

             + ***``maxBy()``*** y ***``minBy()``***, que mediante una función de selección devuelven el elemento con el valor más grande o más pequeño (o en caso de igualdad, el primero que encuentra)
             + ***``maxWith()``*** y ***``minWith()``***, que tomando como referencia un objeto de tipo *Comparator* devuelven el elemento más grande o más pequeño (o en caso de igualdad, el primero que encuentra)
            
             Además, hay funciones de suma avanzada o compleja que toman una función que opera sobre los elementos y devuelven la **suma total** de los valores de retorno:

             + ***``sumBy()``*** se aplica a funciones que devuelven valores de tipo *Int* en los elementos de la colección.
             + ***``sumByDouble()``*** se aplica a funciones que devuelven valores de tipo *Double*.


             ```kotlin
             val numeros = listOf(1, 2, 3, 4, 5)
             val moduloDos = numeros.minBy { it % 2 }
             println(moduloDos) // 2

             val strings = listOf("uno", "dos", "tres", "cuatro", "cinco")
             val masLargo = strings.maxWith(compareBy { it.length })
             println(masLargo) // cuatro

             println(numeros.sumBy { it * 2 }) // 30
             println(numeros.sumByDouble { it.toDouble() / 2 }) // 7.5
             ```

               - 5.11.1.8.1. ***Fold and reduce***: para casos más específicos, existen las **funciones** ***``reduce()``*** y ***``fold()``***, que aplican la operación proporcionada a los elementos de la colección de forma secuencial y devuelven el **resultado acumulado**. La operación toma dos argumentos: el valor acumulado previamente y el elemento de colección, con la diferencia de que *fold()* toma un valor inicial y lo usa como el valor acumulado en el primer paso, mientras que *reduce()* usa en el primer paso el primer y el segundo elemento como argumentos de la operación. Para aplicar estas funciones en el orden inverso, se usan las **funciones** ***``reduceRight()``*** y ***``foldRight()``***, que operan de manera similar a *reduce()* y *fold()* pero comienzan desde el último elemento y avanzan hacia el anterior hasta el primero (además hay que tener en cuenta que en estas funciones los argumentos de la operación cambian su orden: primero va el elemento y luego el valor acumulado). También podemos realizar operaciones que toman los índices de los elementos como parámetros con las **funciones** ***``reduceIndexed()``*** y ***``foldIndexed()``*** pasando el índice del elemento como primer argumento de la operación (y las funciones inversas que aplican las operaciones a los elementos de la colección de derecha a izquierda: ***``reduceRightIndexed()``*** y ***``foldRightIndexed()``***).

                 ```kotlin
                 val numeros = listOf(5, 2, 10, 4)

                 val acumulado = numeros.reduce { sum, element -> sum + element }
                 println(acumulado) // 5 -> (5 + 2) -> (7 + 10) -> (17 + 4) = 21

                 val desdeDiez = numeros.fold(10) { sum, element -> sum + element }
                 println(desdeDiez) // 10 + 5 -> (15 + 2) -> (17 + 10) -> (27 + 4) = 31

                 val acumuladoDoble = numeros.fold(0) { sum, element -> sum + element * 2 }
                 println(acumuladoDoble) // 5x2 -> (10 + 2x2) -> (14 + 10x2) -> (34 + 4x2) = 42

                 // con reduce el primer elemento no se multiplica por 2:
                 val acumuladoDobleReduce = numeros.reduce { sum, element -> sum + element * 2 }
                 println(acumuladoDobleReduce) // 5 -> (5 + 2x2) -> (9 + 10x2) -> (29 + 4x2) = 37

                 val acumuladoDobleDerecha = numeros.foldRight(0) { element, sum -> sum + element * 2 }
                 println(acumuladoDobleDerecha) // 42

                 val sumaPosicionesPares = numeros.foldIndexed(0) { idx, suma, elemento -> if (idx % 2 == 0) suma + elemento else suma }
                 println(sumaPosicionesPares) // 5 -> 5 -> (5 + 10) -> 15 = 15
                 // idx = 0 -> 5 (suma + elemento)
                 // idx = 1 -> 5 (suma)
                 // idx = 2 -> 5 + 10 (suma + elemento)
                 // idx = 3 -> 15 (suma)
                 ```

       - 5.11.2. **Operaciones de escritura**: Para colecciones mutables, hay operaciones de escritura que **cambian el estado de la colección**. Dichas operaciones incluyen agregar, eliminar y actualizar elementos. Para ciertas operaciones, existen pares de funciones para realizar **la misma operación**: una aplica la operación *in situ* y la otra devuelve el resultado como una colección separada. Por ejemplo, ***``sort()``*** ordena una colección mutable en el lugar, por lo que su estado cambia; ***``sorted()``*** crea una nueva colección que contiene los mismos elementos de forma ordenada.

           - 5.11.2.1. ***Adding elements***: para agregar un solo elemento a una lista o a un conjunto se usa la **función** ***``add()``***, que agrega el nuevo elemento al final de la colección.  También está la **función** ***``addAll()``*** que utiliza como argumento un iterable, una secuencia o un *array* para agregar cada elemento de dicho objeto, sin necesidad de que los tipos del argumento y de la colección receptora sean similares, esto es, se pueden agregar todos los elementos de un conjunto (*Set*) a una lista (*List*). Cuando *addAll()* se invoca sobre listas, los nuevos elementos se agregan en el mismo orden en que aparecen en el argumento. También se puede llamar a *addAll()* especificando una posición determinada como primer argumento, y entonces el primer elemento del argumento se inserta en esa posición seguido por el resto de elementos del argumento, desplazando a los elementos de la colección receptora hasta el final.

             ```kotlin
             val numeros = mutableListOf(1, 2, 5, 6)
             numeros.addAll(arrayOf(7, 8))
             println(numeros) // [1, 2, 5, 6, 7, 8]
             numeros.addAll(2, setOf(3, 4)) // el primer argumento es el índice de inserción
             println(numeros) // [1, 2, 3, 4, 5, 6, 7, 8]
             ```

             Otra manera de agregar elementos es utilizando el **operador** **``+=``**, tanto con un elemento individual como con otra colección, insertando los nuevos elementos al final de la colección:

             ```kotlin
             val numeros = mutableListOf("uno", "dos")
             numeros += "tres"
             println(numeros) // [uno, dos, tres]
             numeros += listOf("cuatro", "cinco")
             println(numeros) // [uno, dos, tres, cuatro, cinco]
             ```

           - 5.11.2.2. ***Removing elements***: para eliminar un elemento de una colección mutable se usa la **función** ***``remove()``***, que toma un valor y, en caso de existir en la colección, elimina la primera ocurrencia de ese valor (en caso contrario, no hace nada). Para eliminar múltiples elementos a la vez, existen las siguientes funciones:

             + ***``removeAll()``***, que elimina todos los elementos que están presentes en la colección de argumentos. Alternativamente, se puede llamar con un predicado como argumento y en este caso la función elimina todos los elementos para los cuales el predicado resulta verdadero (*true*)
             + ***``retainAll()``*** que, al contrario que la anterior, elimina todos los elementos excepto los de la colección de argumentos. Cuando se usa con un predicado, deja solo los elementos que coinciden
             + ***``clear()``***, que elimina todos los elementos de una lista y la deja vacía


             ```kotlin
             val numeros = mutableListOf(1, 2, 3, 4, 3)
             numeros.remove(3) // elimina el primer 3
             println(numeros) // [1, 2, 4, 3]
             numeros.remove(5) // no elimina nada
             println(numeros) // [1, 2, 4, 3]

             numeros.retainAll { it >= 3 }
             println(numeros) // [3, 4]
             numeros.clear()
             println(numeros) // []

             val numerosSet = mutableSetOf("uno", "dos", "tres", "cuatro")
             numerosSet.removeAll(setOf("uno", "dos"))
             println(numerosSet) // [tres, cuatro]
             ```

             Y al igual que para agregar elementos de una colección se usa el operador **``+=``**, se puede eliminar elementos con el **operador** **``-=``**, tanto con un elemento individual (elimina la primera aparición del mismo) como con otra colección (se eliminan todas las apariciones de sus elementos).

             ```kotlin
             val numeros = mutableListOf("uno", "dos", "tres", "tres", "cuatro", "cuatro")
             numeros -= "tres"
             println(numeros) // [uno, dos, tres, cuatro, cuatro]
             numeros -= listOf("cuatro", "cinco")
             println(numeros) // [uno, dos, tres]
             ```



---



### 6. Otros

   - 6.1. ***Operator overloading***: La sobrecarga de operadores permite asignar comportamientos a los operadores **para que actúen sobre tipos creados por el programador**. Para implementar un operador, se proporciona una **función miembro** o una **función de extensión** con un nombre fijo, para el tipo correspondiente. Las funciones que **sobrecargan** operadores necesitan ser marcadas con el **modificador** ***``operator``***. 
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

   - 6.2. ***Destructuring declarations***: A veces es conveniente **desestructurar un objeto** en varias variables. Una declaración de desestructuración **crea múltiples variables a la vez** que luego pueden usarse de forma **independiente**. Se compilan utilizando **funciones** ***``componentN()``***, que algunas estructuras implementan por defecto, como las ***data classes*** y las **colecciones** (estas últimas, como **funciones de extensión**). 
     Si una variable no se necesita, se puede colocar un guión bajo (**``_``**) en vez del nombre.

     ```kotlin
     data class Data(val name:String,val age:Int) 

     // Una función que retorna un objeto Data (dos valores)
     fun sendData():Data{ 
       return Data("Jack",30) 
     } 

     fun main(){ 
       val obj = sendData() 
       // Usando la instancia para acceder a las propiedades
       println("Name is ${obj.name}") 
       println("Age is ${obj.age}") 

       // Creando dos variables usando destructuring declaration
       val (name,age) = sendData() 
       println("Name is $name") 
       println("Age is $age") 
     }

     // Internamente, se compila:
     val name = obj.component1()
     val age = obj.component2()
     ```

   - 6.3. ***Type checks*** y ***Casts***: los **operadores** ***``is``*** y ***``!is``*** sirven para **comprobar, en tiempo de ejecución, si un objeto es de un tipo determinado o no**. Esto, que en Kotlin se conoce como ***Smart Cast***, permite que el compilador inserte un *casting* seguro cuando se necesite:

     ```kotlin
     if (obj is String){
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

   - 6.4. ***This expression***: para denotar el receptor (*receiver*) actual, se usan **expresiones con** ***``this``***: en un miembro de una clase, *this* refiere al objeto actual de esa clase; en una función de extensión o una *function literal* con receptor, *this* denota el parámetro receptor (*receiver*) que se pasa en el lado izquierdo del punto.  
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
     fun printLine() { println("Top-level function") }

     class A {
         fun printLine() { println("Member function") }

         fun invokePrintLine(omitThis: Boolean = false)  {
             if (omitThis) printLine()
             else this.printLine()
         }
     }

     A().invokePrintLine() // Member function
     A().invokePrintLine(omitThis = true) // Top-level function
     ```

   - 6.5. ***Null safety***: en Kotlin, el sistema de tipos distingue entre referencias que pueden contener nulos (***nullable references***) y aquellas que no pueden (***non-null references***). Para permitir valores nulos, se puede declarar una variable como anulable con el **operador de llamada seguro** **``?``** (***safe call operator***). Se puede, por ejemplo, colocar una llamada segura en el lado izquierdo de una asignación. Luego, si uno de los receptores en la cadena de llamadas seguras es nulo, la asignación se omite y la expresión de la derecha no se evalúa en absoluto:

     ```kotlin
     // If either 'person' or 'person.department' is null, the function is not called:
     person?.department?.head = managersPool.getManager()
     ```

     Cuando se tiene una referencia *b* que acepta valores nulos, se puede decir "si *b* no es nulo, utilícelo; de lo contrario, use algún valor no nulo". En lugar de usar una **expresión** ***if*** completa, esto se puede expresar con el **operador** ***Elvis*** **``?:``**

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
         // ...
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

     Si se tiene una colección de elementos de un tipo que acepta valores nulos y se desea filtrar elementos no nulos, se puede utilizar la **función** ***``filterNotNull``***:

     ```kotlin
     val nullableList: List<Int?> = listOf(1, 2, null, 4)
     val intList: List<Int> = nullableList.filterNotNull()
     println(intList.joinToString()) // Imprime: 1, 2, 4
     ```

   - 6.6. ***Reflection***: es un conjunto de características de lenguaje y biblioteca que permite realizar una **introspección de la estructura** de su propio programa **en tiempo de ejecución**. Kotlin hace que las funciones y propiedades sean ***ciudadanos de primera clase*** en el lenguaje (ver *funciones de Orden Superior*), y su introspección (es decir, saber un nombre o un tipo de una propiedad o función en tiempo de ejecución) está estrechamente entrelazado con el simple uso de un estilo funcional o reactivo.  
     La característica más básica de la reflexión es **obtener la referencia a una clase** Kotlin, utilizando el **operador** **``::``**. El valor de esa referencia es de tipo ***KClass***. Para obtener una referencia a una clase Java, se debe agregar ***``.java``***

     ```kotlin
     val c = MyClass::class  // Referencia a clase Kotlin
     val c = MyClass::class.java // Referencia a clase Java
     ```

     También se puede **obtener la referencia a una función**, **a una propiedad o a un constructor**. Este tipo de referencias se denominan **invocables** (***Callable***) y pueden usarse como ***function types***. El supertipo común para todas las referencias *callable* es ***KCallable\<out R\>***.

     ```kotlin
     fun main() {

         fun greetFunction() {
             println("Hola")
         }

         val greet = ::greetFunction

         greet() // Imprime: Hola
     }
     ```

     ***Function references***: en el siguiente ejemplo, **se pasa una función como parámetro de otra función**. El operador **``::``** puede usarse con funciones sobrecargadas cuando el tipo esperado se desprende del contexto.

     ```kotlin
     fun isOdd(x: Int) = x % 2 != 0
     fun isOdd(s: String) = s == "brillig" || s == "slithy" || s == "tove"

     val numbers = listOf(1, 2, 3)
     println(numbers.filter(::isOdd)) // refers to isOdd(x: Int)
     ```

     ***Property references***: para acceder a **propiedades como objetos de primera clase** en Kotlin, también podemos usar el **operador** **``::``**

     ```kotlin
     val x = 1

     fun main() {
         println(::x.get()) // 1
         println(::x.name)  // x
     }
     ```

     ***Constructor references***: los constructores pueden referenciarse de la misma forma que las funciones y las propiedades. Además del operador **``::``** se le agrega el **nombre de la clase**. La siguiente función, espera una función sin parámetros como parámetro y retorna un tipo *Foo*:

     ```kotlin
     class Foo

     fun function(factory: () -> Foo) {
         val x: Foo = factory()
     }
     ```

     Utilizando ***``::Foo``***, el constructor sin argumentos de la clase *Foo*, se podría llamar directamente así:

     ```kotlin
     function(::Foo)
     ```

   - 6.7. ***Scope functions*** (***``let``***, ***``run``***, ***``with``***, ***``apply``*** y ***``also``***): Funciones cuyo único propósito es ejecutar un bloque de código **dentro del contexto de un objeto**. Cuando se llama a una de estas funciones en un objeto con una expresión *lambda* proporcionada, forma un **ámbito** (***scope***) **temporal**.  En este ámbito, se puede **acceder al objeto sin su nombre**. 

     ```kotlin
     data class Person(var name: String, var age: Int, var city: String) {
         fun moveTo(newCity: String) { city = newCity }
         fun incrementAge() { age++ }
     }

     fun main() {
         Person("Alice", 20, "Amsterdam").let {
             println(it)
             it.moveTo("London")
             it.incrementAge()
             println(it)
         }
     }

     // Imprime:
     // Person(name=Alice, age=20, city=Amsterdam)
     // Person(name=Alice, age=21, city=London)
     ```

     Hay dos diferencias principales entre cada función de alcance: la forma de referirse al **objeto de contexto** y el **valor de retorno**.

       - 6.7.1. ***Context object*** (***this*** o ***it***): ***``run``***, ***``with``*** y ***``apply``*** utilizan el objeto de contexto como **receptor o destinatario** (***receiver***) del *lambda*, es decir **el objeto mismo**, y para eso usan la palabra reservada ***``this``***. En cambio, ***``let``*** y ***``also``*** utilizan el objeto de contexto como **argumento** del *lambda*, y para eso usan la palabra reservada ***``it``***.

       - 6.7.2. ***Return value***: ***``apply``*** y ***``also``*** retornan el mismo ***context object***. En cambio, ***``let``***, ***``run``*** y ***``with``*** retornan el **resultado del** ***lambda***. 


   - 6.8. ***SAM*** (***Single Abstract Method***) ***conversions***: Esta característica sólo funciona para **interoperar con Java**. Las ***functions literals*** (*lambdas* y funciones anónimas) de Kotlin pueden ser **convertidas** automáticamente en **implementaciones de ***interfaces*** funcionales** (*interfaces* con un único método abstracto) de Java. Y como tiene un único método abstracto, **se puede omitir el nombre al implementarlo**:

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



---
---



### Referencias:

- [Kotlin Documentation](https://kotlinlang.org/docs/reference/)
- [Learn Kotlin by Example](https://play.kotlinlang.org/byExample/overview)
- [Kotlin Koans](https://play.kotlinlang.org/koans/overview)
- [DevExperto](https://devexperto.com/)
- [Kotlin From Scratch](https://code.tutsplus.com/series/kotlin-from-scratch--cms-1209)
- [Kotlin programmer dictionary](https://blog.kotlin-academy.com/kotlin-programmer-dictionary-2cb67fff1fe2)
- [Kotlin Doc Blogspot - Kotlin y Android](https://kotlindoc.blogspot.com/)
