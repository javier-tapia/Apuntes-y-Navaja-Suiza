<h1><i>Strings & RegEx</i></h1>

***Index***:
<!-- TOC -->
  * [Colores ARGB](#colores-argb)
  * [*Regex* para *matchear* cualquier *endpoint* que no sea el especificado](#regex-para-matchear-cualquier-endpoint-que-no-sea-el-especificado)
  * [String extensions](#string-extensions)
    * [``String?.nullIfNullOrEmpty``](#stringnullifnullorempty)
    * [``String?.orDefaultIfNullOrEmpty``](#stringordefaultifnullorempty)
    * [``String?.decodeUrl``](#stringdecodeurl)
    * [``String?.encodeUrl``](#stringencodeurl)
    * [``String.getCurrentTimestamp``](#stringgetcurrenttimestamp)
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

## String extensions

### ``String?.nullIfNullOrEmpty``

```kotlin
/**
 * Returns null whether this String is null or empty
 *
 * @receiver A nullable String
 * @return Same String if neither null nor empty; null otherwise
 */
fun String?.nullIfNullOrEmpty(): String? {
    if (this.isNullOrEmpty()) {
        return null
    }
    return this
}
```

### ``String?.orDefaultIfNullOrEmpty``

```kotlin
/**
 * Returns the string itself if it is not null and not empty, otherwise returns the default value.
 *
 * @receiver The nullable string that might be null or empty.
 * @param default The string to return if the receiver is null or empty.
 * @return The original string if not null and not empty, otherwise the default value.
 *
 * @sample
 * val result = nullString.orDefaultIfNullOrEmpty("Default") // returns "Default"
 * val result = "".orDefaultIfNullOrEmpty("Default") // returns "Default"
 * val result = "Value".orDefaultIfNullOrEmpty("Default") // returns "Value"
 */
fun String?.orDefaultIfNullOrEmpty(default: String): String = if (this.isNullOrEmpty()) default else this
```

### ``String?.decodeUrl``

```kotlin
/** Returns a decoded string if possible
*
* @receiver A nullable String
* @return Decoded string url if is possible decode it otherwise return null
*/
fun String?.decodeUrl(): String? {
    this ?: return null
    val tempString = URLDecoder.decode(this, "UTF-8")
    return tempString.replace("__PERCENT__", "%")
}
```

### ``String?.encodeUrl``

```kotlin
/** Returns an encoded string if possible
*
* @receiver A nullable String
* @return Encoded string url if is possible encode it otherwise return an empty String
*/
fun String?.encodeUrl(): String {
    this ?: return ""
    val tempString = this.replace("%", "__PERCENT__")
    return URLEncoder.encode(tempString, "UTF-8")
}
```

### ``String.getCurrentTimestamp``

```kotlin
fun String.getCurrentTimestamp(): String =
    SimpleDateFormat(this, Locale.getDefault())
        .format(
            Calendar.getInstance().time,
        )
```
