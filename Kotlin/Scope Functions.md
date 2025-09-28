<h1><i>Scope functions</i></h1>

Su único propósito es ejecutar un bloque de código **dentro del contexto de un objeto**. Cuando se llama a una de estas funciones en un objeto con una expresión *lambda* proporcionada, forma un **ámbito** (***scope***) **temporal**.  En este ámbito, se puede **acceder al objeto sin su nombre**.

```kotlin
    data class Person(var name: String, var age: Int, var city: String) {
        fun moveTo(newCity: String) { city = newCity }
        fun incrementAge() { age++ }
    }
    
    fun main() {
        Person("Alice", 20, "Amsterdam").let {
            println(it) // Person(name=Alice, age=20, city=Amsterdam)
            it.moveTo("London")
            it.incrementAge()
            println(it) // Person(name=Alice, age=21, city=London)
        }
    }
```

***Index***:
<!-- TOC -->
  * [Comparativa](#comparativa)
  * [`let`](#let)
  * [`run`](#run)
  * [`with`](#with)
  * [`apply`](#apply)
  * [`also`](#also)
<!-- TOC -->

---

## Comparativa
Hay dos diferencias principales entre cada una de ellas: la forma de referirse al **objeto de contexto** y el **valor de retorno**.
> Sobre el concepto de ***Receiver***, ver [acá](../Glosary%20&%20Core%20Concepts/Software%20in%20general.md#receiver)

<br>

| ***Scope Function*** | **Referencia al Objeto de Contexto (*Context Object*)** | **Valor de Retorno (*Return value*)** |
|----------------------|---------------------------------------------------------|---------------------------------------|
| **`let`**            | `it` (argumento de la *lambda*)                         | Resultado de la *lambda*              |
| **`run`**            | `this` (*receiver*)                                     | Resultado de la *lambda*              |
| **`with`**           | `this` (*receiver*)                                     | Resultado de la *lambda*              |
| **`apply`**          | `this` (*receiver*)                                     | Objeto de contexto                    |
| **`also`**           | `it` (argumento de la *lambda*)                         | Objeto de contexto                    |

## `let`
| **Situación**                                                                                    | **Usar `let`**                                                                                                                                                                                                                                                                       |
|--------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Chequeo de nulabilidad de variable mutable (`var`)**                                           | ✅ **Sí.** `let` crea una copia inmutable (`it`) del valor de la variable mutable en el momento de la verificación, asegurando que el valor no cambie dentro del bloque `let` incluso si la variable original es modificada por otro hilo.                                            |
| **Chequeo de nulabilidad de variable inmutable (`val`)**                                         | ❌ **No.** Para variables inmutables (`val`), un simple `if (variable != null)` es más eficiente y directo, ya que no hay riesgo de que el valor cambie. Usar `let` haría que la JVM introduzca una variable `it` innecesaria.                                                        |
| **Distinguir `it` (objeto de contexto) de `this` (ámbito externo)**                              | ✅ **Sí.** Si dentro del bloque necesitas acceder tanto a miembros del objeto de contexto (`it`) como a miembros de la clase contenedora (`this`), `let` ayuda a clarificar a cuál te refieres, evitando ambigüedades.                                                                |
| **Acceso principal al objeto de contexto (sin necesidad del ámbito externo)**                    | ⚠️ **Considerar `run`.** Aunque `let` funciona, si solo operas sobre el objeto de contexto y no necesitas distinguirlo del ámbito externo, `run` puede ser más conciso, ya que accedes a los miembros del objeto directamente (como `this` implícito) en lugar de usar `it.miembro`. |
| **Simplificar cadenas largas de operadores nulos seguros (`?.`)**                                | ✅ **Sí.** `let` permite ejecutar un bloque de código solo si el resultado de una cadena de llamadas con el operador de nulidad segura (`?.`) no es nulo. Esto hace el código más limpio y legible que anidar múltiples `if` o repetir verificaciones.                                |
| **Obtener el resultado final de una cadena de operaciones / transformación**                     | ✅ **Sí.** `let` devuelve el resultado de la expresión lambda. Esto es útil para transformar un objeto y asignar este nuevo resultado a una variable o usarlo directamente, eliminando la necesidad de una variable temporal intermedia.                                              |
| **Necesidad de encadenar el objeto original después del bloque (ej.: para efectos secundarios)** | ❌ **No (usar `also`).** `let` devuelve el resultado de la lambda. Si necesitas realizar alguna acción con el objeto y luego seguir usando el objeto *original* en una cadena, `also` es más adecuado porque devuelve el objeto de contexto (`this`) original.                        |

## `run`
TODO...

## `with`
TODO...

## `apply`
TODO...

## `also`
TODO...
