## *Filtering*
Las condiciones de filtrado se definen mediante **predicados: funciones** ***lambda*** **que toman un elemento de la colección y devuelven un valor booleano** (***true*** significa que el elemento dado coincide con el predicado, ***false*** significa lo contrario).

- [*Filtering by predicate*](#filtering-by-predicate)
- [*Partitioning*](#partitioning)
- [*Testing predicates*](#testing-predicates)

---

### *Filtering by predicate*
La función básica de filtrado es ***``filter()``***. Cuando se llama con un predicado, **devuelve los elementos de la colección que coinciden**. Tanto para las listas y los conjuntos (*List* y *Set*) la colección resultante es una lista (*List*), mientras que para los diccionarios (*Map*) también es un *Map*.

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
        print("${it.uppercase()} ") // DOS CUATRO
    }
    
    val num2 = listOf(null, "uno", "dos", null)
    num2.filterNotNull().forEach {
        print("$it: ${it.length} ") // uno: 3 dos: 3
    }
```

### *Partitioning*
Otra función de filtrado es ***``partition()``***, que filtra una colección por un predicado y mantiene los elementos que no coinciden en una lista separada. Por lo tanto, **tiene un** ***Pair*** **de listas como valor de retorno**: la primera lista que contiene los elementos que coinciden con el predicado y la segunda que contiene todo lo demás de la colección original.

```kotlin
    val numeros = listOf("uno", "dos", "tres", "cuatro")
    val (match, noMatch) = numeros.partition { it.length > 3 }
    
    println(match)   // [tres, cuatro]
    println(noMatch) // [uno, dos]
```

### *Testing predicates*
Hay funciones que simplemente **prueban** (***test***) **un predicado en los elementos de una colección**. A saber, ***``any()``*** (devuelve *true* si **al menos un elemento coincide con el predicado** dado); ***``none()``*** (devuelve *true* si **ninguno de los elementos coincide con el predicado** dado); ***``all()``*** (devuelve *true* si **todos los elementos coinciden con el predicado** dado).  
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
