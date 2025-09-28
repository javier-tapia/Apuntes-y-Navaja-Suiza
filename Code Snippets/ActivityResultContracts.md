<h1><i>ActivityResultContracts</i></h1>

## `StartActivityForResult`
1. Crear una propiedad de tipo `ActivityResultLauncher<Intent!>` antes de crear la *activity* y registrar el *callback*:

```kotlin
    private val resultLauncher: ActivityResultLauncher<Intent> =
        registerForActivityResult(ActivityResultContracts.StartActivityForResult()) { result: ActivityResult! ->
            // Handle the returned result
            if (result.resultCode == SOME_CODE) {
                val data: Intent? = result.data
                setResult(SOME_CODE, data)
                finish()
            }
        }
```

2. Lanzar otra *activity* (sólo si el ciclo de vida de la *activity* actual alcanzó el estado `CREATED`):

```kotlin
    import android.content.Intent
    import android.net.Uri
    
    private fun navigateToAnotherScreen() {
        // Do something
    
        val intent = Intent(Intent.ACTION_VIEW)
            .apply {
                `package` = packageName
                data = Uri.parse(deeplinkUrl)
                addFlags(Intent.FLAG_ACTIVITY_CLEAR_TOP or Intent.FLAG_ACTIVITY_SINGLE_TOP)
            }
        resultLauncher.launch(intent)
    }
```
