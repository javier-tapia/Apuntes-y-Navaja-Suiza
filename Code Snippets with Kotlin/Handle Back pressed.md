<h1>Handle Back pressed</h1>

***Index***:
<!-- TOC -->
  * [``onBackPressedDispatcher.addCallback``](#onbackpresseddispatcheraddcallback)
<!-- TOC -->

---

## ``onBackPressedDispatcher.addCallback``
```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    // Otro código...
    setupOnBackPressed()
}

private fun setupOnBackPressed() {
    onBackPressedDispatcher.addCallback(
        this,
        object : OnBackPressedCallback(true) {
            override fun handleOnBackPressed() {
                // Impedir redirecciones solicitadas por paquetes no permitidos
                Log.d("log", "🔙 Back button pressed")
                val allowedPackages = listOf("com.example.myapp", "com.other", "com.android.chrome")
                val callingPackage = callingActivity?.packageName
                if (callingPackage != null && allowedPackages.contains(callingPackage)) {
                    val destinationActivityName = intent.getStringExtra("destinationActivity")
                    Log.d("intent:", intent.toString())
                    if (destinationActivityName != null) {
                        val destinationActivity = Class.forName(destinationActivityName)
                        val redirectionIntent = Intent(this@ExampleActivity, destinationActivity)
                        redirectionIntent.putExtra("build", intent.getStringExtra("build"))
                        startActivityForResult(redirectionIntent, LAUNCH_INTERNAL_ACTIVITY)
                    }
                } else {
                    Log.d("ERROR", "No se permite la redirección desde este paquete")
                }
            }
        },
    )
}
```
