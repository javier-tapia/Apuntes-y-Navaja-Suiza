## *Reflection*
Es un conjunto de características de lenguaje y librería que permite realizar una **introspección de la estructura** de su propio programa **en tiempo de ejecución**. Kotlin hace que las funciones y propiedades sean ***ciudadanos de primera clase*** en el lenguaje (pueden usarse como valores: referenciarse, almacenarse y pasarse dinámicamente), y su introspección (es decir, saber un nombre o un tipo de una propiedad o función en tiempo de ejecución) está estrechamente entrelazado con el simple uso de un estilo funcional o reactivo.  
La característica más básica de la reflexión es **obtener la referencia a una clase** Kotlin, utilizando el operador **``::``**. El valor de esa referencia es de tipo ***KClass***. Para obtener una referencia a una clase Java, se debe agregar ***``.java``***.

```kotlin
    val c = MyClass::class  // Referencia a clase Kotlin
    val c = MyClass::class.java // Referencia a clase Java
```

También se puede **obtener la referencia a una función, a una propiedad o a un constructor**. Este tipo de referencias se denominan **invocables** (***Callable***) y pueden usarse como ***function types***. El supertipo común para todas las referencias *callable* es ***``KCallable<out R>``***.

```kotlin
    fun main() {
    
        fun greetFunction() {
            println("Hola")
        }
    
        val greet = ::greetFunction
    
        greet() // Hola
    }
```

### *Function references*
En el siguiente ejemplo, **se pasa una función como parámetro de otra función**. El operador **``::``** puede usarse con funciones sobrecargadas cuando el tipo esperado se desprende del contexto.

```kotlin
    fun isOdd(x: Int) = x % 2 != 0
    fun isOdd(s: String) = s == "brillig" || s == "slithy" || s == "tove"
    
    val numbers = listOf(1, 2, 3)
    println(numbers.filter(::isOdd)) // Refers to isOdd(x: Int)
```

### *Property references*
Para acceder a **propiedades como objetos de primera clase** en Kotlin, también podemos usar el operador **``::``**

```kotlin
    val x = 1
    
    fun main() {
        println(::x.get()) // 1
        println(::x.name)  // x
    }
```

### *Constructor references*
Los constructores pueden referenciarse de la misma forma que las funciones y las propiedades. Además del operador **``::``** se le agrega el **nombre de la clase**. La siguiente función, espera una función sin parámetros como parámetro y retorna un tipo *Foo*:

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

### Ejemplo de uso de `call`
La función ``call()`` es parte de la interfaz ``KCallable``, que representa cualquier invocable reflectivo (función, _getter_, _setter_, constructor, etc.). Permite ejecutar el miembro con argumentos en tiempo de ejecución.  
> ⚠️ ``call()`` no hace verificación de tipos en tiempo de compilación. Si los argumentos no coinciden, se lanza ``IllegalArgumentException`` o ``TypeCastException``.

<br>

| Tipo de Miembro Reflectivo             | `it` es un...       | `call` se usa para...                                       | Argumentos para `call`                                                                       | Alternativa más idiomática (si existe)                                              |
|----------------------------------------|---------------------|-------------------------------------------------------------|----------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------|
| **Propiedad Miembro**                  | `KProperty1`        | - Obtener el valor (invocar `getter`)                       | `it.call(instanciaReceptora)`                                                                | -                                                                                   |
| **Propiedad Miembro (Mutable)**        | `KMutableProperty1` | - Obtener valor (`getter`)<br>- Establecer valor (`setter`) | `it.getter.call(instanciaReceptora)`<br><br>`it.setter.call(instanciaReceptora, nuevoValor)` | -                                                                                   |
| **Función Miembro**                    | `KFunction`         | - Invocar la función                                        | `it.call(instanciaReceptora, arg1, arg2, ...)`                                               | `kFunction.invoke(instanciaReceptora, ...)`<br>(menos común para reflexión directa) |
| **Propiedad Nivel Superior**           | `KProperty0`        | - Obtener el valor (invocar `getter`)                       | `it.call()`                                                                                  | `kProperty0.get()`                                                                  |
| **Propiedad Nivel Superior (Mutable)** | `KMutableProperty0` | - Obtener valor (`getter`)<br>- Establecer valor (`setter`) | `it.getter.call()`<br><br>`it.setter.call(nuevoValor)`                                       | `kMutableProperty0.get()`<br><br>`kMutableProperty0.set(nuevoValor)`                |
| **Función Nivel Superior**             | `KFunction`         | - Invocar la función                                        | `it.call(arg1, arg2, ...)`                                                                   | `kFunction.invoke(arg1, ...)`<br>(más común para referencias a funciones)           |

