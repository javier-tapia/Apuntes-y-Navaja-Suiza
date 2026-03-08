<h1>Asincronía & Concurrencia</h1>

***Index***:
<!-- TOC -->
  * [TL;DR: Corrutinas y *Flows*](#tldr-corrutinas-y-flows)
  * [Jerarquía conceptual de las corrutinas](#jerarquía-conceptual-de-las-corrutinas)
  * [*CoroutineScope*](#coroutinescope)
  * [*CoroutineContext*](#coroutinecontext)
  * [*Job*](#job)
  * [*SupervisorJob*](#supervisorjob)
  * [*Dispatchers*](#dispatchers)
  * [*CoroutineCancellationException*](#coroutinecancellationexception)
  * [*CoroutineName*](#coroutinename)
  * [Algunas comparativas útiles](#algunas-comparativas-útiles)
    * [*Cold Flow* vs *Hot Flow*](#cold-flow-vs-hot-flow)
    * [`StateFlow` vs `SharedFlow`](#stateflow-vs-sharedflow)
    * [``Channel`` vs ``SharedFlow``](#channel-vs-sharedflow)
  * [Operadores de *Flows*: Intermedios vs Terminales](#operadores-de-flows-intermedios-vs-terminales)
  * [Anotaciones & Funciones](#anotaciones--funciones)
    * [Creación y ejecución de *Flows*](#creación-y-ejecución-de-flows)
      * [``asFlow``](#asflow)
      * [``collect``](#collect)
      * [``flow``](#flow)
      * [``flowOf``](#flowof)
    * [Emisión y *Backpressure* en *Flows*](#emisión-y-backpressure-en-flows)
      * [`emit` & `tryEmit`](#emit--tryemit)
    * [Transformación y combinación de *Flows*](#transformación-y-combinación-de-flows)
      * [``combine``](#combine)
      * [`scan`](#scan)
    * [Concurrencia y desacople en *Flows*](#concurrencia-y-desacople-en-flows)
      * [``buffer``](#buffer)
      * [``conflate``](#conflate)
      * [``flatMapMerge``](#flatmapmerge)
      * [``flowOn``](#flowon)
      * [``merge``](#merge)
    * [Operadores de *side-effects*, *lifecycle* y errores en *Flows*](#operadores-de-side-effects-lifecycle-y-errores-en-flows)
      * [``cancellable``](#cancellable)
      * [``catch``](#catch)
      * [``onCompletion``](#oncompletion)
      * [``onEach``](#oneach)
      * [``onStart``](#onstart)
    * [Control de ritmo (*time-based*) en *Flows*](#control-de-ritmo-time-based-en-flows)
      * [``debounce``](#debounce)
      * [``sample``](#sample)
    * [Operadores terminales de obtención de valor en *Flows*](#operadores-terminales-de-obtención-de-valor-en-flows)
      * [``first``](#first)
      * [``single``](#single)
    * [Lanzamiento, *scopes* y cancelación](#lanzamiento-scopes-y-cancelación)
      * [``cancel``](#cancel)
      * [``launch``](#launch)
      * [``launchIn``](#launchin)
    * [Sincronización y concurrencia](#sincronización-y-concurrencia)
      * [`Mutex`](#mutex)
      * [`synchronized()`](#synchronized)
      * [`@Synchronized`](#synchronized-1)
      * [`@ThreadLocal`](#threadlocal)
      * [`@Volatile`](#volatile)
      * [Comparativa](#comparativa)
    * [Serialización y modelo de datos](#serialización-y-modelo-de-datos)
      * [`@Transient`](#transient)
    * [Primitivas de suspensión y cooperación en corrutinas](#primitivas-de-suspensión-y-cooperación-en-corrutinas)
      * [``delay``](#delay)
      * [``yield``](#yield)
      * [``async`` & ``await``](#async--await)
      * [``withContext``](#withcontext)
      * [``join``](#join)
      * [``resume``](#resume)
  * [*Testing* en corrutinas: ``StandardTestDispatcher`` y ``UnconfinedTestDispatcher``](#testing-en-corrutinas-standardtestdispatcher-y-unconfinedtestdispatcher)
    * [``StandardTestDispatcher``](#standardtestdispatcher)
    * [``UnconfinedTestDispatcher``](#unconfinedtestdispatcher)
    * [Modelo de ``TestDispatcherRule`` (o ``MainCoroutineRule``)](#modelo-de-testdispatcherrule-o-maincoroutinerule)
  * [Referencias y Fuentes](#referencias-y-fuentes)
<!-- TOC -->

---

## TL;DR: Corrutinas y *Flows*
> 🔍 Ver también [Manejo de *Flows* en la UI](../Apuntes-Android.md#manejo-de-flows-en-la-ui)

**Las Corrutinas** están diseñadas para ejecutar **operaciones asíncronas** de forma limpia y estructurada. Permiten escribir código que **se lee de forma secuencial**, pero que se ejecuta de manera **no bloqueante**. La principal herramienta para lograrlo son las **funciones de suspensión** ([***suspending functions***](https://kotlinlang.org/docs/coroutines-basics.html#suspending-functions)).

Cuando una corrutina encuentra una `suspend fun`:

- **Se suspende**: "Pausa" su propia ejecución en ese punto exacto, guardando su estado.
- **Libera el hilo**: El hilo en el que corría queda libre para hacer otro trabajo.
- **Se reanuda**: Una vez que la `suspend fun` termina su tarea (ej: una llamada a la red), la corrutina **se reanuda exactamente donde se detuvo** para continuar con la siguiente línea.

El mecanismo de reanudación ([`resume`](#resume)) depende de cómo se estructura la concurrencia y se relaciona directamente con el control del flujo de ejecución en las corrutinas:

1. **Reanudación tras finalizar el trabajo (Comportamiento de espera):**
    - En este modelo, una corrutina necesita el **resultado** de una operación asíncrona para poder continuar su procesamiento. Esto se ejemplifica mediante el uso de `async` para iniciar la operación, que devuelve un `Deferred` representando el resultado de esa operación que se completará en el futuro.
    - Cuando la corrutina encuentra la función de suspensión `await()` en ese `Deferred`, se **suspende** en ese punto. Esto significa que su contexto se guarda (como el estado de las variables locales y la posición en el código) y se libera el hilo para que realice otras tareas.
    - Solo cuando la operación asíncrona ha finalizado y el resultado está disponible, la corrutina se **reanuda** en el punto donde se detuvo, permitiendo que el procesamiento continúe sin perder su estado anterior. Este enfoque garantiza un flujo ordenado y controlado, evitando condiciones de carrera.
2. **Reanudación tras despachar el trabajo (Comportamiento "*fire-and-forget*"):**
    - En este caso, una corrutina solo necesita **iniciar una tarea sin esperar un resultado inmediato**. Un ejemplo común es el uso de `launch`, donde se ejecuta una tarea de forma paralela.
    - La corrutina se suspende lo suficiente para **despachar** la tarea hacia otro hilo o contexto, permitiendo que se ejecute independientemente. Después de este despacho, la corrutina original puede **reanudar su ejecución** inmediatamente, continuando con las siguientes líneas de código.
    - Este enfoque puede ser riesgoso si no se gestiona adecuadamente, especialmente en flujos donde el **orden de las operaciones** es crítico. Si la corrutina reanuda su ejecución sin esperar a que la tarea despachada termine, puede producirse una condición de carrera (*race condition*). Esto ocurre cuando múltiples corrutinas acceden o modifican recursos compartidos simultáneamente, lo que puede provocar resultados inesperados y comportamientos indeseables.

Las `suspend fun` más comunes son [`delay()`](#delay), [`withContext()`](#withcontext) (para cambiar de hilo de forma segura, garantizando *main-safety*), y [`await()`](#async--await) (para esperar un resultado). A las funciones de suspensión sólo se las puede llamar **desde una corrutina** o **desde otra función de suspensión** y retornan asincrónicamente un solo valor.

**Los *Flows***, a diferencia de las funciones de suspensión que devuelven solo un único valor, se utilizan para **emitir múltiples valores secuencialmente, computados asincrónicamente**. Están diseñados explícitamente para manejar **operaciones asíncronas** complejas de forma efectiva y **emitir varias veces según se requiera**.  
Los _Flows_ son ***cold streams***, al igual que las [Secuencias (*Sequences*)](../Apuntes-Kotlin.md#410-sequencet). El código dentro del constructor de un *flow* (*flow builder*), no se ejecuta hasta que el *flow* es recolectado.  
En el mundo Android, estas características hacen de *Flow* una excelente alternativa a *LiveData*, ya que ofrece una funcionalidad similar: *builders*, *cold streams* y auxiliares útiles (por ejemplo, transformación de datos). Y a diferencia de *LiveData*, **no están vinculados al ciclo de vida y brindan más control sobre el contexto de ejecución**.

## Jerarquía conceptual de las corrutinas

1. **_CoroutineScope_** :arrow_right: Es un concepto de nivel superior que proporciona un ámbito para lanzar corrutinas.
    - Tiene métodos como **`launch`**, **`async`**, etc.
    - Contiene un _CoroutineContext_ que define cómo se comportarán las corrutinas lanzadas.
2. **_CoroutineContext_** :arrow_right: Es un conjunto de elementos que definen el comportamiento de las corrutinas.
    - Es parte de un _CoroutineScope_.
    - Contiene elementos como _Job_, _Dispatcher_, etc.
3. **Elementos del _CoroutineContext_ (_Job_, _Dispatcher_, etc.)** :arrow_right: Son componentes individuales que definen aspectos específicos del comportamiento de las corrutinas.
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

## Algunas comparativas útiles

### *Cold Flow* vs *Hot Flow*

| ***Cold Flow***                                                                                                                                                                                               | ***Hot Flow***                                                                                                                                                                                                                                                                                                  |
|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Emite valores **solo cuando hay un collector activo**.<br>Cada collector **recibe los valores emitidos de forma independiente**,<br>y comienza a recibir emisiones **desde el principio** cuando se suscribe. | Emite valores **sin importar si hay collectors activos**.<br>Puede producir valores incluso si no hay suscriptores.<br>Los nuevos collectors que se unan a un hot flow pueden **perder emisiones** que ocurrieron antes de empezar a recolectar, si el flow no guarda estado o `replay` (ver siguiente cuadro). |
| Un cold flow **no almacena los datos**.                                                                                                                                                                       | Un hot flow **puede almacenar los datos**.                                                                                                                                                                                                                                                                      |
|                                                                                                                                                                                                               | Se puede convertir un cold flow en un hot flow usando los operadores intermedios `stateIn` y `sharedIn`.                                                                                                                                                                                                        |
| Ejemplos: `Flow`                                                                                                                                                                                              | Ejemplos: `SharedFlow` y `StateFlow`                                                                                                                                                                                                                                                                            |

### `StateFlow` vs `SharedFlow`

| ***StateFlow***                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            | ***SharedFlow***                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
|----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| ***Hot** Flow*                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | ***Hot** Flow*                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| interface [StateFlow](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-state-flow/index.html)<out [T](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-state-flow/index.html)> : [SharedFlow](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-shared-flow/index.html)<[T](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-state-flow/index.html)>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                | interface [SharedFlow](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-shared-flow/index.html)<out [T](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-shared-flow/index.html)> : [Flow](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-flow/index.html)<[T](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-shared-flow/index.html)>                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                   |
| **TL;DR**<br><br>Ideal para **_estado persistente o continuo_**, como el estado de la UI (`UiState`). Siempre mantiene un **valor actual (vigente)**, que cualquier nuevo collector recibe inmediatamente al suscribirse.<br>Si se necesita **_un valor vigente y consistente para todos los observers_**, usar `StateFlow`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | **TL;DR**<br><br>Ideal para **_eventos efímeros o transmisiones_**, como toasts, navegación o logs. No guarda estado a menos que se configure `replay`, y permite **_controlar buffers y políticas de overflow_**.<br>Si se necesita **_emitir eventos que pueden ser ignorados por los que lleguen tarde_**, usar `SharedFlow`.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                     |
| Requiere que se le pase un **estado inicial** en el constructor                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                            | Permite personalizar el comportamiento de las emisiones mediante:<br>• `replay` → Cantidad de valores previamente emitidos que se reenviarán a nuevos suscriptores.<br>• `extraBufferCapacity` → Cantidad de valores que pueden almacenarse en el buffer además de los de ``replay``. Mientras haya espacio, ``emit`` no se suspende. Opcional, no puede ser negativo, por defecto es 0.<br>• `onBufferOverflow` → Política a aplicar cuando el buffer está lleno:<br>&nbsp;&nbsp;&nbsp;&nbsp;• `SUSPEND` → Suspende hasta que haya espacio libre. Crea [_backpressure_](../Glosary%20&%20Core%20Concepts/Software%20in%20general.md#backpressure), haciendo que el productor desacelere si los consumidores no procesan los valores a tiempo. Recomendado cuando todos los elementos deben procesarse eventualmente.**<br>&nbsp;&nbsp;&nbsp;&nbsp;• `DROP_OLDEST` → Descarta el valor más antiguo del buffer, agrega el nuevo y no suspende. Útil cuando solo interesan los valores más recientes.**<br>&nbsp;&nbsp;&nbsp;&nbsp;• `DROP_LATEST` → Descarta el valor nuevo (no modifica el buffer actual). Útil en casos avanzados donde los elementos son equivalentes y no importa cuál se descarte.                                                                                                                                                                                                                                                                                                                                                                                                                               |
| `stateIn`<br><br>`fun <T> MutableStateFlow<T>.stateIn(`<br>&nbsp;&nbsp;&nbsp;&nbsp;`scope: CoroutineScope,`<br>&nbsp;&nbsp;&nbsp;&nbsp;`started: SharingStarted = SharingStarted.WhileSubscribed(),`<br>&nbsp;&nbsp;&nbsp;&nbsp;`initialValue: T`<br>`): StateFlow<T>`<br><br>Crea una versión compartida del flow que mantiene su estado dentro del ``scope`` indicado.<br><br>• `scope` → El coroutine scope que el Flow compartido debe usar.<br>• `started` → El comportamiento de compartir que determina cuándo el Flow debe empezar y dejar de emitir valores.<br>&nbsp;&nbsp;&nbsp;&nbsp;• `SharingStarted.Lazily`: El flujo comienza a compartir datos cuando el primer collector empieza a recolectar y se detiene cuando el último collector deja de recolectar. Útil para compartir el estado **solo cuando hay un collector activo**.<br>&nbsp;&nbsp;&nbsp;&nbsp;• `SharingStarted.WhileSubscribed`: El flujo comienza a compartir datos cuando el primer collector empieza a recolectar y se detiene tras un período de inactividad (cuando no hay collectors activos). Útil para **compartir el estado mientras se está usando y detenerlo después de cierto tiempo, cuando ya no se necesita**.<br>&nbsp;&nbsp;&nbsp;&nbsp;• `SharingStarted.Eagerly`: El flujo comienza a compartir datos inmediatamente y continúa compartiéndolos indefinidamente, aunque no haya collectors activos. Útil para **compartir el estado todo el tiempo**.<br>• `initialValue` → El valor inicial del `StateFlow` que se emite cuando aún no se ha producido ningún valor. | `sharedIn`<br><br>`fun <T> Flow<T>.shareIn(`<br>&nbsp;&nbsp;&nbsp;&nbsp;`scope: CoroutineScope,`<br>&nbsp;&nbsp;&nbsp;&nbsp;`started: SharingStarted = SharingStarted.WhileSubscribed(),`<br>&nbsp;&nbsp;&nbsp;&nbsp;`replay: Int = 0`<br>`): SharedFlow<T>`<br><br>Crea una versión compartida del flow que mantiene su estado dentro del ``scope`` indicado.<br><br>• `scope` → El coroutine scope que el Flow compartido debe usar.<br>• `started` → El comportamiento de compartir que determina cuándo el Flow debe empezar y dejar de emitir valores.<br>&nbsp;&nbsp;&nbsp;&nbsp;• `SharingStarted.Lazily`: El flujo comienza a compartir datos cuando el primer collector empieza a recolectar y se detiene cuando el último collector deja de recolectar. Útil para compartir el estado **solo cuando hay un collector activo**.<br>&nbsp;&nbsp;&nbsp;&nbsp;• `SharingStarted.WhileSubscribed`: El flujo comienza a compartir datos cuando el primer collector empieza a recolectar y se detiene tras un período de inactividad (cuando no hay collectors activos). Útil para **compartir el estado mientras se está usando y detenerlo después de cierto tiempo, cuando ya no se necesita**.<br>&nbsp;&nbsp;&nbsp;&nbsp;• `SharingStarted.Eagerly`: El flujo comienza a compartir datos inmediatamente y continúa compartiéndolos indefinidamente, aunque no haya collectors activos. Útil para **compartir el estado todo el tiempo**.<br>• `replay` → Número de emisiones previas que se deben reenviar a nuevos suscriptores. Útil para **proveer a los nuevos suscriptores con un cierto número de emisiones pasadas**. |
| **Casos de uso más comunes**:<br><br>✅ **Representar el estado actual de la UI** que cambia con el tiempo (por ejemplo, `UiState` en `ViewModel`).<br>✅ **Sustituir LiveData** en arquitecturas reactivas (MVVM/MVI).<br>✅ **Exponer un valor observable que siempre tiene un estado actual.** Todo nuevo collector recibe inmediatamente el valor actual almacenado (no espera a que se emita uno nuevo).<br>✅ **Sincronizar estado entre múltiples collectors** que necesitan conocer el valor más reciente (ej. pantalla + test + logger). **Todos los collectors reciben el mismo valor actual al mismo tiempo**, pero `StateFlow` **siempre mantiene un valor vigente**.                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                              | **Casos de uso más comunes**:<br><br>✅ **Emitir eventos efímeros o de una sola vez**, como *navegaciones, mensajes, toasts o errores.*<br>✅ **Transmitir flujos de datos que no representan estado**, sino una **secuencia de valores** (por ejemplo, actualizaciones de ubicación, resultados de red o logs).<br>✅ Difusión (_multicast_) de eventos a múltiples collectors (todos reciben el mismo evento simultáneamente, si están suscritos en ese momento). **A diferencia de StateFlow, los valores no se almacenan**; los collectors que se suscriben tarde no reciben los eventos previos, salvo que se configure `replay`.<br>✅ **Configurar buffers o políticas de desborde (overflow)** cuando se necesita controlar la presión del flujo (cuando los productores generan eventos más rápido de lo que los consumidores pueden procesar).                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |

### ``Channel`` vs ``SharedFlow``
- `Channel`
    - **Cola** :arrow_right: **“Uno toma un _ticket_”** 
    - **_Fan-out “single consumer”_** :arrow_right: **Cada mensaje lo consume SOLO UN _receiver_/consumidor**, aunque haya varios posibles.
    - En Kotlin, un `Channel` se comporta así: si dos corrutinas están recibiendo del mismo _channel_, **cada elemento lo recibe una u otra, NO ambas**. Esto se decide por **orden de espera + _scheduling_** de corrutinas. 
    - Es ideal para “**eventos _one-shot_ a un consumidor**” o **_work-queue_**.
- `SharedFlow`
    - **Altavoz** :arrow_right: **“Todos escuchan el anuncio”**
    - **_Fan-out “broadcast”_** :arrow_right: **Cada mensaje se entrega A TODOS los consumidores activos** (_collectors_ que ya están colectando). 
    - Un **`SharedFlow`** se comporta así: todos los _collectors_ reciben la misma emisión.

## Operadores de *Flows*: Intermedios vs Terminales

- **Intermedios** :arrow_right: Transforman el ``Flow`` y retornan otro ``Flow`` (``Flow<T>``).
- **Terminales** :arrow_right: Consumen el ``Flow`` y disparan la ejecución (devuelve ``Unit``, ``T`` o lanza una corrutina)

> ℹ️ **Nota:**  
> El siguiente cuadro agrupa los operadores principales. Algunos operadores tienen variantes (``mapNotNull``, ``filterIsInstance``, etc.).  
> Los operadores marcados con (``⇉``) introducen un **_concurrency boundary_**: **corrutinas separadas** + **_buffer_** + **desacople** entre producción y consumo.

| Tipo            | Operadores                                                                                                                                                                                                                                                                               |
|-----------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| **Intermedios** | `buffer` (``⇉``), `cancellable`, `catch`, `combine`, `conflate` (``⇉``), `debounce`, `distinctUntilChanged`, `filter`, `flatMapMerge` (``⇉``), `flowOn` (``⇉``), `map`, `merge` (``⇉``), `onCompletion`, `onEach`, `onStart`, `retry`, `retryWhen`, `sample`, `scan`, `transform`, `zip` |
| **Terminales**  | `all`, `any`, `collect`, `collectLatest`, `count`, `first`, `firstOrNull`, `fold`, `last`, `lastOrNull`, `launchIn`, `none`, `produceIn`, `reduce`, `single`, `singleOrNull`, `toList`, `toSet`                                                                                          |

## Anotaciones & Funciones
### Creación y ejecución de *Flows*
> 👉 Dónde nace el flujo y cuándo se ejecuta

#### [``asFlow``](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/as-flow.html)
Convierte una **fuente existente** (colecciones, secuencias, rangos, etc.) en un ``Flow`` frío (**_cold stream_**), emitiendo sus elementos de **forma secuencial al ser colectados**. Se usa principalmente para **adaptar estructuras ya existentes** a APIs basadas en ``Flow``, especialmente en **ejemplos, tests o _pipelines_ simples**.  
No introduce concurrencia ni _buffer_. La emisión ocurre en el **contexto del _collector_**, salvo que se use [``flowOn``](#flowon).

📌 **Ejemplo**:  
Una lista se convierte en ``Flow`` y se consume al colectar

```kotlin
listOf(1, 2, 3)
    .asFlow()
    .collect { println(it) }

// 1
// 2
// 3

// También se puede aplicar sobre rangos o secuencias
(1..3).asFlow()
    .collect { println(it) }
```

#### [``collect``](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-flow/collect.html)
Operador terminal que **inicia la ejecución del _Flow_** y **consume todos los valores emitidos**, ejecutando una acción por cada uno. Es suspendido, respeta cancelación y [_backpressure_](../Glosary%20&%20Core%20Concepts/Software%20in%20general.md#backpressure), y no devuelve un valor (``Unit``).

📌 **Ejemplo**:  
El ``Flow`` se ejecuta, emite ``1``, ``2``, ``3``, y cada valor es consumido en orden
```kotlin
flowOf(1, 2, 3)
    .collect { value ->
        println(value)
    }
```

#### [``flow``](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/flow.html)
Crea un ``Flow`` frío (**_cold stream_**), donde el bloque **se ejecuta al colectar**. Permite **emitir valores suspendibles, cancelables y secuenciales**.

📌 **Ejemplo**:

```kotlin
flow {
    emit(loadFromDisk()) // Bloque de Emisión/Producción de valores
}
```

#### [``flowOf``](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/flow-of.html)
Crea un ``Flow`` frío (**_cold stream_**) a partir de **valores conocidos**, que se **emiten secuencialmente al ser colectados**. Se usa principalmente para **ejemplos, tests y flujos simples**, donde los valores ya están disponibles.  
No introduce concurrencia ni _buffer_, y no ejecuta lógica suspendida.

📌 **Ejemplo**:  
El flujo no se ejecuta hasta que se llama a ``collect``

```kotlin
val flow = flowOf(1, 2, 3)

// Nada se emite aún

flow.collect { println(it) }

// 1
// 2
// 3

// Equivalente conceptual a:
flow {
    emit(1)
    emit(2)
    emit(3)
}
```

### Emisión y *Backpressure* en *Flows*
> 👉 Cómo se producen eventos y quién controla el ritmo

#### [`emit`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-flow-collector/emit.html) & [`tryEmit`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/-mutable-shared-flow/try-emit.html)

- **`emit`**: Es una **función suspendida** que **respeta [_backpressure_](../Glosary%20&%20Core%20Concepts/Software%20in%20general.md#backpressure)**: se suspende hasta que el evento pueda ser emitido (cuando haya un _collector_ activo y/o espacio disponible en el _buffer_), permitiendo que **el consumidor regule el ritmo de producción**.
- **`tryEmit`**: Es una **_función no suspendida_** usada con ``MutableSharedFlow`` y ``MutableStateFlow`` que **_intenta emitir el evento inmediatamente_**. Si el evento fue emitido con éxito (haya o no haya _collectors_), devuelve **`true`**. Si el evento no pudo ser emitido (porque el _buffer_ está lleno y se está usando la estrategia de _overflow_ ``BufferOverflow.SUSPEND``), los eventos adicionales simplemente se descartan (retornando **`false`**), sin suspender ni esperar. Esta es la principal diferencia con ``emit`` y la razón por la que **_se usa para eventos de alta frecuencia y no críticos_**. La función ofrece una solución de **_"intentar y olvidar" (fire-and-forget)_**, donde no es necesario bloquear el hilo emisor si un evento no se puede entregar de inmediato.

📌 **Ejemplo**:

```kotlin
val sharedFlow = MutableSharedFlow<Int>(
    replay = 0,
    extraBufferCapacity = 1
)

coroutineScope.launch {
    sharedFlow.collect { value ->
        delay(100)          // Simula consumidor lento
        println(value)
    }
}

coroutineScope.launch {
    sharedFlow.emit(1)      // Suspende si el buffer está lleno
    sharedFlow.emit(2)
}

coroutineScope.launch {
    val emitted = sharedFlow.tryEmit(3) // No suspende
    println("tryEmit success: $emitted")
}
```

### Transformación y combinación de *Flows*
> 👉 Qué se emite y cómo se combinan los datos

#### [``combine``](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/combine.html)
Operador intermedio que retorna un ``Flow`` cuyos valores se generan con la función ``transform`` (la _lambda_), **combinando los valores emitidos más recientemente por cada _flow_**.

📌 **Ejemplo**:

```kotlin
val flow = flowOf(1, 2).onEach { delay(10) }
val flow2 = flowOf("a", "b", "c").onEach { delay(15) }

combine(flow, flow2) { i, s ->
    i.toString() + s
}.collect { combinedValue ->
    println(combinedValue) // 1a 2a 2b 2c
}
```

#### [`scan`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/scan.html)
Operador intermedio que **acumula estado** a partir de los valores emitidos y **emite cada estado intermedio**, comenzando por un valor inicial.  
Es el equivalente reactivo de un [``fold``](../Kotlin/Colecciones/Aggregate%20operations.md#fold-and-reduce), pero **sin esperar al final del flujo**. Cada emisión depende del valor anterior acumulado.

📌 **Ejemplo**:  
Se emite el estado inicial (``0``) y luego cada acumulación parcial

```kotlin
flowOf(1, 2, 3)
    .scan(0) { acc, value -> acc + value }
    .collect { println(it) }

// 0
// 1
// 3
// 6
```

### Concurrencia y desacople en *Flows*
> 👉 Dónde se separan productor y consumidor (**_concurrency boundaries_**)

#### [``buffer``](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/buffer.html)
Operador intermedio que introduce un _concurrency boundary_ entre el productor y el consumidor, **desacoplándolos mediante un _buffer_** (se ejecutan en **corrutinas separadas**).  
Permite que el **productor emita más rápido** sin esperar a que el consumidor procese cada valor.

📌 **Ejemplo**:  
Sin ``buffer``, cada ``emit`` esperaría al ``collect``.  
Con ``buffer``, la emisión continúa mientras haya espacio disponible en el _buffer_.

```kotlin
flow {
    repeat(3) {
        emit(it)
        println("emit $it")
    }
}
    .buffer()
    .collect {
        delay(100)
        println("collect $it")
    }
```

#### [``conflate``](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/conflate.html)
Operador intermedio que introduce un _concurrency boundary_ y **desacopla productor y consumidor**, pero **solo conserva el valor más reciente** cuando el consumidor es más lento.  
Los valores intermedios **se descartan**, reduciendo la presión sin suspender al productor.

📌 **Ejemplo**:  
El productor emite más rápido que el consumidor.  
El consumidor recibe únicamente el último valor disponible.

```kotlin
flow {
    repeat(5) {
        emit(it)
        delay(10)
    }
}
    .conflate()
    .collect {
        delay(100)
        println(it)
    }

// 0
// 4
```

#### [``flatMapMerge``](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/flat-map-merge.html)
Operador intermedio que **transforma cada valor emitido en un nuevo ``Flow`` y fusiona (mergea) sus emisiones de forma concurrente**.  
Introduce un _concurrency boundary_: cada _flow_ interno se colecta en **corrutinas separadas**, permitiendo que **sus emisiones se intercalen según estén disponibles**.

📌 **Ejemplo**:  
Cada valor se transforma en un ``Flow`` que emite dos valores con demora.  
Las emisiones de ambos flujos se intercalan.

```kotlin
flowOf(1, 2)
    .flatMapMerge { value ->
        flow {
            emit(value * 10)
            delay(50)
            emit(value * 10 + 1)
        }
    }
    .collect { println(it) }

// 10
// 20
// 11
// 21
```

#### [``flowOn``](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/flow-on.html)
Operador intermedio que **cambia el ``CoroutineContext`` del flujo ascendente (_upstream_)**, es decir, el bloque de emisión y los operadores intermedios **definidos antes** de ``flowOn``.

📌 **Ejemplo**:

```kotlin
flow {
    emit(loadFromDisk())
}
    .flowOn(Dispatchers.IO) // Producción/Emisión en IO
    .collect { result ->
        println(result)     // Colección en el contexto actual
    }
```

#### [``merge``](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/merge.html)
Operador intermedio que **combina múltiples _Flows_ en uno solo, emitiendo los valores de todos a medida que llegan, sin preservar orden entre flujos**.  
Cada ``Flow`` se colecta **concurrentemente**, introduciendo un **_concurrency boundary_**.

A diferencia de [``flatMapMerge``](#flatmapmerge), **no transforma valores**: simplemente **fusiona emisiones existentes**.

📌 **Ejemplo**:  
Los valores se emiten en el orden en que cada ``Flow`` los produce.  
Cada flujo se ejecuta en su propia corrutina, y el _collector_ recibe los valores **tan pronto como están disponibles**, sin esperar a que los demás flujos emitan.

```kotlin
val flowA = flow {
    emit("A1")
    delay(30)
    emit("A2")
}

val flowB = flow {
    delay(10)
    emit("B1")
    emit("B2")
}

merge(flowA, flowB)
    .collect { println(it) }

// B1
// B2
// A1
// A2
```

### Operadores de *side-effects*, *lifecycle* y errores en *Flows*
> 👉 No cambian datos, **cambian comportamiento**. Ideales para _logging_, métricas, _setup_, _fallback_

#### [``cancellable``](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/cancellable.html)
Operador intermedio que **hace explícita la cooperación con la cancelación** durante la recolección del ``Flow``.  
Garantiza que el flujo **verifique el estado de cancelación entre emisiones**, incluso cuando el _upstream_ no tiene puntos de suspensión frecuentes.

No introduce _concurrency boundaries_ ni _buffers_: **solo afecta el comportamiento de cancelación**.

📌 **Ejemplo**:  
La recolección se detiene inmediatamente cuando se cancela la corrutina.  
Sin ``cancellable``, un flujo puramente CPU-bound podría continuar emitiendo valores hasta llegar a un punto de suspensión. Con ``cancellable``, la cancelación se respeta de forma cooperativa entre emisiones.

```kotlin
val job = CoroutineScope(Dispatchers.Default).launch {
    flow {
        repeat(1_000) {
            emit(it)
        }
    }
        .cancellable()
        .collect { value ->
            println(value)
        }
}

delay(10)
job.cancel()
```

#### [``catch``](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/catch.html)
Operador intermedio que **intercepta excepciones lanzadas en el flujo ascendente (_upstream_)**, permitiendo **manejar errores o emitir valores alternativos** sin cancelar la recolección. Es el equivalente reactivo de un bloque ``try / catch`` y suele combinarse con [``onCompletion``](#oncompletion) para lógica de cierre.

No captura excepciones del bloque ``collect`` ni excepciones de cancelación (``CancellationException``).

📌 **Ejemplo**:  
Si ocurre un error durante la emisión, se maneja y el flujo continúa con un valor de respaldo.

```kotlin
flow {
    emit(1)
    error("Algo falló")
}
    .catch { e ->
        emit(-1)
    }
    .collect {
        println(it)
    }

// 1
// -1
```

#### [``onCompletion``](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/on-completion.html)
Operador intermedio que **se ejecuta cuando el flujo termina**, ya sea por **finalización normal, error o cancelación**. es análogo a un bloque ``finally`` en un ``try / catch / finally``.  
Permite ejecutar lógica de limpieza, _logging_ o métricas **sin alterar los valores emitidos**.

Recibe como parámetro opcional la causa de finalización (``Throwable?``), que es ``null`` si el flujo terminó correctamente.

📌 **Ejemplo**:  
Se ejecuta siempre al finalizar el flujo, independientemente del motivo

```kotlin
flowOf(1, 2, 3)
    .onCompletion { cause ->
        println("Flow finalizado. Error = $cause")
    }
    .collect {
        println(it)
    }

// 1
// 2
// 3
// Flow finalizado. Error = null
```

#### [``onEach``](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/on-each.html)
Operador intermedio que **ejecuta una acción para cada valor emitido** sin modificar el flujo. Es útil para **_side-effects_**, como _logging_, métricas, guardar en _cache_, _debug_, etc.  
No detiene ni transforma los elementos, **simplemente los observa**.

📌 **Ejemplo**:  
Permite ejecutar un efecto secundario mientras los valores siguen fluyendo hacia ``collect``

```kotlin
flowOf(1, 2, 3)
    .onEach { println("Valor emitido: $it") }
    .collect { println("Procesando: $it") }

// Output:
// Valor emitido: 1
// Procesando: 1
// Valor emitido: 2
// Procesando: 2
// Valor emitido: 3
// Procesando: 3
```

#### [``onStart``](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/on-start.html)
Operador intermedio que **ejecuta una acción antes de que el _Flow_ comience a emitir valores**.  
Se usa para inicialización, emitir valores iniciales o realizar _side-effects_ previos al flujo principal.

📌 **Ejemplo**:  
Se ejecuta primero el bloque de ``onStart``, luego las emisiones originales del _Flow_.

```kotlin
flowOf(1, 2, 3)
    .onStart {
        println("Starting flow")
    }
    .collect {
        println(it)
    }

// Starting flow
// 1
// 2
// 3
```

### Control de ritmo (*time-based*) en *Flows*
> 👉 Cuándo se permite emitir

#### [``debounce``](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/debounce.html)
Operador intermedio que **filtra emisiones rápidas**, manteniendo solo el valor más reciente después de que haya pasado un tiempo sin nuevas emisiones.  
Se usa para **evitar procesar eventos demasiado frecuentes** (por ejemplo, búsquedas mientras se escribe).

📌 **Ejemplo**:  
Solo se emite el último valor si no llegan otros dentro del intervalo.

```kotlin
flow {
    emit(1)
    delay(50)
    emit(2)
    delay(200)
    emit(3)
}
    .debounce(100)
    .collect { println(it) }

// 2
// 3
```

#### [``sample``](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/sample.html)
Operador intermedio que **emite periódicamente el valor más reciente** del flujo, según un intervalo fijo.  
A diferencia de [``debounce``](#debounce), **no espera silencio**, sino que toma “muestras” del estado actual.

📌 **Ejemplo**:  
Se emite el último valor disponible en cada intervalo.

```kotlin
flow {
    repeat(5) {
        emit(it)
        delay(50)
    }
}
    .sample(100)
    .collect { println(it) }

// 1
// 3
// 4
```

### Operadores terminales de obtención de valor en *Flows*
> 👉 Consumir parcialmente el ``Flow``

#### [``first``](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/first.html)
Operador terminal que **obtiene el primer valor emitido** por el _Flow_ y **cancela la colección inmediatamente después**.  
Si el flujo está vacío, lanza una excepción (``NoSuchElementException``). Existe una variante con predicado.

📌 **Ejemplo**:  
Solo se consume la primera emisión.

```kotlin
val result = flowOf(10, 20, 30).first()

println(result)
// 10
```

#### [``single``](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/single.html)
Operador terminal que **obtiene el único valor emitido** por el _Flow_.  
Lanza excepción si el flujo está vacío o si emite más de un valor. Existe una variante con predicado.

📌 **Ejemplo**:  
El flujo debe emitir exactamente un valor.

```kotlin
val result = flowOf(42).single()

println(result)
// 42
```

### Lanzamiento, *scopes* y cancelación
> 👉 Cómo se ejecutan corrutinas

#### [``cancel``](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/cancel.html)
Función que **cancela una corrutina o un ``CoroutineScope``**, marcando su ``Job`` como cancelado.  
La cancelación es **cooperativa**: la corrutina se detiene en el próximo punto de suspensión (``delay``, ``collect``, ``yield``, etc.), lanzando internamente una ``CancellationException``.

📌 **Ejemplo**:  
Se cancela el ``Job``, deteniendo la corrutina antes de que termine.

```kotlin
val job = CoroutineScope(Dispatchers.Default).launch {
    repeat(5) {
        delay(100)
        println("Trabajo $it")
    }
}

delay(250)
job.cancel()

println("Cancelado")

// Trabajo 0
// Trabajo 1
// Cancelado
```

#### [``launch``](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/launch.html)
_Coroutine builder_ que **inicia una nueva corrutina** dentro de un ``CoroutineScope``.  
Se usa para tareas asíncronas que **no devuelven resultado** (retorna un ``Job``) y cuyo ciclo de vida queda ligado al ``scope``. Las excepciones no capturadas cancelan el ``Job`` padre (salvo que se use ``SupervisorJob``).

📌 **Ejemplo**:  
Se lanza una corrutina que se ejecuta en paralelo al hilo principal.

```kotlin
val scope = CoroutineScope(Dispatchers.Default)

val job = scope.launch {
    delay(100)
    println("Corrutina terminada")
}

println("Sigue el flujo principal")
job.join()

// Sigue el flujo principal
// Corrutina terminada
```

#### [``launchIn``](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.flow/launch-in.html)
Operador terminal que **inicia la recolección de un ``Flow`` dentro de un ``CoroutineScope``**, sin necesidad de llamar explícitamente a ``collect``.  
Se usa junto con operadores como ``onEach``, ``catch``, etc. Retorna un ``Job`` que representa la recolección activa.

📌 **Ejemplo**:  
El ``Flow`` comienza a ejecutarse en el ``scope`` y cada valor se maneja en ``onEach``.

```kotlin
val scope = CoroutineScope(Dispatchers.Main)

flowOf(1, 2, 3)
    .onEach { println(it) }
    .launchIn(scope)

// 1
// 2
// 3
```

### Sincronización y concurrencia
> 👉 Protección de estado compartido

#### [`Mutex`](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines.sync/-mutex.html)
Permite detener (suspender) la ejecución de corrutinas, a diferencia de los bloqueos tradicionales que son para hilos como [`synchronized`](#synchronized).  
Se utiliza a menudo en lugar de `synchronized` para manejar el acceso a recursos compartidos de manera más efectiva, ya que permite que las corrutinas sean "despertadas" una vez que se libera el bloqueo.

📌 **Ejemplo**:

```kotlin
val mutex = Mutex()

coroutineScope.launch {
    mutex.lock()
    try {
        // Código crítico
    } finally {
        mutex.unlock()
    }
}
```

#### [`synchronized()`](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin/synchronized.html)
Bloquea el acceso a una sección crítica del código en un contexto de múltiples hilos (*threads*). Esto asegura que **solo un hilo pueda ejecutar el bloque de código que está dentro de `synchronized` al mismo tiempo**. Se pueden sincronizar bloques de código dentro de métodos, funciones o incluso *lambdas*.  
Es útil cuando se necesita controlar el acceso a recursos compartidos desde múltiples hilos.

📌 **Ejemplo**:

```kotlin
private val lock = Any()

fun safeFunction() {
    synchronized(lock) {
        // Código crítico que debe ser ejecutado por un solo hilo a la vez
    }
}
```

#### [`@Synchronized`](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.jvm/-synchronized/)
Marca **métodos que necesitan ser ejecutados de manera sincronizada**, es decir, que **sólo un hilo puede ejecutarlo a la vez**.  
Al agregar `@Synchronized` a un método, Kotlin genera un bloqueo en un objeto interno (el objeto receptor del método) para que solo un hilo pueda ejecutarlo en un momento dado.

📌 **Ejemplo**:

```kotlin
@Synchronized
fun threadSafeMethod() {
    // Código seguro para varios hilos
}
```

#### [`@ThreadLocal`](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.native.concurrent/-thread-local/)
Marca **campos que deben tener un valor único para cada hilo**. Permite que **cada hilo tenga su propia copia de la variable**, lo que ayuda a evitar interferencias.  
Es útil para mantener variables que son específicas a un hilo sin interferencias entre hilos.

📌 **Ejemplo**:

```kotlin
@ThreadLocal
private var threadSpecificVariable: Int = 0
```

#### [`@Volatile`](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.concurrent/-volatile/)
Indica que **el valor de una variable puede ser modificado por varios hilos (threads)** y que la visibilidad de esa variable debe ser consistente entre ellos. También evita el cacheo de la variable para asegurar que siempre se lea el valor más reciente.  
Al marcar una variable como `@Volatile`, Kotlin asegura que cualquier operación de lectura/escritura en esa variable se refleje inmediatamente en todos los hilos, eliminando posibles problemas de _cache_ de CPU.

📌 **Ejemplo**:

```kotlin
@Volatile
var sharedResource: Int = 0
```

#### Comparativa
> ℹ️ **Nota:**  
> `@ThreadLocal` no comparte estado entre hilos, por lo que **Visibilidad** y **Exclusión mutua** no aplican (ver [Atomicidad](../Glosary%20&%20Core%20Concepts/Software%20in%20general.md#atomicidad-atomicity)).  
> En corrutinas, `@ThreadLocal` puede perder el valor si la corrutina cambia de hilo al reanudarse. Para ese caso existe `ThreadLocal.asContextElement()`.

| Mecanismo       | Visibilidad | Exclusión mutua | Estrategia                |
|-----------------|-------------|-----------------|---------------------------|
| `@Volatile`     | ✅           | ❌               | Coordinación (memoria)    |
| `synchronized`  | ✅           | ✅               | Coordinación (bloqueo)    |
| `@Synchronized` | ✅           | ✅               | Coordinación (bloqueo)    |
| `Mutex`         | ✅           | ✅               | Coordinación (suspensión) |
| `@ThreadLocal`  | N/A         | N/A             | Aislamiento               |

### Serialización y modelo de datos
> 👉 Persistencia / representación

#### [`@Transient`](https://kotlinlang.org/api/core/kotlin-stdlib/kotlin.jvm/-transient/)
Indica que **una propiedad de una clase no debe ser serializada**. Es útil para propiedades que no son relevantes para la persistencia de datos.  
Es común utilizarla en clases que implementan la interfaz `Serializable` y para propiedades sensibles, como contraseñas. Al marcar una propiedad como `@Transient`, se excluye de la serialización, lo que significa que no se guardará cuando el objeto sea convertido a un formato serializado.

📌 **Ejemplo**:

```kotlin
data class User(@Transient val password: String, val username: String)
```

### Primitivas de suspensión y cooperación en corrutinas
> 👉 Bajo nivel; base del modelo. Son las operaciones fundamentales que permiten que las corrutinas sean **asíncronas, cancelables y no bloqueantes**, y sobre las cuales se construyen los operadores de ``Flow`` y otras abstracciones de Kotlin

#### [``delay``](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/delay.html)
Función de suspensión que **pausa la corrutina sin bloquear el hilo** durante un tiempo determinado.  
Es un punto de **cancelación cooperativa**: si la corrutina es cancelada mientras está suspendida, lanza ``CancellationException``.

📌 **Ejemplo**:  

```kotlin
launch {
    println("Inicio")
    delay(1000)
    println("Fin")
}

// Inicio
// (~1s de diferencia)
// Fin
```

#### [``yield``](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/yield.html)
Función de suspensión que **cede voluntariamente la ejecución** para que otras corrutinas puedan correr.  
No implica necesariamente un cambio de hilo, pero sí le da al _scheduler_ la oportunidad de reordenar tareas.  
Es un punto de **cancelación cooperativa**: si la corrutina fue cancelada, lanza ``CancellationException``.

📌 **Ejemplo**:  

```kotlin
launch {
    repeat(3) { i ->
        println("A $i")
        yield()
    }
}

launch {
    repeat(3) { i ->
        println("B $i")
        yield()
    }
}

// Posible salida (puede variar):
// A 0
// B 0
// A 1
// B 1
// A 2
// B 2
```

#### [``async``](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/async.html) & [``await``](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-deferred/await.html)
- **``async``**: Lanza una corrutina que **produce un resultado futuro**. Devuelve un objeto ``Deferred<T>`` (una promesa de valor). 
- **``await``**: Función de suspensión que **espera el resultado del ``Deferred``**. Si aún no terminó, se suspende sin bloquear el hilo. 

Ambas forman la base del modelo **concurrente con resultados** en corrutinas.  
También son puntos de **cancelación cooperativa**: si el ``Deferred`` es cancelado, ``await()`` lanza ``CancellationException``.

📌 **Ejemplo**:  

```kotlin
launch {
    val deferred1 = async {
        delay(1000)
        10
    }

    val deferred2 = async {
        delay(500)
        20
    }

    val result = deferred1.await() + deferred2.await()
    println(result)
}

// (~1s de diferencia total, no 1.5s)
// 30
```

#### [``withContext``](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/with-context.html)
Función de suspensión que **cambia el contexto de la corrutina** (por ejemplo, el ``Dispatcher``) para ejecutar un bloque de código y **devuelve su resultado**.  
Se usa para **mover trabajo a otro hilo o _pool_** (I/O, CPU, etc.) sin romper la concurrencia estructurada.

A diferencia de [``async``](#async--await), **no crea una corrutina hija independiente**: simplemente **suspende la actual**, ejecuta el bloque en el nuevo contexto y luego vuelve al contexto original.

📌 **Ejemplo**:  

```kotlin
launch(Dispatchers.Main) {
    println("Main thread")

    val result = withContext(Dispatchers.IO) {
        println("Doing IO work")
        "Data loaded"
    }

    println(result)
}

// Main thread
// Doing IO work
// Data loaded
```

#### [``join``](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-job/join.html)
Función de suspensión que **espera a que una corrutina (``Job``) finalice** sin bloquear el hilo.  
Se usa cuando **solo interesa la finalización**, no un resultado (a diferencia de [``await``](#async--await)).

Respeta cancelación: si la corrutina que llama a ``join`` es cancelada, también se cancela la espera.

📌 **Ejemplo**:  

```kotlin
val job = launch {
    delay(500)
    println("Trabajo terminado")
}

println("Esperando...")
job.join()
println("Continúa ejecución")

// Esperando...
// (≈500 ms)
// Trabajo terminado
// Continúa ejecución
```

#### [``resume``](https://kotlinlang.org/api/kotlinx.coroutines/kotlinx-coroutines-core/kotlinx.coroutines/-cancellable-continuation/resume.html)
Función de bajo nivel que **reanuda una corrutina previamente suspendida**, entregándole un valor y continuando su ejecución desde el punto donde se había detenido. No se usa en código de alto nivel con ``Flow`` o ``suspend`` comunes, sino en **integraciones**, librerías o código de infraestructura.  
Se usa a través de una ``Continuation`` (normalmente ``CancellableContinuation``) y es clave para **adaptar APIs basadas en _callbacks_** al modelo de corrutinas. Es parte del mecanismo interno de suspensión:

``suspend`` **guarda el estado** (el compilador transforma la función en una **máquina de estados**, y la ``Continuation`` mantiene ese estado) :arrow_right: ``resume`` **lo restaura y continúa**

> ℹ️ **Nota:**  
> - ``resume`` **solo debe llamarse una vez** por continuación. 
> - Si la corrutina fue cancelada antes de reanudar, ``resume`` **puede fallar o ser ignorado** según el estado de la continuación.
> - Existe ``resumeWithException()`` para **reanudar señalando un error**.

📌 **Ejemplo**:  
Reanudar una corrutina desde un _callback_.

```kotlin
suspend fun fetchData(): String =
    suspendCancellableCoroutine { cont ->

        println("Esperando resultado")

        // Simula API async con callback
        Thread {
            Thread.sleep(300)
            cont.resume("DATA")
        }.start()
    }

launch {
    val result = fetchData()
    println(result)
}

// Esperando resultado
// (~300 ms)
// DATA
```

## *Testing* en corrutinas: ``StandardTestDispatcher`` y ``UnconfinedTestDispatcher``
> ℹ️ **Nota:**  
> Ver también [acá](../Android/Testing/Testing%20Tools.md#testing-con-corrutinas)

Ambos son *dispatchers* utilizados en pruebas de corrutinas en Kotlin, pero tienen comportamientos diferentes:

### ``StandardTestDispatcher``

1. **Ejecución controlada**: Las tareas se encolan pero no se ejecutan automáticamente. Se necesita llamar explícitamente a métodos como **`advanceUntilIdle()`** o **`advanceTimeBy()`** para que se ejecuten las tareas pendientes.
2. **Control de tiempo**: Permite controlar exactamente cuándo se ejecutan las tareas, lo que facilita probar comportamientos que dependen del tiempo.
3. **Determinismo**: Proporciona un comportamiento determinista y predecible, ideal para pruebas unitarias.
4. **Caso de uso ideal**: Pruebas donde se necesita verificar el orden exacto de ejecución o controlar precisamente cuándo se ejecutan las corrutinas.

📌 **Ejemplo**:

```kotlin
val testDispatcher = StandardTestDispatcher()
launch(testDispatcher) {
    // Este código no se ejecutará inmediatamente
    println("Tarea ejecutada")
}
// Necesitas llamar a esto para ejecutar la tarea
testDispatcher.scheduler.advanceUntilIdle()
```

### ``UnconfinedTestDispatcher``

1. **Ejecución inmediata**: Las tareas se ejecutan inmediatamente, sin necesidad de avanzar manualmente el tiempo.
2. **Comportamiento "ansioso" (*eager behavior*)**: Ejecuta las corrutinas tan pronto como se lanzan, hasta que alcanzan un punto de suspensión. La documentación de Kotlin también usa el término *"unconfined"* para describir este comportamiento, indicando que la ejecución no está confinada o restringida a un orden específico controlado por el programador, sino que fluye libremente.
3. **Simplicidad**: Más simple de usar cuando no se necesita controlar el tiempo de ejecución.
4. **Caso de uso ideal**: Pruebas donde solo interesa el resultado final y no el orden o tiempo de ejecución.

📌 **Ejemplo**:

```kotlin
val testDispatcher = UnconfinedTestDispatcher()
launch(testDispatcher) {
    // Este código se ejecutará inmediatamente
    println("Tarea ejecutada")
}
// No se necesita avanzar el tiempo manualmente
```

### Modelo de ``TestDispatcherRule`` (o ``MainCoroutineRule``)

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

## Referencias y Fuentes
- [Kotlin Docs - Coroutines guide](https://kotlinlang.org/docs/coroutines-guide.html)
- [Kotlin Docs - Asynchronous Flow](https://kotlinlang.org/docs/flow.html)
- [Kotlin Docs - Channels](https://kotlinlang.org/docs/channels.html)
- [Android Docs - Kotlin flows on Android](https://developer.android.com/kotlin/flow)
