# AndroidStudio, APK's, JaCoCo, etc...

- [Transferir *settings* de AS a IntelliJ y viceversa](#transferir-settings-de-as-a-intellij-y-viceversa)
- [Cambiar de ubicación las carpetas `.android`, `.gradle`, `Sdk`](#cambiar-de-ubicación-las-carpetas-android-gradle-sdk)
- [Generar un APK](#generar-un-apk)
  - [Desde AndroidStudio](#desde-androidstudio)
  - [Con línea de comandos](#con-línea-de-comandos)
- [Agregar recursos exportados de Figma a un proyecto](#agregar-recursos-exportados-de-figma-a-un-proyecto)
- [JaCoCo en AndroidStudio](#jacoco-en-androidstudio)
  - [Agregar paquetes y/o clases al filtro de JaCoCo](#agregar-paquetes-yo-clases-al-filtro-de-jacoco)
  - [Correr reporte de *coverage* de JaCoCo con Gradle](#correr-reporte-de-coverage-de-jacoco-con-gradle)
- [Salir del editor VIM integrado en la terminal de AS (Windows)](#salir-del-editor-vim-integrado-en-la-terminal-de-as-windows)

---

## Transferir *settings* de AS a IntelliJ y viceversa
> Ref: https://stackoverflow.com/a/25416163/12935412

In order to transfer your existing settings from IntelliJ to Android Studio (or vice versa) select **File** ▶️ **Manage IDE Settings** ▶️ **Export Settings...** and select an appropriate path.  
Then in the other IDE you wish to transfer the settings to, select **File** ▶️ **Manage IDE Settings** ▶️ **Import Settings** and locate the previously exported settings file.

## Cambiar de ubicación las carpetas `.android`, `.gradle`, `Sdk`
https://moradiemails.medium.com/relocating-sdk-and-gradle-directories-for-better-usage-of-android-studio-9d4d45e6ef90

## Generar un APK

### Desde AndroidStudio
- Ve a "Build" ▶️ "Build Bundle(s) / APK(s)" ▶️ "Build APK(s)".
- Después de que el _build_ se complete, Android Studio mostrará una notificación con un enlace "locate". Hacer clic en ese enlace para abrir el directorio que contiene el APK (que estará en `*<módulo>/build/outputs/apk/<build_type>*`, donde `*<build_type>*` es "debug" o "release" dependiendo de la configuración del _build_).

### Con línea de comandos
- `./gradlew <MODULO>:packageDebugUniversalApk` o `./gradlew <MODULO>:packageReleaseUniversalApk`
- Se guarda en `*<módulo>/build/outputs/universal_apk/debug/app-debug-universal.apk*` o `*<módulo>/build/outputs/universal_apk/release/app-release-universal.apk`*

## Agregar recursos exportados de Figma a un proyecto
- `1x` -> `/android/mdpi/{RESOURCE-NAME}`
- `1.5x` -> `/android/hdpi/{RESOURCE-NAME}`
- `2x` -> `/android/xhdpi/{RESOURCE-NAME}`
- `3x` -> `/android/xxhdpi/{RESOURCE-NAME}`
- `4x` -> `/android/xxxhdpi/{RESOURCE-NAME}`

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
    <MODULO>/build/reports/jacoco/jacocoTestDebugUnitTestReport/html/index.html
```

## Salir del editor VIM integrado en la terminal de AS (Windows)
- Para salir del editor y proceder a guardar y salir (`:wq`), hay que apretar `Ctrl + C`
