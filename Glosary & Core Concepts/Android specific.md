<h1>Glosario de términos del mundo Android</h1>

***Index***:
<!-- TOC -->
  * [*Desugaring*](#desugaring)
  * [*Insets*](#insets)
  * [`MessageQueue`, `Looper` and `Handler`](#messagequeue-looper-and-handler)
  * [*Screen compositing*](#screen-compositing)
    * [Componentes Principales](#componentes-principales)
    * [Proceso](#proceso)
    * [Beneficios](#beneficios)
  * [*Stack Frames*](#stack-frames)
<!-- TOC -->

---

## *Desugaring*
Transformar características del lenguaje Java moderno (como _lambdas_ o API’s de `java.time`) en código compatible con versiones antiguas de Android, cuyos entornos de ejecución (ART/Dalvik) no las soportan de forma nativa. Ver también la definición para el [_Software_ en general](Software%20in%20general.md#desugaring)

## *Insets*
> 🔍 Referencias:
> - https://proandroiddev.com/understanding-window-insets-in-jetpack-compose-46245b9ceffa
> - https://developer.android.com/develop/ui/compose/quick-guides/content/video/insets-in-compose

Describen **_las áreas que la app debe respetar para evitar superponerse con partes de la UI del Sistema (System UI) o con características físicas del dispositivo_**.  

Como **_la UI del Sistema es dinámica_** (sus componentes pueden aparecer o desaparecer en distintos momentos), **_los insets también cambian en función de las áreas que esta genera_**. Entre ellas se encuentran las zonas asociadas a **_la barra de estado (status bar), la barra de navegación (navigation bar), el teclado en pantalla (software keyboard) y otros componentes del sistema_**.  
El tamaño, la posición y la manera en que varían los _insets_ dependen de la configuración del dispositivo y del entorno de ventanas (_windowing environment_).

```kotlin
Column(
    modifier = Modifier
        .fillMaxSize()
        .padding(paddingValues)
        .consumeWindowInsets(paddingValues)
) { ... }
```

1. `padding(paddingValues)`: Este modificador aplica el espaciado recibido. En el contexto de un `Scaffold`, `paddingValues` contiene los valores necesarios para evitar que el contenido quede oculto detrás de las barras del sistema. Al aplicarlo, "se empuja" el contenido del composable para que sea visible.
2. `consumeWindowInsets(paddingValues)`: Después de haber aplicado el `padding`, este modificador **_"consume" esos _insets_ y evita su propagación hacia los hijos_**. Indica a los composables internos que ya se encargó de aplicar un espaciado para las barras del sistema, por lo cual no necesitan volver a hacerlo. Básicamente, **_evita que se aplique un espaciado doble_**.

## `MessageQueue`, `Looper` and `Handler`
TODO...

## *Screen compositing*
Es el proceso mediante el cual se combinan diferentes fuentes de contenido visual en la pantalla para formar una única imagen que se presenta al usuario.

### Componentes Principales
1. **Capas**: Cada elemento visual (como ventanas, íconos, menús y efectos gráficos) se puede tratar como una capa independiente. Estas capas pueden tener diferentes propiedades, como transparencia.
2. **Fusión (Blending)**: Cuando se combinan capas, los valores de color de cada pixel se combinan según su nivel de transparencia (alpha). Este proceso permite que las capas superiores interactúen visualmente con las capas inferiores, dando como resultado efectos como sombras o mosaicos.
3. **Buffering**: Generalmente, el contenido de las capas se dibuja primero en buffers (áreas de memoria de video) antes de ser compuestos en la pantalla. Esto ayuda a reducir el flickering y mejora la eficiencia de renderizado.
4. **GPU y Hardware Acceleration**: El screen compositing a menudo se maneja mediante la GPU, especialmente cuando se utiliza aceleración por hardware. Esto permite que el proceso sea más eficiente, aprovechando el procesamiento paralelo que ofrece la GPU.

### Proceso
1. **Renderizar Capas**: Cada capa se dibuja en su buffer correspondiente.
2. **Composición de la Pantalla**: La GPU toma estas capas, las combina según sus propiedades y las renderiza en el buffer de salida (la pantalla).
3. **Visualización**: La imagen compuesta se muestra en la pantalla, proporcionando al usuario una interfaz interactiva.

### Beneficios
- **Eficiencia**: Permite un manejo más eficiente de múltiples elementos visuales al permitir que cada uno se dibuje de manera independiente.
- **Efectos Visuales**: Facilita la creación de efectos visuales complejos, como sombras, transparencias y animaciones suaves.
- **Mejor Experiencia de Usuario**: Al permitir que las interfaces sean más dinámicas y responden mejor a las interacciones del usuario.

## *Stack Frames*
Al colocar un _breakpoint_ y correr la app en modo _debug_, cuando se detiene en dicho _breakpoint_, se pueden visualizar los **_Frames_** del hilo pausado.  
**Cada _frame_ representa una llamada a una función en la pila de llamadas (_call stack_) que llevó hasta la línea actual**.

- El _frame_ superior suele ser el método/línea donde se detuvo la ejecución.
- Los _frames_ inferiores son los “llamadores” (la cadena de llamadas) y permiten navegar el *call stack* para ver variables/estado en cada nivel.
