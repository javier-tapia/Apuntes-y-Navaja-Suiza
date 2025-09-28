<h1>Glosario de términos del mundo Android</h1>

***Index***:
<!-- TOC -->
  * [*Desugaring*](#desugaring)
  * [*Insets*](#insets)
  * [`MessageQueue`, `Looper` and `Handler`](#messagequeue-looper-and-handler)
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
