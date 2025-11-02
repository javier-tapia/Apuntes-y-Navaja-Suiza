<h1>Kotlin Multiplatform (KMP) y Compose Multiplatform (CMP)</h1>

***Index***:
<!-- TOC -->
  * [0. ¿Qué son KMP y CMP?](#0-qué-son-kmp-y-cmp)
    * [0.1. Kotlin Multiplatform (KMP)](#01-kotlin-multiplatform-kmp)
    * [0.2. Compose Multiplatform (CMP)](#02-compose-multiplatform-cmp)
    * [0.3. Ventajas Principales](#03-ventajas-principales)
  * [1. Arquitectura de un Proyecto KMP](#1-arquitectura-de-un-proyecto-kmp)
    * [1.1. Estructura Modular Típica](#11-estructura-modular-típica)
    * [1.2. Flujo de Interacción](#12-flujo-de-interacción)
    * [1.3. El Rol de los Módulos de Plataforma](#13-el-rol-de-los-módulos-de-plataforma)
  * [2. Conceptos Fundamentales de KMP](#2-conceptos-fundamentales-de-kmp)
    * [2.1. *Targets*: Los destinos de compilación](#21-targets-los-destinos-de-compilación)
    * [2.2. *Source Sets*: Las carpetas con el código](#22-source-sets-las-carpetas-con-el-código)
    * [2.3. La Relación entre *Target* y *Source Set*](#23-la-relación-entre-target-y-source-set)
    * [2.4. El Mecanismo `expect` y `actual`](#24-el-mecanismo-expect-y-actual)
  * [3. Gestión de Dependencias](#3-gestión-de-dependencias)
    * [3.1. Dependencias `api` vs. `implementation`](#31-dependencias-api-vs-implementation)
  * [4. Testing en KMP](#4-testing-en-kmp)
    * [4.1. Tests en el código común (`commonTest`)](#41-tests-en-el-código-común-commontest)
    * [4.2. Tests específicos de plataforma (`androidTest`, `iosTest`, etc.)](#42-tests-específicos-de-plataforma-androidtest-iostest-etc)
  * [5. Nota sobre el desarrollo en Windows](#5-nota-sobre-el-desarrollo-en-windows)
  * [6. *Troubleshooting*](#6-troubleshooting)
    * [6.1. *Warning: The Kotlin Hierarchy Template*](#61-warning-the-kotlin-hierarchy-template)
    * [6.2. Error: `No actual for expect` en iOS](#62-error-no-actual-for-expect-en-ios)
    * [6.3. *Warning: Variable de source set "nunca usada"*](#63-warning-variable-de-source-set-nunca-usada)
  * [7. Referencias y Recursos](#7-referencias-y-recursos)
<!-- TOC -->

---

## 0. ¿Qué son KMP y CMP?
### 0.1. Kotlin Multiplatform (KMP)
Kotlin Multiplatform (KMP) es una funcionalidad del lenguaje Kotlin que permite escribir y reutilizar código a través de múltiples plataformas. Su objetivo principal es compartir la lógica de negocio, el acceso a datos y los modelos desde una única base de código, mientras se mantiene la flexibilidad para escribir código específico de cada plataforma cuando sea necesario para acceder a APIs nativas.

### 0.2. Compose Multiplatform (CMP)
Compose Multiplatform es un _framework_ de UI declarativo moderno creado por JetBrains, que extiende Jetpack Compose de Android a otras plataformas. Permite que la interfaz de usuario (UI) también se defina una sola vez en el código común y se renderice de forma nativa en cada plataforma de destino, como _desktop_ (Windows, macOS, Linux), Web, Android y iOS.

### 0.3. Ventajas Principales
- **Reutilización de código**: La lógica de negocio y, en gran medida, la UI, solo necesitan escribirse una vez.
- **Consistencia y Mantenimiento**: Asegura un comportamiento coherente de la aplicación en todas las plataformas y simplifica la corrección de errores y las actualizaciones.
- **Rendimiento Nativo**: El código Kotlin se compila al formato esperado por la plataforma de destino (ej. bytecode de JVM, binario nativo, JavaScript), por lo que no hay una capa de abstracción que penalice el rendimiento.

## 1. Arquitectura de un Proyecto KMP
### 1.1. Estructura Modular Típica
Un proyecto KMP se organiza comúnmente en los siguientes módulos:

- **``shared``**: Es el corazón del proyecto. Contiene la lógica de negocio, el acceso a datos, los modelos y, si se usa CMP, la UI que se comparte entre todas las plataformas.
- **``androidApp``**: La aplicación para Android. Este módulo consume el código del módulo ``shared`` y proporciona el punto de entrada (``Activity``) y la configuración específica para Android (``AndroidManifest.xml``).
- **``desktopApp``**: La aplicación de escritorio (JVM). De forma similar, consume el módulo ``shared`` y gestiona el lanzamiento de la aplicación de escritorio a través de su función ``main``.
- **``iosApp``**: Un proyecto de Xcode que consume el _framework_ nativo generado por el módulo `shared`. Proporciona el punto de entrada de la aplicación Swift/Objective-C y gestiona el ciclo de vida de la aplicación iOS.
- **``webApp``**: La aplicación web (JS). Consume la salida de JavaScript del módulo ``shared``. Generalmente contiene el archivo `index.html` y los recursos web, y se encarga de iniciar la aplicación en el navegador. Se lo suele llamar también `jsApp`.

### 1.2. Flujo de Interacción
1. El módulo **``shared``** define las interfaces, la lógica principal y la UI compartida.
2. Los módulos de plataforma (``androidApp``, ``desktopApp``, ``iosApp``, ``webApp``) dependen del módulo ``shared``.
3. Cada aplicación de plataforma inicializa la capa compartida y utiliza sus componentes para construir la aplicación final y manejar las interacciones del usuario.

### 1.3. El Rol de los Módulos de Plataforma
Aunque `shared` contiene la mayor parte del código (lógica y UI), los módulos específicos de la plataforma son esenciales por varias razones:

- **Punto de Entrada**: Cada plataforma inicia una aplicación de forma diferente. El módulo `shared` se configura como una librería, no como una aplicación ejecutable. Los módulos de plataforma proveen el código de arranque, que es una pequeña capa de código nativo que se encarga de lanzar la UI compartida.
  - En ``androidApp``: Es una ``Activity`` de Android que, en su ``onCreate``, llama a ``setContent`` para mostrar la UI de Compose del módulo ``shared``.
  - En ``iosApp``: Es un ``UIViewController`` en Swift que carga y presenta la UI de Compose Multiplatform.
  - En ``desktopApp``: Es la función ``main`` que crea la ``Window`` y le dice que dibuje la UI del módulo ``shared``.
  - En ``webApp``: Es el archivo ``index.html`` que carga el ``.js`` compilado y un pequeño script que monta la aplicación en un elemento del DOM (como un ``<div>``).
- **Dependencias y SDK's específicos**: Permiten incluir dependencias que solo son relevantes para su plataforma (ej. librerías de Jetpack en Android).
- **Empaquetado**: El proceso para generar el artefacto final es radicalmente diferente en cada plataforma (APK/AAB para Android, JAR/MSI/DMG para _desktop_, un paquete `.app` para iOS, o un conjunto de archivos estáticos HTML/JS/CSS para Web). Los módulos de aplicación se encargan de esta configuración.

## 2. Conceptos Fundamentales de KMP
### 2.1. *Targets*: Los destinos de compilación
En KMP, un _target_ representa una plataforma específica para la que se compilará el código. Se declaran en el archivo `build.gradle.kts` del módulo `shared`. Ejemplos comunes son `android`, `jvm` (para _desktop_), `iosArm64`, o `js`.

### 2.2. *Source Sets*: Las carpetas con el código
Un _source set_ es un conjunto de carpetas con código fuente. Al definir un _target_, el _plugin_ de Kotlin crea automáticamente un _source set_ para él.
- **`commonMain`**: Contiene el código que es 100% independiente de la plataforma y se comparte con todos los _targets_.
- **`androidMain`**, **`desktopMain`**, etc.: Contienen la implementación específica para una plataforma concreta.

### 2.3. La Relación entre *Target* y *Source Set*
> **Los _Targets_ definen el "para qué" (para qué plataformas compilar). Los _Source Sets_ organizan el "dónde" (dónde vive el código para esas plataformas).**

El código en `commonMain` se compila para todos los _targets_. El código en un _source set_ específico (como `androidMain`) solo se compila para el _target_ correspondiente.

### 2.4. El Mecanismo `expect` y `actual`
Este mecanismo permite que el código común declare una funcionalidad que necesita, dejando que cada plataforma provea su propia implementación.
- En `commonMain`, se define una declaración con la palabra clave `expect` (ej. `expect fun getPlatformName(): String`).
- En cada _source set_ de plataforma (`androidMain`, `desktopMain`), se provee la implementación `actual` correspondiente (ej. `actual fun getPlatformName(): String = "Android"`).

Así, el código común puede llamar a `getPlatformName()` sin conocer la plataforma subyacente.

## 3. Gestión de Dependencias
Las dependencias se gestionan en los archivos `build.gradle.kts`. Dentro del módulo `shared`, es posible declararlas para cada _source set_:
- **`commonMain`**: Para librerías multiplataforma como **_Ktor_** (red), **_SQLDelight_** (base de datos) o **_Kotlinx Serialization_**.
- **`androidMain`**, **`desktopMain`**, etc.: Para dependencias específicas de una plataforma, como motores concretos para una librería o acceso a una API nativa. Por ejemplo, para hacer una petición de red con **_Ktor_**, en Android (``androidMain``) se usaría el motor **_OkHttp_**, pero en la app de escritorio (``desktopMain``) se usaría **_CIO_**.

### 3.1. Dependencias `api` vs. `implementation`
Cuando se añade una dependencia a un _source set_ específico (ej. a `desktopMain` en el módulo `shared`), por defecto, esa dependencia es privada y no es visible para los módulos que consumen `shared` (como `desktopApp`). Este es un mecanismo de encapsulamiento para evitar que las dependencias de una plataforma se filtren a otra.

Para controlar esta visibilidad, se usan dos tipos de declaraciones:

- **`implementation`**: Es la opción por defecto. La dependencia es un detalle de implementación privado y no se expone a los módulos consumidores. Es ideal para dependencias internas que los módulos de aplicación no necesitan conocer.
  - *Ejemplo*: El motor específico de **_Ktor_** (como **_OkHttp_** o **_CIO_**). El módulo `desktopApp` no necesita saber qué motor se está usando, solo quiere un `HttpClient` funcional.

- **`api`**: La dependencia se convierte en parte de la "Interfaz Pública de Programación" (*API*) del módulo y se exporta a los módulos consumidores. Esto es necesario solo cuando las clases o funciones de la dependencia deben ser usadas directamente en el módulo consumidor.
  - *Ejemplo*: En una aplicación Compose for Desktop, el módulo `desktopApp` necesita usar las funciones `Window()` y `application()` de la librería `compose.desktop.currentOs`. Por lo tanto, el módulo `shared` debe declarar esta dependencia usando `api` para "exportar" esa capacidad.

**Regla general**: Usar siempre `implementation` a menos que sea estrictamente necesario que el módulo consumidor interactúe directamente con las clases de la dependencia. Esto mejora el encapsulamiento y los tiempos de compilación.

## 4. Testing en KMP
La estrategia de testing en KMP sigue la estructura de los _source sets_ para maximizar la reutilización y, al mismo tiempo, garantizar el correcto funcionamiento en cada plataforma.

### 4.1. Tests en el código común (`commonTest`)
Este es el lugar ideal para la gran mayoría de los tests unitarios. El código escrito aquí se compila y ejecuta en todas las plataformas _target_.
- **Qué testear aquí**: Lógica de negocio pura (casos de uso, _ViewModels_/_Presenters_), validaciones, _mappers_ de datos, y cualquier componente que no tenga dependencias de APIs de plataforma.
- **Ventaja principal**: Se escriben una sola vez y verifican que el comportamiento central de la aplicación sea consistente en todos los _targets_.
- **Limitación**: No se puede acceder a APIs específicas de una plataforma (como el `Context` de Android o `NSUserDefaults` de iOS).

### 4.2. Tests específicos de plataforma (`androidTest`, `iosTest`, etc.)
Estos _source sets_ se utilizan para testear código que sí depende de una plataforma concreta.

**Qué testear aquí**:
  - Las implementaciones `actual` de las declaraciones `expect`. Por ejemplo, si se tiene una `expect` para leer/escribir en disco, aquí se testea que la implementación `actual` para Android use `SharedPreferences` correctamente y la de iOS use `NSUserDefaults`.
  - El código que interactúa directamente con los SDKs nativos.
  - Tests de instrumentación o UI que necesitan un emulador, simulador o dispositivo físico para ejecutarse. Por ejemplo, los tests de UI de Compose que verifican que un `@Composable` se renderiza correctamente en una pantalla de Android.

## 5. Nota sobre el desarrollo en Windows
En Windows, Android Studio no ofrece directamente algunas plantillas para crear un proyecto KMP desde cero. Esto no impide usar la tecnología, pero implica que la configuración inicial puede requerir más pasos manuales, como:
- Crear manualmente la estructura de módulos (`shared`, `androidApp`, etc.).
- Configurar los archivos de Gradle y las dependencias.

El proyecto resultante, sin embargo, funciona de la misma manera que uno creado en Mac o Linux.

## 6. *Troubleshooting*
### 6.1. *Warning: The Kotlin Hierarchy Template*
**El _warning_**:  
> The Default Kotlin Hierarchy Template was not applied to 'project ':shared'':
> Explicit .dependsOn() edges were configured for the following source sets: [desktopMain]
> Consider removing dependsOn-calls or disabling the default template...

**¿Qué significa?**  
En las versiones modernas de KMP, Gradle intenta crear automáticamente una "jerarquía de _source sets_". Por ejemplo, sabe que `androidMain` y `desktopMain` son plataformas JVM y podría crear un _source set_ intermedio `jvmMain` del que ambos dependan para reducir duplicación de código.
El _warning_ aparece porque en el archivo `build.gradle.kts` del módulo `shared` existe una configuración manual como esta:
```kotlin
val desktopMain by getting {
    dependsOn(commonMain) // Esta línea causa el warning
}
```
Al escribir manualmente `dependsOn(commonMain)`, se deshabilita la plantilla de jerarquía (_hierarchy template_) automática. Gradle simplemente está informando que el usuario tomó el control manual.

**¿Es un problema?**  
No, para la mayoría de los proyectos simples no es un problema. Una configuración manual es perfectamente clara y correcta. La plantilla automática es más útil en proyectos masivos con muchos _targets_.

**¿Cómo solucionarlo?**  
Existen dos opciones:
1.  **(Recomendado) Silenciar el _warning_**: Si la configuración manual es la deseada, simplemente se le indica a Gradle que no advierta sobre ello.
    -   Abrir el archivo `gradle.properties`.
    -   Añadir la siguiente línea:
        ```groovy
        kotlin.mpp.applyDefaultHierarchyTemplate=false
        ```
2.  **(Alternativa) Usar la plantilla**: Eliminar las llamadas `dependsOn(commonMain)` de los _source sets_ (`desktopMain`, `androidMain`, etc.). La plantilla por defecto los conectará automáticamente a `commonMain`. Esta es la forma considerada "moderna".

### 6.2. Error: `No actual for expect` en iOS
**El error**:  
> No actual for expect declaration in module(s): iosSimulatorArm64Main, iosX64Main, iosArm64Main

**¿Qué significa?**  
Este error ocurre porque una declaración `expect` en `commonMain` no tiene su correspondiente implementación `actual` en los _targets_ finales de iOS.  
La razón principal es la diversidad de arquitecturas de _hardware_ en el ecosistema de Apple:
- **Android y _Desktop_ (JVM)**: Al compilar para estas plataformas, el resultado es _bytecode_ de la JVM, que es universal. Una Máquina Virtual de Java (como ART en Android) se encarga de interpretarlo para la arquitectura específica del dispositivo (ARM, x86, etc.). Por ello, solo se necesita un único _target_.
- **iOS (Kotlin/Native)**: Kotlin/Native compila el código Kotlin directamente a código máquina nativo, sin una máquina virtual intermedia. Esto obliga a generar un binario diferente para cada arquitectura de procesador que se quiera soportar:
    - `iosArm64`: Para iPhones y iPads físicos modernos (procesadores Apple Silicon).
    - `iosX64`: Para simuladores de iOS en Macs con procesadores Intel.
    - `iosSimulatorArm64`: Para simuladores de iOS en Macs con procesadores Apple Silicon (M1, M2, etc.).

La parte sutil es que, aunque se haya colocado la implementación `actual` en un _source set_ común para iOS como `iosMain`, Gradle no sabe automáticamente que estos _targets_ específicos deben usar ese código.

**¿Es un problema?**  
Sí, es un error de compilación que impide generar el _framework_ para iOS. El compilador necesita saber exactamente dónde encontrar la implementación `actual` para cada _target_ final.

**¿Cómo solucionarlo?**  
La solución es configurar explícitamente en el archivo `build.gradle.kts` del módulo `shared` que los _source sets_ de los _targets_ específicos de iOS dependen de `iosMain`.

```kotlin
// Se obtiene una referencia al source set iosMain (o se crea si no existe)
val iosMain by creating {
    // Se vincula explícitamente con commonMain
    dependsOn(commonMain)
}

// Se vincula cada target específico de iOS con iosMain
val iosX64Main by getting {
    dependsOn(iosMain)
}
val iosArm64Main by getting {
    dependsOn(iosMain)
}
val iosSimulatorArm64Main by getting {
    dependsOn(iosMain)
}
```

### 6.3. *Warning: Variable de source set "nunca usada"*
**El _warning_**  
El IDE marca una variable de _source set_ como `androidMain` como si nunca fuera usada.

**¿Qué significa?**  
Es una peculiaridad del editor de código de Android Studio y el DSL de Gradle que se puede ignorar de forma segura.

- **Lo que realmente pasa**: Aunque parece que la variable `androidMain` no se usa, en realidad se está utilizando para configurar el _source set_ a través de la función `by getting`. El _plugin_ de Kotlin Multiplatform utiliza internamente esta declaración para localizar las carpetas (ej. `src/androidMain/kotlin`), configurar dependencias y conectar todo el grafo de compilación.
- **Por qué el IDE se confunde**: El análisis estático del IDE no siempre es lo suficientemente inteligente como para entender que estas declaraciones `val` en el DSL de Gradle son, de hecho, "usadas" por el sistema de _build_ en segundo plano. Simplemente ve una variable a la que no se hace referencia explícita y la marca como no utilizada.

**En resumen**: Es un falso positivo del IDE. El código es correcto y necesario.

## 7. Referencias y Recursos
- [Documentación oficial de Kotlin Multiplatform](https://kotlinlang.org/docs/multiplatform-get-started.html)
- [Documentación oficial de Compose Multiplatform](https://www.jetbrains.com/lp/compose-multiplatform/)
- [Proyecto de práctica - KmpClientSample](https://github.com/javier-tapia/KmpClientSample)
