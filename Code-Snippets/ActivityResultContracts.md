# ActivityResultContracts

1. Crear variable de tipo `ActivityResultLauncher<Intent!>`:

```kotlin
private val resultLauncher = registerForActivityResult(ActivityResultContracts.StartActivityForResult()) { _ -> }
```

2. 