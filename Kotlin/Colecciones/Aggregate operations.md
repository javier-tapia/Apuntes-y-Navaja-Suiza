## *Aggregate operations*
Las colecciones de Kotlin contienen funciones para operaciones de agrupación (o agregación) de uso común: operaciones que **devuelven un valor único en función del contenido** de la colección, como por ejemplo, los valores mínimo y máximo (***``min()``*** y ***``max()``***), el número de elementos (***``count()``***), la suma de todos los elementos (***``sum()``***) o el valor promedio (***``average()``***).  
Un ejemplo de operación de agrupamiento es el cálculo del promedio de temperatura a partir de los valores de temperatura diaria durante un mes.

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

- [`max...`, `min...`, `sum...`](#max-min-sum)
- [*Fold and reduce*](#fold-and-reduce)

---

### `max...`, `min...`, `sum...`
Hay varias funciones que sirven para recuperar los elementos más pequeños y más grandes mediante funciones de selección o mediante un comparador personalizado, como son:

+ ***``maxBy()``*** y ***``minBy()``***, que mediante una función de selección devuelven el elemento con el valor más grande o más pequeño (o en caso de igualdad, el primero que encuentra)
+ ***``maxWith()``*** y ***``minWith()``***, que tomando como referencia un objeto de tipo *Comparator* devuelven el elemento más grande o más pequeño (o en caso de igualdad, el primero que encuentra)

También hay funciones de suma avanzada o compleja que toman una función que opera sobre los elementos y devuelven la **suma total** de los valores de retorno:

+ ***``sumOf()``*** devuelve la suma de todos los valores producidos por la función selectora aplicada a cada elemento de la colección.

```kotlin
    data class Book(val title: String, val publishYear: Int, val rating: Double)
    
    fun main() {
        val numbers = listOf(6, 4, 5, 3)
        val moduleTwo = numbers.maxBy { it % 2 }
        println(moduleTwo) // 5
        println(numbers.sumOf { it * 2 }) // 36
        println(numbers.sumOf { it.toDouble() / 2 }) // 9.0
    
        val strings0 = listOf("uno", "dos", "tres", "cuatro", "cinco")
        val longer = strings0.maxWith(compareBy { it.length })
        println(longer) // cuatro
    
        val strings1 = listOf("one", "two", "three", "four")
        println(strings1.maxWithOrNull(compareBy { it.length })) // three
    
        val strings2 = listOf("one", "two", "three", "four")
        println(strings2.maxByOrNull { it.length }) // three
    
        val strings3 = listOf("one", "two", "three", "four")
        println(strings3.maxOfOrNull { it.length }) // 5
    
        // A custom Comparator that orders strings by their length
        val lengthComparator = compareBy<String> { it.length }
    
        // The longest and shortest book titles
        val books = listOf(
            Book("Red Sand", 2004, 3.5),
            Book("Silver Bullet", 2009, 4.4),
            Book("Clear Water", 2018, 4.1),
            Book("Night Sky", 2023, 3.8)
        )
        // 1. Selecciona por título
        // 2. Compara el largo del título
        // 3. Retorna el título más largo
        println(books.maxOfWithOrNull(lengthComparator) { it.title }) // Silver Bullet
    }
```

### *Fold and reduce*
Para casos más específicos, existen las funciones ***``reduce()``*** y ***``fold()``***, que aplican la operación proporcionada a los elementos de la colección de forma secuencial y devuelven el **resultado acumulado**. La operación toma dos argumentos: el valor acumulado previamente y el elemento de colección, con la diferencia de que *fold()* toma un valor inicial y lo usa como el valor acumulado en el primer paso, mientras que *reduce()* usa en el primer paso el primer y el segundo elemento como argumentos de la operación. Para aplicar estas funciones en el orden inverso, se usan las funciones ***``reduceRight()``*** y ***``foldRight()``***, que operan de manera similar a *reduce()* y *fold()* pero comienzan desde el último elemento y avanzan hacia el anterior hasta el primero (además hay que tener en cuenta que en estas funciones los argumentos de la operación cambian su orden: primero va el elemento y luego el valor acumulado). También podemos realizar operaciones que toman los índices de los elementos como parámetros con las funciones ***``reduceIndexed()``*** y ***``foldIndexed()``*** pasando el índice del elemento como primer argumento de la operación (y las funciones inversas que aplican las operaciones a los elementos de la colección de derecha a izquierda: ***``reduceRightIndexed()``*** y ***``foldRightIndexed()``***).

```kotlin
    val numeros = listOf(5, 2, 10, 4)
    
    val acumulado = numeros.reduce { sum, element -> sum + element }
    println(acumulado) // 5 -> (5 + 2) -> (7 + 10) -> (17 + 4) = 21
    
    val desdeDiez = numeros.fold(10) { sum, element -> sum + element }
    println(desdeDiez) // 10 + 5 -> (15 + 2) -> (17 + 10) -> (27 + 4) = 31
    
    val acumuladoDoble = numeros.fold(0) { sum, element -> sum + element * 2 }
    println(acumuladoDoble) // 5x2 -> (10 + 2x2) -> (14 + 10x2) -> (34 + 4x2) = 42
    
    // Con reduce el primer elemento no se multiplica por 2:
    val acumuladoDobleReduce = numeros.reduce { sum, element -> sum + element * 2 }
    println(acumuladoDobleReduce) // 5 -> (5 + 2x2) -> (9 + 10x2) -> (29 + 4x2) = 37
    
    val acumuladoDobleDerecha = numeros.foldRight(0) { element, sum -> sum + element * 2 }
    println(acumuladoDobleDerecha) // 42
    
    val sumaPosicionesPares =
        numeros.foldIndexed(0) { idx, suma, elemento ->
            if (idx % 2 == 0) suma + elemento else suma
        }
    println(sumaPosicionesPares) // 5 -> 5 -> (5 + 10) -> 15 = 15
    // idx = 0 -> 5 (suma + elemento)
    // idx = 1 -> 5 (suma)
    // idx = 2 -> 5 + 10 (suma + elemento)
    // idx = 3 -> 15 (suma)
```
