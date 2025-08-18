# _Serializable_ / _Bundle_

Para reemplazar `getSerializableExtra` y `getParcelableExtra` cuando se marcan como **deprecados**, según la versión de Android:

```kotlin
inline fun <reified T : Serializable> Bundle.serializable(key: String): T? = when {
  Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU -> getSerializable(key, T::class.java)
  else -> @Suppress("DEPRECATION") getSerializable(key) as? T
}

inline fun <reified T : Serializable> Intent.serializable(key: String): T? = when {
  Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU -> getSerializableExtra(key, T::class.java)
  else -> @Suppress("DEPRECATION") getSerializableExtra(key) as? T
}
```
