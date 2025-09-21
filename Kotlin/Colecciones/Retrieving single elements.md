## *Retrieving single elements*
Una lista es una colección ordenada, por lo que cada elemento tiene una posición que se puede usar para referir al elemento. En cambio, un *set* no es una colección ordenada, aunque aún así los elementos se almacenan en cierto orden (orden de inserción en *LinkedHashSet*, orden natural en *SortedSet*, etc.). Por lo tanto, los resultados que se obtienen de una posición son impredecibles, a menos que se conozca la implementación específica de *Set* utilizada.

- [*Retrieving by position*](#retrieving-by-position)
- [*Retrieving by condition*](#retrieving-by-condition)
- [*Random element*](#random-element)
- [*Checking existence*](#checking-existence)

---

### *Retrieving by position*
Para recuperar un elemento en una posición específica, existe la función ***``elementAt()``*** que se llama con un número entero como argumento, y devuelve el elemento de la colección en esa posición (recordando que el primer elemento está en la posición 0 y el último en la posición tamaño - 1). Es útil para colecciones que no proporcionan (o que no se sabe que proporcionen) acceso indexado, si bien en el caso de *List* se suele utilizar el operador de acceso indexado *``get()``* o el índice entre ``[]``.También hay herramientas útiles para recuperar el primer y último elemento de la colección: ***``first()``*** y ***``last()``***.

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

### *Retrieving by condition*
Las **funciones** *``first()``* y *``last()``* también permiten buscar elementos en una colección que coincidan con un predicado determinado. Así, cuando se llama a *first()* con un predicado que comprueba un elemento de la colección, devuelve el primer elemento en el que el predicado se traduce en verdadero. A su vez, *last()* con un predicado devuelve el último elemento que coincide con él. Si ningún elemento coincide con el predicado, ambas funciones lanzan excepciones, y para evitarlo se usan ***``firstOrNull()``*** y ***``lastOrNull()``***, que devuelven *null* si no encuentran elementos coincidentes. Otra alternativa para evitar estas excepciones es usar ***``find()``*** en lugar de *``firstOrNull()``* y ***``findLast()``*** en lugar de *``lastOrNull()``*.

```kotlin
    val numeros = listOf("uno", "dos", "tres", "cuatro", "cinco", "seis")
    println(numeros.first { it.length > 3 }) // tres
    println(numeros.last { it.startsWith("c") }) // cinco
    
    println(numeros.firstOrNull { it.length > 6 }) // null
    
    println(numeros.find { it.length > 6 }) // null
```

### *Random element*
Si se necesita recuperar un elemento arbitrario de una colección, se puede usar la función ***``random()``***, que se puede llamar sin argumentos o con un objeto tipo *Random* como fuente de la aleatoriedad.

```kotlin
    import kotlin.random.Random
    import kotlin.random.Random.Default.nextInt
    
    fun main() {
        val numeros = listOf("uno", "dos", "tres", "cuatro", "cinco", "seis")
        println(numeros.random())
        println(numeros.random(Random(nextInt(0, numeros.size))))
    }
```

### *Checking existence*
Para verificar si un elemento está presente en la colección, se puede usar la función ***``contains()``***, que devuelve *true* si existe un elemento en la colección igual al argumento utilizado en la llamada a la función. También se puede llamar a *contains()* con la palabra reservada ***``in``***. Además, para comprobar la presencia de varios argumentos, se puede utilizar ***``containsAll()``*** con una colección de elementos que queremos verificar como argumento. También se puede verificar si una colección contiene algún elemento o está vacía con ***``isNotEmpty()``*** y con ***``isEmpty()``***.

```kotlin
    val numeros = listOf("uno", "dos", "tres", "cuatro", "cinco", "seis")
    println(numeros.contains("cuatro")) // true
    println("cero" in numeros)          // false
    
    println(numeros.containsAll(listOf("cuatro", "dos"))) // true
    println(numeros.containsAll(listOf("uno", "cero")))   // false
    
    println(numeros.isEmpty())    // false
    println(numeros.isNotEmpty()) // true
```
