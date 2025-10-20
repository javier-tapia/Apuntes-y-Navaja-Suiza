<h1>Flows</h1>

***Index***:
<!-- TOC -->
  * [Internal event bus with ``MutableSharedFlow``](#internal-event-bus-with-mutablesharedflow)
<!-- TOC -->

---

## Internal event bus with ``MutableSharedFlow``

```kotlin
/**
 * A private [MutableSharedFlow] that acts as an internal event bus for this ViewModel.
 *
 * This flow is used to receive [SampleUiEvent]s from the UI (via the public `processUiEvent` method)
 * and process them sequentially in a single collector coroutine launched in the `init` block.
 *
 * Using a `SharedFlow` here helps to decouple the event emission from its processing,
 * ensuring that events are handled one by one, preventing race conditions or complex concurrent logic.
 *
 * - `extraBufferCapacity = 1`: Allows one event to be buffered if the collector is busy.
 * - `onBufferOverflow = BufferOverflow.DROP_LATEST`: If more events arrive while the buffer is full,
 *   the newest event is dropped.
 */
private val eventsFlow = MutableSharedFlow<SampleUiEvent>(
    extraBufferCapacity = 1,
    onBufferOverflow = BufferOverflow.DROP_LATEST
)

init {
    viewModelScope.launch {
        eventsFlow.collect { event ->
            processEventInternal(event)
        }
    }
}

fun processUiEvent(uiEvent: SampleUiEvent) {
    viewModelScope.launch {
        eventsFlow.emit(uiEvent)
    }
}

private suspend fun processEventInternal(uiEvent: SampleUiEvent) {
    when (uiEvent) {
        is InitialUiEvent -> doSomething(uiEvent)
    }
}
```