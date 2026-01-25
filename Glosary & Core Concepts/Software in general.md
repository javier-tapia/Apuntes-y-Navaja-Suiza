<h1>Glosario de términos de <i>Software</i> en general</h1>

***Index***:
<!-- TOC -->
  * [*Abstraction*](#abstraction)
  * [ABI (*Application Binary Interface*)](#abi-application-binary-interface)
  * [🛠 API vs Librería vs *Framework*](#-api-vs-librería-vs-framework)
  * [Asincronía (*Asynchrony*)](#asincronía-asynchrony)
  * [*Atomicity*](#atomicity)
  * [*Backpressure*](#backpressure)
  * [*Build*](#build)
  * [*Bytecode*](#bytecode)
  * [*Callback*](#callback)
  * [*Casting* (*cast*)](#casting-cast)
  * [CDN (*Content Delivery Network*)](#cdn-content-delivery-network)
  * [*Claims* (reclamos)](#claims-reclamos)
  * [Código binario](#código-binario)
  * [Código máquina](#código-máquina)
  * [Compatibilidad binaria](#compatibilidad-binaria)
  * [Compilación](#compilación)
  * [Concurrencia (*Concurrency*)](#concurrencia-concurrency)
  * [Copia profunda (*deep copy*)](#copia-profunda-deep-copy)
  * [Copia superficial (*shallow copy*)](#copia-superficial-shallow-copy)
  * [Deserializar (*deserialize*)](#deserializar-deserialize)
  * [*Desugaring*](#desugaring)
  * [DSL (*Domain-Specific Language*)](#dsl-domain-specific-language)
  * [Expresión (*expression*)](#expresión-expression)
  * [Funciones vs Métodos](#funciones-vs-métodos)
  * [HTTP (*HyperText Transfer Protocol*)](#http-hypertext-transfer-protocol)
  * [JWT (*JSON Web Token*)](#jwt-json-web-token)
  * [Latencia](#latencia)
  * [Objeto compuesto](#objeto-compuesto)
  * [Paralelismo (*Parallelism*)](#paralelismo-parallelism)
  * [Parseo (*parsing*)](#parseo-parsing)
  * [Paso por Valor vs Paso por Referencia](#paso-por-valor-vs-paso-por-referencia)
    * [Paso por Valor](#paso-por-valor)
    * [Paso por Referencia](#paso-por-referencia)
  * [*Polymorphism*](#polymorphism)
  * [Predicado (*predicate*)](#predicado-predicate)
  * [*Principle of Least Privilege* (PoLP)](#principle-of-least-privilege-polp)
  * [Punteros](#punteros)
  * [*Receiver*](#receiver)
  * [Sentencia (*statement*)](#sentencia-statement)
  * [Serializar (*serialize*)](#serializar-serialize)
  * [TLS/SSL](#tlsssl)
<!-- TOC -->

---

## *Abstraction*
Principio fundamental en la **Programación Orientada a Objetos** (**OOP**) que permite ocultar los detalles de implementación y resaltar las características esenciales de un objeto.  
En el contexto de un sistema, al hablar de **componentes de alto nivel** y **bajo nivel**, nos referimos al **grado de abstracción** que cada componente representa. Los componentes de **alto nivel** suelen estar más alejados de los detalles específicos y operan con conceptos más generales (mayor abstracción). Por ejemplo, una interfaz que define métodos comunes para varias clases es un componente de alto nivel, porque permite a otros componentes interactuar con una amplia gama de objetos sin conocer sus detalles de implementación específicos. Por otro lado, los componentes de **bajo nivel** manejan detalles específicos y son más concretos (menor abstracción), como una clase que implementa funcionalidades particulares de un sistema.  
En *Clean Architecture*, esta jerarquía de niveles de abstracción se traduce en un esquema de capas circulares donde las capas más internas son más abstractas (alto nivel) y representan la lógica de negocio, mientras que las capas externas manejan los detalles concretos y la implementación (bajo nivel). Este enfoque no solo mejora la organización y la mantenibilidad del código, sino que también facilita la prueba y evolución de cada componente por separado.

## ABI (*Application Binary Interface*)
Conjunto de reglas que describe cómo una aplicación (por ejemplo, un APK de Android) debe interactuar con el código a nivel de máquina en la CPU de un dispositivo. Es el "contrato" a bajo nivel entre el _software_ y el _hardware_.  
Se puede pensar **como una API, pero para el código binario**. Mientras que una API define cómo interactuar con el código fuente (nombres de funciones, parámetros, etc.), una ABI define:
1. **El Conjunto de Instrucciones de la CPU**: Qué "idioma" habla el procesador (por ejemplo, ARM o x86).
2. **El Orden de los Bytes (_Endianness_)**: Cómo se almacenan los números en la memoria (_little-endian_ o _big-endian_).
3. **Convenciones de Llamada a Funciones**: Cómo se pasan los parámetros a las funciones y cómo se devuelven los valores a nivel de ensamblador.
4. **Formato de los Archivos Binarios**: Cómo se estructuran los archivos ejecutables y las librerías compartidas (como los archivos ``.so`` en Android).

## 🛠 API vs Librería vs *Framework*
- **API (_Application Programming Interface_)** ➡️ **_Define QUÉ se puede hacer, pero NO CÓMO se hace_**.  
  **_Interfaz pública (o contrato)_** que determina cómo interactuar con un sistema (por ejemplo, qué funciones tiene y cómo se utilizan) sin exponer su implementación interna (qué hacen esas funciones por debajo).  
  ⚡ **Relación:** Una API puede ser implementada por una librería o un _framework_ y consumida por una aplicación cliente.  
  📌 **Ejemplos:** Android SDK (`MediaPlayer`, `CameraX`), REST API de GitHub, OpenGL.
- **Librería** ➡️ **_Define CÓMO se hace_**.  
  Código reutilizable que proporciona **_implementaciones concretas_** y que **_el desarrollador puede invocar directamente_**.  
  ⚡ **Relación:** Puede definir cómo cumplir el contrato de una API (implementación); puede ser usada dentro de un _framework_; puede exponer su propia API.  
  📌 **Ejemplos:** ExoPlayer, Retrofit, Room, TensorFlow, React.
- ***Framework*** ➡️ **_Define la arquitectura y orquesta el flujo de control_**.  
  Conjunto estructurado de librerías y APIs que **_llama al código del desarrollador_** (**_inversión de control_**).  
  ⚡ **Relación:** Integra librerías internas o externas y expone APIs; el desarrollador adapta su código al marco definido.  
  📌 **Ejemplos:** Jetpack Compose, Spring, Django, Angular.

## Asincronía (*Asynchrony*)
Modelo de ejecución donde una operación inicia y **no bloquea** al llamador mientras espera su resultado. El trabajo continúa “en segundo plano” y la finalización se notifica más tarde (por _callbacks_, promesas/futuros, eventos o reanudación de una corrutina). Se usa especialmente **para I/O y tareas de larga duración**.  
Ver [Concurrencia](#concurrencia-concurrency)

## *Atomicity*
Atomic implies indivisibility and irreducibility, so _**an atomic operation must be performed entirely or not performed at all**_. An operation that is atomic on one machine may not be on another.  
An atomic operation is an operation during which a processor can simultaneously read a location and write it in the same bus operation. This prevents any other processor or I/O device from writing or reading memory until the operation is complete.  
Atomicity is a trait that defines whether an operation can be interrupted or not. It matters because _**if something can't be interrupted, it's intrinsically thread safe**_.

## *Backpressure*
Mecanismo de control de flujo que **regula la velocidad de producción de datos cuando el consumidor no puede procesarlos al mismo ritmo**, evitando saturación, pérdida de información o fallas por sobrecarga. Permite aplicar estrategias como espera, almacenamiento en _buffer_, descarte o reducción de datos para mantener la estabilidad del sistema.

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

## CDN (*Content Delivery Network*)
Red distribuida de servidores que **almacena, replica y entrega contenido** (como videos, imágenes o archivos estáticos) desde ubicaciones geográficas cercanas al usuario.  
Su objetivo es **reducir la [latencia](#latencia)** y **optimizar la entrega de recursos** a través del protocolo [HTTP (o HTTPS)](#http-hypertext-transfer-protocol), mejorando la velocidad, estabilidad y escalabilidad del servicio.

## *Claims* (reclamos)
Son simplemente **_pares clave-valor incluidos dentro de un token_** (generalmente un [JWT](#jwt-json-web-token)) que describen propiedades verificadas sobre el usuario o sobre el _token_ mismo.

Ejemplos de _claims_:

- ``sub``: ID único del usuario. 
- ``email``: Correo. 
- ``iss``: Quién emitió el _token_. 
- ``exp``: Cuándo expira el _token_.

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

## Concurrencia (*Concurrency*)
Capacidad de un sistema para **gestionar varias tareas en progreso** durante el mismo intervalo de tiempo. Las tareas pueden ejecutarse **intercaladas** (_time-slicing_) en un solo hilo o repartidas entre múltiples hilos/procesos. Es principalmente un concepto de **estructura y coordinación** (composición, sincronización, comunicación, control de acceso a recursos).  
Ver [Paralelismo](#paralelismo-parallelism) y [Asincronía](#asincronía-asynchrony)

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

## DSL (*Domain-Specific Language*)
> 👉 Idea clave: **menos “cómo hacerlo”, más “qué se quiere expresar”**.

Lenguaje diseñado para resolver problemas dentro de un **dominio específico**, en lugar de ser de propósito general.  
Ofrece una sintaxis y abstracciones cercanas al problema que se quiere modelar, permitiendo expresar soluciones de forma **más declarativa, legible y concisa** que con un lenguaje general.

📌 **Ejemplos**:  
- **SQL** :arrow_right: consultas a bases de datos 
- **_Regex_** :arrow_right: patrones de texto 
- **Gradle Kotlin DSL** :arrow_right: configuración de builds 
- **DSLs embebidas en código** (como _builders_ en Kotlin)

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

## HTTP (*HyperText Transfer Protocol*)
Protocolo que define cómo se **transfieren y solicitan datos** entre un cliente (por ejemplo, un navegador o aplicación móvil) y un servidor web.  
**HTTPS** (***HTTP Secure***) es su versión segura, la cual utiliza [**_TLS/SSL_**](#tlsssl) para **cifrar la comunicación** y garantizar la **confidencialidad, integridad y autenticación** de los datos transmitidos.

## JWT (*JSON Web Token*)
Es un formato estándar de _token_, compuesto por tres partes codificadas en Base64:

1. **_Header_** :arrow_right: Metadatos (algoritmo, tipo de _token_)
2. **_Payload_** :arrow_right: Los _claims_ ([reclamos](#claims-reclamos)) 
3. **_Signature_** :arrow_right: Firma digital para verificar autenticidad

Ventajas:

- Portable 
- Firmado digitalmente 
- No requiere consultar al servidor para validar su integridad (solo la firma)

## Latencia
Tiempo que transcurre entre que se envía una señal o dato y el momento en que se recibe o procesa la respuesta. En el contexto de un _streaming_, representa el retraso entre la captura del contenido y su visualización en el dispositivo del usuario.

## Objeto compuesto
Objetos que contienen otros objetos, como listas o instancias de clase

## Paralelismo (*Parallelism*)
Caso particular de concurrencia donde hay **ejecución simultánea real** de múltiples tareas o partes de una tarea, típicamente en **múltiples cores/CPUs** (o unidades de cómputo). Es principalmente un concepto de **rendimiento**: aumentar _throughput_ o reducir el tiempo total de cómputo ejecutando trabajo al mismo tiempo.  
Ver [Concurrencia](#concurrencia-concurrency)

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

- ***Dynamic Polymorphism*** :arrow_right: Capacidad de un objeto para comportarse de diferentes maneras en tiempo de ejecución (*runtime*). Generalmente se logra a través de la herencia (con clases abstractas o implementando interfaces) y sobreescritura (*overriding*) de funciones.  
- ***Static Polymorphism*** :arrow_right: Capacidad de la misma función de manejar diferentes tipos de datos en tiempo de compilación (*compile-time*). Se implementa principalmente a través de la sobrecarga (*overloading*) de funciones y el compilador determina cuál usar basándose en su firma (número, orden y tipos de parámetros).

## Predicado (*predicate*)
Representan funciones de argumento único que devuelven un valor booleano.

## *Principle of Least Privilege* (PoLP)
Principio general de seguridad que implica que un componente, proceso o app solo debe tener los **permisos mínimos necesarios**, **durante el menor tiempo posible**, para cumplir su función.

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

## TLS/SSL
> 👉 Hoy en día, TLS reemplaza completamente a SSL, que está obsoleto.

**TLS** (**_Transport Layer Security_**) y su predecesor **SSL** (**_Secure Sockets Layer_**) son protocolos criptográficos que protegen la comunicación en redes, como Internet (ver también [HTTP](#http-hypertext-transfer-protocol)).  
Proveen tres garantías principales:
- **Cifrado**: los datos transmitidos no pueden ser leídos por terceros. 
- **Integridad**: los datos no pueden ser modificados sin ser detectado. 
- **Autenticación**: el cliente puede verificar la identidad del servidor (y viceversa, si se requiere).
