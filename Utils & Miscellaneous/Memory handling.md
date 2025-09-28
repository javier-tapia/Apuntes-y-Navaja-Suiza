<h1><i>Memory handling</i></h1>

***Index***:
<!-- TOC -->
  * [*Memory leaks*](#memory-leaks)
  * [*Heap dump*](#heap-dump)
  * [Zonas de memoria que usa la JVM](#zonas-de-memoria-que-usa-la-jvm)
    * [*Stack*](#stack)
    * [*Heap*](#heap)
<!-- TOC -->

---

## *Memory leaks*
Se producen cuando la app ya no usa los objetos pero todavía están siendo referenciados, por lo que el ***Garbage Collector*** no los puede eliminar de la memoria.

## *Heap dump*
Es una captura de la memoria de un proceso en un momento determinado. Algo así como un _screenshot_ de la memoria.  
Contiene datos sobre objetos, clases, campos, referencias, contenido estático, _thread stacks_, etc.

## Zonas de memoria que usa la JVM
### *Stack*
Almacena resultados parciales, variables locales, variables de referencia, parámetros y valores de retorno. También se usa para controlar la invocación y retorno de los métodos.  
Cada _thread_ en la JVM tiene asignado un _stack_ privado desde el momento que se crea.

### *Heap*
Almacena objetos y sus variables de instancia. Es un espacio de memoria dinámica que se crea al inicio de la JVM y es único.  
El _heap_ es administrado por el _Garbage Collector_.
