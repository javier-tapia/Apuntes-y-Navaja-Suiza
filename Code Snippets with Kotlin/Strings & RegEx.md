<h1><i>Strings & RegEx</i></h1>

***Index***:
<!-- TOC -->
  * [Colores ARGB](#colores-argb)
  * [*Format Strings*](#format-strings)
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

## *Format Strings*
Se utiliza el símbolo ``%`` como **operador de formato** (o _**format specifier starter**_), el cual sirve para **indicar el inicio de una instrucción de formateo**.  
Luego, se pueden agregar _placeholders_ con diferentes tipos de datos, como pueden ser _Strings_ (``s``), Números decimales (``f``), Números enteros (``d``), etc. También se pueden definir la cantidad de decimales (si corresponde).  

```text
%<PARAMETER-NUMBER>$<PARAMETER-TYPE>

%<PARAMETER-NUMBER>$.<NUMBER-OF-DECIMALS><PARAMETER-TYPE>
```

Es posible utilizar sólo ``%<PARAMETER-TYPE>`` (sin el número de parámetro ni el operador aritmético `$`), pero para eso, **el orden de los argumentos debe coincidir exactamente**.  
Esta estructura se puede usar tanto con un archivo de recursos (``strings.xml``) como directamente en código.

📌 Ejemplo con ``strings.xml``:

En el archivo ``strings.xml``:
```xml
<resources>
    <string name="app_name">SampleApp</string>
    <string name="badge_notifications_v1">%s notificaciones, %s sin leer</string>
    <!-- Alternativa: usar formato con el número y tipo de parámetros -->
    <string name="badge_notifications_v2">%1$s notificaciones, %2$s sin leer</string>
    
    <!-- Y si hubiera que formatear un número con dos decimales -->
    <string name="price">%1$.2f dólares</string>
    
    <!-- Para mostrar el carácter % de forma literal en una format string, se debe escribir %%  -->
    <string name="progress">Progreso: %s%%</string>
</resources>
```

Y en código (ejemplo en Compose):
```kotlin
val badgeNumber = "8"
val otherValue = "4"
Text(
    // Equivalente a Context.getString(...) si se trabajara con XML y ViewBinding
    text = stringResource(R.string.badge_notifications, badgeNumber, otherValue) // 8 notificaciones, 4 sin leer
)

val price = 2.5F
Text(text = stringResource(R.string.price, price)) // 2.50 dólares

val progress = 50
Text(text = stringResource(R.string.progress, progress)) // Progreso: 50%
```

📌 Ejemplo con ``.format``:

```kotlin
val numbers = listOf(25.5, 26.1, 24.3, 25.2)
println("Máximo: %.1f".format(numbers.max())) // Máximo: 26.1
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
