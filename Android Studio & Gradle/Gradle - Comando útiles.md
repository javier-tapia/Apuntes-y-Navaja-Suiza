# Comandos útiles - Gradle

## Árbol de dependencias
Obtener el árbol de dependencias que se están resolviendo. Útil para los errores de tipo **`NoSuchMethodError`**.  
Opcionalmente, se le puede agregar `--refresh-dependencies` para refrescar las dependencias.

````bash
    ./gradlew <MÓDULO>:dependencies --refresh-dependencies
````

## Dependencia específica
Obtener información de una dependencia específica (en el ejemplo, `something`).  
También se puede obtener información de un módulo (por ejemplo, `com.example.something:example-module`)

````bash
  ./gradlew <MÓDULO>:dependencyInsight  --configuration releaseRuntimeClasspath --dependency com.example.something
````
