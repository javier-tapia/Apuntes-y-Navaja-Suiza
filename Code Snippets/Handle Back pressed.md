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
    // Whatever
    setupOnBackPressed()
}

private fun setupOnBackPressed() {
    onBackPressedDispatcher.addCallback(
        this,
        object : OnBackPressedCallback(true) {
            override fun handleOnBackPressed() {
                // Do something when Back is pressed
            }
        },
    )
}
```
