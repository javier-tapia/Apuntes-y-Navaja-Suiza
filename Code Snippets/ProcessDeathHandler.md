<h1>ProcessDeathHandler</h1>

***Index***:
<!-- TOC -->
  * [Create the ProcessDeathHandler interface](#create-the-processdeathhandler-interface)
  * [Implement the interface](#implement-the-interface)
<!-- TOC -->

---

## Create the ProcessDeathHandler interface

```kotlin
import android.os.Bundle

private const val PID = "pid"
private const val DEFAULT_PID = -1

/**
 * This interface has the responsibility to detect if the process has died and give a callback
 * to handle it by the activity that implements this.
 */
interface ProcessDeathHandler {
    fun onProcessDeath()

    fun onSaveState(outState: Bundle) {
        outState.putInt(PID, android.os.Process.myPid())
    }

    fun onRestoreState(savedInstanceState: Bundle) {
        val pid = savedInstanceState.getInt(PID, DEFAULT_PID)
        if (pid != DEFAULT_PID && pid != android.os.Process.myPid()) {
            onProcessDeath()
        }
    }
}
```

## Implement the interface

```kotlin
class SampleActivity : AppCompatActivity(), ProcessDeathHandler {
		// onCreate() and whatever...
		
		override fun onProcessDeath() {
        // Reset values, navigate to another screen, finish(), etc...
    }
    
    override fun onSaveInstanceState(outState: Bundle) {
        super.onSaveInstanceState(outState)
        onSaveState(outState)
    }
    
    override fun onRestoreInstanceState(savedInstanceState: Bundle) {
        super.onRestoreInstanceState(savedInstanceState)
        onRestoreState(savedInstanceState)
    }
}
```
