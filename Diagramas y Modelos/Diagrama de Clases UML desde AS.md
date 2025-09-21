# Generar un diagrma de Clases desde AS con PlantUML

## Instalación y configuración de [*PlantUML*](https://plantuml.com/es/class-diagram)
- Instalar ***PlantUML*** y ***Graphviz*** (ver [acá](https://medium.com/@shivam.gosavi340_58315/productivity-hack-visual-documentation-using-plantuml-2f9562890a42))
  - *Plugin* → `PlantUML integration`
  - [*Mac*] → `brew install graphviz`
- Configuración de [***MetaView***](https://github.com/xcporter/MetaView):
  - En el `build.gradle.kts(:module)`. Tomar la versión preferiblemente de https://plugins.gradle.org/plugin/com.xcporter.metaview

```kotlin
  plugins {
    // (...)
    id("com.xcporter.metaview") version "<VERSION>"
  }
  
  android {
    // (...)
  
    generateUml {
      classTree {
        outputFile = "<NOMBRE DEL ARCHIVO>.md"
        style = listOf("skinparam BackgroundColor LightBlue")
        target =
          file(projectDir.path + "/src/main/<RUTA DEL PAQUETE DE LOS ARCHIVOS A GENERAR EL DIAGRAMA>")
      }
    }
  }
```

## Generar un diagrama
- Generar diagrama (se guarda en `<MODULO>/build/docs/`):

```bash
  ./gradlew generateUmlDiagrams
```

- Adicionalmente, se puede agregar un tamaño en DPI's para guardar el diagrama como imagen:

```plantuml
  @startuml
  skinparam dpi 300
  (...)
  @enduml
```

## *Troubleshooting*
- Problemas con versiones de Java al correr el comando para generar un diagrama → Validar si se está usando la última versión del *plugin*

```bash
com/xcporter/KotlinLexer has been compiled by a more recent version of the Java Runtime (class file version 58.0), this version of the Java Runtime only recognizes class file versions up to 55.0
```

- No se genera el diagrama por algún error en la ruta → Agregar la ruta correcta del archivo `dot` ejecutable en `Open Settings` > `Graphviz dot executable`

<img src="../images/graphviz-dot-executable-path.png" width="1264" alt="">
