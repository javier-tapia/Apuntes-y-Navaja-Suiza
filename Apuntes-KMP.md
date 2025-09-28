<h1>Kotlin Multiplatform (KMP) y Compose Multiplatform (CMP)</h1>

***Index***:
<!-- TOC -->
  * [0. Introducción a Kotlin Multiplatform](#0-introducción-a-kotlin-multiplatform)
    * [0.1. Qué es KMP](#01-qué-es-kmp)
    * [0.2. Ventajas de KMP](#02-ventajas-de-kmp)
    * [0.3. Casos de uso comunes](#03-casos-de-uso-comunes)
  * [1. Configuración de un proyecto KMP](#1-configuración-de-un-proyecto-kmp)
    * [1.1. Estructura de módulos](#11-estructura-de-módulos)
      * [1.1.1. Módulo común (*shared*)](#111-módulo-común-shared)
      * [1.1.2. Módulos específicos de plataforma](#112-módulos-específicos-de-plataforma)
    * [1.2. Gradle y Kotlin DSL](#12-gradle-y-kotlin-dsl)
      * [1.2.1. Configuración de plugins](#121-configuración-de-plugins)
      * [1.2.2. Dependencias multiplataforma](#122-dependencias-multiplataforma)
    * [1.3. Targets soportados](#13-targets-soportados)
      * [1.3.1. Android](#131-android)
      * [1.3.2. iOS](#132-ios)
      * [1.3.3. JavaScript](#133-javascript)
      * [1.3.4. Desktop (JVM/Native)](#134-desktop-jvmnative)
  * [2. Módulo compartido (*shared module*)](#2-módulo-compartido-shared-module)
    * [2.1. Declaración de código común](#21-declaración-de-código-común)
      * [2.1.1. Funciones comunes](#211-funciones-comunes)
      * [2.1.2. Clases y data classes comunes](#212-clases-y-data-classes-comunes)
    * [2.2. Expect / Actual](#22-expect--actual)
      * [2.2.1. Ejemplo de función expect/actual](#221-ejemplo-de-función-expectactual)
      * [2.2.2. Ejemplo de clase expect/actual](#222-ejemplo-de-clase-expectactual)
    * [2.3. Librerías multiplataforma](#23-librerías-multiplataforma)
      * [2.3.1. Ktor](#231-ktor)
      * [2.3.2. SQLDelight](#232-sqldelight)
      * [2.3.3. Kotlinx Serialization](#233-kotlinx-serialization)
    * [2.4. Manejo de dependencias](#24-manejo-de-dependencias)
      * [2.4.1. Dependencias comunes](#241-dependencias-comunes)
      * [2.4.2. Dependencias específicas](#242-dependencias-específicas)
  * [3. Interoperabilidad con plataformas](#3-interoperabilidad-con-plataformas)
    * [3.1. Android](#31-android)
      * [3.1.1. Llamadas a código compartido](#311-llamadas-a-código-compartido)
    * [3.2. iOS](#32-ios)
      * [3.2.1. Uso de framework compartido](#321-uso-de-framework-compartido)
      * [3.2.2. Swift interoperability](#322-swift-interoperability)
    * [3.3. JavaScript / Web](#33-javascript--web)
      * [3.3.1. Exportar funciones y clases](#331-exportar-funciones-y-clases)
  * [4. Compose Multiplatform (CMP)](#4-compose-multiplatform-cmp)
    * [4.1. Qué es CMP](#41-qué-es-cmp)
    * [4.2. Configuración del proyecto CMP](#42-configuración-del-proyecto-cmp)
      * [4.2.1. Plugins y dependencias](#421-plugins-y-dependencias)
      * [4.2.2. Targets soportados](#422-targets-soportados)
    * [4.3. Componentes básicos](#43-componentes-básicos)
      * [4.3.1. Text, Button, Image](#431-text-button-image)
      * [4.3.2. Column, Row, Box](#432-column-row-box)
      * [4.3.3. Scrollable y LazyLists](#433-scrollable-y-lazylists)
    * [4.4. State y eventos](#44-state-y-eventos)
      * [4.4.1. State hoisting](#441-state-hoisting)
      * [4.4.2. MutableState y remember](#442-mutablestate-y-remember)
    * [4.5. Navegación multiplataforma](#45-navegación-multiplataforma)
    * [4.6. Temas y estilos](#46-temas-y-estilos)
      * [4.6.1. Colores y tipografía](#461-colores-y-tipografía)
      * [4.6.2. Light/Dark mode](#462-lightdark-mode)
  * [5. Testing en KMP](#5-testing-en-kmp)
    * [5.1. Tests en código común](#51-tests-en-código-común)
      * [5.1.1. Unit tests con Kotlin Test](#511-unit-tests-con-kotlin-test)
    * [5.2. Tests específicos de plataforma](#52-tests-específicos-de-plataforma)
      * [5.2.1. Android tests](#521-android-tests)
      * [5.2.2. iOS tests](#522-ios-tests)
  * [6. Buenas prácticas y consejos](#6-buenas-prácticas-y-consejos)
    * [6.1. Mantener separado el código común y específico](#61-mantener-separado-el-código-común-y-específico)
    * [6.2. Evitar dependencias no multiplataforma](#62-evitar-dependencias-no-multiplataforma)
    * [6.3. Modularización y escalabilidad](#63-modularización-y-escalabilidad)
  * [7. Referencias y recursos](#7-referencias-y-recursos)
    * [7.1. Documentación oficial KMP](#71-documentación-oficial-kmp)
    * [7.2. Documentación Compose Multiplatform](#72-documentación-compose-multiplatform)
    * [7.3. Tutoriales y ejemplos](#73-tutoriales-y-ejemplos)
<!-- TOC -->

---

## 0. Introducción a Kotlin Multiplatform
TODO...

### 0.1. Qué es KMP
TODO...

### 0.2. Ventajas de KMP
TODO...

### 0.3. Casos de uso comunes
TODO...

## 1. Configuración de un proyecto KMP
TODO...

### 1.1. Estructura de módulos
TODO...

#### 1.1.1. Módulo común (*shared*)
TODO...

#### 1.1.2. Módulos específicos de plataforma
TODO...

### 1.2. Gradle y Kotlin DSL
TODO...

#### 1.2.1. Configuración de plugins
TODO...

#### 1.2.2. Dependencias multiplataforma
TODO...

### 1.3. Targets soportados
TODO...

#### 1.3.1. Android
TODO...

#### 1.3.2. iOS
TODO...

#### 1.3.3. JavaScript
TODO...

#### 1.3.4. Desktop (JVM/Native)
TODO...

## 2. Módulo compartido (*shared module*)
TODO...

### 2.1. Declaración de código común
TODO...

#### 2.1.1. Funciones comunes
TODO...

#### 2.1.2. Clases y data classes comunes
TODO...

### 2.2. Expect / Actual
TODO...

#### 2.2.1. Ejemplo de función expect/actual
TODO...

#### 2.2.2. Ejemplo de clase expect/actual
TODO...

### 2.3. Librerías multiplataforma
TODO...

#### 2.3.1. Ktor
TODO...

#### 2.3.2. SQLDelight
TODO...

#### 2.3.3. Kotlinx Serialization
TODO...

### 2.4. Manejo de dependencias
TODO...

#### 2.4.1. Dependencias comunes
TODO...

#### 2.4.2. Dependencias específicas
TODO...

## 3. Interoperabilidad con plataformas
TODO...

### 3.1. Android
TODO...

#### 3.1.1. Llamadas a código compartido
TODO...

### 3.2. iOS
TODO...

#### 3.2.1. Uso de framework compartido
TODO...

#### 3.2.2. Swift interoperability
TODO...

### 3.3. JavaScript / Web
TODO...

#### 3.3.1. Exportar funciones y clases
TODO...

## 4. Compose Multiplatform (CMP)
TODO...

### 4.1. Qué es CMP
TODO...

### 4.2. Configuración del proyecto CMP
TODO...

#### 4.2.1. Plugins y dependencias
TODO...

#### 4.2.2. Targets soportados
TODO...

### 4.3. Componentes básicos
TODO...

#### 4.3.1. Text, Button, Image
TODO...

#### 4.3.2. Column, Row, Box
TODO...

#### 4.3.3. Scrollable y LazyLists
TODO...

### 4.4. State y eventos
TODO...

#### 4.4.1. State hoisting
TODO...

#### 4.4.2. MutableState y remember
TODO...

### 4.5. Navegación multiplataforma
TODO...

### 4.6. Temas y estilos
TODO...

#### 4.6.1. Colores y tipografía
TODO...

#### 4.6.2. Light/Dark mode
TODO...

## 5. Testing en KMP
TODO...

### 5.1. Tests en código común
TODO...

#### 5.1.1. Unit tests con Kotlin Test
TODO...

### 5.2. Tests específicos de plataforma
TODO...

#### 5.2.1. Android tests
TODO...

#### 5.2.2. iOS tests
TODO...

## 6. Buenas prácticas y consejos
TODO...

### 6.1. Mantener separado el código común y específico
TODO...

### 6.2. Evitar dependencias no multiplataforma
TODO...

### 6.3. Modularización y escalabilidad
TODO...

## 7. Referencias y recursos
TODO...

### 7.1. Documentación oficial KMP
TODO...

### 7.2. Documentación Compose Multiplatform
TODO...

### 7.3. Tutoriales y ejemplos
TODO...
