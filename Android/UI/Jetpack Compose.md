# Jetpack Compose (https://developer.android.com/compose)
**Jetpack Compose** es el _toolkit_ moderno de Android para construir interfaces de usuario de forma **declarativa**. En lugar de manipular vistas imperativamente, se describe **cómo debería verse la UI según su estado**, y Compose se encarga de renderizarla y actualizarla automáticamente.

- [*State* in *Compose*](#state-in-compose-httpsdeveloperandroidcomdevelopuicomposestate)
  - [*UI State* vs *UI Events*](#ui-state-vs-ui-events)
  - [*Compose State - Flow - LiveData*](#compose-state---flow---livedata)
    - [Compose State](#compose-state)
      - [`mutableStateOf`](#mutablestateof)
      - [`produceState`](#producestate)
      - [`derivedStateOf`](#derivedstateof)
    - [*Flow*](#flow)
    - [*LiveData*](#livedata)
  - [`remember` and `rememberSaveable`](#remember-and-remembersaveable)
    - [`remember` with a `key` vs `remember` in conjunction with `derivedStateOf`](#remember-with-a-key-vs-remember-in-conjunction-with-derivedstateof)
  - [*Side Effects*](#side-effects)
- [Animaciones](#animaciones)
  - [*Tween*](#tween)
- [Previews](#previews)
  - [Live Template](#live-template--prevcol)
  - [Using `PreviewParameterProvider`](#using-previewparameterprovider)

---

## *State* in *Compose* (https://developer.android.com/develop/ui/compose/state)

### *UI State* vs *UI Events*

- ***UI State*** → **Qué se muestra** en la pantalla y **cómo se muestra** (propiedades intrínsecas a los elementos de la UI que influyen en cómo se renderizan, como el tamaño de fuente, el color, etc.)
- ***UI Events*** → **Acciones** (del usuario o del sistema) que deberían gestionarse en la capa de UI (se incluye al *ViewModel*)

### *Compose State - Flow - LiveData*

#### Compose State

##### [`mutableStateOf`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#mutableStateOf(kotlin.Any,androidx.compose.runtime.SnapshotMutationPolicy))

Creates an observable [`MutableState<T>`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/MutableState) (which is an observable type integrated with the compose runtime, i.e. `MutableState` class is a single **value holder whose reads and writes are observed by Compose**) initialized with the passed in [`value`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#mutableStateOf(kotlin.Any,androidx.compose.runtime.SnapshotMutationPolicy)). **Any changes to `value` (reads and writes) schedules recomposition of any composable functions that read `value`**.

```kotlin
  interface State<T : Any?>
  
  interface MutableState<T> : State<T> {
    override var value: T
  }
  
  fun <T : Any?> mutableStateOf(
    value: T,
    policy: SnapshotMutationPolicy<T> = structuralEqualityPolicy()
  ): MutableState<T>
```

**Code snippet examples**:

```kotlin
  val mutableState = remember { mutableStateOf(default) }
  
  var value by remember { mutableStateOf(default) }
  
  val (value, setValue) = remember { mutableStateOf(default) }
  
  var username by mutableStateOf("")
      private set
```

Compose doesn't require that you use `MutableState<T>` to hold state; it supports other observable types. **Before reading another observable type in Compose, you must convert it to a `State<T>` so that composables can automatically recompose when the state changes**.  
Compose ships with functions to create [`State<T>`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/State) from common observable types used in Android apps. Before using these integrations, add the appropriate [artifact(s)](https://developer.android.com/jetpack/androidx/releases/compose-runtime#declaring_dependencies).

**Key Differences with StateFlow** (refer to [***Flow***](#flow))

| ***Feature***       | `MutableState`                                                                                                                                                                 | `StateFlow`                                                                                                                                               |
|---------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------|
| Purpose             | Managing _**synchronous**_, recomposition-triggering state                                                                                                                     | Managing _**asynchronous**_, observable state                                                                                                             |
| Mutability          | _**Directly mutable**_. You can change its value using its value property                                                                                                      | _**Read-only**_ (requires `MutableStateFlow` for updates)                                                                                                 |
| Asynchronous        | No                                                                                                                                                                             | Yes                                                                                                                                                       |
| Backing Property    | Not typical                                                                                                                                                                    | Common (with `.asStateFlow()`)                                                                                                                            |
| Initial Value       | Required                                                                                                                                                                       | Required                                                                                                                                                  |
| Lifecycle           | Inherently tied to composable lifecycle. When the composable is recomposed, the MutableState (if remembered correctly) retains its value                                       | Not inherently lifecycle-aware. You need to use functions like `collectAsStateWithLifecycle()` in Compose to ensure proper lifecycle handling             |
| Compose Integration | Automatic recomposition on value change                                                                                                                                        | Requires `collectAsStateWithLifecycle()` (or `collectAsState()`)                                                                                          |
| **_Use Cases_**     | _**Local composable state, UI element values, such as text field input, checkbox states, visibility flags, and other values that directly affect the composable's rendering**_ | _**ViewModel state, asynchronous data streams (e.g., network requests, database queries), user interactions, or other events that can change over time**_ |

##### [`produceState`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#produceState(kotlin.Any,kotlin.coroutines.SuspendFunction1))
> Also, refer to [Side Effects](#side-effects)

Return an observable [`snapshot`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/snapshots/Snapshot) [`State`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/State) that produces values over time without a defined data source.  
[`producer`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#produceState(kotlin.Any,kotlin.coroutines.SuspendFunction1)) is launched when [`produceState`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#produceState(kotlin.Any,kotlin.coroutines.SuspendFunction1)) enters the composition and is cancelled when [`produceState`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#produceState(kotlin.Any,kotlin.coroutines.SuspendFunction1)) leaves the composition. In other words, it launches a coroutine scoped to the Composition that can push values into a returned [`State`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/State). **_Use it to convert non-Compose state into Compose state_**, for example bringing external subscription-driven state such as `Flow`, `LiveData`, or `RxJava` into the Composition.  
[`producer`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#produceState(kotlin.Any,kotlin.coroutines.SuspendFunction1)) should use [`ProduceStateScope.value`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/ProduceStateScope#value()) to set new values on the returned [`State`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/State).  
The returned [`State`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/State) conflates values; no change will be observable if [`ProduceStateScope.value`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/ProduceStateScope#value()) is used to set a value that is [`equal`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/-any/equals.html) to its old value, and observers may only see the latest value if several values are set in rapid succession.  
Even though `produceState` creates a coroutine, it can also be used to observe non-suspending sources of data. To remove the subscription to that source, use the [`awaitDispose`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/ProduceStateScope#awaitDispose(kotlin.Function0)) function.

```kotlin
  interface State<T : Any?>
  
  @Composable
  fun <T : Any?> produceState(
    initialValue: T,
    producer: suspendProduceStateScope<T>.() -> Unit
  ): State<T>
```

**Code snippet example**:

```kotlin
  val uiState by
  produceState<UiState<List<Person>>>(UiState.Loading, viewModel) {
    viewModel.people.map { UiState.Data(it) }.collect { value = it }
  }
  
  when (val state = uiState) {
    is UiState.Loading -> _root_ide_package_.org.w3c.dom.Text("Loading...")
    is UiState.Data ->
      Column {
        for (person in state.data) {
          _root_ide_package_.org.w3c.dom.Text("Hello, ${person.name}")
        }
      }
  }
  
----------
  
  val uiState by produceState<TasksUiState>(
    initialValue = TasksUiState.Loading,
    key1 = lifecycle,
    key2 = tasksViewModel
  ) {
    lifecycle.repeatOnLifecycle(
      state = Lifecycle.State.STARTED
    ) {
      tasksViewModel.uiState.collect {
        value = it
      }
    }
  }
```

##### [`derivedStateOf`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#derivedStateOf(kotlin.Function0))
> Also, refer to [Side Effects](#side-effects) and [`remember` with a `key` vs `remember` in conjunction with `derivedStateOf`](#remember-with-a-key-vs-remember-in-conjunction-with-derivedstateof)

**_Convert one or multiple state objects into another state_**.  
In Compose, [recomposition](https://developer.android.com/develop/ui/compose/mental-model#recomposition) occurs each time an observed state object or composable input changes. A state object or input may be changing more often than the UI actually needs to update, leading to unnecessary recomposition.  
You should use the [`derivedStateOf`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#derivedStateOf(kotlin.Function0)) function when your inputs to a composable are changing more often than you need to recompose. This often occurs when something is frequently changing, such as a scroll position, but the composable only needs to react to it once it crosses a certain threshold. `derivedStateOf` **_creates a new Compose state object you can observe that only updates as much as you need_**. In this way, it acts similarly to the Kotlin Flows [`distinctUntilChanged()`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/distinct-until-changed.html#:%7E:text=Returns%20flow%20where%20all%20subsequent,a%20StateFlow%20has%20no%20effect.) operator. However, **`derivedStateOf`** is expensive, and you should only use it to avoid unnecessary recomposition when a result hasn't changed.  
**_Creates a [`State`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/State) object whose [`State.value`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/State#value()) is the result of [`calculation`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#derivedStateOf(kotlin.Function0))_**. The result of calculation will be cached in such a way that calling [`State.value`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/State#value()) repeatedly will not cause [`calculation`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#derivedStateOf(kotlin.Function0)) to be executed multiple times, but reading [`State.value`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/State#value()) will cause all [`State`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/State) objects that got read during the [`calculation`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#derivedStateOf(kotlin.Function0)) to be read in the current [`Snapshot`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/snapshots/Snapshot), meaning that this will correctly subscribe to the derived state objects if the value is being read in an observed context such as a [`Composable`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/Composable) function. Derived states without mutation policy trigger updates on each dependency change. To avoid invalidation on update, provide suitable [`SnapshotMutationPolicy`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/SnapshotMutationPolicy) through [`derivedStateOf`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#derivedStateOf(kotlin.Function0)) overload.

```kotlin
  interface State<T : Any?>
  
  fun <T : Any?> derivedStateOf(calculation: () -> T): State<T>
```

**Code snippet example**:

```kotlin
  @Composable
  fun CountDisplay(count: State<Int>) {
    _root_ide_package_.org.w3c.dom.Text("Count: ${count.value}")
  }
  
  @Composable
  fun Example() {
    var a by remember { mutableStateOf(0) }
    var b by remember { mutableStateOf(0) }
    val sum = remember { derivedStateOf { a + b } }
    // Changing either a or b will cause CountDisplay to recompose but not trigger Example
    // to recompose.
    CountDisplay(sum)
  }
  
------------------------------------------------------
  
  @Composable
  // When the messages parameter changes, the MessageList
  // composable recomposes. derivedStateOf does not
  // affect this recomposition.
  fun MessageList(messages: List<Message>) {
    Box {
      val listState = rememberLazyListState()
  
      LazyColumn(state = listState) {
        // ...
      }
  
      // Show the button if the first visible item is past
      // the first item. We use a remembered derived state to
      // minimize unnecessary compositions
      val showButton by remember {
        derivedStateOf {
          listState.firstVisibleItemIndex > 0
        }
      }
  
      AnimatedVisibility(visible = showButton) {
        ScrollToTopButton()
      }
    }
  }
```

#### ***Flow***
[`collectAsState()`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#(kotlinx.coroutines.flow.StateFlow).collectAsState(kotlin.coroutines.CoroutineContext)) is similar to `collectAsStateWithLifecycle` (see below), because it also collects values from a `Flow` and transforms it into Compose [`State`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/State). Use `collectAsState` for platform-agnostic code instead of `collectAsStateWithLifecycle`, which is Android-only.  
[`collectAsStateWithLifecycle()`](https://developer.android.com/reference/kotlin/androidx/lifecycle/compose/package-summary#extension-functions) collects values from a [`Flow`](https://kotlin.github.io/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-flow/index.html) in a lifecycle-aware manner, allowing your app to conserve app resources. It represents the latest emitted value from the Compose [`State`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/State). Use this API as the recommended way to collect flows on Android apps.

```kotlin
  import kotlin.coroutines.CoroutineContext
  import kotlin.coroutines.EmptyCoroutineContext
  
  interface State<T : Any?>
  
  @Composable
  fun <T : Any?> StateFlow<T>.collectAsStateWithLifecycle(
    lifecycle: Lifecycle,
    minActiveState: Lifecycle.State = Lifecycle.State.STARTED,
    context: CoroutineContext = EmptyCoroutineContext
  ): State<T>
```

> *Under the hood, the implementation of `collectAsStateWithLifecycle` uses the `repeatOnLifecycle` API which is the recommended way to collect flows in Android using the View system.
`collectAsStateWithLifecycle` saves you from typing the boilerplate code shown below that also collects flows in a lifecycle-aware manner from a composable function (...)
The UI can help free up resources by collecting the UI state using `collectAsStateWithLifecycle`. The ViewModel can do the same by producing the UI state in a collector-aware manner. If there are no collectors , such as when the UI isn’t visible on screen, stop the upstream flows coming from the data layer. You can do so using the [`.stateIn(WhileSubscribed)`](https://github.com/android/nowinandroid/blob/main/feature-author/src/main/java/com/google/samples/apps/nowinandroid/feature/author/AuthorViewModel.kt#L104) flow API when producing the UI state.*  
> Source: https://manuelvivo.dev/consuming-flows-compose || https://medium.com/androiddevelopers/consuming-flows-safely-in-jetpack-compose-cde014d0d5a3

[`.asStateFlow()`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/as-state-flow.html) → La extensión `.asStateFlow()` en Kotlin sirve para **exponer una versión inmutable** de un `MutableStateFlow`.  
Dicho de otra forma, si se hiciera `val uiState: StateFlow<LoginUiState> = _uiState`, el compilador **permitirá eso sin problemas**, porque `MutableStateFlow` **es un `StateFlow`.** Pero se sigue teniendo acceso a los métodos mutables si se hace un *cast* o se pasa la instancia a algún otro componente (`(uiState as MutableStateFlow).value = otroValor`), **_pudiendo mutar el estado desde fuera, lo cual rompe el principio de encapsulamiento_**.  
`.asStateFlow()` devuelve una instancia de `StateFlow` que **_oculta los métodos de mutabilidad_** (como `.value =`, `.emit()`, etc.), aunque internamente siga siendo la misma instancia. Con esto, se **_evita la posibilidad de que alguien haga *cast* o acceda directamente a la mutabilidad_** desde fuera del scope controlado.

**Code snippet example**:

```kotlin
  // ViewModel
  
  private val _uiState = MutableStateFlow(InterestsUiState(loading = true))
  val uiState: StateFlow<InterestsUiState> = _uiState.asStateFlow()
  
  fun onInterestChanged(interest: Int) {
    _uiState.update { uiState ->
      uiState.copy(
        interest = interest
      )
    }
  }
  
  val selectedTopics =
    interestsRepository.observeTopicsSelected().stateIn(
      viewModelScope,
      SharingStarted.WhileSubscribed(5000),
      emptySet()
    )
  
------------
  
  // Composable
  
  val uiState by interestsViewModel.uiState.collectAsStateWithLifecycle()
  
  val selectedTopics by interestsViewModel.selectedTopics.collectAsStateWithLifecycle()
```

#### ***LiveData***
[`observeAsState()`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/livedata/package-summary#(androidx.lifecycle.LiveData).observeAsState(kotlin.Any)) starts observing this [`LiveData`](https://developer.android.com/reference/kotlin/androidx/lifecycle/LiveData) and represents its values via [`State`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/State). Every time there would be new value posted into the [`LiveData`](https://developer.android.com/reference/kotlin/androidx/lifecycle/LiveData) the returned [`State`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/State) will be updated causing recomposition of every [`State.value`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/State#value()) usage.  
The [`initial`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/livedata/package-summary#(androidx.lifecycle.LiveData).observeAsState(kotlin.Any)) value will be used only if this LiveData is not already [`initialized`](https://kotlinlang.org/api/latest/jvm/stdlib/kotlin/index.html). Note that if [`T`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/livedata/package-summary#(androidx.lifecycle.LiveData).observeAsState(kotlin.Any)) is a non-null type, it is your responsibility to ensure that any value you set on this LiveData is also non-null.  
Uses `Lifecyle` internally for safely observing the data. The inner observer will automatically be removed when this composable disposes or the current [`LifecycleOwner`](https://developer.android.com/reference/kotlin/androidx/lifecycle/LifecycleOwner) moves to the [`Lifecycle.State.DESTROYED`](https://developer.android.com/reference/kotlin/androidx/lifecycle/Lifecycle.State#DESTROYED) state.

```kotlin
  interface State<T : Any?>
  
  @Composable
  fun <R : Any?, T : R> LiveData<T>.observeAsState(initial: R): State<R>
```

**Code snippet example**:

```kotlin
  // ViewModel
  
  private var _textFieldHelperErrorMessage = MutableLiveData<String>()
  val textFieldHelperErrorMessage: LiveData<String> = _textFieldHelperErrorMessage
  
-------------
  
  // Composable
  
  val textFieldHelperErrorMessage by viewModel.textFieldHelperErrorMessage.observeAsState(initial = "")
```

### `*remember*` and `*rememberSaveable*`
Composable functions can use the [`remember`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#remember(kotlin.Function0)) API to store an object in memory. `remember` can be used to store both mutable and immutable objects. You can use the remembered value as a parameter for other composables or even as logic in statements to change which composables are displayed.  
The [`remember`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#remember(kotlin.Any,kotlin.Any,kotlin.Any,kotlin.Function0)) API is frequently used together with [`MutableState`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/MutableState) (see previous section about states). In general, **_`remember` takes a `calculation` lambda parameter. When `remember` is first run, it invokes the `calculation` lambda and stores its result. During recomposition, `remember` returns the value that was last stored_**.  
`remember` stores the value until it leaves the Composition. However, there is a way to invalidate the cached value. The `remember` API also takes a `key` or `keys` parameter. ***If any of these keys change, the next time the function recomposes*, `remember` *invalidates the cache and executes the calculation lambda block again***.

```kotlin
  @Composable
  inline fun <T : Any?> remember(
    vararg keys: Any?,
    crossinline calculation: @DisallowComposableCalls () -> T
  ): T
```

**Code snippet example**:

```kotlin
  import android.graphics.Color
  import android.widget.Button
  
  var isVisible by remember { mutableStateOf(true) }
  
  Column(Modifier.fillMaxSize()) {
    Button(onClick = { isVisible = !isVisible }) {
      _root_ide_package_.org.w3c.dom.Text("Show/Hide")
    }
  
    Spacer(modifier = Modifier.size(50.dp))
  
    AnimatedVisibility(
      isVisible,
      enter = slideInHorizontally(),
      exit = slideOutHorizontally()
    ) {
      Box(
        Modifier
          .size(150.dp)
          .background(Color.Red)
      )
    }
  }
```

The [`rememberSaveable`](https://developer.android.com/reference/kotlin/androidx/compose/runtime/saveable/package-summary#rememberSaveable(kotlin.Array,androidx.compose.runtime.saveable.Saver,kotlin.String,kotlin.Function0)) API behaves similarly to `remember` because it retains state across recompositions, and also across activity or process recreation using the saved instance state mechanism. For example, this happens, when the screen is rotated.  
`rememberSaveable` automatically saves any value that can be saved in a `Bundle`. For other values, you can pass in a **custom saver** object, for example, a parcelable object, a MapSaver or a ListSaver (ref [here](https://developer.android.com/develop/ui/compose/state#ways-to-store)).  
`rememberSaveable` receives `input` parameters for the same purpose that `remember` receives `keys`. ***The cache is invalidated when any of the inputs change***. The next time the function recomposes, `rememberSaveable` re-executes the calculation lambda block.

```kotlin
  @Composable
  fun <T : Any> rememberSaveable(
    vararg inputs: Any?,
    saver: Saver<T, Any> = autoSaver(),
    key: String? = null,
    init: () -> T
  ): T
```

**Code snippet example**:

```kotlin
  // rememberSaveable stores userTypedQuery until typedQuery changes
  var userTypedQuery by rememberSaveable(typedQuery, stateSaver = TextFieldValue.Saver) {
    mutableStateOf(
      TextFieldValue(text = typedQuery, selection = TextRange(typedQuery.length))
    )
  }
```

#### `remember` with a `key` vs `remember` in conjunction with `derivedStateOf`
They both tend to do the same: **_listen for changes in another state and then derive a state off of that_**.

| `val something = remember(key = someKey) {…}`                                                                                                                                                                                                               | `val something = remember { derivedStateOf {…} }`                                                                                                                                                                                                                                                                                                |
|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `remember` is a ***Composable* function** → Compose will take a look at the parameters that we pass (the `keys`) and **_as soon as any of those parameters (keys) change, then Compose will recompose the composable_** in which `remember` is implemented. | `derivedStateOf` is a ***non-Composable* function** → Only will update the resulting state (of `something`) if the end result of the calculation (`derivedStateOf` block) actually changed. And this ***end result*** changes pretty infrequently. That means, **_Compose will recompose when the end result changes, not just the parameter_**. |

### [*Side Effects*](https://developer.android.com/develop/ui/compose/side-effects)
A **side-effect** is a **_change to the state of the app that happens outside the scope of a composable function_**. Examples include network requests, database operations, updating a ViewModel, etc.  
Sometimes side-effects are necessary, for example, to trigger a one-off event such as showing a snackbar or navigate to another screen given a certain state condition. These actions should be called from a controlled environment that is aware of the lifecycle of the composable.  
**_An effect is a composable function that doesn't emit UI and causes side effects to run when a composition completes._**

- **`LaunchedEffect`:** composable function that **_runs suspend functions_** in the scope of a composable. It is **_triggered when it enters the composition and will be restarted if any of its key parameters change_**. If no keys are provided, it will be restarted on every recomposition. It's primarily used for side effects that need to be **_tied to the lifecycle of a composable_** and potentially need to be re-executed when certain state changes occur. Examples include launching a coroutine to fetch data when a screen is displayed or starting an animation that should restart when a specific value changes.  
- **`SideEffect`:** composable function that **_runs a block of code after every successful recomposition_**. It is **_not tied to any specific state changes within the composable_**. It simply executes its code block after the UI has been updated and is stable. It's typically used for side effects that need to be synchronized with the Compose runtime but don't necessarily need to be re-executed when specific state changes happen. A common use case is to publish Compose state to non-Compose code, ensuring that the external code always has the latest UI state.  
- **`DisposableEffect`:** effects that require cleanup.  
- **`rememberCoroutineScope`:** obtain a composition-aware scope to launch a coroutine outside a composable.  
- **`rememberUpdatedState`:** reference a value in an effect that shouldn't restart if the value changes.  
- **`produceState`:** convert non-Compose state into Compose state (also, refer to [`produceState`](#producestate).  
- **`derivedStateOf`:** convert one or multiple state objects into another state (also, refer to [`derivedStateOf`](#derivedstateof).  
- **`snapshotFlow`:** convert Compose's State into Flows.

**Key differences between `LaunchedEffect` and `SideEffect`:**

| ***Feature*** | `LaunchedEffect`                                                | `SideEffect`                                                                                                                   |
|---------------|-----------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------|
| Trigger       | Enters composition, restarts on key changes                     | After every successful recomposition                                                                                           |
| Recomposition | Restarted if keys change (or on every recomposition if no keys) | Runs after every recomposition, not tied to specific state changes                                                             |
| Use Case      | Side effects tied to specific state changes                     | Side effects synchronized with the Compose runtime                                                                             |
| Example       | Fetching data on screen load, restarting animation              | Publishing Compose state to non-Compose code, i. e. making Compose state changes visible or accessible to the non-Compose code |

## Animaciones

### *Tween*
`Tween` viene de "in-betweening", un término usado en animación tradicional para referirse a los cuadros intermedios entre dos estados clave (_keyframes_).  
En Jetpack Compose, un **`tween`** es una especificación de animación que define:

- **Duración**: Cuánto tiempo tarda la animación.  
- **Easing** (curva de aceleración): Cómo cambia el valor a lo largo del tiempo (lineal, aceleración, desaceleración, etc.).  
- **Delay** (opcional): Espera antes de que comience la animación.

**Code snippet example**:

```kotlin
  // Anima el valor de opacidad (alpha) de 0f a 1f o viceversa.
  // Usa un tween con:
  //  - Duración de 1 segundo (1000 ms);
  //  - Retardo de 300 ms antes de comenzar;
  //  - Y una curva de desaceleración (acelera rápido y se desacelera al final).
  val animatedAlpha by animateFloatAsState(
    targetValue = if (isVisible) 1f else 0f,
    animationSpec = tween(
      durationMillis = 1000,
      easing = LinearOutSlowInEasing,
      delayMillis = 300
    )
  )
```

## Previews

### Live Template → `prevCol`

```kotlin
  // Creates a CollectionPreviewParameterProvider
  
  class $NAME$: PreviewParameterProvider < $TYPE$> {
    override val values = sequenceOf(
      $END$
    )
  }
```

### Using `PreviewParameterProvider`

```kotlin
  // ExampleParameterProvider.kt
  
  data class ExampleParameters(
    val name: String?,
    val lastName: String?,
    val onTextFieldValueChange: (String) -> Unit
  )
  
  class ExamplePreviewParameterProvider : PreviewParameterProvider<ExampleParameters> {
    override val values = sequenceOf(
      ExampleParameters(
        name = "Javi",
        lastName = "Fulanito",
        onTextFieldValueChange = { }
      ),
      ExampleParameters(
        name = "Juan",
        lastName = "Polainas",
        onTextFieldValueChange = { }
      ),
      ExampleParameters(
        name = "Jhon",
        lastName = "Connor",
        onTextFieldValueChange = { }
      )
    )
  }
  
-------------
  
  // Example.kt
  
  @Composable
  fun Example(
    name: String?,
    lastName: String?,
    val onTextFieldValueChange: (String) -> Unit
  ) {
    // Do something
  }
  
  @Preview
  @Composable
  private fun ExamplePreview(
    @PreviewParameter(ExamplePreviewParameterProvider::class)
    exampleParameters: ExampleParameters
  ) {
    Example(
      name = exampleParameters.name,
      lastName = exampleParameters.lastName,
      onTextFieldValueChange = exampleParameters.onTextFieldValueChange
    )
  }
```
