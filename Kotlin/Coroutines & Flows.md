<h1>Corrutinas (<i>Coroutines</i>) & <i>Flows</i></h1>

**Las corrutinas** están diseñadas para ejecutar **operaciones asíncronas** complejas de forma limpia y **secuencialmente**, lo que significa que el código de la corrutina espera a que regrese lo que invocó antes de continuar. Esto permite, entre otras cosas, **no bloquear el hilo principal**. Para eso, se utilizan **funciones de suspensión** (***suspension functions***), como ***``delay()``***, ***``await()``*** (que se utiliza junto con el *builder ``async{}``*) y ***``withContext()``*** (una práctica recomendada consiste en usar ``withContext()`` a fin de garantizar que todas las funciones sean seguras para el subproceso principal (*main-safe*), lo cual significa que se puede llamar a la función desde el subproceso principal). En esencia, **_la función de suspensión realiza una acción asíncrona, pero para la corrutina que la invoca, se considera síncrona_**.  
También se puede indicar que una función personalizada es de suspensión anteponiéndole la palabra reservada ***``suspend``*** (pausa la ejecución de la corrutina actual y guarda todas las variables locales) o ***``resume``*** (continúa la ejecución de una corrutina suspendida desde donde se detuvo). A las funciones de suspensión sólo se las puede llamar **desde una corrutina** o **desde otra función de suspensión** y **retornan asincrónicamente un solo valor**.

**Los *Flows*** se utilizan para retornar múltiples valores computados asincrónicamente. Están diseñados explícitamente para manejar **operaciones asíncronas** complejas de forma efectiva y **emitir varias veces según se requiera**.  
A diferencia de los canales (*channels*), los _flows_ son *cold streams*, al igual que las secuencias (*sequences*) de Kotlin. El código dentro del constructor de un *flow* (*flow builder*), no se ejecuta hasta que el *flow* es recolectado.  
En el mundo Android, estas características hacen de *Flow* una excelente alternativa a *LiveData*. *Flow* ofrece una funcionalidad similar: *builders*, *cold streams* y auxiliares útiles (por ejemplo, transformación de datos). Y a diferencia de *LiveData*, **no están vinculados al ciclo de vida y brindan más control sobre el contexto de ejecución**.

