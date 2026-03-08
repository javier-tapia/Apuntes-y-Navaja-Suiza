<h1>Tips Android Studio</h1>

***Index***:
<!-- TOC -->
  * [Transferir *settings* de AS a IntelliJ y viceversa](#transferir-settings-de-as-a-intellij-y-viceversa)
  * [Cambiar de ubicación las carpetas `.android`, `.gradle`, `Sdk`](#cambiar-de-ubicación-las-carpetas-android-gradle-sdk)
  * [Generar *Bundles* y APKs por terminal (con *Gradle*)](#generar-bundles-y-apks-por-terminal-con-gradle)
    * [APK “Directo” (sin *Android App Bundle*)](#apk-directo-sin-android-app-bundle)
    * [APK Universal desde un *Android App Bundle*](#apk-universal-desde-un-android-app-bundle)
    * [*Android App Bundle* (`.aab`)](#android-app-bundle-aab)
  * [Agregar recursos exportados de Figma a un proyecto](#agregar-recursos-exportados-de-figma-a-un-proyecto)
    * [Densidad de pantalla y conversión](#densidad-de-pantalla-y-conversión)
  * [JaCoCo en AndroidStudio](#jacoco-en-androidstudio)
    * [Agregar paquetes y/o clases al filtro de JaCoCo](#agregar-paquetes-yo-clases-al-filtro-de-jacoco)
    * [Correr reporte de *coverage* de JaCoCo con Gradle](#correr-reporte-de-coverage-de-jacoco-con-gradle)
  * [Salir del editor VIM integrado en la terminal de AS (Windows)](#salir-del-editor-vim-integrado-en-la-terminal-de-as-windows)
<!-- TOC -->

---

## Transferir *settings* de AS a IntelliJ y viceversa

> 🔍 Referencia:  
> https://stackoverflow.com/a/25416163/12935412
> 

In order to transfer your existing settings from IntelliJ to Android Studio (or vice versa) select **File** ▶️ **Manage IDE Settings** ▶️ **Export Settings...** and select an appropriate path.  
Then in the other IDE you wish to transfer the settings to, select **File** ▶️ **Manage IDE Settings** ▶️ **Import Settings** and locate the previously exported settings file.

## Cambiar de ubicación las carpetas `.android`, `.gradle`, `Sdk`
https://moradiemails.medium.com/relocating-sdk-and-gradle-directories-for-better-usage-of-android-studio-9d4d45e6ef90

## Generar *Bundles* y APKs por terminal (con *Gradle*)
> 🔍 Ver también:  
> - [Tipos de módulos](Librería%20-%20Creación%20&%20Uso.md#conceptos-previos-tipos-de-módulos)
> - [*Android App Bundle* y *Performance*](../Android/UI/Performance.md#aab---android-app-bundle)

### APK “Directo” (sin *Android App Bundle*)
Esta forma se considera ***legacy*** y ha sido la forma de construir APKs desde los inicios de Android Studio y _Gradle_. Sigue existiendo por compatibilidad y porque es perfecto para el ciclo de desarrollo rápido (compilar -> instalar en el emulador).

Genera un único APK que puede ser universal o no, dependiendo de la configuración del `build.gradle.kts` (por ejemplo, si se usa `splits`).

**Comandos**:
1. *Debug*:
    - `./gradlew <MODULE>:assembleDebug`
2. *Release*:
    - `./gradlew <MODULE>:assembleRelease`

**Ubicación típica**:
- `<MODULE>/build/outputs/apk/<BUILD-TYPE>/<MODULE>-<BUILD-TYPE>.apk`
    - Ejemplo: con `./gradlew <MODULE>:assembleDebug`, se generaría `app/build/outputs/apk/debug/app-debug.apk`

### APK Universal desde un *Android App Bundle*
La tarea `package<BUILD-TYPE>UniversalApk` es muy útil porque crea **un único APK universal** a partir de un *Android App Bundle* (`.aab`). Este APK contiene todo el código y todos los recursos (para todas las arquitecturas de CPU, densidades de pantalla, idiomas, etc.) en un solo archivo.

**¿Para qué sirve?** :arrow_right: Es ideal para distribuciones internas, pruebas de QA o para subir a tiendas de aplicaciones de terceros que aún no soportan el formato `.aab`. No está optimizado para el tamaño de descarga como los APKs que genera _Google Play_, pero es muy conveniente por ser **un único archivo autoinstalable**.

**Comandos**:
1. *Debug*:
    - `./gradlew <MODULE>:packageDebugUniversalApk`
2. *Release*:
  - `./gradlew <MODULE>:packageReleaseUniversalApk`

**Ubicación típica**:
- `<MODULE>/build/outputs/apk_from_bundle/<BUILD-TYPE>/<MODULE>-<BUILD-TYPE>-universal.apk`
    - Ejemplo: con `./gradlew app:packageDebugUniversalApk`, se generaría `app/build/outputs/apk_from_bundle/debug/app-debug-universal.apk`

### *Android App Bundle* (`.aab`)
Cuando se ejecuta la tarea `bundle`, Gradle compila todo el código y agrupa todos los recursos (imágenes para todas las densidades, *strings* para todos los idiomas, código nativo para todas las arquitecturas de CPU, etc.) en un solo archivo: el `.aab`. Este archivo está diseñado para ser una representación completa y modular de la aplicación.

El flujo recomendado para *Google Play* es que primero se genere un `.aab` y luego, si es necesario, se extraigan APKs a partir de él.  
Desde un `.aab` se pueden obtener:
- **_Universal APK_** :arrow_right: Un único `*-universal.apk` (ver apartado anterior).
- **_Split APKs_** :arrow_right: Varios APKs (base + configs por [ABI](../Glosary%20&%20Core%20Concepts/Software%20in%20general.md#abi-application-binary-interface)/_density_/idiomas), típicamente para distribución eficiente.

**Comandos**:
1. *Debug*:
    - `./gradlew <MODULE>:bundleDebug`
2. *Release*:
    - `./gradlew <MODULE>:bundleRelease`

**Ubicación típica**:
- `<MODULE>/build/outputs/bundle/<BUILD-TYPE>/<MODULE>-<BUILD-TYPE>.aab`
    - Ejemplo: con `./gradlew app:bundleDebug`, se generaría `app/build/outputs/bundle/debug/app-debug.aab`

## Agregar recursos exportados de Figma a un proyecto
- `1x` -> `/android/mdpi/{RESOURCE-NAME}`
- `1.5x` -> `/android/hdpi/{RESOURCE-NAME}`
- `2x` -> `/android/xhdpi/{RESOURCE-NAME}`
- `3x` -> `/android/xxhdpi/{RESOURCE-NAME}`
- `4x` -> `/android/xxxhdpi/{RESOURCE-NAME}`

### Densidad de pantalla y conversión
En Android, las dimensiones se expresan en **dp (_density-independent pixels_)** para que la UI se vea igual en diferentes densidades de pantalla. Cuando un diseño de Figma está en píxeles reales, hay que convertirlos a dp según la densidad del dispositivo destino.  
Para eso, se utiliza la siguiente fórmula:

```text
dp = px / (dpi / 160)
```

Donde:
- **px** :arrow_right: Valor en píxeles (del diseño de Figma)
- **dpi** :arrow_right: Densidad del dispositivo en _dots per inch_ (se puede obtener con este [comando](Comandos%20útiles%20-%20ADB.md#obtener-la-densidad-en-dpi-dots-per-inch-de-un-dispositivo))
- **160** :arrow_right: Densidad base de Android (mdpi = 160 dpi, donde 1dp = 1px)

📌 **Ejemplo**:  

Si Figma muestra **32px** y el dispositivo tiene **320 dpi** (xhdpi):
```text
dp = 32 / (320 / 160)
dp = 32 / 2
dp = 16dp
```

## JaCoCo en AndroidStudio

### Agregar paquetes y/o clases al filtro de JaCoCo
The name is the JVM class name, e.g. `org/example/AppIdentifier` (no `.class` suffix).  
Classes and packages can be **excluded** using standard **`*`** and **`?`** wildcard syntax in the plugin configuration:    
    - `*` -> matches zero or more characters  
    - **`**`** -> matches zero or more directories  
    - **`?`** -> matches a single character

### Correr reporte de *coverage* de JaCoCo con Gradle
  1. En el `build.gradle(:{MODULE})`

```kotlin
    jacoco {
        toolVersion = "<VERSION>"
    }
```

  2. Correr reporte de *coverage*:

```bash
  ./gradlew jacocoFullReport --build-cache --no-daemon --parallel -Dorg.gradle.jvmargs="-Xms4g -Xmx4g -Xss16m -XX:+TieredCompilation -XX:+UseParallelGC -XX:+HeapDumpOnOutOfMemoryError -Dfile.encoding=UTF-8"
```

  3. Ruta del reporte de *coverage*:

```
    <MODULE>/build/reports/jacoco/jacocoTestDebugUnitTestReport/html/index.html
```

## Salir del editor VIM integrado en la terminal de AS (Windows)
- Para salir del editor y proceder a guardar y salir (`:wq`), hay que apretar `Ctrl + C`
