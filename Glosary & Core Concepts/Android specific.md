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
<!-- TOC -->

---

## *Desugaring*
Transformar características del lenguaje Java moderno (como _lambdas_ o API’s de `java.time`) en código compatible con versiones antiguas de Android, cuyos entornos de ejecución (ART/Dalvik) no las soportan de forma nativa. Ver también la definición para el [_Software_ en general](Software%20in%20general.md#desugaring)

## *Insets*
Insets describe **_how much the content of your app needs to be padded to avoid overlapping with parts of the system UI or physical device features_**. Because there are different parts of the system UI that may be visible at any given time, there are also different types of insets. **_These include the status bars, the navigations bars, the software keyboard, and more_**.  
System UI is dynamic, and therefore, insets are dynamic as well. How big they are, where they are, and how they change dependes on the system configuration and windowing environment.

> 🔍 References:  
> - https://proandroiddev.com/understanding-window-insets-in-jetpack-compose-46245b9ceffa
> - https://developer.android.com/develop/ui/compose/quick-guides/content/video/insets-in-compose
> 

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