***Index***:
<!-- TOC -->
  * [Jerarquía conceptual de las corrutinas](#jerarquía-conceptual-de-las-corrutinas)
  * [*CoroutineScope*](#coroutinescope)
  * [*CoroutineContext*](#coroutinecontext)
  * [*Job*](#job)
  * [*SupervisorJob*](#supervisorjob)
  * [*Dispatchers*](#dispatchers)
  * [*CoroutineCancellationException*](#coroutinecancellationexception)
  * [*CoroutineName*](#coroutinename)
  * [*Testing* en corrutinas: ``StandardTestDispatcher`` y ``UnconfinedTestDispatcher``](#testing-en-corrutinas-standardtestdispatcher-y-unconfinedtestdispatcher)
    * [**StandardTestDispatcher**](#standardtestdispatcher)
    * [**UnconfinedTestDispatcher**](#unconfinedtestdispatcher)
    * [Ejemplo para crear una clase ``TestDispatcherRule`` (o ``MainCoroutineRule``)](#ejemplo-para-crear-una-clase-testdispatcherrule-o-maincoroutinerule)
  * [*Cold Flow* vs *Hot Flow*](#cold-flow-vs-hot-flow)
  * [*`StateFlow` vs `SharedFlow`*](#stateflow-vs-sharedflow)
<!-- TOC -->

---

## Jerarquía conceptual de las corrutinas

1. **CoroutineScope**: Es un concepto de nivel superior que proporciona un ámbito para lanzar corrutinas.
    - Tiene métodos como **`launch`**, **`async`**, etc.
    - Contiene un _CoroutineContext_ que define cómo se comportarán las corrutinas lanzadas.
2. **CoroutineContext**: Es un conjunto de elementos que definen el comportamiento de las corrutinas.
    - Es parte de un _CoroutineScope_.
    - Contiene elementos como _Job_, _Dispatcher_, etc.
3. **Elementos del CoroutineContext** (_Job_, _Dispatcher_, etc.): Son componentes individuales que definen aspectos específicos del comportamiento de las corrutinas.
    - Son parte de un _CoroutineContext_.

```
CoroutineScope
├── CoroutineContext
│   ├── Job / SupervisorJob
│   ├── Dispatcher
│   ├── CoroutineCancellationException
│   ├── CoroutineName
```

## *CoroutineScope*
**¿Qué es?**

- Un ámbito o contenedor para corrutinas que define su ciclo de vida.
- Una interfaz que proporciona el método **`launch`** y otros constructores de corrutinas.

**Características principales:**

- Permite agrupar corrutinas relacionadas bajo un mismo paraguas.
- Facilita la cancelación en grupo de todas las corrutinas lanzadas dentro del scope.
- Hereda el contexto de corrutina que se le proporciona al crearlo.

**Ejemplos comunes:**

- **`lifecycleScope`**: Vinculado al ciclo de vida de un componente de Android.
- **`viewModelScope`**: Vinculado al ciclo de vida de un _ViewModel_.
- **`CoroutineScope(Dispatchers.Main)`**: Un scope personalizado que ejecuta corrutinas en el hilo principal.

**Uso típico:**

```kotlin
  val scope = CoroutineScope(Dispatchers.Main)
  scope.launch {
      // Corrutina que se ejecutará en el hilo principal
  }

  // Más tarde, cuando ya no se necesite:
  scope.cancel() // Cancela todas las corrutinas lanzadas en este scope
```

## *CoroutineContext*
**¿Qué es?**

- Un conjunto de elementos que definen el comportamiento de una corrutina.
- Una estructura de datos indexada similar a un mapa, donde cada elemento tiene una clave única.

**Elementos principales que puede contener:**

- ``Job``: Controla el ciclo de vida de la corrutina.
- ``Dispatcher``: Define en qué hilo se ejecutará la corrutina.
- ``CoroutineExceptionHandler``: Define cómo se manejan las excepciones.
- ``CoroutineName``: Proporciona un nombre para depuración.

**Características:**

- Es inmutable, pero se puede crear uno nuevo combinando contextos existentes.
- Se puede acceder a elementos individuales usando corchetes: **`context[Job]`**.
- Se pueden combinar contextos usando el operador **`+`**.

**Uso típico:**

```kotlin
  val context = Dispatchers.IO + Job() + CoroutineName("MiCorrutina")

  launch(context) {
      // Esta corrutina se ejecutará en el hilo IO, con un Job específico y un nombre
  }
```

## *Job*
**¿Qué es?**

- Representa una tarea cancelable con un ciclo de vida.
- Es el componente que controla el ciclo de vida de una corrutina.
- Cada corrutina está asociada a un _Job_ específico.

**Estados de un _Job_:**

1. _New_ (Nuevo): Cuando se crea pero aún no ha comenzado.
2. _Active_ (Activo): Cuando está ejecutándose.
3. _Completing_ (Completando): Estado transitorio cuando está terminando su ejecución.
4. _Completed_ (Completado): Cuando ha terminado con éxito.
5. _Cancelling_ (Cancelando): Estado transitorio durante la cancelación.
6. _Cancelled_ (Cancelado): Cuando ha sido cancelado.

**Características:**

- Organiza las corrutinas en una jerarquía padre-hijo.
- La cancelación se propaga de padres a hijos (pero no al revés).
- Si un hijo falla, la excepción se propaga al padre y cancela a todos los hermanos.

**Uso típico:**

```kotlin
  val job = Job()
  launch(job) {
      // Esta corrutina está asociada al job
  }
  // Más tarde:
  job.cancel() // Cancela la corrutina
```

## *SupervisorJob*

**¿Qué es?**

- Una variante especial de _Job_ que cambia el comportamiento de propagación de fallos.

**Diferencia clave con _Job_:**

- En un _Job_ normal, si un hijo falla, la excepción se propaga al padre y cancela a todos los hermanos.
- En un _SupervisorJob_, si un hijo falla, la excepción NO se propaga al padre y NO cancela a los hermanos.

**Casos de uso:**

- Cuando se tiene múltiples tareas independientes y se quiere que el fallo de una no afecte a las demás.
- Ideal para componentes de UI donde cada elemento puede fallar independientemente.

**Uso típico:**

```kotlin
  val supervisorJob = SupervisorJob()

  launch(supervisorJob) {
      // Esta corrutina puede fallar sin afectar a otras
  }

  launch(supervisorJob) {
      // Esta corrutina seguirá ejecutándose incluso si la anterior falla
  }
```

## *Dispatchers*

**¿Qué es?**

- Determina en qué hilo o hilos se ejecutará una corrutina.
- Define la política de distribución de trabajo para las corrutinas.

**Tipos principales:**

1. **``Dispatchers.Main``**: Ejecuta corrutinas en el hilo principal de UI.
    - **`.immediate`**: Ejecuta inmediatamente si ya se está en el hilo principal.
2. **``Dispatchers.IO``**: Optimizado para operaciones de entrada/salida (red, archivos).
    - Utiliza un _pool_ de hilos compartido.
    - Limitado a 64 hilos o el número de núcleos, lo que sea mayor.
3. **``Dispatchers.Default``**: Optimizado para operaciones intensivas de CPU.
    - Utiliza un _pool_ de hilos igual al número de núcleos de CPU.
    - Ideal para procesamiento de listas, cálculos complejos, etc.
4. **``Dispatchers.Unconfined``**: No está confinado a ningún hilo específico.
    - Comienza en el hilo del llamador.
    - Después de la primera suspensión, continúa en el hilo que completó la suspensión.
    - Raramente usado en código de aplicación.

**Uso típico:**

```kotlin
  // Para operaciones de UI
  launch(Dispatchers.Main) {
      updateUI()
  }
  
  // Para operaciones de red
  launch(Dispatchers.IO) {
      val data = fetchDataFromNetwork()
      withContext(Dispatchers.Main) {
          updateUI(data)
      }
  }
  
  // Para cálculos intensivos
  launch(Dispatchers.Default) {
      val result = processLargeList()
      withContext(Dispatchers.Main) {
          showResult(result)
      }
  }
```

## *CoroutineCancellationException*

**Terminología**:

- Los ***“puntos de suspensión”*** son lugares en el código de una corrutina donde la ejecución puede pausarse y luego reanudarse más tarde. En el contexto de las corrutinas de Kotlin, estos son:
  1. **Funciones de suspensión**: Cualquier función marcada con la palabra clave **`suspend`**, como:
    - **`delay(timeMillis)`**: Pausa la corrutina por un tiempo específico
    - **`withContext(dispatcher)`**: Cambia el contexto de ejecución
    - **`join()`**: Espera a que otra corrutina termine
    - **`await()`**: Espera el resultado de un **`Deferred`**
  2. **Funciones de suspensión personalizadas**: Cualquier función que se haya creado con la palabra clave **`suspend`**
  3. ***Flow Operators***: Operadores como **`collect`**, **`first`**, **`single`** en los flujos de Kotlin
- La ***"cancelación cooperativa"*** en corrutinas de Kotlin significa que una corrutina no se detiene inmediatamente cuando se cancela, sino que debe "cooperar" para terminar adecuadamente.

  En pocas palabras:

  1. Cuando se llama a **`job.cancel()`**, la corrutina no se detiene instantáneamente.
  2. En su lugar, se marca como "cancelando" y se lanza un **`CancellationException`** en el próximo punto de suspensión (como **`delay()`**, **`yield()`**, etc.).
  3. La corrutina tiene la oportunidad de ejecutar código de limpieza (cerrar recursos, etc.) antes de terminar completamente.
  4. Es "cooperativa" porque la corrutina debe verificar periódicamente si ha sido cancelada y responder adecuadamente, generalmente a través de puntos de suspensión que automáticamente verifican el estado de cancelación.

**¿Qué es?**

**`CancellationException`** es una excepción especial que se usa para señalar la cancelación cooperativa de una corrutina. A diferencia de otras excepciones, no se considera un error y no se propaga a los padres en la jerarquía de corrutinas.

**Características principales:**

- Es una señal de cancelación, no un error real
- Indica que una corrutina ha sido cancelada intencionalmente
- Se usa internamente para implementar la cancelación cooperativa
- No se propaga a los padres en la jerarquía de corrutinas
- No causa que el _scope_ padre cancele a otros hijos
- Generalmente no debe ser capturada, o si se captura, debe ser relanzada

**Manejo adecuado:**

```kotlin
  try {
      // Código que puede ser cancelado
  } catch (e: CancellationException) {
      // Limpieza específica para cancelación
      throw e  // IMPORTANTE: re-lanzar la excepción para que la cancelación continúe
  }
```

## *CoroutineName*

**¿Qué es?**

**`CoroutineName`** es un elemento del **`CoroutineContext`** que proporciona un nombre para la corrutina, útil para depuración.

**Características:**

- Ayuda a identificar corrutinas en _logs_ y _dumps_ de hilos
- No afecta el comportamiento de la corrutina, solo es para depuración
- Se puede combinar con otros elementos del contexto

**Uso típico:**

```kotlin
  val scope = CoroutineScope(SupervisorJob() + Dispatchers.Main + CoroutineName("MyFragmentScope"))
  
  // O para una corrutina específica
  scope.launch(CoroutineName("CargarDatos")) {
      // Esta corrutina tendrá un nombre en los logs
  }
```

## *Testing* en corrutinas: ``StandardTestDispatcher`` y ``UnconfinedTestDispatcher``
Ambos son *dispatchers* utilizados en pruebas de corrutinas en Kotlin, pero tienen comportamientos diferentes:

### **StandardTestDispatcher**

1. **Ejecución controlada**: Las tareas se encolan pero no se ejecutan automáticamente. Se necesita llamar explícitamente a métodos como **`advanceUntilIdle()`** o **`advanceTimeBy()`** para que se ejecuten las tareas pendientes.
2. **Control de tiempo**: Permite controlar exactamente cuándo se ejecutan las tareas, lo que facilita probar comportamientos que dependen del tiempo.
3. **Determinismo**: Proporciona un comportamiento determinista y predecible, ideal para pruebas unitarias.
4. **Caso de uso ideal**: Pruebas donde se necesita verificar el orden exacto de ejecución o controlar precisamente cuándo se ejecutan las corrutinas.

```kotlin
  // Ejemplo con StandardTestDispatcher
  val testDispatcher = StandardTestDispatcher()
  launch(testDispatcher) {
      // Este código no se ejecutará inmediatamente
      println("Tarea ejecutada")
  }
  // Necesitas llamar a esto para ejecutar la tarea
  testDispatcher.scheduler.advanceUntilIdle()
```

### **UnconfinedTestDispatcher**

1. **Ejecución inmediata**: Las tareas se ejecutan inmediatamente, sin necesidad de avanzar manualmente el tiempo.
2. **Comportamiento "ansioso" (*eager behavior*)**: Ejecuta las corrutinas tan pronto como se lanzan, hasta que alcanzan un punto de suspensión. La documentación de Kotlin también usa el término *"unconfined"* para describir este comportamiento, indicando que la ejecución no está confinada o restringida a un orden específico controlado por el programador, sino que fluye libremente.
3. **Simplicidad**: Más simple de usar cuando no se necesita controlar el tiempo de ejecución.
4. **Caso de uso ideal**: Pruebas donde solo interesa el resultado final y no el orden o tiempo de ejecución.

```kotlin
  // Ejemplo con UnconfinedTestDispatcher
  val testDispatcher = UnconfinedTestDispatcher()
  launch(testDispatcher) {
      // Este código se ejecutará inmediatamente
      println("Tarea ejecutada")
  }
  // No se necesita avanzar el tiempo manualmente
```

### Ejemplo para crear una clase ``TestDispatcherRule`` (o ``MainCoroutineRule``)

```kotlin
  class TestDispatcherRule(
    private val testDispatcher: TestDispatcher = UnconfinedTestDispatcher(),
  ) : TestWatcher() {
    val testDispatcherProvider = mockk<DispatcherProvider>()
  
    init {
      every { testDispatcherProvider.main() } returns testDispatcher
      every { testDispatcherProvider.io() } returns testDispatcher
    }
  
    override fun starting(description: Description) {
      super.starting(description)
      Dispatchers.setMain(testDispatcher)
    }
  
    override fun finished(description: Description) {
      super.finished(description)
      Dispatchers.resetMain()
    }
  }
```

## *Cold Flow* vs *Hot Flow*

| ***Cold Flow***                                                                                                                                                                                 | ***Hot Flow***                                                                                                                                                                                                                    |
|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Emits values only when there is an active collector.<br>Each collector receives the emitted values independently,<br>and they start receiving emissions from the beginning when they subscribe. | Emits values regardless of whether there are active collectors.<br>It can produce values even if there are no subscribers.<br>New collectors joining a hot flow might miss emissions that occurred before they started listening. |
| A cold flow does not store the data.                                                                                                                                                            | A hot flow can store the data.                                                                                                                                                                                                    |
|                                                                                                                                                                                                 | You can convert cold flow to hot flow, using `stateIn` and `sharedIn` intermediate operators.                                                                                                                                     |
| Examples: `Flow`                                                                                                                                                                                | Examples: `SharedFlow` and `StateFlow`                                                                                                                                                                                            |

## *`StateFlow` vs `SharedFlow`*

| ***StateFlow***                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            | ***SharedFlow***                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |
|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ***Hot** Flow*                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | ***Hot** flow*                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
| interface [StateFlow](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-state-flow/index.html)<out [T](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-state-flow/index.html)> : [SharedFlow](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-shared-flow/index.html)<[T](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-state-flow/index.html)>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | interface [SharedFlow](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-shared-flow/index.html)<out [T](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-shared-flow/index.html)> : [Flow](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-flow/index.html)<[T](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-shared-flow/index.html)>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| Will cache the latest emission → Commonly used for **displaying UI State**                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 | Emits values to all consumers that collect from it                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                |
| Requires an **initial state** to be passed in to the constructor                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | Allows customized behavior in the following ways:<br>• `replay` → Allows resend a number of previously-emitted values for new subscribers.<br>• `extraBufferCapacity` → The number of values buffered in addition to replay. `emit` does not suspend while there is a buffer space remaining (optional, cannot be negative, defaults to zero).<br>• `onBufferOverflow` → Allows specify a policy for when the buffer is full of items to be sent.<br>&nbsp;&nbsp;&nbsp;&nbsp;• `SUSPEND` → Suspend until free space appears in the buffer. Use this to create backpressure, forcing the producers to slow down creation of new values in response to consumers not being able to process the incoming values in time. `SUSPEND` **is a good choice when all elements must eventually be processed.**<br>&nbsp;&nbsp;&nbsp;&nbsp;• `DROP_OLDEST` → Drop the oldest value in the buffer on overflow, add the new value to the buffer, do not suspend. **Use this in scenarios when only the last few values are important and skipping the processing of severely outdated ones is desirable.**<br>&nbsp;&nbsp;&nbsp;&nbsp;• `DROP_LATEST` → Leave the buffer unchanged on overflow, dropping the value that we were going to add, do not suspend. This option can be used in **rare advanced scenarios** where all elements that are expected to enter the buffer are equal, so it is **not important which of them get thrown away**.                                                                                                             |
| `stateIn`<br><br>`fun <T> MutableStateFlow<T>.stateIn(`<br>&nbsp;&nbsp;&nbsp;&nbsp;`scope: CoroutineScope,`<br>&nbsp;&nbsp;&nbsp;&nbsp;`started: SharingStarted = SharingStarted.WhileSubscribed(),`<br>&nbsp;&nbsp;&nbsp;&nbsp;`initialValue: T`<br>`): StateFlow<T>`<br><br>• `scope` → The coroutine scope that the shared Flow should use.<br>• `started` →The sharing behavior that determines when the Flow should start and stop emitting values.<br>&nbsp;&nbsp;&nbsp;&nbsp;• `SharingStarted.Lazily`: The flow starts sharing data when the first collector starts collecting and stops when the last collector stops. This is useful when you want to share the state **only when there’s an active collector**.<br>&nbsp;&nbsp;&nbsp;&nbsp;• `SharingStarted.WhileSubscribed`: The flow begins sharing data when the first collector starts collecting and stops after a specified period of inactivity (i.e. when there are no active collectors). This is useful when you want to **share the state while it’s being used but stop after some time when it’s no longer needed.**<br>&nbsp;&nbsp;&nbsp;&nbsp;• `SharingStarted.Eagerly`: The flow starts sharing data immediately and continues sharing indefinitely, regardless of whether there are any active collectors. This is useful when you want to **share the state all the time**.<br>• `initialValue` → The initial value of the *StateFlow* that is emitted when no value has been produced yet. | `sharedIn`<br><br>`fun <T> Flow<T>.shareIn(`<br>&nbsp;&nbsp;&nbsp;&nbsp;`scope: CoroutineScope,`<br>&nbsp;&nbsp;&nbsp;&nbsp;`started: SharingStarted = SharingStarted.WhileSubscribed(),`<br>&nbsp;&nbsp;&nbsp;&nbsp;`replay: Int = 0`<br>`): SharedFlow<T>`<br><br>• `scope` → The coroutine scope that the shared Flow should use.<br>• `started` →The sharing behavior that determines when the Flow should start and stop emitting values.<br>&nbsp;&nbsp;&nbsp;&nbsp;• `SharingStarted.Lazily`: The flow starts sharing data when the first collector starts collecting and stops when the last collector stops. This is useful when you want to share the state **only when there’s an active collector**.<br>&nbsp;&nbsp;&nbsp;&nbsp;• `SharingStarted.WhileSubscribed`: The flow begins sharing data when the first collector starts collecting and stops after a specified period of inactivity (i.e. when there are no active collectors). This is useful when you want to **share the state while it’s being used but stop after some time when it’s no longer needed.**<br>&nbsp;&nbsp;&nbsp;&nbsp;• `SharingStarted.Eagerly`: The flow starts sharing data immediately and continues sharing indefinitely, regardless of whether there are any active collectors. This is useful when you want to **share the state all the time**.<br>• `replay` → The number of previous emissions that should be replayed to new subscribers. This is useful when we want to **provide new subscribers with a certain number of past emissions.** |


