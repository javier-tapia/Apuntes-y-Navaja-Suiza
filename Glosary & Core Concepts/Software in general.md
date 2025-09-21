# Glosario de términos de _Software_ en general

- [*Abstraction*](#abstraction)
- [*Atomicity*](#atomicity)
- [*Desugaring*](#desugaring)
- [*Polymorphism*](#polymorphism)
- [*Receiver*](#receiver)

---

## *Abstraction*
Principio fundamental en la **Programación Orientada a Objetos** (**OOP**) que permite ocultar los detalles de implementación y resaltar las características esenciales de un objeto.  
En el contexto de un sistema, al hablar de **componentes de alto nivel** y **bajo nivel**, nos referimos al **grado de abstracción** que cada componente representa. Los componentes de **alto nivel** suelen estar más alejados de los detalles específicos y operan con conceptos más generales (mayor abstracción). Por ejemplo, una interfaz que define métodos comunes para varias clases es un componente de alto nivel, porque permite a otros componentes interactuar con una amplia gama de objetos sin conocer sus detalles de implementación específicos. Por otro lado, los componentes de **bajo nivel** manejan detalles específicos y son más concretos (menor abstracción), como una clase que implementa funcionalidades particulares de un sistema.  
En *Clean Architecture*, esta jerarquía de niveles de abstracción se traduce en un esquema de capas circulares donde las capas más internas son más abstractas (alto nivel) y representan la lógica de negocio, mientras que las capas externas manejan los detalles concretos y la implementación (bajo nivel). Este enfoque no solo mejora la organización y la mantenibilidad del código, sino que también facilita la prueba y evolución de cada componente por separado.

## *Atomicity*
Atomic implies indivisibility and irreducibility, so _**an atomic operation must be performed entirely or not performed at all**_. An operation that is atomic on one machine may not be on another.  
An atomic operation is an operation during which a processor can simultaneously read a location and write it in the same bus operation. This prevents any other processor or I/O device from writing or reading memory until the operation is complete.  
Atomicity is a trait that defines whether an operation can be interrupted or not. It matters because _**if something can't be interrupted, it's intrinsically thread safe**_.

## *Desugaring*
Transformación de una construcción de lenguaje de alto nivel (más "dulce" o conveniente para el programador) en una construcción equivalente de nivel inferior (más "cruda" o básica) que puede ser entendida por un intérprete o compilador que no soporta la construcción de alto nivel directamente. En resumen, se trata de **_traducir características de un lenguaje a una forma más básica_**. Ver también la definición específica para [Android](Android%20specific.md#desugaring).

## *Polymorphism*
Concepto fundamental en la **Programación Orientada a Objetos** que permite que un objeto o función tome múltiples formas. En términos más simples, el polimorfismo permite que una sola interfaz se use para representar diferentes tipos de datos o acciones. Esto es útil porque promueve la flexibilidad y la reutilización del código, permitiendo que una función o método opere sobre distintos tipos de datos según el contexto en que se utilice.

- ***Dynamic Polymorphism*** → Capacidad de un objeto para comportarse de diferentes maneras en tiempo de ejecución (*runtime*). Generalmente se logra a través de la herencia (con clases abstractas o implementando interfaces) y sobreescritura (*overriding*) de funciones.  
- ***Static Polymorphism*** → Capacidad de la misma función de manejar diferentes tipos de datos en tiempo de compilación (*compile-time*). Se implementa principalmente a través de la sobrecarga (*overloading*) de funciones y el compilador determina cuál usar basándose en su firma (número, orden y tipos de parámetros).

## *Receiver*
Es el objeto sobre el cual se invoca un método o se accede a una propiedad. Dicho de otra forma, es el objeto que "recibe" el mensaje (la llamada al método o el acceso a la propiedad).
