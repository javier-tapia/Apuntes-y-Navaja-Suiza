<h1><i>Retrieving collection parts</i></h1>

Estas funciones utilizan varias formas, como listar sus posiciones explícitamente o especificar el tamaño del resultado, para obtener una colección resultante que **selecciona y toma ciertos elementos** de la colección a la que se aplican.

***Index***:
<!-- TOC -->
  * [*Slice*](#slice)
  * [*Take and drop*](#take-and-drop)
  * [*Chunked*](#chunked)
  * [*Windowed*](#windowed)
<!-- TOC -->

---

## *Slice*
Devuelve una lista de elementos de la colección con unos índices determinados que se pasan como argumento, ya sea como un rango o como una colección de valores enteros.

```kotlin
    val numeros = listOf("uno", "dos", "tres", "cuatro", "cinco", "seis")
    println(numeros.slice(1..3))            // [dos, tres, cuatro]
    println(numeros.slice(0..4 step 2))     // [uno, tres, cinco]
    println(numeros.slice(setOf(3, 5, 0)))  // [cuatro, seis, uno]
```

## *Take and drop*
Para obtener un número específico de elementos se usan las **funciones** ***``take()``*** y ***``takeLast()``***, con la diferencia de que *take()* cuenta a partir del primer elemento y *takeLast()* a partir del último, hacia atrás. Cuando se llama a estas funciones con un número mayor que el tamaño de la colección, ambas devuelven la colección completa. Y para tomar todos los elementos excepto un número dado de elementos, ya sea desde el primer o último elemento, se usan las **funciones** ***``drop()``*** y ***``dropLast()``*** respectivamente.

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

## *Chunked*
Para dividir una colección en partes de un tamaño determinado se usa la función ***``chunked()``***, que toma un solo argumento (el tamaño del fragmento) y devuelve una lista de listas del tamaño dado. La primera sublista contiene los primeros *n* elementos, la segunda los *n* siguientes y así hasta el final de la colección, por lo que la última sublista puede ser más pequeña si no hay suficientes elementos para completarla.

```kotlin
    val numeros = (0..13).toList()
    println(numeros.chunked(3))  // [[0, 1, 2], [3, 4, 5], [6, 7, 8], [9, 10, 11], [12, 13]]
```

También se puede aplicar una transformación a cada fragmento o sublista utilizando una función *lambda* con *chunked()*.

```kotlin
    val numeros = (0..13).toList()
    println(numeros.chunked(3) { it.sum() })  // [3, 12, 21, 30, 25]
```

## *Windowed*
La función ***``windowed()``*** también devuelve rangos de elementos de un tamaño determinado, pero a diferencia de ``chunked()``, cada uno de los fragmentos o ventanas (*windows*) empieza a partir de cada elemento de la colección.

```kotlin
    val numeros = listOf("uno", "dos", "tres", "cuatro", "cinco")
    println(numeros.windowed(3))  // [[uno, dos, tres], [dos, tres, cuatro], [tres, cuatro, cinco]]
```

``windowed()`` también admite **parámetros** opcionales: 

* ***``step``*** define la distancia entre los primeros elementos de dos ventanas adyacentes. Por defecto el valor es 1, por lo que el resultado contiene ventanas que comienzan con todos los elementos. Si incrementa el paso a 2, solo recibirá ventanas a partir de elementos impares: primero, tercero, y así sucesivamente.
* ***``partialWindows``***, por defecto *true*, incluye la ventana final de tamaño más pequeño, y si es *false* la excluye.

Con *windowed()* también se puede aplicar una transformación a los rangos devueltos con una función *lambda*.

```kotlin
    val numeros = (1..10).toList()
    println(numeros.windowed(3, step = 2, partialWindows = true)) // [[1, 2, 3], [3, 4, 5], [5, 6, 7], [7, 8, 9], [9, 10]]
    println(numeros.windowed(3) { it.sum() })  // [6, 9, 12, 15, 18, 21, 24, 27]
```

Por último, para construir ventanas de dos elementos hay una función específica: ***``zipWithNext()``***, que para cada elemento crea pares de elementos adyacentes. También se puede llamar con una función de transformación tomando dos elementos de la colección como argumentos.

```kotlin
    val numeros = listOf("uno", "dos", "tres", "cuatro", "cinco")
    println(numeros.zipWithNext()) // [(uno, dos), (dos, tres), (tres, cuatro), (cuatro, cinco)]
    println(numeros.zipWithNext() { s1, s2 -> s1.length > s2.length }) // [false, false, false, true]
```
