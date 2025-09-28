<h1><i>Adding elements</i></h1>

Para agregar un solo elemento a una lista o a un conjunto se usa la función ***``add()``***, que agrega el nuevo elemento al final de la colección.  También está la función ***``addAll()``*** que utiliza como argumento un iterable, una secuencia o un *array* para agregar cada elemento de dicho objeto, sin necesidad de que los tipos del argumento y de la colección receptora sean similares, esto es, se pueden agregar todos los elementos de un conjunto (*Set*) a una lista (*List*). Cuando *addAll()* se invoca sobre listas, los nuevos elementos se agregan en el mismo orden en que aparecen en el argumento. También se puede llamar a *addAll()* especificando una posición determinada como primer argumento, y entonces el primer elemento del argumento se inserta en esa posición seguido por el resto de elementos del argumento, desplazando a los elementos de la colección receptora hasta el final.

```kotlin
    val numeros = mutableListOf(1, 2, 5, 6)
    numeros.addAll(arrayOf(7, 8))
    println(numeros) // [1, 2, 5, 6, 7, 8]
    numeros.addAll(2, setOf(3, 4)) // El primer argumento es el índice de inserción
    println(numeros) // [1, 2, 3, 4, 5, 6, 7, 8]
```

Otra manera de agregar elementos es utilizando el operador **``+=``**, tanto con un elemento individual como con otra colección, insertando los nuevos elementos al final de la colección:

```kotlin
    val numeros = mutableListOf("uno", "dos")
    numeros += "tres"
    println(numeros) // [uno, dos, tres]
    numeros += listOf("cuatro", "cinco")
    println(numeros) // [uno, dos, tres, cuatro, cinco]
```
