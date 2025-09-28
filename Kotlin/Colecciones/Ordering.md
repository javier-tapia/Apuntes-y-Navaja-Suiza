<h1><i>Ordering</i></h1>

El orden de los elementos es un aspecto importante en ciertos tipos de colecciones. Por ejemplo, dos listas con los mismos elementos no son iguales si sus elementos están ordenados de manera diferente.  
El paquete de colecciones de Kotlin proporciona funciones específicas para ordenar los elementos de las colecciones por su **orden natural**, por **criterios personalizados** o **de manera aleatoria**. Estas funciones **se aplican a las colecciones de solo lectura** y devuelven como resultado una nueva colección que contiene los mismos elementos que la colección original pero presentados en el orden requerido.

***Index***:
<!-- TOC -->
  * [*Natural order*](#natural-order)
  * [*Custom order*](#custom-order)
  * [*Reverse order*](#reverse-order)
  * [*Random order*](#random-order)
<!-- TOC -->

---

## *Natural order*
En Kotlin, los objetos se pueden ordenar siguiendo distintos criterios. Primero, hay un orden natural para los objetos que heredan la interfaz ***``Comparable``*** y que se utiliza para clasificar los objetos del mismo tipo cuando no se especifica ningún otro criterio. Por ejemplo, los tipos numéricos utilizan el orden numérico tradicional (1 es mayor que 0 y -3.4f es mayor que -5f) y *Char* y *String* usan el orden lexicográfico ('b' es mayor que 'a' y 'mundo' es mayor que 'hola').
Además, el usuario puede definir un orden natural para otro tipo haciendo que herede de la interfaz *Comparable*, lo que requiere implementar la función ***``compareTo()``***. Esta función debe tomar otro objeto del mismo tipo como argumento y devuelve un valor entero que muestra qué objeto es mayor: **si es positivo el objeto receptor es mayor, si es negativo el argumento es mayor y si es cero ambos objetos son iguales**. Un ejemplo de una clase que se puede usar para ordenar versiones:

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

Las funciones básicas ***``sorted()``*** y ***``sortedDescending()``*** devuelven los elementos de una colección ordenados en secuencia ascendente y descendente **según su orden natural**. Estas funciones se aplican a las colecciones de elementos comparables.

```kotlin
    val numeros = listOf("uno", "dos", "tres", "cuatro")
    println("En orden ascendente: ${numeros.sorted()}")
    println("En orden descendente: ${numeros.sortedDescending()}")
```

## *Custom order*
Los ordenamientos personalizados permiten ordenar instancias de cualquier tipo de la forma que uno quiera, pudiendo definir un orden para objetos no comparables o bien definir un orden no natural para un tipo comparable. Para definir un orden determinado para un tipo, se debe crear un ***Comparator*** para él. *Comparator* contiene la función ***``compare()``*** que toma dos instancias de una clase y devuelve un resultado entero que se interpreta como se vio antes en la función *compareTo()*.

```kotlin
    val ordenLongitud = Comparator { str1: String, str2: String -> str1.length - str2.length }
    println(listOf("aaa", "bb", "c").sortedWith(ordenLongitud)) // [c, bb, aaa]
```

En el ejemplo anterior se organizan las palabras por su longitud en lugar del orden lexicográfico natural. Se puede definir un **comparador** de manera más concisa con la función ***``compareBy()``***, que toma una función *lambda* que produce un valor comparable de una instancia y define el orden personalizado como el orden natural de los valores producidos, por lo que se puede escribir el código anterior así:

```kotlin
    println(listOf("aaa", "bb", "c").sortedWith(compareBy { it.length }))
```

Para utilizar **criterios personalizados** u **ordenar objetos no comparables** podemos usar las **funciones** ***``sortedBy()``*** y ***``sortedByDescending()``***, que toman una función que asigna los elementos de la colección a valores de *Comparable* y ordena la colección en el orden natural de esos valores. Para definir un criterio personalizado para ordenar una colección, se puede proporcionar un *Comparator* propio, llamando a la función ***``sortedWith()``***.

```kotlin
    val numeros = listOf("uno", "dos", "tres", "cuatro", "cinco")
    
    val ordenLongitud = numeros.sortedBy { it.length }
    println("Orden ascendente por longitud: $ordenLongitud")
    
    val ultimaLetra = numeros.sortedByDescending { it.last() }
    println("Orden descendente por última letra: $ultimaLetra")
    
    println("Orden ascendente por longitud: ${numeros.sortedWith(compareBy { it.length })}")
```

## *Reverse order*
Se puede recuperar la colección en orden inverso utilizando la función ***``reversed()``***. Hay que tener en cuenta que ``reversed()`` devuelve una nueva colección con copias de los elementos y, por tanto, si más adelante cambia la colección original, esto no afectará a los resultados obtenidos previamente con ``reversed()``. En cambio, la función ***``asReversed()``*** devuelve una vista invertida de la misma instancia de la colección, por lo que es preferible frente a ``reversed()`` al requerir menos recursos siempre que la lista original no vaya a cambiar o sea inmutable.

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

## *Random order*
La función ***``shuffled()``*** devuelve una nueva lista que contiene los elementos de la colección mezclados en un **orden aleatorio**, pudiéndose llamar sin argumentos o con un objeto tipo ***``Random``***.

```kotlin
    val numeros = mutableListOf("uno", "dos", "tres", "cuatro")
    println(numeros.shuffled())  // [tres, uno, cuatro, dos]
```
