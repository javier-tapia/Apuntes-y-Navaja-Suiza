<h1><i>Memory handling</i></h1>

***Index***:
<!-- TOC -->
  * [*Stack*](#stack)
  * [*Heap*](#heap)
    * [*Heap Dump*](#heap-dump)
  * [*Garbage Collector*](#garbage-collector)
  * [*Memory Leaks*](#memory-leaks)
  * [*Memory Churn*](#memory-churn)
<!-- TOC -->

---

## *Stack*
Zona de la memoria en la que la JVM almacena resultados parciales, variables locales, variables de referencia, parámetros y valores de retorno. También se usa para controlar la invocación y retorno de los métodos.  
**Cada _thread_** en la JVM **tiene asignado un _stack_** privado desde el momento que se crea.

## *Heap*
Zona de la memoria en la que la JVM almacena objetos y sus variables de instancia. Es un **_espacio de memoria dinámica_** (aloja los **_datos en ejecución_**) que se crea al inicio de la JVM, es administrado por el _**Garbage Collector**_ y es único.  
Su tamaño es limitado y depende de la RAM disponible, por lo que si no está bien gestionada la memoria, puede provocar un **OOM** (**_``OutOfMemoryError``_**).

### *Heap Dump*
Es una captura de la memoria de un proceso en un momento determinado. Algo así como un **_screenshot_ de la memoria**.  
Contiene datos sobre objetos, clases, campos, referencias, contenido estático, _thread stacks_, etc.

## *Garbage Collector*
Se encarga de encontrar recursos que no son necesarios y liberarlos del _Heap_. El sistema operativo tiene un conjunto de criterios para determinar cuándo debe realizar el proceso de *Garbage Collector* (no es controlado por el desarrollador).  
Por cada iteración del GC, se puede producir una pérdida de memoria, ya que Android tiene que pausar el código de la app para ejecutarlo. El usuario puede notar un "tartamudeo" en la aplicación cuando hay frecuentes recolecciones de basura.

Consta de tres fases:
1. **_Before GC_**: La raíz del GC (GC _Roots_) es el nodo del que parte el GC.
2. **_Marking_**: El GC recorre el grafo de objetos en memoria desde el nodo GC _Roots_ y marca los objetos que están siendo referenciados por otros objetos en el _Heap_.
3. **_Sweep complete_**: Elimina los objetos que ya no son accesibles (los que no están marcados) y libera la memoria.

## *Memory Leaks*
Se producen cuando la app ya no usa los objetos pero todavía están siendo referenciados, por lo que el ***Garbage Collector*** no los puede eliminar de la memoria.  
El resultado final de las _memory leaks_ es que una app se quede sin memoria disponible y, desde la perspectiva del usuario, se bloquea repentinamente (**ANR o _Application Not Responding_**), conduciendo a una mala experiencia.  
Los más comunes son:
1. _Singletons_ referenciando componentes de Android.
2. Suscribirse a _listeners_ y no desuscribirse.
3. Variables estáticas referenciando componentes de Android.
4. _Inner classes_ y Clases anónimas (tienen una referencia implícita y fuerte a su clase contenedora).

## *Memory Churn*
Se refiere a la creación y liberación rápida de muchos objetos temporales, lo que puede causar problemas de rendimiento al sobrecargar el gestor de memoria.  
Cuando ocurre, el sistema necesita realizar recolecciones de basura con mayor frecuencia para liberar memoria, lo que puede generar lentitud y otros problemas de rendimiento. Por ejemplo, puede causar _jank_ y aumentar el consumo de batería.
