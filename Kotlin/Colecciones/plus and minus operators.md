## *plus and minus operators*
En Kotlin, los operadores ***``plus``*** (**``+``**) y ***``minus``*** (**``-``**) se definen para las colecciones. Toman una colección como primer operando; el segundo operando puede ser un elemento u otra colección. El valor de retorno es una **nueva colección de solo lectura**.

```kotlin
    val numbers = listOf("one", "two", "three", "four")
    
    val plusList = numbers + "five"
    val minusList = numbers - listOf("three", "four")
    println(plusList)  // [one, two, three, four, five]
    println(minusList) // [one, two]
```
