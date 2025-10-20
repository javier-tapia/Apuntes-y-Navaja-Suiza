<h1>JSON operations & Error handling with Either</h1>

***Index***:
<!-- TOC -->
  * [``sealed class Either`` (``Left`` and ``Right``)](#sealed-class-either-left-and-right)
  * [``getGsonInstance``](#getgsoninstance)
  * [``<reified T> String?.jsonToObject``](#reified-t-stringjsontoobject)
  * [``<reified T> Any.jsonResourceToObject``](#reified-t-anyjsonresourcetoobject)
  * [``<reified T> Gson.fromJson``](#reified-t-gsonfromjson)
<!-- TOC -->

---

## ``sealed class Either`` (``Left`` and ``Right``)

```kotlin
sealed class Either<out L, out R> {
    data class Left<out L>(val left: L) : Either<L, Nothing>()

    data class Right<out R>(val right: R) : Either<Nothing, R>()
}
```

## ``getGsonInstance``

```kotlin
fun getGsonInstance(): Gson {
    return GsonBuilder()
        .setFieldNamingPolicy(FieldNamingPolicy.LOWER_CASE_WITH_UNDERSCORES)
        .create()
}
```

## ``<reified T> String?.jsonToObject``

```kotlin
/**
 * Constructs a [Gson] object and calls its _fromJson_ method
 * with the receiver object -a JSON- as a parameter
 *
 * @receiver A nullable JSON String representation of an object
 * @param T The type of the object to be returned
 * @param key Optional value for the query param key
 * @return The object of type [T] after deserializing its JSON String representation; an exception otherwise
 */
inline fun <reified T> String?.jsonToObject(key: String? = null): Either<T, Exception> {
    val includeKeyInMessage =
        if (key.isNullOrEmpty()) {
            ""
        } else {
            "For the \"$key\" query param. "
        }

    if (this.isNullOrEmpty()) {
        return Either.Right(
            NullPointerException(
                "${includeKeyInMessage}The receiver object for String?.jsonToObject is null or empty.",
            ),
        )
    }

    return try {
        val gson = getGsonInstance()
        Either.Left(gson.fromJson(this, object : TypeToken<T>() {}.type))
    } catch (e: JsonSyntaxException) {
        Either.Right(JsonSyntaxException("$includeKeyInMessage${e.message}. ${e.stackTrace}"))
    } catch (e: JsonParseException) {
        Either.Right(JsonParseException("$includeKeyInMessage${e.message}. ${e.stackTrace}"))
    }
}

// Whatever the function or property in which jsonToObject extension function is used
private val getSample: Sample?
    get() {
        // Do something to decode a String and save it in a stringDecoded variable
    
        return when (val either = stringDecoded?.jsonToObject<Sample>()) {
            is Either.Left -> either.left
            is Either.Right -> throw either.right
        }
    }
```

## ``<reified T> Any.jsonResourceToObject``

```kotlin
inline fun <reified T> Any.jsonResourceToObject(res: String): T {
    return getGsonInstance()
        .fromJson(
            InputStreamReader(this::class.java.classLoader!!.getResourceAsStream(res)),
        )
}
```

## ``<reified T> Gson.fromJson``

```kotlin
inline fun <reified T> Gson.fromJson(json: InputStreamReader) =
    this.fromJson<T>(json, object : TypeToken<T>() {}.type)!!
```
