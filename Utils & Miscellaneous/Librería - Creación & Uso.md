<h1>Proceso de Creación y Uso de una Librería</h1>

***Index***:
<!-- TOC -->
  * [1. Creación de una Librería](#1-creación-de-una-librería)
    * [1.1. Subir un módulo 'Android Library' a GitHub](#11-subir-un-módulo-android-library-a-github)
    * [1.2. ¿Es necesairo un módulo 'Android App'?](#12-es-necesairo-un-módulo-android-app)
    * [1.3. Crear una versión de la librería en GitHub y en el propio proyecto](#13-crear-una-versión-de-la-librería-en-github-y-en-el-propio-proyecto)
  * [2. Consumo de una librería desde otro proyecto](#2-consumo-de-una-librería-desde-otro-proyecto)
    * [2.1. Agregar Dependencias Usando `libs.versions.toml` con JitPack](#21-agregar-dependencias-usando-libsversionstoml-con-jitpack)
    * [2.2. Consumo de Módulos sin JitPack](#22-consumo-de-módulos-sin-jitpack)
<!-- TOC -->

---

## 1. Creación de una Librería

### 1.1. Subir un módulo 'Android Library' a GitHub

1. Navegar a la carpeta del módulo:

    ```bash
    cd ruta/al/modulo
    ```

2. Inicializar el repositorio de Git:

    ```bash
    git init
    ```

3. Agregar archivos y commitear:

    ```bash
    git add .
    git commit -m "Initial commit of my Android library module"
    ```

4. Conectar al repositorio remoto:

    ```bash
    git remote add origin https://github.com/usuario/nombre-del-repo.git
    ```

5. Pushear al repositorio de GitHub:

    ```bash
    git push -u origin main
    ```


### 1.2. ¿Es necesairo un módulo 'Android App'?

- **Preferible para pruebas**: Tener un módulo de aplicación es útil para desarrollar y probar las librerías creadas.
- **No subir el módulo 'Android App' a GitHub**: Generalmente no se sube como parte del repositorio de la librería; se suelen compartir solo los módulos que se desea que otros usen.

### 1.3. Crear una versión de la librería en GitHub y en el propio proyecto

1. **Modificar la versión en el código fuente**:

   En el archivo **`build.gradle.kts`** del módulo que se desea actualizar, buscar (o agregar) el campo **`version`** y actualizar la versión:

    ```kotlin
    plugins {
        (...)
    }
    
    android {
    		(...)
    		
        defaultConfig {
            // ID único del módulo
            applicationId = "com.example.mimodulo"
    
            version = "1.1.0" // Cambiar acá la versión para este módulo
            (...)
        }
    }
    
    dependencies {
        (...)
    }
    ```

2. **Commit de los cambios**:

   Commit del cambio de versión:

    ```bash
    git add .
    git commit -m "Actualización de versión a 1.1.0"
    ```

3. **Crear un tag para la nueva versión**:

   Navegar a la carpeta del módulo y crear un tag con una etiqueta anotada (`-a`) para representar la nueva versión:

    ```bash
    git tag -a v1.1.0 -m "Actualización a la versión 1.1.0"
    ```

4. **Hacer push del tag y del commit**:

   Push del tag al repositorio remoto:

    ```bash
    git push origin v1.1.0
    ```

   Push de los cambios del commit a la rama principal:

    ```bash
    git push origin main
    ```

5. **Crear un release en GitHub**:
    - Ir al repositorio en GitHub.
    - Entrar a la pestaña **Releases** en la parte superior del repositorio.
    - Clic en **Draft a new release**.
    - En el campo **Tag version**, seleccionar el tag que se creó (por ejemplo, **`v1.1.0`**).
    - Completar el título y la descripción del release, proporcionando información sobre los cambios o características nuevas.
    - Finalmente, clic en **Publish release**.

## 2. Consumo de una librería desde otro proyecto

### 2.1. Agregar Dependencias Usando `libs.versions.toml` con JitPack

1. **Agregar la dependencia en `libs.versions.toml`**:

   Si se utiliza **JitPack** para consumir librerías de un repositorio de GitHub, se deben definir las versiones y las dependencias de los módulos en el archivo **`libs.versions.toml`**:

    ```toml
    [versions]
    miLibreria = "1.1.0"
    
    [libraries]
    module_A = { module = "com.github.usuario:nombre-del-repo:module_A", version.ref = "miLibreria" }
    module_B = { module = "com.github.usuario:nombre-del-repo:module_B", version.ref = "miLibreria" }
    ```

2. **Agregar el Repositorio JitPack al `build.gradle.kts` del proyecto**:

   El archivo de configuración de Gradle debe incluir el repositorio de JitPack. Esto permite que Gradle busque y descargue las dependencias desde JitPack:

    ```kotlin
    pluginManagement {
        repositories {
            google()
            mavenCentral()
            maven { 
    	          url = uri("https://jitpack.io")
    	          // Esto, hay que ver si es necesario o no
    	          content {
    		            includeGroupByRegex = "com\\.github\\..*"
    		        }
    	      }
        }
    }
    ```

3. **Agregar las Dependencias en `build.gradle.kts`**:

   Luego, en el archivo **`build.gradle.kts`** del módulo donde se desea consumir las librerías, se agregan las dependencias como sigue:

    ```kotlin
    dependencies {
        implementation(libs.module_A) // Referencia a module_A en libs.versions.toml
        implementation(libs.module_B) // Referencia a module_B en libs.versions.toml
    }
    ```


### 2.2. Consumo de Módulos sin JitPack

Si se decide **no usar JitPack**, se puede clonar el repositorio localmente y referenciar los módulos desde allí.

1. **Clonar el Repositorio Localmente**:

   Primero, clonar el repositorio que contiene los módulos en la máquina local:

    ```bash
    git clone https://github.com/usuario/nombre-del-repo.git
    ```

2. **Agregar Módulos en el Proyecto de Android**:

   Abrir el archivo **`settings.gradle`** (o **`settings.gradle.kts`**) en el proyecto de Android e incluir los módulos que se acaban de clonar:

    ```groovy
    include ':module_A', ':module_B'
    project(':module_A').projectDir = file('ruta/a/nombre-del-repo/module_A')
    project(':module_B').projectDir = file('ruta/a/nombre-del-repo/module_B')
    ```

3. **Agregar las Dependencias Locales**:

   Luego, en el **`build.gradle.kts`** de la aplicación, agregar las dependencias de los módulos locales:

    ```kotlin
    dependencies {
        implementation(project(":module_A")) // Referencia al módulo A
        implementation(project(":module_B")) // Referencia al módulo B
    }
    ```
