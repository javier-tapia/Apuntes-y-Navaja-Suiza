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
println(buttonClassic.render())   // Esto se imprime: Renderizando botón claro
println(checkboxClassic.render()) // Esto se imprime: Renderizando checkbox claro

// ============================================================
// 2️⃣ Uso idiomático (con lambdas)
// ============================================================
val factoryLambda = LambdaUIFactory(
    buttonCreator = { DarkButton() },
    checkboxCreator = { DarkCheckbox() }
)
val buttonLambda = factoryLambda.createButton()
val checkboxLambda = factoryLambda.createCheckbox()
println(buttonLambda.render())   // Esto se imprime: Renderizando botón oscuro
println(checkboxLambda.render()) // Esto se imprime: Renderizando checkbox oscuro

// ============================================================
// 3️⃣ Uso DSL-style
// ============================================================
val factoryDSL = uiFactory {
    theme("dark")
}
val buttonDSL = factoryDSL.createButton()
val checkboxDSL = factoryDSL.createCheckbox()
println(buttonDSL.render())   // Esto se imprime: Renderizando botón oscuro
println(checkboxDSL.render()) // Esto se imprime: Renderizando checkbox oscuro
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
// 2️⃣ Versión idiomática - En lugar de heredar y sobrescribir `createProduct()`, se pasa una **lambda** que cumple el mismo rol
// ============================================================
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

```

### Uso
```kotlin

```

## *Facade*
### Implementación
```kotlin

```

### Uso
```kotlin

```

## *Decorator*
### Implementación
```kotlin

```

### Uso
```kotlin

```

## *Strategy*
### Implementación
```kotlin

```

### Uso
```kotlin

```

## *Observer*
### Implementación
```kotlin

```

### Uso
```kotlin

```

## *State*
### Implementación
```kotlin

```

### Uso
```kotlin

```
