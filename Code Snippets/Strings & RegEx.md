<h1><i>Strings & RegEx</i></h1>

***Index***:
<!-- TOC -->
  * [Colores ARGB](#colores-argb)
  * [*Regex* para *matchear* cualquier *endpoint* que no sea el especificado](#regex-para-matchear-cualquier-endpoint-que-no-sea-el-especificado)
<!-- TOC -->

---

## Colores ARGB
En la mayoría de las situaciones al tratar con colores en formato ARGB (*Alpha, Red, Green, Blue*) en Android, si no proporcionas un componente alfa explícito en el recurso de color, el comportamiento común es que Android asume que el componente alfa es `FF` (completamente opaco) para que el color se represente plenamente.  
Al hacer `and 0xffffffff`, se asegura que el color final solo retiene los *bits* de color RGB (rojo, verde, azul) y establece el canal alfa en `FF` o `255` (completamente opaco), en caso de que el color de entrada tuviera un componente alfa diferente. Esto es útil si necesitas asegurarte de que el color resultante sea completamente opaco.

```kotlin
    String.format(
        ColorResourceProvider.HEXADECIMAL_COLOR_FORMAT,
        ContextCompat.getColor(
            this,
            ColorResourceProvider.SOME_SCREEN_BACKGROUND_COLOR_CONTAINER
        ) and 0xffffffff.toInt()
    )
```

## *Regex* para *matchear* cualquier *endpoint* que no sea el especificado
Matchear todo excepto `https://mobile.example.com/some-path/something`

```
    // RegEx
    
    ^(?!https:\/\/mobile\.example\.com\/some-path\/something$).*$
    
    // Matches 2 and 3
    
    1. https://mobile.example.com/some-path/something
    
    2. https://mobile.example.com/some-path/somethingggg
    
    3. https://mobile.example.com/some-path/other
```
