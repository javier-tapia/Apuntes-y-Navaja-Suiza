<h1>Design Patterns with Kotlin</h1>

***Index***:
<!-- TOC -->
  * [*Singleton*](#singleton)
    * [Implementación](#implementación)
    * [Uso](#uso)
  * [*Builder*](#builder)
    * [Implementación](#implementación-1)
    * [Uso](#uso-1)
  * [*Abstract Factory*](#abstract-factory)
    * [Implementación](#implementación-2)
    * [Uso](#uso-2)
  * [*Factory Method*](#factory-method)
    * [Implementación](#implementación-3)
    * [Uso](#uso-3)
  * [*Adapter*](#adapter)
    * [Implementación](#implementación-4)
    * [Uso](#uso-4)
  * [*Facade*](#facade)
    * [Implementación](#implementación-5)
    * [Uso](#uso-5)
  * [*Decorator*](#decorator)
    * [Implementación](#implementación-6)
    * [Uso](#uso-6)
  * [*Strategy*](#strategy)
    * [Implementación](#implementación-7)
    * [Uso](#uso-7)
  * [*Observer*](#observer)
    * [Implementación](#implementación-8)
    * [Uso](#uso-8)
  * [*State*](#state)
    * [Implementación](#implementación-9)
    * [Uso](#uso-9)
<!-- TOC -->

---

## *Singleton*
### Implementación
```kotlin
// Singleton clásico orientado a Android
class DatabaseHelper private constructor(context: Context) {
    init {
        // Inicialización de la base de datos, por ejemplo:
        println("Base de datos inicializada con: $context")
    }

    // ============================================================
    // 1️⃣ Opción con `synchronized` - Bloquea el hilo actual. Es suficiente para garantizar exclusión mutua en entornos sin concurrencia por corutinas.
    // ============================================================
    companion object {
        @Volatile
        private var instance: DatabaseHelper? = null

        fun getInstance(context: Context): DatabaseHelper =
            instance ?: synchronized(this) {
                instance ?: DatabaseHelper(context.applicationContext).also { instance = it }
            }
    }

    // ============================================================
    // 2️⃣ Opción con `Mutex` - Evita bloqueos de hilo. Suspende la coroutine mientras otra tiene el bloqueo. Ideal para Android y apps con Dispatchers.IO, ViewModelScope, etc.
    // ============================================================
    companion object {
        @Volatile
        private var instance: DatabaseHelper? = null
        private val mutex = Mutex()

        suspend fun getInstance(context: Context): DatabaseHelper {
            return instance ?: mutex.withLock {
                instance ?: DatabaseHelper(context.applicationContext).also { instance = it }
            }
        }
    }
}
```

### Uso
```kotlin
val db = DatabaseHelper.getInstance(context)
```

## *Builder*
### Implementación
```kotlin
// ============================================================
// 1️⃣ Builder clásico (patrón tradicional / multiplataforma)
// ============================================================
data class Notification(
    val title: String?,
    val message: String?,
    val icon: Int?,
    val isPersistent: Boolean
)

class NotificationBuilder {
    private var title: String? = null
    private var message: String? = null
    private var icon: Int? = null
    private var isPersistent: Boolean = false

    fun setTitle(title: String) = apply { this.title = title }
    fun setMessage(message: String) = apply { this.message = message }
    fun setIcon(icon: Int) = apply { this.icon = icon }
    fun setPersistent(persistent: Boolean) = apply { this.isPersistent = persistent }

    fun build(): Notification = Notification(title, message, icon, isPersistent)
}

// ============================================================
// 2️⃣ Builder con copy() (aprovechando data class inmutable)
// ============================================================
data class NotificationCopy(
    val title: String? = null,
    val message: String? = null,
    val icon: Int? = null,
    val isPersistent: Boolean = false
) {
    fun withTitle(title: String) = copy(title = title)
    fun withMessage(message: String) = copy(message = message)
    fun withIcon(icon: Int) = copy(icon = icon)
    fun withPersistent(persistent: Boolean) = copy(isPersistent = persistent)
}

// ============================================================
// 3️⃣ Builder DSL-style (lambda con receptor / idiomático Kotlin)
// ============================================================
data class NotificationDSL(
    var title: String? = null,
    var message: String? = null,
    var icon: Int? = null,
    var isPersistent: Boolean = false
)

// Función DSL
// Alternativamente, se puede extraer como: `typealias NotificationBlock = NotificationDSL.() -> Unit`
fun notification(block: NotificationDSL.() -> Unit): NotificationDSL {
    return NotificationDSL().apply(block)
}
```

### Uso
```kotlin
// ============================================================
// 1️⃣ Builder clásico
// ============================================================
val notificationClassic = NotificationBuilder()
    .setTitle("Nueva tarea")
    .setMessage("Tienes una tarea pendiente")
    .setIcon(android.R.drawable.ic_dialog_info)
    .setPersistent(true)
    .build()

// ============================================================
// 2️⃣ Builder con copy()
// ============================================================
val notificationCopy = NotificationCopy()
    .withTitle("Nueva tarea")
    .withMessage("Tienes una tarea pendiente")
    .withIcon(android.R.drawable.ic_dialog_info)
    .withPersistent(true)

// ============================================================
// 3️⃣ Builder DSL-style
// ============================================================
val notificationDSL = notification {
    title = "Nueva tarea"
    message = "Tienes una tarea pendiente"
    icon = android.R.drawable.ic_dialog_info
    isPersistent = true
}
```

## *Abstract Factory*
### Implementación
```kotlin
// ============================================================
// 1️⃣ Implementación clásica (estructural)
// ============================================================
// Productos abstractos
interface Button {
    fun render(): String
}

interface Checkbox {
    fun render(): String
}

// Productos concretos
class LightButton : Button {
    override fun render() = "Renderizando botón claro"
}

class DarkButton : Button {
    override fun render() = "Renderizando botón oscuro"
}

class LightCheckbox : Checkbox {
    override fun render() = "Renderizando checkbox claro"
}

class DarkCheckbox : Checkbox {
    override fun render() = "Renderizando checkbox oscuro"
}

// Fábrica abstracta
interface UIFactory {
    fun createButton(): Button
    fun createCheckbox(): Checkbox
}

// Fábricas concretas
class LightUIFactory : UIFactory {
    override fun createButton(): Button = LightButton()
    override fun createCheckbox(): Checkbox = LightCheckbox()
}

class DarkUIFactory : UIFactory {
    override fun createButton(): Button = DarkButton()
    override fun createCheckbox(): Checkbox = DarkCheckbox()
}

// ============================================================
// 2️⃣ Versión idiomática (uso de lambdas en lugar de subclases)
// ============================================================
// Alternativamente, se pueden extraer como: 
// `typealias ButtonCreator = () -> Button` y `typealias CheckboxCreator = () -> Checkbox`
class LambdaUIFactory(
    private val buttonCreator: () -> Button,
    private val checkboxCreator: () -> Checkbox
) : UIFactory {
    override fun createButton(): Button = buttonCreator()
    override fun createCheckbox(): Checkbox = checkboxCreator()
}

// ============================================================
// 3️⃣ Versión DSL-style (más declarativa y expresiva)
// ============================================================
fun uiFactory(block: UIFactoryScope.() -> Unit): UIFactory =
    UIFactoryScope().apply(block).build()

class UIFactoryScope {
    private var theme: String = "light"

    fun theme(theme: String) = apply { this.theme = theme }

    fun build(): UIFactory = when (theme.lowercase()) {
        "dark" -> DarkUIFactory()
        else -> LightUIFactory()
    }
}
```

### Uso
```kotlin
// ============================================================
// 1️⃣ Uso clásico
// ============================================================
val factoryClassic: UIFactory = LightUIFactory()
val buttonClassic = factoryClassic.createButton()
val checkboxClassic = factoryClassic.createCheckbox()
println(buttonClassic.render())   // Renderizando botón claro
println(checkboxClassic.render()) // Renderizando checkbox claro

// ============================================================
// 2️⃣ Uso idiomático (con lambdas)
// ============================================================
val factoryLambda = LambdaUIFactory(
    buttonCreator = { DarkButton() },
    checkboxCreator = { DarkCheckbox() }
)
val buttonLambda = factoryLambda.createButton()
val checkboxLambda = factoryLambda.createCheckbox()
println(buttonLambda.render())   // Renderizando botón oscuro
println(checkboxLambda.render()) // Renderizando checkbox oscuro

// ============================================================
// 3️⃣ Uso DSL-style
// ============================================================
val factoryDSL = uiFactory {
    theme("dark")
}
val buttonDSL = factoryDSL.createButton()
val checkboxDSL = factoryDSL.createCheckbox()
println(buttonDSL.render())   // Renderizando botón oscuro
println(checkboxDSL.render()) // Renderizando checkbox oscuro
```

## *Factory Method*
### Implementación
```kotlin
// ============================================================
// 1️⃣ Implementación clásica con subclases
// ============================================================
// Producto abstracto
interface Notification {
    fun send()
}

// Productos concretos
class EmailNotification(private val message: String) : Notification {
    override fun send() = println("📧 Enviando email: $message")
}

class PushNotification(private val message: String) : Notification {
    override fun send() = println("📲 Mostrando notificación push: $message")
}

// Creador abstracto
abstract class NotificationFactory {
    abstract fun createNotification(message: String): Notification
}

// Creadores concretos
class EmailNotificationFactory : NotificationFactory() {
    override fun createNotification(message: String): Notification = EmailNotification(message)
}

class PushNotificationFactory : NotificationFactory() {
    override fun createNotification(message: String): Notification = PushNotification(message)
}

// ============================================================
// 2️⃣ Versión idiomática - En lugar de heredar y sobrescribir `createProduct()`, se pasa una _lambda_ que cumple el mismo rol
// ============================================================
// Alternativamente, se puede extraer como: `typealias NotificationCreator = (String) -> Notification`
class LambdaNotificationFactory(
    private val creator: (String) -> Notification
) {
    fun create(message: String): Notification = creator(message)
}

// ============================================================
// 3️⃣ Versión DSL-style (más expresiva y fluida)
// ============================================================
fun notification(block: NotificationCreator.() -> Unit): Notification =
    NotificationCreator().apply(block).build()

class NotificationCreator {
    private var type: String = "push"
    private var message: String = ""

    fun type(type: String) = apply { this.type = type }
    fun message(message: String) = apply { this.message = message }

    fun build(): Notification = when (type.lowercase()) {
        "email" -> EmailNotification(message)
        else -> PushNotification(message)
    }
}
```

### Uso
```kotlin
// --- Uso común en Android ---
// Podría usarse dentro de un ViewModel o UseCase, por ejemplo.
// ============================================================
// 1️⃣ Usando fábricas concretas
// ============================================================
val emailFactory = EmailNotificationFactory()
val pushFactory = PushNotificationFactory()

val email = emailFactory.createNotification("Nuevo correo recibido")
val push = pushFactory.createNotification("Tienes una nueva tarea pendiente")

email.send() // 📧 Enviando email: Nuevo correo recibido
push.send() // 📲 Mostrando notificación push: Tienes una nueva tarea pendiente

// ============================================================
// 2️⃣ Usando lambda factory (más conciso)
// ============================================================
val lambdaFactory = LambdaNotificationFactory { message ->
    if ("@" in message) EmailNotification(message)
    else PushNotification(message)
}

val lambdaNotification = lambdaFactory.create("Recordatorio diario")
lambdaNotification.send() // 📲 Mostrando notificación push: Recordatorio diario

// ============================================================
// 3️⃣ Usando DSL-style builder
// ============================================================
val dslNotification = notification {
    type("email")
    message("Reporte mensual disponible")
}
dslNotification.send() // 📧 Enviando email: Reporte mensual disponible
```

## *Adapter*
### Implementación
```kotlin
// ============================================================
// 1️⃣ Implementación clásica (estructura tradicional)
// ============================================================
// Target
interface MediaPlayer {
    fun play(fileName: String)
}

// Adaptee (librería externa)
class AdvancedMediaPlayer {
    fun playFile(filePath: String) {
        println("Reproduciendo archivo avanzado: $filePath")
    }
}

// Adapter
class MediaPlayerAdapter(
    private val advancedPlayer: AdvancedMediaPlayer
) : MediaPlayer {

    override fun play(fileName: String) {
        val formatted = "/storage/media/$fileName"
        advancedPlayer.playFile(formatted)
    }
}

// ============================================================
// 2️⃣ Versión idiomática (Adapter como lambda wrapper)
// ============================================================
fun interface SimpleMediaPlayer {
    fun play(fileName: String)
}

class AdvancedMediaPlayerV2 {
    fun playFile(filePath: String) {
        println("SDK reproduciendo: $filePath")
    }
}

fun advancedPlayerAdapter(player: AdvancedMediaPlayerV2): SimpleMediaPlayer =
    SimpleMediaPlayer { fileName ->
        player.playFile("/sdk/media/$fileName")
    }

// ============================================================
// 3️⃣ DSL-style Adapter (configurable)
// ============================================================
fun mediaPlayerAdapter(block: MediaAdapterScope.() -> Unit): MediaPlayer =
    MediaAdapterScope().apply(block).build()

class MediaAdapterScope {
    private var basePath: String = ""
    private var player: AdvancedMediaPlayer? = null

    fun basePath(path: String) = apply { basePath = path }
    fun adaptee(player: AdvancedMediaPlayer) = apply { this.player = player }

    fun build(): MediaPlayer = object : MediaPlayer {
        override fun play(fileName: String) {
            val formatted = "$basePath/$fileName"
            player?.playFile(formatted)
                ?: error("AdvancedMediaPlayer no definido")
        }
    }
}
```

### Uso
```kotlin
// ============================================================
// 1️⃣ Uso clásico
// ============================================================
val classicPlayer: MediaPlayer =
    MediaPlayerAdapter(AdvancedMediaPlayer())

classicPlayer.play("song.mp3") // Reproduciendo archivo avanzado: /storage/media/song.mp3

// ============================================================
// 2️⃣ Uso idiomático (lambda adapter)
// ============================================================
val simplePlayer = advancedPlayerAdapter(AdvancedMediaPlayerV2())
simplePlayer.play("podcast.mp3") // SDK reproduciendo: /sdk/media/podcast.mp3

// ============================================================
// 3️⃣ Uso DSL-style
// ============================================================
val dslPlayer = mediaPlayerAdapter {
    basePath("/dsl/media")
    adaptee(AdvancedMediaPlayer())
}

dslPlayer.play("audiobook.mp3") // Reproduciendo archivo avanzado: /dsl/media/audiobook.mp3
```

## *Facade*
### Implementación
```kotlin
// ============================================================
// 1️⃣ Implementación clásica (Object-Oriented tradicional)
// ============================================================
class AudioService {
    fun load(file: String) = println("AudioService → cargando audio $file")
}

class VideoService {
    fun load(file: String) = println("VideoService → cargando video $file")
}

class RenderService {
    fun draw() = println("RenderService → dibujando en pantalla")
}

class MediaFacade(
    private val audio: AudioService,
    private val video: VideoService,
    private val render: RenderService
) {
    fun play(file: String) {
        audio.load(file)
        video.load(file)
        render.draw()
    }
}

// ============================================================
// 2️⃣ Implementación idiomática (funciones como subsistema)
// ============================================================
class MediaFacadeV2(
    private val loadAudio: (String) -> Unit,
    private val loadVideo: (String) -> Unit,
    private val render: () -> Unit
) {
    fun play(file: String) {
        loadAudio(file)
        loadVideo(file)
        render()
    }
}

// ============================================================
// 3️⃣ Implementación DSL-style (configurable)
// ============================================================
class MediaFacadeDsl private constructor(
    private val audio: AudioService,
    private val video: VideoService,
    private val render: RenderService,
    private val basePath: String
) {
    fun play(file: String) {
        val fullPath = "$basePath/$file"
        audio.load(fullPath)
        video.load(fullPath)
        render.draw()
    }

    class Builder {
        var audio: AudioService = AudioService()
        var video: VideoService = VideoService()
        var render: RenderService = RenderService()
        var basePath: String = "/media"

        fun build() = MediaFacadeDsl(audio, video, render, basePath)
    }
}

fun mediaFacade(block: MediaFacadeDsl.Builder.() -> Unit): MediaFacadeDsl =
    MediaFacadeDsl.Builder().apply(block).build()
```

### Uso
```kotlin
// ============================================================
// 1️⃣ Uso clásico
// ============================================================
val facade = MediaFacade(AudioService(), VideoService(), RenderService())
facade.play("movie.mp4")

// AudioService → cargando audio movie.mp4
// VideoService → cargando video movie.mp4
// RenderService → dibujando en pantalla


// ============================================================
// 2️⃣ Uso idiomático (subsistema como lambdas)
// ============================================================
val facadeV2 = MediaFacadeV2(
    loadAudio = { println("λAudio → $it") },
    loadVideo = { println("λVideo → $it") },
    render = { println("λRender → pintando frame") }
)

facadeV2.play("trailer.mp4")

// λAudio → trailer.mp4
// λVideo → trailer.mp4
// λRender → pintando frame

// ============================================================
// 3️⃣ Uso DSL-style
// ============================================================
val dslFacade = mediaFacade {
    basePath = "/dsl/content"
}

dslFacade.play("documentary.mp4")

// AudioService → cargando audio /dsl/content/documentary.mp4
// VideoService → cargando video /dsl/content/documentary.mp4
// RenderService → dibujando en pantalla
```

## *Decorator*
### Implementación
```kotlin
// ============================================================
// 1️⃣ Implementación clásica (Object-Oriented tradicional)
// ============================================================
interface TextProcessor {
    fun process(text: String): String
}

class PlainTextProcessor : TextProcessor {
    override fun process(text: String) = text
}

open class TextDecorator(protected val wrappee: TextProcessor) : TextProcessor {
    override fun process(text: String) = wrappee.process(text)
}

class TrimDecorator(processor: TextProcessor) : TextDecorator(processor) {
    override fun process(text: String) = super.process(text).trim()
}

class UppercaseDecorator(processor: TextProcessor) : TextDecorator(processor) {
    override fun process(text: String) = super.process(text).uppercase()
}


// ============================================================
// 2️⃣ Implementación idiomática (funciones como componente)
// ============================================================
// ``typealias`` permite dar nombre semántico a un tipo complejo (como una función), 
// ayudando a modelar roles de un patrón sin introducir nuevas clases
typealias TextOp = (String) -> String

fun trimDecorator(op: TextOp): TextOp = { text ->
    op(text).trim()
}

fun uppercaseDecorator(op: TextOp): TextOp = { text ->
    op(text).uppercase()
}


// ============================================================
// 3️⃣ Implementación DSL-style
// ============================================================
class TextPipelineBuilder {
    private val decorators = mutableListOf<TextOp>()

    fun trim() {
        decorators += { it.trim() }
    }

    fun uppercase() {
        decorators += { it.uppercase() }
    }

    fun build(): TextOp = { text ->
        decorators.fold(text) { acc, op -> op(acc) }
    }
}

fun textPipeline(block: TextPipelineBuilder.() -> Unit): TextOp {
    val builder = TextPipelineBuilder()
    builder.block()
    return builder.build()
}
```

### Uso
```kotlin
// ============================================================
// 1️⃣ Uso clásico
// ============================================================
val classicProcessor: TextProcessor =
    UppercaseDecorator(
        TrimDecorator(
            PlainTextProcessor()
        )
    )

val result1 = classicProcessor.process("  hola mundo  ")
println(result1) // HOLA MUNDO


// ============================================================
// 2️⃣ Uso idiomático
// ============================================================
val base: TextOp = { it }
val processor = uppercaseDecorator(trimDecorator(base))

val result2 = processor("  kotlin decorator  ")
println(result2)  // KOTLIN DECORATOR


// ============================================================
// 3️⃣ Uso DSL-style
// ============================================================
val pipeline = textPipeline {
    trim()
    uppercase()
}

val result3 = pipeline("  patrones de diseño  ")
println(result3) // PATRONES DE DISEÑO
```

## *Strategy*
### Implementación
```kotlin
// ============================================================
// 1️⃣ Implementación clásica (Object-Oriented tradicional)
// ============================================================
interface PricingStrategy {
    fun calculate(price: Double): Double
}

class RegularPricing : PricingStrategy {
    override fun calculate(price: Double) = price
}

class DiscountPricing(private val discount: Double) : PricingStrategy {
    override fun calculate(price: Double) = price * (1 - discount)
}

class Checkout(
    private var strategy: PricingStrategy
) {
    fun setStrategy(strategy: PricingStrategy) {
        this.strategy = strategy
    }

    fun total(price: Double): Double = strategy.calculate(price)
}


// ============================================================
// 2️⃣ Implementación idiomática (funciones como estrategia)
// ============================================================
// ``typealias`` permite dar nombre semántico a un tipo complejo (como una función), 
// ayudando a modelar roles de un patrón sin introducir nuevas clases
typealias PricingStrategyFn = (Double) -> Double

fun regularPricing(): PricingStrategyFn = { it }

fun discountPricing(discount: Double): PricingStrategyFn = { price ->
    price * (1 - discount)
}


// ============================================================
// 3️⃣ Implementación DSL-style
// ============================================================
class PricingStrategyBuilder {
    private var discount: Double = 0.0

    fun discount(value: Double) {
        discount = value
    }

    fun build(): PricingStrategyFn = { price ->
        price * (1 - discount)
    }
}

fun pricingStrategy(block: PricingStrategyBuilder.() -> Unit): PricingStrategyFn {
    val builder = PricingStrategyBuilder()
    builder.block()
    return builder.build()
}
```

### Uso
```kotlin
// ============================================================
// 1️⃣ Uso clásico
// ============================================================
val checkout = Checkout(RegularPricing())

println(checkout.total(100.0)) // 100.0

checkout.setStrategy(DiscountPricing(0.2))
println(checkout.total(100.0)) // 80.0


// ============================================================
// 2️⃣ Uso idiomático
// ============================================================
var strategyFn: PricingStrategyFn = regularPricing()
println(strategyFn(200.0)) // 200.0

strategyFn = discountPricing(0.1)
println(strategyFn(200.0)) // 180.0


// ============================================================
// 3️⃣ Uso DSL-style
// ============================================================
val blackFridayStrategy = pricingStrategy {
    discount(0.5)
}

println(blackFridayStrategy(300.0)) // 150.0
```

## *Observer*
### Implementación
```kotlin
// ============================================================
// 1️⃣ Implementación clásica (Object-Oriented tradicional)
// ============================================================
interface EventListener<T> {
    fun onEvent(data: T)
}

class EventPublisher<T> {
    private val listeners = mutableSetOf<EventListener<T>>()

    fun subscribe(listener: EventListener<T>) {
        listeners += listener
    }

    fun unsubscribe(listener: EventListener<T>) {
        listeners -= listener
    }

    fun notify(data: T) {
        listeners.forEach { it.onEvent(data) }
    }
}


// ============================================================
// 2️⃣ Implementación idiomática (funciones como observers)
// ============================================================
// ``typealias`` permite dar nombre semántico a un tipo complejo (como una función), 
// ayudando a modelar roles de un patrón sin introducir nuevas clases
typealias Observer<T> = (T) -> Unit

class EventBus<T> {
    private val observers = mutableSetOf<Observer<T>>()

    fun subscribe(observer: Observer<T>) {
        observers += observer
    }

    fun unsubscribe(observer: Observer<T>) {
        observers -= observer
    }

    fun emit(data: T) {
        observers.forEach { it(data) }
    }
}


// ============================================================
// 3️⃣ Implementación DSL-style
// ============================================================
class ObservableBuilder<T> {
    private val observers = mutableListOf<Observer<T>>()

    fun onEvent(block: Observer<T>) {
        observers += block
    }

    fun build(): (T) -> Unit = { data ->
        observers.forEach { it(data) }
    }
}

fun <T> observable(block: ObservableBuilder<T>.() -> Unit): (T) -> Unit {
    val builder = ObservableBuilder<T>()
    builder.block()
    return builder.build()
}
```

### Uso
```kotlin
// ============================================================
// 1️⃣ Uso clásico
// ============================================================
val publisher = EventPublisher<String>()

val emailListener = object : EventListener<String> {
    override fun onEvent(data: String) {
        println("Email recibido: $data")
    }
}

publisher.subscribe(emailListener)
publisher.notify("Pedido enviado") // Email recibido: Pedido enviado


// ============================================================
// 2️⃣ Uso idiomático
// ============================================================
val bus = EventBus<Int>()

val logger: Observer<Int> = { println("Log: valor = $it") }
bus.subscribe(logger)

bus.emit(42) // Log: valor = 42


// ============================================================
// 3️⃣ Uso DSL-style
// ============================================================
val notifier = observable<String> {
    onEvent { println("Listener A: $it") }
    onEvent { println("Listener B: $it") }
}

notifier("Actualización disponible")

// Listener A: Actualización disponible
// Listener B: Actualización disponible
```

## *State*
### Implementación
```kotlin

```

### Uso
```kotlin

```
