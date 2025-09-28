<h1><i>Removing elements</i></h1>

Para eliminar un elemento de una colección mutable se usa la función ***``remove()``***, que toma un valor y, en caso de existir en la colección, elimina la primera ocurrencia de ese valor (en caso contrario, no hace nada). Para eliminar múltiples elementos a la vez, existen las siguientes funciones:

+ ***``removeAll()``***, que elimina todos los elementos que están presentes en la colección de argumentos. Alternativamente, se puede llamar con un predicado como argumento y en este caso la función elimina todos los elementos para los cuales el predicado resulta verdadero (*true*)
+ ***``retainAll()``*** que, al contrario que la anterior, elimina todos los elementos excepto los de la colección de argumentos. Cuando se usa con un predicado, deja solo los elementos que coinciden
+ ***``clear()``***, que elimina todos los elementos de una lista y la deja vacía

```kotlin
    fun main() {
        val numeros = mutableListOf(1, 2, 3, 4, 3)
        numeros.remove(3) // Elimina el primer 3
        println(numeros) // [1, 2, 4, 3]
        numeros.remove(5) // No elimina nada
        println(numeros) // [1, 2, 4, 3]
    
        numeros.retainAll { it >= 3 }
        println(numeros) // [4, 3]
        numeros.clear()
        println(numeros) // []
    
        val numerosSet = mutableSetOf("uno", "dos", "tres", "cuatro")
        numerosSet.removeAll(setOf("uno", "dos"))
        println(numerosSet) // [tres, cuatro]
    }
```

Y al igual que para agregar elementos de una colección se usa el operador **``+=``**, se puede eliminar elementos con el operador **``-=``**, tanto con un elemento individual (elimina la primera aparición del mismo) como con otra colección (se eliminan todas las apariciones de sus elementos).

```kotlin
    fun main() {
        val numeros = mutableListOf("uno", "dos", "tres", "tres", "cuatro", "cuatro")
        numeros -= "tres"
        println(numeros) // [uno, dos, tres, cuatro, cuatro]
        numeros -= listOf("cuatro", "cinco")
        println(numeros) // [uno, dos, tres]
    }
```
