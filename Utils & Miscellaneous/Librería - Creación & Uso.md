<h1>Proceso de Creación y Uso de una Librería</h1>

***Index***:
<!-- TOC -->
  * [Conceptos previos: Tipos de módulos](#conceptos-previos-tipos-de-módulos)
  * [TL;DR — Flujo rápido](#tldr--flujo-rápido)
    * [Crear la librería](#crear-la-librería)
    * [Distribuir la librería](#distribuir-la-librería)
    * [Consumir la librería](#consumir-la-librería)
    * [Regla clave](#regla-clave)
  * [1. Creación de una Librería](#1-creación-de-una-librería)
    * [1.1. Subir el proyecto de la librería a *GitHub*](#11-subir-el-proyecto-de-la-librería-a-github)
    * [1.2. ¿Es necesario un módulo de aplicación?](#12-es-necesario-un-módulo-de-aplicación)
    * [1.3. Crear una versión de la librería en *GitHub*](#13-crear-una-versión-de-la-librería-en-github)
      * [1.3.1. *Commitear* cambios y Crear un *tag* para la versión](#131-commitear-cambios-y-crear-un-tag-para-la-versión)
      * [1.3.2. Crear un *release* en *GitHub*](#132-crear-un-release-en-github)
    * [1.4. Distribución en *Maven* (si se utiliza *Maven*)](#14-distribución-en-maven-si-se-utiliza-maven)
      * [1.4.1. Configurar publicación en ``build.gradle.kts(:module)``](#141-configurar-publicación-en-buildgradlektsmodule)
      * [1.4.2. Publicar en un repositorio](#142-publicar-en-un-repositorio)
  * [2. Consumo de una librería desde otro proyecto](#2-consumo-de-una-librería-desde-otro-proyecto)
    * [2.1. Sin *JitPack*](#21-sin-jitpack)
      * [2.1.1. Clonar repo](#211-clonar-repo)
      * [2.1.2. Incluir módulos en `settings.gradle.kts`](#212-incluir-módulos-en-settingsgradlekts)
      * [2.1.3. Agregar dependencias locales en ``build.gradle.kts(:module)``](#213-agregar-dependencias-locales-en-buildgradlektsmodule)
    * [2.2. Con *JitPack*](#22-con-jitpack)
      * [2.2.1. Declarar repo de *JitPack*](#221-declarar-repo-de-jitpack)
      * [2.2.2. Declarar dependencias en ``libs.versions.toml``](#222-declarar-dependencias-en-libsversionstoml)
      * [2.2.3. Agregar dependencias en ``build.gradle.kts(:module)``](#223-agregar-dependencias-en-buildgradlektsmodule)
    * [2.3. Con *Maven*](#23-con-maven)
      * [2.3.1. Declarar repo de *Maven*](#231-declarar-repo-de-maven)
      * [2.3.2. Declarar dependencias en ``libs.versions.toml``](#232-declarar-dependencias-en-libsversionstoml)
      * [2.3.3. Agregar dependencias en ``build.gradle.kts(:module)``](#233-agregar-dependencias-en-buildgradlektsmodule)
    * [2.4. Comparativa: *JitPack* vs *Maven*](#24-comparativa-jitpack-vs-maven)
<!-- TOC -->

---

## Conceptos previos: Tipos de módulos
> ℹ️ **Nota:**  
> Un proyecto Android puede contener múltiples módulos de aplicación y librería en el mismo repositorio.

En el ecosistema de Android y Gradle, existen principalmente **dos tipos de módulos**:

1. **_Application Module_** (**Módulo de Aplicación**):
    - **Propósito**: Es una unidad de código que se puede compilar y ejecutar directamente como una aplicación independiente.
    - **Resultado**: Genera un archivo ``.apk`` (o ``.aab`` para la _Play Store_).
    - **_Plugin_ de Gradle**: Utiliza el _plugin_ ``com.android.application``. En el ``build.gradle.kts`` del módulo ``app``, esto se ve como ``plugins { id("com.android.application") ... }``.
    - **Ejemplo**: El módulo ``:app`` que se suele crear por defecto al crear un nuevo proyecto.

2. **_Library Module_** (**Módulo de Librería**):
    - **Propósito**: Contiene código y recursos reutilizables que no se pueden ejecutar por sí solos. Están diseñados para ser incluidos como una dependencia en otros módulos (ya sean de aplicación o de librería).
    - **Resultado**: Genera un archivo ``.aar`` (_Android Archive_) consumible.
    - **_Plugin_ de Gradle**: Utiliza el _plugin_ ``com.android.library``.
    - **Ejemplo**: Un módulo ``:core`` para la lógica de negocio, ``:common_ui`` para componentes de UI compartidos, etc.

## TL;DR — Flujo rápido
Los módulos de aplicación (`com.android.application`) **nunca se publican como dependencias**.

**Crear una librería Android y consumirla** implica separar claramente tres cosas: 
- **Versionado** :arrow_right: _Tags_ de Git y Control de versiones
- **Publicación** :arrow_right: _Release_ en _GitHub_ (metadata, _changelog_). No afecta el consumo técnico; es una **capa de comunicación para desarrolladores**
- **Distribución** :arrow_right: **Proceso técnico** que determina cómo se entrega el artefacto (``.aar``)

### Crear la librería
1. Crear uno o más **módulos de tipo `com.android.library`**
2. (**Opcional**) Usar un módulo `:sample` o `:demo` para pruebas
3. Hacer _commit_ del código y crear un **_tag_ de Git** (`v1.1.0`)
4. (**Opcional**) Crear un **_release_ en _GitHub_** para documentar cambios

### Distribuir la librería
- **_JitPack_**
    - Consume directamente un _tag_ o _commit_
    - Construye el `.aar` **bajo demanda**
    - No hay publicación manual ni artefactos persistentes

- **_Maven_**
    - Requiere configurar `maven-publish` en cada módulo
    - El `.aar` se **publica explícitamente** en un repositorio _Maven_
    - El repositorio _Maven_ es la **fuente de verdad**

### Consumir la librería
- Declarar el repositorio (`jitpack.io` o repositorio _Maven_)
- Declarar la dependencia:
    - **_JitPack_** → `com.github.user:repo:module:tag`
    - **_Maven_** → `groupId:artifactId:version`

### Regla clave
> Un módulo **solo es consumible** si fue:
> - Incluido como módulo local (código)
> - Publicado explícitamente (artefacto en _Maven_)
> - Construido bajo demanda (_JitPack_)

## 1. Creación de una Librería
### 1.1. Subir el proyecto de la librería a *GitHub*
Inicializar el repositorio Git en la **raíz del proyecto** y subirlo a _GitHub_.

```bash
git init
git add .
git commit -m "Initial commit of my Android library"
git remote add origin https://github.com/user/repo-name.git
git push -u origin main
```

### 1.2. ¿Es necesario un módulo de aplicación?
- **Preferible para pruebas** :arrow_right: Es muy útil para desarrollar y validar la librería (UI, flujos y _debugging_).
- **Publicación** :arrow_right: Está bien que viva en el repositorio, ya que no forma parte del API distribuible (**no se publica como dependencia**). Suele nombrarse como `:sample` o `:demo`.

### 1.3. Crear una versión de la librería en *GitHub*
> ⚠️ **Importante**:  
> La versión que consumen herramientas como **_JitPack_** se define mediante **_tags_ de Git**.  
> ``namespace`` define el paquete generado para ``R`` y ``BuildConfig``, y no afecta a las coordenadas _Maven_ o _JitPack_.

📌 Ejemplo de `build.gradle.kts` de librería:

```kotlin
plugins {
    id("com.android.library")
    kotlin("android")
}

android {
    namespace = "com.example.mymodule"

    defaultConfig {
        minSdk = 21
        consumerProguardFiles("consumer-rules.pro")
    }
}
```

#### 1.3.1. *Commitear* cambios y Crear un *tag* para la versión
```bash
git add .
git commit -m "Prepare release 1.1.0"
git push origin main
git tag -a v1.1.0 -m "Release 1.1.0"
git push origin v1.1.0
```

#### 1.3.2. Crear un *release* en *GitHub*
Se asocia el _tag_, se agrega metadata (texto, _changelog_) y se marca como "versión estable".  
Este paso no publica artefactos; la publicación técnica ocurre cuando una herramienta externa (por ejemplo, _JitPack_) consume el _tag_.

- Ir a la pestaña **_Releases_**
- Crear un nuevo _release_ usando el _tag_ `v1.1.0`
- Agregar descripción de cambios
- Publicar

### 1.4. Distribución en *Maven* (si se utiliza *Maven*)
#### 1.4.1. Configurar publicación en ``build.gradle.kts(:module)``
> ⚠️ Importante:  
> Se debe realizar esta configuración para cada módulo que se quiera publicar, ya que _Maven_ considera **cada módulo publicable como un artefacto independiente**.

La publicación Maven se configura **en el módulo de librería** (no en el proyecto raíz). En este paso se define **a qué repositorio _Maven_** se sube el artefacto.

> ℹ️ **Nota:**  
> El repo _Maven_ no lo crea Gradle, lo provee una plataforma (_GitHub Packages_, _Sonatype_, _Artifactory_).  
> En _GitHub Packages_, el repositorio _Maven_ se crea automáticamente cuando se [publica el ``.aar``](#142-publicar-en-un-repositorio) cuya URL es del estilo ``https://maven.pkg.github.com/USER/REPO``.  
> A su vez, se puede crear un **_Personal Access Token_** (**PAT**, que actúa como credencial de autenticación para publicar y consumir paquetes) con permisos ``read:packages``, ``write:packages`` y ``repo`` (si el repo es privado).

**Sobre ``publications {}``**:
- `groupId`, `artifactId` y `version` definen las **coordenadas Maven** que identifican de forma única al artefacto publicado.
- El artefacto `.aar` que se publica corresponde al **_component_ `release`** generado por el _plugin_ de Android.
    - `afterEvaluate` se usa porque `components["release"]` solo **existe una vez evaluado el _plugin_ de Android**.

```kotlin
import java.util.Properties

// Asumiendo que se use un archivo ``secrets.properties``
val secrets = Properties().apply {
    load(file("secrets.properties").inputStream())
}

plugins {
    id("com.android.library")
    kotlin("android")
    id("maven-publish")
}

android {
    namespace = "com.example.mylibrary"
}

afterEvaluate {
    publishing {
        // Qué se publica
        publications {
            create<MavenPublication>("release") {
                from(components["release"])
                groupId = "com.example"
                artifactId = "mylibrary"
                version = "1.1.0"
            }
        }
        
        // Dónde se publica
        repositories {
            maven {
                name = "privateRepo"
                url = uri("https://maven.pkg.github.com/USER/REPO")

                credentials {
                    // En GitHub Packages:
                    // username = Usuario de GitHub
                    // password = Personal Access Token (PAT)
                    username = secrets["mavenUser"] as String
                    password = secrets["mavenPassword"] as String

                    // Alternativa si no se usa ``secrets.properties``:
                    // username = findProperty("mavenUser") as String
                    // password = findProperty("mavenPassword") as String
                }
            }
        }
    }
}
```

#### 1.4.2. Publicar en un repositorio
La **publicación técnica real** (subir el `.aar` al repositorio _Maven_) se ejecuta explícitamente con:

```bash
./gradlew publish
```

## 2. Consumo de una librería desde otro proyecto
### 2.1. Sin *JitPack*
Se puede clonar el repositorio y usar los módulos de forma local. Técnicamente, no hay “publicación”, es inclusión directa de código.

**Cuándo se usa**:
- Desarrollo activo
- _Debug_ conjunto
- Monorepos
- _Forks_

**Características**:
- ❌ No hay versionado real
- ❌ No hay artefactos
- ✔️ Cambios inmediatos
- ✔️ Ideal para trabajar la lib

#### 2.1.1. Clonar repo
```bash
git clone https://github.com/user/repo-name.git
```

#### 2.1.2. Incluir módulos en `settings.gradle.kts`

```kotlin
include(":module_A", ":module_B")

project(":module_A").projectDir = file("path/to/repo-name/module_A")
project(":module_B").projectDir = file("path/to/repo-name/module_B")
```

#### 2.1.3. Agregar dependencias locales en ``build.gradle.kts(:module)``

```kotlin
dependencies {
    implementation(project(":module_A"))
    implementation(project(":module_B"))
}
```

### 2.2. Con *JitPack*
> ℹ️ **Nota:**  
> _JitPack_ es un **_build service_**, no un repositorio _Maven_ clásico.  
> A su vez, publica ***artefactos efímeros***: no garantiza que el mismo `.aar` exista para siempre si el repo cambia o deja de compilar.

**Qué hace _JitPack_ realmente**:
1. Detecta un _tag_ o _commit_
2. Clona el repositorio
3. Ejecuta _Gradle_
4. Genera el `.aar`
5. Sirve el artefacto **bajo demanda**

**Cuándo se usa**:
- Librerías internas 
- Prototipos 
- Open source pequeños 
- Evitar burocracia

**Características**:
- ✔️ Versionado por _tags_ de Git 
- ✔️ Cero infraestructura 
- ❌ No apto para entornos regulados 
- ❌ Dependencia externa implícita

#### 2.2.1. Declarar repo de *JitPack*
Se agrega al archivo `settings.gradle.kts`:

📌 Ejemplo:

```kotlin
pluginManagement {
    repositories {
        gradlePluginPortal()
        google()
        mavenCentral()
    }
}

dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)

    repositories {
        google()
        mavenCentral()
        maven(url = "https://jitpack.io")
    }
}

rootProject.name = "MyProject"

include(":app")
include(":core")
include(":feature_login")
```

#### 2.2.2. Declarar dependencias en ``libs.versions.toml``
Al usar **_JitPack_**, la versión corresponde al **_tag_ de Git** (coincidir con el nombre real del _tag_ con la `v` inicial evita confusión).

```toml
[versions]
miLibrary = "v1.1.0"

[libraries]
moduleA = { module = "com.github.user:repo-name:module_A", version.ref = "miLibrary" }
moduleB = { module = "com.github.user:repo-name:module_B", version.ref = "miLibrary" }
```

#### 2.2.3. Agregar dependencias en ``build.gradle.kts(:module)``

```kotlin
dependencies {
    implementation(libs.moduleA)
    implementation(libs.moduleB)
}
```

### 2.3. Con *Maven*
En este enfoque, la librería se **publica explícitamente como un artefacto `.aar`** en un **repositorio _Maven_** (privado o público). Aquí, la **fuente de verdad no es Git**, sino el repositorio _Maven_ que almacena los artefactos generados.

**Qué implica usar publicación _Maven_**:
- El `.aar` se construye de forma explícita
- Se sube manualmente o vía CI
- Queda **almacenado de forma persistente**
- El consumo **no depende del repositorio Git**

**Cuándo se usa**:
- Librerías internas estables
- Proyectos empresariales
- SDKs reutilizables
- Casos donde se requiere control estricto de versiones

**Características**:
- ✔️ Artefactos versionados y persistentes
- ✔️ Control total del proceso de publicación
- ✔️ Independiente de _GitHub_ o _JitPack_
- ❌ Requiere configuración inicial (ver [Publicación en *Maven*](#14-distribución-en-maven-si-se-utiliza-maven))

#### 2.3.1. Declarar repo de *Maven*
Se agrega al archivo `settings.gradle.kts`:

📌 Ejemplo:

```kotlin
pluginManagement {
    repositories {
        gradlePluginPortal()
        google()
        mavenCentral()
    }
}

dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)

    repositories {
        google()
        mavenCentral()
        maven(url = "https://maven.example.com/releases")
    }
}

rootProject.name = "MyProject"

include(":app")
include(":core")
include(":feature_login")
```

#### 2.3.2. Declarar dependencias en ``libs.versions.toml``
En _Maven_, cada dependencia se identifica mediante **una única coordenada**:

```text
groupId:artifactId:version
```

A diferencia de *JitPack*, **no existen submódulos en la coordenada**. Solo se puede consumir **aquello que fue publicado explícitamente como artefacto _Maven_**.

> ℹ️ **Nota**:  
> Si se desean consumir múltiples módulos (`module_A`, `module_B`) usando _Maven_, **cada módulo debe publicarse como un artefacto independiente**, con su propio `artifactId` (ver [acá](#141-configurar-publicación-en-buildgradlektsmodule)).
>
> Ejemplo:
> - `com.example:module-a:1.1.0`
> - `com.example:module-b:1.1.0`

En este ejemplo, se asume que:
- Se publicó **un único módulo de librería**
- El `artifactId` publicado es `mylibrary`

```toml
[versions]
miLibrary = "1.1.0"

[libraries]
mylibrary = { module = "com.example:mylibrary", version.ref = "miLibrary" }
```

#### 2.3.3. Agregar dependencias en ``build.gradle.kts(:module)``

```kotlin
dependencies {
    implementation(libs.mylibrary)
}
```

### 2.4. Comparativa: *JitPack* vs *Maven*

| Aspecto                                                    | JitPack                                                                      | Maven tradicional                                                                       |
|------------------------------------------------------------|------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------|
| Fuente de verdad                                           | Repositorio Git                                                              | Repositorio Maven                                                                       |
| Generación del `.aar`                                      | Bajo demanda                                                                 | Explícita                                                                               |
| Publicación manual                                         | No                                                                           | Sí                                                                                      |
| Persistencia del artefacto                                 | No garantizada (Puede borrar _cache_, recompilar o fallar si el repo cambia) | Permanente (El ``.aar`` ya existe, se descarga tal cual y no depende del código fuente) |
| Versionado                                                 | Tags de Git                                                                  | Coordenadas Maven                                                                       |
| Infraestructura (Servidores, Credenciales, Storage, CI/CD) | Ninguna                                                                      | Requerida                                                                               |
| Control del proceso                                        | Bajo                                                                         | Alto                                                                                    |

