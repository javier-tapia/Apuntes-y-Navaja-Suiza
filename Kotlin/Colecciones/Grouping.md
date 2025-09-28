<h1><i>Grouping</i></h1>

La función básica ***``groupBy()``*** **toma una función** ***lambda*** **y devuelve un** ***Map***. En este *map*, cada clave es el resultado del *lambda* y el valor correspondiente es la lista de elementos en los que se devuelve este resultado. Esta función se puede utilizar, por ejemplo, para agrupar una lista de *strings* por su primera letra. También se puede llamar a *groupBy()* con un **segundo argumento** ***lambda***: **una función de transformación** de valor. En el *map* resultante de *groupBy()* con dos *lambdas*, las claves producidas por la función ***keySelector*** se mapean con los resultados de la **función de transformación** de valor en lugar de los elementos originales.

```kotlin
    val numbers = listOf("one", "two", "three", "four", "five")
    
    println(numbers.groupBy { it.first().uppercase() }) // {O=[one], T=[two, three], F=[four, five]}
    
    println(
        numbers.groupBy(
            keySelector = { it.first() },
            valueTransform = { it.uppercase() })
    ) // {o=[ONE], t=[TWO, THREE], f=[FOUR, FIVE]}
```

Si se desea agrupar elementos y luego **aplicar una operación a todos los grupos a la vez**, se usa la función ***``groupingBy()``***, que devuelve una instancia de tipo ***Grouping***. La instancia de *Grouping* permite aplicar operaciones a todos los grupos de manera *lazy*: los grupos se crean **antes de la ejecución de la operación**.  
*Grouping* admite las siguientes operaciones: ***``eachCount()``*** cuenta los elementos de cada grupo; ***``fold()``*** y ***``reduce()``*** realizan operaciones de plegado (*fold*) y reducción (*reduce*) en cada grupo como una colección separada y devuelven los resultados; ***``aggregate()``*** aplica una operación dada después de todos los elementos de cada grupo y devuelve el resultado. Esta es la forma genérica de realizar cualquier operación en un *Grouping*. Se usa para implementar operaciones personalizadas cuando *fold* o *reduce* no es suficiente.

```kotlin
    val numbers = listOf("one", "two", "three", "four", "five", "six")
    println(numbers.groupingBy { it.first() }.eachCount()) // {o=1, t=2, f=2, s=1}
```
