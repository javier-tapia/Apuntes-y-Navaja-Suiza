<h1><i>Transformations</i></h1>

Estas funciones crean nuevas colecciones a partir de las existentes según las reglas de transformación proporcionadas.

***Index***:
<!-- TOC -->
  * [*Mapping*](#mapping)
  * [*Zipping*](#zipping)
  * [*Association*](#association)
  * [*Flattening*](#flattening)
  * [*String representation*](#string-representation)
<!-- TOC -->

---

## *Mapping*
Crea una nueva colección a partir de los resultados de una función sobre los elementos de una colección. La función básica de *mapping* es ***``map()``***, que aplica una determinada función *lambda* a cada elemento y devuelve una lista de los resultados respetando el orden original de los elementos en la colección original. Si además se quiere usar los índices de los elementos como argumento, se usa ***``mapIndexed()``***.Si la transformación produce *null* en ciertos elementos, se pueden filtrar los *null* de la colección de resultados llamando a la función ***``mapNotNull()``*** en lugar de ***``map()``*** y ***``mapIndexedNotNull()``*** en lugar de ***``mapIndexed()``***.

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
    println(numerosDic.mapKeys { it.key.uppercase() + it.value})
    // {KEY11=1, KEY22=2, KEY33=3, KEY1111=11}
    println(numerosDic.mapValues { it.value + it.key.length })
    // {key1=5, key2=6, key3=7, key11=16}
```

## *Zipping*
Consiste en **construir pares a partir de elementos con las mismas posiciones en dos colecciones**. En la librería estándar de Kotlin esto se realiza mediante la **función de extensión** ***``zip()``*** (comprimir), que cuando se utiliza sobre una colección con otra colección como argumento, **devuelve una lista de objetos** ***Pair*** en la que los elementos de la primera colección (receptor) son los primeros elementos en estos pares. Si las colecciones tienen diferentes tamaños, el resultado de *zip()* es el tamaño más pequeño y los últimos elementos de la colección más grande no se incluyen en el resultado. La función *zip()* también puede ser llamada en forma *infix*: ***``a zip b``***. También se puede hacer la transformación inversa (descomprimir) con la función ***``unzip()``*** que genera dos listas a partir de pares de elementos, y obtiene así las listas originales.

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

## *Association*
Permiten **construir diccionarios** (***Map***) a partir de los elementos de una colección y ciertos valores asociados con ellos y, según el tipo de asociación, los elementos pueden ser claves o valores en el diccionario. La función básica ***``associateWith()``*** crea un *Map* en el que los elementos de la colección original son claves y los valores se generan a partir de ellos mediante una función de transformación determinada (si dos elementos son iguales, solo el último permanece).

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
    
    println(numeros.associateBy { it.first().uppercase() })
    // {U=uno, D=dos, T=tres, C=cinco} // C=cuatro se omite
    
    println(numeros.associateBy(keySelector = { it.first().uppercase() }, valueTransform = { it.length }))
    // {U=3, D=3, T=4, C=5} // C=6 se omite
```

Otra forma de crear diccionarios en los que tanto las claves como los valores se producen a partir de los elementos de una colección es la función ***``associate()``***, que utiliza una función *lambda* que devuelve un *Pair* con la clave y el valor correspondientes. Si cualquiera de los pares tuviera la misma clave, solo la última se agregará al mapa.

```kotlin
    val lista = listOf("a", "b", "c", "d")
    val dict = lista.associate { it.capitalize() to it }
    println(dict) // {A=a, B=b, C=c, D=d}
    
    val intList = listOf(1, 2, 3)
    println(intList.associate { Pair(it.toString(), it) })
    // {1=1, 2=2, 3=3}
```

## *Flattening*
Cuando se utilizan **colecciones anidadas** (por ejemplo una lista de *sets*) pueden ser útiles algunas funciones de la librería estándar que proporcionan acceso a sus elementos, como la función ***``flatten()``*** que **devuelve una lista única de todos los elementos** de las colecciones anidadas.

```kotlin
    // List<Set<Int>>
    val listaSet = listOf(setOf(1, 2, 3), setOf(4, 5, 6), setOf(1, 2))
    println(listaSet.flatten()) // [1, 2, 3, 4, 5, 6, 1, 2]
```

Por su parte, la función ***``flatMap()``*** toma el resultado que devuelve una función que opera sobre cada uno de los elementos de las colecciones anidadas para obtener una **lista única de todos los valores** de los elementos de las colecciones anidadas.

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

## *String representation*
Cuando se necesita recuperar el contenido de una colección en un formato legible, se pueden usar **funciones** que transforman las colecciones en *String*, como ***``joinToString()``*** y ***``joinTo()``***. Mientras que *joinToString()* construye un único String a partir de los elementos de la colección en función de los argumentos proporcionados, la función *joinTo()* hace lo mismo pero agrega el resultado a un objeto determinado. Cuando se utilizan con los argumentos por defecto, estas funciones devuelven un resultado similar a la función *toString()* sobre una colección.

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
    println(numeros.joinToString { "${it.uppercase()}: ${it.length} letras" })
    // UNO: 3 letras, DOS: 3 letras, TRES: 4 letras, CUATRO: 6 letras
```
