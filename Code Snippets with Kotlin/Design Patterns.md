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
// Productos abstractos
interface Button {
    fun render(): String
}

interface Checkbox {
    fun render(): String
}

// Variantes concretas
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

// Cliente
class SettingsScreen(private val factory: UIFactory) {
    private val button = factory.createButton()
    private val checkbox = factory.createCheckbox()

    fun renderUI() {
        println(button.render())
        println(checkbox.render())
    }
}
```

### Uso
```kotlin
val factory: UIFactory = DarkUIFactory()
val screen = SettingsScreen(factory)
screen.renderUI()
```

## *Factory Method*
### Implementación
```kotlin

```

### Uso
```kotlin

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
