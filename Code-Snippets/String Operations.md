# Operaciones con Strings

En la mayoría de las situaciones al tratar con colores en formato ARGB (*Alpha, Red, Green, Blue*) en Android, si no se proporciona un componente alfa explícito en el recurso de color, el comportamiento común es que Android asume que el componente alfa es `FF` (completamente opaco) para que el color se represente plenamente.  
Al hacer `and 0xffffffff`, se asegura que el color final solo retiene los *bits* de color RGB (rojo, verde, azul) y establece el canal alfa en `FF` o `255` (completamente opaco), en caso de que el color de entrada tuviera un componente alfa diferente. Esto es útil si se necesita asegurar de que el color resultante sea completamente opaco.

```kotlin
String.format(
    ColorResourceProvider.HEXADECIMAL_COLOR_FORMAT,
    ContextCompat.getColor(
        this,
        ColorResourceProvider.RYC_BACKGROUND_COLOR_CONTAINER
    ) and 0xffffffff.toInt()
)
```
