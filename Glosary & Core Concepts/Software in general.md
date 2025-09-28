<h1>Glosario de términos de <i>Software</i> en general</h1>

***Index***:
<!-- TOC -->
  * [*Abstraction*](#abstraction)
  * [*Atomicity*](#atomicity)
  * [*Build*](#build)
  * [*Bytecode*](#bytecode)
  * [*Callback*](#callback)
  * [*Casting* (*cast*)](#casting-cast)
  * [Código binario](#código-binario)
  * [Código máquina](#código-máquina)
  * [Compatibilidad binaria](#compatibilidad-binaria)
  * [Compilación](#compilación)
  * [Copia profunda (*deep copy*)](#copia-profunda-deep-copy)
  * [Copia superficial (*shallow copy*)](#copia-superficial-shallow-copy)
  * [Deserializar (*deserialize*)](#deserializar-deserialize)
  * [*Desugaring*](#desugaring)
  * [Expresión (*expression*)](#expresión-expression)
  * [Funciones vs Métodos](#funciones-vs-métodos)
  * [Objeto compuesto](#objeto-compuesto)
  * [Parseo (*parsing*)](#parseo-parsing)
  * [Paso por Valor vs Paso por Referencia](#paso-por-valor-vs-paso-por-referencia)
    * [Paso por Valor](#paso-por-valor)
    * [Paso por Referencia](#paso-por-referencia)
  * [*Polymorphism*](#polymorphism)
  * [Predicado (*predicate*)](#predicado-predicate)
  * [Punteros](#punteros)
  * [*Receiver*](#receiver)
  * [Sentencia (*statement*)](#sentencia-statement)
  * [Serializar (*serialize*)](#serializar-serialize)
<!-- TOC -->

---

## *Abstraction*
Principio fundamental en la **Programación Orientada a Objetos** (**OOP**) que permite ocultar los detalles de implementación y resaltar las características esenciales de un objeto.  
En el contexto de un sistema, al hablar de **componentes de alto nivel** y **bajo nivel**, nos referimos al **grado de abstracción** que cada componente representa. Los componentes de **alto nivel** suelen estar más alejados de los detalles específicos y operan con conceptos más generales (mayor abstracción). Por ejemplo, una interfaz que define métodos comunes para varias clases es un componente de alto nivel, porque permite a otros componentes interactuar con una amplia gama de objetos sin conocer sus detalles de implementación específicos. Por otro lado, los componentes de **bajo nivel** manejan detalles específicos y son más concretos (menor abstracción), como una clase que implementa funcionalidades particulares de un sistema.  
En *Clean Architecture*, esta jerarquía de niveles de abstracción se traduce en un esquema de capas circulares donde las capas más internas son más abstractas (alto nivel) y representan la lógica de negocio, mientras que las capas externas manejan los detalles concretos y la implementación (bajo nivel). Este enfoque no solo mejora la organización y la mantenibilidad del código, sino que también facilita la prueba y evolución de cada componente por separado.

## *Atomicity*
Atomic implies indivisibility and irreducibility, so _**an atomic operation must be performed entirely or not performed at all**_. An operation that is atomic on one machine may not be on another.  
An atomic operation is an operation during which a processor can simultaneously read a location and write it in the same bus operation. This prevents any other processor or I/O device from writing or reading memory until the operation is complete.  
Atomicity is a trait that defines whether an operation can be interrupted or not. It matters because _**if something can't be interrupted, it's intrinsically thread safe**_.

## *Build*
Proceso global que incluye varios pasos para preparar un software para su ejecución, incluyendo compilación del código fuente, ejecución de pruebas unitarias, empaquetado de la aplicación (por ejemplo, crear un archivo JAR o WAR en Java), generación de documentación, creación de artefactos para despliegue.

## *Bytecode*
Forma intermedia de código que ha sido compilada a partir del código fuente y representada en un formato binario, que no está directamente en la forma que entiende el procesador de la máquina (código máquina). En el caso de Java y Kotlin, el código fuente se compila a un formato de bytecode que es independiente de la plataforma, lo que permite que este bytecode sea ejecutado en cualquier sistema que tenga una máquina virtual correspondiente (como la Java Virtual Machine - JVM).

Cuando una aplicación se ejecuta, la JVM convierte el bytecode a código máquina que puede ser entendido por la CPU del sistema. Este proceso se llama ***compilación Just-In-Time (JIT)***.

Algunas características del bytecode son:  
- **Portabilidad**: Permite que el mismo código se ejecute en diferentes plataformas, siempre que exista la máquina virtual adecuada.
- **Optimización**: Puede ser optimizado en el momento de la ejecución por la máquina virtual.
- **Intermediación**: Actúa como un ***nivel de separación entre el código fuente y el código máquina***, permitiendo que se realicen optimizaciones y verificación en tiempo de ejecución.

## *Callback*
Es una función que se pasa como parámetro a otra función y a la que se invoca una vez que el proceso se haya completado.

## *Casting* (*cast*)
Conversión explícita de un tipo de dato a otro. Es decir, se le avisa al compilador que un objeto es de un tipo de dato diferente.

## Código binario
Cualquier ***representación de datos que pueda ser entendida directamente por una computadora***, que típicamente es en forma de ***ceros y unos***. Esto incluye, entre otras cosas:

- **Ejecutables**: Programas compilados que el sistema operativo puede ejecutar directamente (por ejemplo, archivos **`.exe`** en Windows).  
- **Librerías**: Archivos de librerías que también están compilados en una forma que la computadora puede usar directamente (por ejemplo, archivos **`.dll`** o **`.so`**).  
- **Datos**: Cualquier tipo de información que pueda ser almacenada y procesada por una computadora en formato binario, como imágenes, textos, archivos de audio, etc. Aunque estos datos se representan en formato binario, ***no son ejecutables en sí mismos***.

## Código máquina
***Forma específica de código binario*** que es el ***resultado final de la compilación del código fuente*** y que es directamente entendida y ejecutada por la CPU de una computadora. Dicho de otra forma, el código máquina consiste en un ***conjunto de datos binarios estructurados como instrucciones ejecutables*** que la CPU puede interpretar y ejecutar de manera inmediata.

## Compatibilidad binaria
Capacidad de un programa o componente de software para interactuar con otro en su forma compilada (código máquina) ***sin necesidad de recompilación***. La compatibilidad binaria asegura que los cambios en la implementación interna de una librería no rompan el comportamiento de las partes del código que dependen de ella, siempre que se mantengan las APIs públicas. Esto es crucial para garantizar la estabilidad y funcionalidad de las aplicaciones que utilizan librerías externas y permite a los desarrolladores actualizar las librerías ***sin modificar su código fuente***.

## Compilación
Proceso que implica transformar código fuente (como el escrito en un lenguaje de programación de alto nivel) en código máquina que la CPU puede ejecutar.

## Copia profunda (*deep copy*)
Construye un nuevo objeto compuesto y luego, recursivamente, inserta copias de los objetos encontrados en el original.  
Ver [Copia superficial](#copia-superficial-shallow-copy)

## Copia superficial (*shallow copy*)
Construye un nuevo objeto compuesto y luego (en la medida de lo posible) inserta referencias a los objetos encontrados en el original.  
Ver [Copia profunda](#copia-profunda-deep-copy)

## Deserializar (*deserialize*)
Reconstruir el objeto a partir de la información recuperada, ya sea una serie de bytes o un JSON o XML.  
Ver [Serializar](#serializar-serialize)

## *Desugaring*
Transformación de una construcción de lenguaje de alto nivel (más "dulce" o conveniente para el programador) en una construcción equivalente de nivel inferior (más "cruda" o básica) que puede ser entendida por un intérprete o compilador que no soporta la construcción de alto nivel directamente. En resumen, se trata de ***traducir características de un lenguaje a una forma más básica***. Ver también la definición específica para [Android](Android%20specific.md#desugaring).

## Expresión (*expression*)
Debe ser evaluada y, por lo tanto, retorna un valor.  
Ver [Sentencia](#sentencia-statement)

## Funciones vs Métodos
Partiendo de esta distinción en cuanto al ámbito en el cual se puede declarar una función o una propiedad:  
- "*Top-level*" (fuera de una clase, directamente dentro del paquete)
- "Miembro" (dentro de una clase)
- "Local" (dentro de una función)

Si una función se declara como *top-level*, se denomina "Función". No tienen relacionada una instancia y son accesibles directamente.  
En cambio, si se declara como miembro, se denomina "Método". Y puede operar sobre las propiedades de la clase.  
Dicho de otra forma, los "Métodos" son un sub-conjunto de las "Funciones" que se declaran dentro de una clase.

Las *extension functions* se utilizan como si fueran métodos y proporcionan una sintaxis similar, pero siguen siendo funciones en el sentido de que son *top-level functions* que no se definen dentro de la clase que extienden. En esencia, ofrecen una forma de agregar funcionalidad de manera fluida a otras clases, pero sin ser realmente parte de la herencia o definición de esas clases.

## Objeto compuesto
Objetos que contienen otros objetos, como listas o instancias de clase

## Parseo (*parsing*)
También llamado análisis de sintaxis. Analiza un texto y decide qué significan las diferentes partes (*pars*) del mismo para luego procesarlas.

## Paso por Valor vs Paso por Referencia

### Paso por Valor
- Se aplica a tipos de datos primitivos y objetos inmutables.
- El valor asignado es una copia del original, que vive en su propia ubicación de memoria.
- La modificación de la copia no afecta al original.
- Ejemplo:

    ```kotlin
    // Tipo primitivo
    fun modificarValor(x: Int) {
        // Se modifica la copia, no el valor original
        x += 5
        println("Dentro de la función: $x") // Imprime: 15
    }
    
    fun main() {
        val numero = 10
        modificarValor(numero)
        println("Fuera de la función: $numero") // Imprime: 10 (sin cambios)
    }
    
    // Objeto inmutable
    data class Persona(val nombre: String, val edad: Int)
    
    fun main() {
        val persona1 = Persona("Carlos", 30)
    
        // Crear una copia de persona1 con el nombre cambiado
        val persona2 = persona1.copy(nombre = "Juan") 
    
        println("Persona 1: Nombre = ${persona1.nombre}, Edad = ${persona1.edad}") // Imprime: Carlos, 30
        println("Persona 2: Nombre = ${persona2.nombre}, Edad = ${persona2.edad}") // Imprime: Juan, 30
    }
    ```

### Paso por Referencia
- Se aplica a objetos y estructuras de datos mutables.
- El valor asignado es una referencia a la ubicación en memoria donde se almacena el objeto original.
- La modificación a través de la referencia (por ejemplo, utilizando métodos o propiedades del objeto) afecta al objeto original, y todas las referencias al objeto (incluida la original) reflejarán esos cambios.
- Ejemplo:

    ```kotlin
    // Objeto mutable
    data class Persona(var nombre: String)
    
    fun modificarNombre(persona: Persona) {
        // Se modifica el objeto original
        persona.nombre = "Juan"
        println("Dentro de la función: ${persona.nombre}") // Imprime: Juan
    }
    
    fun main() {
        val persona = Persona("Carlos")
        modificarNombre(persona)
        println("Fuera de la función: ${persona.nombre}") // Imprime: Juan (cambiado)
    }
    ```

## *Polymorphism*
Concepto fundamental en la **Programación Orientada a Objetos** que permite que un objeto o función tome múltiples formas. En términos más simples, el polimorfismo permite que una sola interfaz se use para representar diferentes tipos de datos o acciones. Esto es útil porque promueve la flexibilidad y la reutilización del código, permitiendo que una función o método opere sobre distintos tipos de datos según el contexto en que se utilice.

- ***Dynamic Polymorphism*** → Capacidad de un objeto para comportarse de diferentes maneras en tiempo de ejecución (*runtime*). Generalmente se logra a través de la herencia (con clases abstractas o implementando interfaces) y sobreescritura (*overriding*) de funciones.  
- ***Static Polymorphism*** → Capacidad de la misma función de manejar diferentes tipos de datos en tiempo de compilación (*compile-time*). Se implementa principalmente a través de la sobrecarga (*overloading*) de funciones y el compilador determina cuál usar basándose en su firma (número, orden y tipos de parámetros).

## Predicado (*predicate*)
Representan funciones de argumento único que devuelven un valor booleano.

## Punteros
Un **puntero** (o **referencia**) es una variable que almacena la dirección de memoria de otra variable. En lugar de contener un valor directamente, un puntero apunta a la ubicación en memoria donde se encuentra ese valor. Ver [Paso por Valor vs Paso por Referencia](https://www.notion.so/Paso-por-Valor-vs-Paso-por-Referencia-27205e8a20bc805fbc9be97fc3160141?pvs=21)

## *Receiver*
Es el objeto sobre el cual se invoca un método o se accede a una propiedad. Dicho de otra forma, es el objeto que "recibe" el mensaje (la llamada al método o el acceso a la propiedad).


## Sentencia (*statement*)
Instrucción para realizar una acción. Puede ser imperativa, condicional, iterativa o componer bloques (funciones).  
Ver [Expresión](#expresión-expression)

## Serializar (*serialize*)
Codificar un objeto con el fin de transmitirlo, ya sea como una serie de *bytes* o en un formato como XML o JSON.  
Ver [Deserializar](#deserializar-deserialize)
