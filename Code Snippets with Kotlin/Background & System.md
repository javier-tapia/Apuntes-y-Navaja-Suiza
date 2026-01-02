<h1><i>Background & System</i></h1>

***Index***:
<!-- TOC -->
  * [*WorkManager*](#workmanager)
    * [Dependencias & *Plugins* (opcional)](#dependencias--plugins-opcional)
    * [Implementación con DI (*Hilt*)](#implementación-con-di-hilt)
      * [1. Configurar el `WorkerFactory`](#1-configurar-el-workerfactory)
      * [2. Crear el *Worker*](#2-crear-el-worker)
      * [3. Crear el *Scheduler*](#3-crear-el-scheduler)
    * [Implementación sin DI](#implementación-sin-di)
      * [1. Crear el *Worker*](#1-crear-el-worker)
    * [Uso con DI (*Hilt*)](#uso-con-di-hilt)
      * [Configurar restricciones y programar tarea](#configurar-restricciones-y-programar-tarea)
    * [Uso sin DI](#uso-sin-di)
      * [Configurar restricciones y programar tarea](#configurar-restricciones-y-programar-tarea-1)
  * [*Services*](#services)
    * [Dependencias & *Plugins* (opcional)](#dependencias--plugins-opcional-1)
    * [Implementación: *Foreground Service*](#implementación-foreground-service)
      * [1. Declarar servicio y permisos en el *Manifest*](#1-declarar-servicio-y-permisos-en-el-manifest)
      * [2. Crear el servicio](#2-crear-el-servicio)
    * [Uso: Iniciar el *Foreground Service*](#uso-iniciar-el-foreground-service)
    * [Implementación: *Bound Service*](#implementación-bound-service)
      * [1. Crear el servicio](#1-crear-el-servicio)
      * [2. Declarar servicio en el Manifest](#2-declarar-servicio-en-el-manifest)
    * [Uso: Conectar el *Bound Service*](#uso-conectar-el-bound-service)
  * [*AlarmManager*](#alarmmanager)
    * [Implementación](#implementación)
      * [1. Declarar receptor y permisos en el *Manifest*](#1-declarar-receptor-y-permisos-en-el-manifest)
      * [2. Crear el *BroadcastReceiver* de la alarma](#2-crear-el-broadcastreceiver-de-la-alarma)
    * [Uso: Programar la alarma](#uso-programar-la-alarma)
  * [*BroadcastReceiver*](#broadcastreceiver)
    * [Implementación: Registro estático](#implementación-registro-estático)
      * [1. Declarar receptor en el *Manifest*](#1-declarar-receptor-en-el-manifest)
      * [2. Definir el Receptor](#2-definir-el-receptor)
    * [Implementación: Registro dinámico](#implementación-registro-dinámico)
    * [Uso: Registro dinámico](#uso-registro-dinámico)
  * [*Notifications*](#notifications)
    * [Dependencias & *Plugins* (opcional)](#dependencias--plugins-opcional-2)
    * [Implementación](#implementación-1)
      * [1. Declarar permiso en el *Manifest*](#1-declarar-permiso-en-el-manifest)
      * [2. Crear el Canal de Notificación](#2-crear-el-canal-de-notificación)
    * [Uso: Construir y mostrar la notificación](#uso-construir-y-mostrar-la-notificación)
  * [*Deep Links*](#deep-links)
    * [Implementación](#implementación-2)
    * [Uso: Invocar *deeplink*](#uso-invocar-deeplink)
      * [1. Desde código (App :arrow_right: App)](#1-desde-código-app-arrow_right-app)
      * [2. Desde otra app o navegador](#2-desde-otra-app-o-navegador)
      * [3. Desde ``adb`` (testing / QA)](#3-desde-adb-testing--qa)
      * [Recibir y procesar el *Deep Link* en la *Activity*](#recibir-y-procesar-el-deep-link-en-la-activity)
  * [*App Links*](#app-links)
    * [Implementación](#implementación-3)
    * [Uso: Invocar *app link*](#uso-invocar-app-link)
      * [1. Desde código (App :arrow_right: App)](#1-desde-código-app-arrow_right-app-1)
      * [2. Desde navegador (caso principal de uso)](#2-desde-navegador-caso-principal-de-uso)
      * [3. Desde otra app (ej. email, _WhatsApp_, etc.)](#3-desde-otra-app-ej-email-_whatsapp_-etc)
      * [4. Desde ``adb`` (testing / QA)](#4-desde-adb-testing--qa)
      * [Recibir y procesar el *App Link* en la *Activity*](#recibir-y-procesar-el-app-link-en-la-activity)
  * [*Shortcuts*](#shortcuts)
    * [Implementación & Uso: *Shortcut* dinámico](#implementación--uso-shortcut-dinámico)
    * [Implementación: *Shortcut* estático](#implementación-shortcut-estático)
      * [1. Declarar el *Shortcut*](#1-declarar-el-shortcut)
      * [2. Registrar el *Shortcut* en el *Manifest*](#2-registrar-el-shortcut-en-el-manifest)
    * [Uso: Recibir el *Shortcut* estático en la *Activity*](#uso-recibir-el-shortcut-estático-en-la-activity)
  * [*In-App Updates*](#in-app-updates)
    * [Dependencias y *Plugins*](#dependencias-y-plugins)
    * [Implementación & Uso: Verificar y lanzar la actualización](#implementación--uso-verificar-y-lanzar-la-actualización)
      * [Modo Inmediato](#modo-inmediato)
      * [Modo Flexible con monitoreo de estado](#modo-flexible-con-monitoreo-de-estado)
  * [*In-App Reviews*](#in-app-reviews)
    * [Dependencias y *Plugins*](#dependencias-y-plugins-1)
    * [Implementación & Uso: Solicitar y lanzar el flujo de reseña](#implementación--uso-solicitar-y-lanzar-el-flujo-de-reseña)
  * [*Widgets*](#widgets)
    * [Implementación](#implementación-4)
      * [1. Crear el *AppWidgetProvider*](#1-crear-el-appwidgetprovider)
      * [2. Definir configuración del *Widget*](#2-definir-configuración-del-widget)
      * [3. Declarar receptor en el *Manifest*](#3-declarar-receptor-en-el-manifest)
    * [Uso: Actualizar *Widget* manualmente](#uso-actualizar-widget-manualmente)
<!-- TOC -->

---

## *WorkManager*
### Dependencias & *Plugins* (opcional)
No es necesario usar inyección de dependencias para _WorkManager_. Pero, en caso que se quiera utilizar _Hilt_, se debe agregar al proyecto:

1. En el archivo `libs.versions.toml`:

```kotlin
[versions]
workManager = "<VERSION>"
hilt = "<VERSION>"
ksp = "<VERSION>"

[libraries]
work-manager = { module = "androidx.work:work-runtime-ktx", version.ref = "workManager" }
hilt-android = { module = "com.google.dagger:hilt-android", version.ref = "hilt" }
hilt-compiler = { module = "com.google.dagger:hilt-compiler", version.ref = "hilt" }
hilt-work = { module = "androidx.hilt:hilt-work", version.ref = "hilt" }

[plugins]
hilt = { id = "com.google.dagger.hilt.android", version.ref = "hilt" }
ksp = { id = "com.google.devtools.ksp", version.ref = "ksp" }
```

2. En el archivo `build.gradle.kts` del proyecto (*root*):

```kotlin
plugins {
    alias(libs.plugins.hilt) apply false
    alias(libs.plugins.ksp) apply false
}
```

3. En el archivo `build.gradle.kts` del módulo que lo requiera:

```kotlin
plugins {
    alias(libs.plugins.hilt)
    alias(libs.plugins.ksp)
}

dependencies {
    // WorkManager
    implementation(libs.work.manager)

    // DI
    implementation(libs.hilt.android)
    implementation(libs.hilt.work)
    ksp(libs.hilt.compiler)
}
```

### Implementación con DI (*Hilt*)
#### 1. Configurar el `WorkerFactory`
> ℹ️ **Nota:**  
> Este paso solo es necesario si se usan _Workers_ con constructor inyectado.

_WorkManager_ **no puede instanciar _Workers_ con constructores personalizados** sin un `WorkerFactory`. Al usar _Hilt_, es obligatorio proveer `HiltWorkerFactory` desde la clase `Application`.

```kotlin
@HiltAndroidApp
class MyApplication : Application(), Configuration.Provider {
    @Inject
    lateinit var workerFactory: HiltWorkerFactory

    override val workManagerConfiguration: Configuration
        get() = Configuration.Builder()
            .setWorkerFactory(workerFactory)
            .build()
}
```

#### 2. Crear el *Worker*
```text
app/src/main/java/com/tuapp/workers/SyncWorker.kt
```

```kotlin
@HiltWorker
class SyncWorker @AssistedInject constructor(
    @Assisted appContext: Context,
    @Assisted workerParams: WorkerParameters,
    private val repository: SyncRepository
) : CoroutineWorker(appContext, workerParams) {
    override suspend fun doWork(): Result {
        return try {
            // Lógica de la tarea (ej. sincronizar base de datos)
            repository.sync()
            Result.success()
        } catch (e: Exception) {
            // Indica que la tarea falló y debe reintentarse más tarde
            Result.retry()
        }
    }
}
```

#### 3. Crear el *Scheduler*
```text
app/src/main/java/com/tuapp/schedulers/SyncScheduler.kt
```

```kotlin
class SyncScheduler @Inject constructor(
    private val workManager: WorkManager
) {
    fun schedule() {
        val constraints = Constraints.Builder()
            .setRequiredNetworkType(NetworkType.UNMETERED)
            .setRequiresCharging(true)
            .build()

        val request = OneTimeWorkRequestBuilder<SyncWorker>()
            .setConstraints(constraints)
            .build()

        workManager.enqueue(request)
    }
}
```

### Implementación sin DI
#### 1. Crear el *Worker*
```text
app/src/main/java/com/tuapp/workers/SyncWorker.kt
```

```kotlin
class SyncWorker(appContext: Context, workerParams: WorkerParameters): CoroutineWorker(appContext, workerParams) {
    override suspend fun doWork(): Result {
        return try {
            // Lógica de la tarea (ej. sincronizar base de datos)
            Result.success()
        } catch (e: Exception) {
            // Indica que la tarea falló y debe reintentarse más tarde
            Result.retry()
        }
    }
}
```

### Uso con DI (*Hilt*)
#### Configurar restricciones y programar tarea
```kotlin
@HiltViewModel
class MainViewModel @Inject constructor(
    private val syncScheduler: SyncScheduler
) : ViewModel() {

    fun onUserRequestedSync() {
        syncScheduler.schedule()
    }
}

// ============================================================
// Y desde la UI
// ============================================================
viewModel.onUserRequestedSync()
```

### Uso sin DI
#### Configurar restricciones y programar tarea
```kotlin
// ============================================================
// 1️⃣ ViewModel Factory
// ============================================================
class MainViewModelFactory(
    private val appContext: Context
) : ViewModelProvider.Factory {
    override fun <T : ViewModel> create(modelClass: Class<T>): T {
        if (modelClass.isAssignableFrom(MainViewModel::class.java)) {
            @Suppress("UNCHECKED_CAST")
            return MainViewModel(appContext) as T
        }
        throw IllegalArgumentException("Unknown ViewModel class")
    }
}

// ============================================================
// 2️⃣ El VM se instancia desde la UI
// ============================================================
private val viewModel: MainViewModel by viewModels {
    MainViewModelFactory(applicationContext)
}

// Alternativa, sin `by viewModels`
private val viewModel = ViewModelProvider(
    this,
    MainViewModelFactory(applicationContext)
)[MainViewModel::class.java]

// ============================================================
// 3️⃣ ViewModel
// ============================================================
class MainViewModel(
    private val context: Context
) : ViewModel() {
    // Esta lógica también se podría extraer a un Caso de Uso/Scheduler y evitaría
    // tener que pasarle el Context a la instancia del VM
    fun enqueueSync() {
        val constraints = Constraints.Builder()
            // Requiere Wi-Fi
            .setRequiredNetworkType(NetworkType.UNMETERED)
            // Requiere estar cargando
            .setRequiresCharging(true)
            .build()

        val syncRequest = OneTimeWorkRequestBuilder<SyncWorker>()
            .setConstraints(constraints)
            .setBackoffCriteria(
                BackoffPolicy.EXPONENTIAL,
                30,
                TimeUnit.SECONDS
            )
            .build()

        WorkManager.getInstance(context).enqueue(syncRequest)
    }
}

// ============================================================
// 4️⃣ Y desde la UI
// ============================================================
viewModel.enqueueSync()
```

## *Services*
### Dependencias & *Plugins* (opcional)
No requiere dependencias porque forma parte del _Android Framework_.  
Sin embargo, en caso de requerir el módulo _Core KTX_ de _AndroidX_, se debe agregar:

```kotlin
implementation("androidx.core:core-ktx:<VERSION>")
```

### Implementación: *Foreground Service*
#### 1. Declarar servicio y permisos en el *Manifest*

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <!-- Obligatorio solo cuando se declara foregroundServiceType="mediaPlayback" -->
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE_MEDIA_PLAYBACK" />

    <!-- Requerido para Android 13+ -->
    <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />

    <service 
        android:name=".MusicService" 
        android:exported="false"
        android:enabled="true"
        android:foregroundServiceType="mediaPlayback" >
    </service>
</manifest>
```

#### 2. Crear el servicio
```text
app/src/main/java/com/tuapp/services/MusicService.kt
```

```kotlin
class MusicService : Service() {
    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        // Crear NotificationChannel si aún no existe (Android 8+)
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val channel = NotificationChannel(
                "MUSIC_CHANNEL",
                "Reproducción de música",
                NotificationManager.IMPORTANCE_LOW
            )

            getSystemService(NotificationManager::class.java)
                ?.createNotificationChannel(channel)
        }

        val notification = NotificationCompat.Builder(this, "MUSIC_CHANNEL")
            .setContentTitle("Reproduciendo")
            .setSmallIcon(R.drawable.ic_music)
            .build()

        // Obligatorio: iniciar en primer plano antes de los 5 segundos
        startForeground(1, notification)

        // El servicio se reinicia si el sistema lo mata
        return START_STICKY
    }

    override fun onBind(intent: Intent?) = null
}
```

### Uso: Iniciar el *Foreground Service*
Se realiza habitualmente desde una _Activity_ o _Fragment_ (por ejemplo, al presionar un botón).  
Antes de iniciar un _Foreground Service_, se debe verificar que el usuario haya concedido el permiso de notificaciones (Android 13+).  
Si el permiso no está concedido, se solicita y se aborta el inicio del servicio hasta recibir la respuesta del usuario.


```kotlin
class MainActivity : AppCompatActivity() {
    // Launcher para solicitar el permiso de notificaciones (Android 13+)
    private val requestNotificationPermission =
        registerForActivityResult(ActivityResultContracts.RequestPermission()) { isGranted ->
            if (isGranted) {
                // El usuario concedió el permiso :arrow_right: Se puede iniciar el servicio
                startMusicService()
            } else {
                // El usuario rechazó el permiso :arrow_right: No se inicia el servicio
                // (Opcional: mostrar un mensaje explicativo)
            }
        }
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            ExampleTheme {
                // Whatever...
            }
        }
    }

    // Punto de entrada (ej. click de botón)
    fun onPlayButtonClicked() {
        // Validar permiso de notificaciones solo en Android 13+
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.TIRAMISU &&
            checkSelfPermission(Manifest.permission.POST_NOTIFICATIONS)
            != PackageManager.PERMISSION_GRANTED
        ) {
            // Solicita permiso y delega la decisión al callback
            requestNotificationPermission.launch(Manifest.permission.POST_NOTIFICATIONS)
            return
        }

        // Permiso ya concedido o no requerido :arrow_right: Iniciar servicio directamente
        startMusicService()
    }

    // Encapsula el inicio del Foreground Service
    private fun startMusicService() {
        val intent = Intent(this, MusicService::class.java)

        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            startForegroundService(intent)
        } else {
            startService(intent)
        }
    }
}
```

### Implementación: *Bound Service*
#### 1. Crear el servicio
```text
app/src/main/java/com/tuapp/services/DownloadService.kt
```

```kotlin
class DownloadService : Service() {
    // Binder expuesto a los clientes
    private val binder = LocalBinder()

    // Clase Binder que devuelve la instancia del servicio
    inner class LocalBinder : Binder() {
        fun getService(): DownloadService = this@DownloadService
    }

    override fun onBind(intent: Intent?): IBinder {
        return binder
    }

    // ============================================================
    // API pública del servicio
    // ============================================================
    fun startDownload(url: String) {
        // Lógica de descarga. No debería hacerse en el main thread del Service
    }

    fun getProgress(): Int {
        // Retornar progreso (0–100)
        return 0
    }
}
```

#### 2. Declarar servicio en el Manifest
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <service
        android:name=".services.DownloadService"
        android:exported="false" />
</manifest>
```

### Uso: Conectar el *Bound Service*
Un _Bound Service_ no se “inicia” como un _Service_ tradicional.  
En su lugar, un componente cliente (_Activity_ / _Fragment_) se vincula al servicio mediante ``bindService()`` y obtiene una referencia directa a él.

```kotlin
class MainActivity : AppCompatActivity() {
    // 1. Crear la conexión (``ServiceConnection``)
    private var downloadService: DownloadService? = null
    private var isBound = false

    private val serviceConnection = object : ServiceConnection {
        override fun onServiceConnected(
            name: ComponentName?,
            service: IBinder?
        ) {
            // El IBinder es provisto por el Service a través de onBind()
            val binder = service as DownloadService.LocalBinder
            downloadService = binder.getService()
            isBound = true
        }

        // Llamado solo en casos excepcionales (ej. crash del servicio)
        override fun onServiceDisconnected(name: ComponentName?) {
            downloadService = null
            isBound = false
        }
    }

    override fun onStart() {
        super.onStart()
        
        // 2. Vincular el servicio (``bindService``)
        val intent = Intent(this, DownloadService::class.java)
        // Context.BIND_AUTO_CREATE crea el servicio si aún no existe
        bindService(intent, serviceConnection, Context.BIND_AUTO_CREATE)
    }

    override fun onStop() {
        // 3. Desvincular el servicio (``unbindService``) para evitar memory leaks
        if (isBound) {
            unbindService(serviceConnection)
            isBound = false
            downloadService = null
        }
        super.onStop()
    }
    
    // 4. Usar el servicio desde la UI
    fun onDownloadButtonClicked() {
        if (isBound) {
            downloadService?.startDownload("https://example.com/file.zip")
        }
    }

    fun refreshProgress() {
        if (isBound) {
            val progress = downloadService?.getProgress()
            // Actualizar UI
        }
    }
}
```

## *AlarmManager*
### Implementación
#### 1. Declarar receptor y permisos en el *Manifest*
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
xmlns:tools="http://schemas.android.com/tools">

<uses-permission android:name="android.permission.SCHEDULE_EXACT_ALARM" />

<application>
    <!-- Otros componentes... -->

    <receiver 
        android:name=".alarms.AlarmReceiver" 
        android:exported="false">
    </receiver>
</application>

</manifest>
```

#### 2. Crear el *BroadcastReceiver* de la alarma
Este componente será el que ejecute la acción (ej. mostrar una notificación) cuando el tiempo se cumpla.

```text
app/src/main/java/com/alarms/AlarmReceiver.kt
```

```kotlin
class AlarmReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent?) {
        // Lógica a ejecutar cuando suena la alarma
        // Ej: Mostrar una notificación o disparar un Worker
    }
}
```

### Uso: Programar la alarma
```kotlin
class MainActivity : ComponentActivity() {
    private val alarmManager by lazy {
        getSystemService(Context.ALARM_SERVICE) as AlarmManager
    }
    private val intent by lazy {
        Intent(this, AlarmReceiver::class.java)
    }
    private val pendingIntent by lazy {
        PendingIntent.getBroadcast(
            this,
            0,
            intent,
            PendingIntent.FLAG_IMMUTABLE or PendingIntent.FLAG_UPDATE_CURRENT
        )
    }
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            ExampleTheme {
                // Whatever...
            }
        }
    }

    override fun onResume() {
        super.onResume()
        // También se puede llamar desde una acción explícita
        // Ej.: click de botón, confirmación de usuario, flow puntual.
        scheduleAlarm()
    }

    // Programar para el tiempo exacto incluso en reposo profundo (RTC_WAKEUP)
    // triggerTimeMillis es el tiempo futuro en milisegundos (System.currentTimeMillis() + delay)
    private fun scheduleAlarm() {
        val triggerTimeMillis = System.currentTimeMillis() + TimeUnit.MINUTES.toMillis(10)
        
        if (alarmManager.canScheduleExactAlarms()) {
            alarmManager.setExactAndAllowWhileIdle(
                AlarmManager.RTC_WAKEUP,
                triggerTimeMillis,
                pendingIntent
            )
        } else {
            // Android 12+ :arrow_right: Redirige al usuario a habilitar alarmas exactas
            val intent = Intent(Settings.ACTION_REQUEST_SCHEDULE_EXACT_ALARM)
            startActivity(intent)
        }
    }
}
```

## *BroadcastReceiver*
### Implementación: Registro estático
#### 1. Declarar receptor en el *Manifest*
Se coloca dentro de la etiqueta ``<application>``. Si el evento es el inicio del sistema, es obligatorio solicitar el permiso correspondiente.  
Siempre que sea posible, mantener ``exported=false`` para reducir superficie de ataque.

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <!-- Permiso necesario para escuchar el inicio del sistema -->
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />

    <application>
        <!-- Otros componentes... -->
        
        <receiver 
            android:name=".BootReceiver" 
            android:exported="false">
            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED" />
            </intent-filter>
        </receiver>
    </application>

</manifest>
```

#### 2. Definir el Receptor
```text
app/src/main/java/com/receivers/BootReceiver.kt
```

```kotlin
class BootReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent?) {
        if (intent?.action == Intent.ACTION_BOOT_COMPLETED) {
            // El sistema nos despertó. 
            // Aprovechamos para volver a setear las alarmas que se perdieron
            val scheduler = MyAlarmScheduler(context)
            scheduler.reprogramAllAlarms()
        }
    }
}
```

### Implementación: Registro dinámico
```text
app/src/main/java/com/receivers/MyReceiver.kt
```

```kotlin
class MyReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent?) {
        when (intent?.action) {
            Intent.ACTION_AIRPLANE_MODE_CHANGED -> {
                val isEnabled = intent.getBooleanExtra("state", false)
                // Reaccionar al cambio de modo avión mientras la app está abierta
            }
            // Aquí irían otros eventos que SÍ permiten registro dinámico 
            // (ej. cambios de batería, conectividad, etc.)
        }
    }
}
```

### Uso: Registro dinámico
Generalmente se realiza en una _Activity_ o _Service_, respetando el ciclo de vida (registrar en `onStart` u `onResume`, liberar en `onStop` u `onPause`).

```kotlin
class MainActivity : AppCompatActivity() {
    private var isReceiverRegistered = false
    private val receiver = MyReceiver()
    private val filter = IntentFilter().apply {
        addAction(Intent.ACTION_AIRPLANE_MODE_CHANGED)
    }
    
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            ExampleTheme {
                // Whatever...
            }
        }
    }

    override fun onStart() {
        super.onStart()

        // En Android 14+ (API 34), se debe especificar la visibilidad del receptor
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.UPSIDE_DOWN_CAKE) {
            registerReceiver(receiver, filter, Context.RECEIVER_NOT_EXPORTED)
        } else {
            registerReceiver(receiver, filter)
        }

        isReceiverRegistered = true
    }

    override fun onStop() {
        // IMPORTANTE: Desregistrar para evitar Memory Leaks
        if (isReceiverRegistered) {
            unregisterReceiver(receiver)
            isReceiverRegistered = false
        }
        super.onStop()
    }
}
```

## *Notifications*
### Dependencias & *Plugins* (opcional)
No requiere dependencias porque forma parte del _Android Framework_.  
Sin embargo, en caso de requerir el módulo _Core KTX_ de _AndroidX_, se debe agregar:

```kotlin
implementation("androidx.core:core-ktx:<VERSION>")
```

### Implementación
#### 1. Declarar permiso en el *Manifest*

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <!-- Requerido para Android 13+ -->
    <uses-permission android:name="android.permission.POST_NOTIFICATIONS" />
    
</manifest>
```

#### 2. Crear el Canal de Notificación
Se debe ejecutar una sola vez, habitualmente al iniciar la aplicación (clase `Application`) o antes de lanzar la primera notificación.

- Opción sin DI

```kotlin
class MyApplication : Application() {
    override fun onCreate() {
        super.onCreate()

        // Obligatorio en Android 8+ crear el NotificationChannel
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val name = "Notificaciones de Recordatorio"
            val descriptionText = "Canal utilizado para alarmas y tareas próximas"
            val importance = NotificationManager.IMPORTANCE_DEFAULT
            val channel = NotificationChannel("REMINDER_CHANNEL_ID", name, importance).apply {
                description = descriptionText
            }

            // Registrar el canal en el sistema
            val notificationManager: NotificationManager =
                getSystemService(Context.NOTIFICATION_SERVICE) as NotificationManager
            notificationManager.createNotificationChannel(channel)
        }
    }
}
```

### Uso: Construir y mostrar la notificación
Se puede ejecutar desde un `Service`, un `Worker`, un `BroadcastReceiver` o directamente desde una `Activity`.

```kotlin
class MainActivity : AppCompatActivity() {
    // Int único para cada notificación
    private val notificationId = 1

    // Launcher para solicitar el permiso de notificaciones (Android 13+)
    private val requestNotificationPermission =
        registerForActivityResult(ActivityResultContracts.RequestPermission()) { isGranted ->
            if (isGranted) {
                showNotification()
            }
            // Si no fue concedido, no se notifica (decisión de UX)
            // (Opcional: mostrar un mensaje explicativo)
        }
    
    private val builder by lazy {
        NotificationCompat.Builder(this, "REMINDER_CHANNEL_ID")
            .setSmallIcon(R.drawable.ic_notification)
            .setContentTitle("Tarea pendiente")
            .setContentText("Tienes una nueva tarea para revisar hoy.")
            .setPriority(NotificationCompat.PRIORITY_DEFAULT)
            .setAutoCancel(true) // Se cierra al tocarla
    }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setupNotificationsPermissions()
        enableEdgeToEdge()
        setContent {
            ExampleTheme {
                // Whatever...
            }
        }
    }

    private fun setupNotificationsPermissions() {
        // Validar permiso de notificaciones solo en Android 13+. En versiones anteriores no hay permiso runtime
        if (Build.VERSION.SDK_INT < Build.VERSION_CODES.TIRAMISU) {
            showNotification()
            return
        }

        when {
            ContextCompat.checkSelfPermission(
                this,
                Manifest.permission.POST_NOTIFICATIONS
            ) == PackageManager.PERMISSION_GRANTED -> {
                showNotification()
            }

            // Si el permiso no está concedido, se debe solicitar usando ActivityResultContracts
            else -> {
                requestNotificationPermission.launch(
                    Manifest.permission.POST_NOTIFICATIONS
                )
            }
        }
    }

    private fun showNotification() {
        NotificationManagerCompat.from(this)
            .notify(notificationId, builder.build())
    }
}
```

## *Deep Links*
### Implementación
Declarar el _deeplink_ en el *Manifest* para la *Activity* que corresponda.  
Por ejemplo, para que el _deeplink_ ``example://screen/user/detail`` pueda abrir la ``MainActivity``.

```xml
<activity
    android:name=".ui.auth.MainActivity"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.VIEW"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <category android:name="android.intent.category.BROWSABLE"/>

        <data
            android:scheme="example"
            android:host="screen"
            android:path="user/detail" />
    </intent-filter>
</activity>
```

### Uso: Invocar *deeplink*
#### 1. Desde código (App :arrow_right: App)
Ejemplo desde una _Activity_ o _Fragment_:

```kotlin
val intent = Intent(
    Intent.ACTION_VIEW,
    Uri.parse("example://screen/user/detail")
)

startActivity(intent)
```

#### 2. Desde otra app o navegador
Por ejemplo, escribiendo la URL manualmente:

```text
example://screen/user/detail
```

#### 3. Desde ``adb`` (testing / QA)
Muy útil para probar Deep Links sin UI (ver también [acá](../Utils%20&%20Miscellaneous/Comandos%20útiles%20-%20ADB.md#iniciar-activity-con-un-deeplink)).

```bash
adb shell am start \
    -a android.intent.action.VIEW \
    -d "example://screen/user/detail"
```

#### Recibir y procesar el *Deep Link* en la *Activity*
Una vez abierta la _Activity_, se accede al ``Uri`` desde el ``Intent``:

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val data: Uri? = intent.data

        if (data != null) {
            // example://screen/user/detail?id=42&source=push
            val path = data.path
            val host = data.host

            // Usar el deeplink para navegar a otra pantalla, recuperar datos
            // de sus query params, etc.
            val userId = data.getQueryParameter("id")
            val source = data.getQueryParameter("source")
        }
    }

    // Clave si la Activity usa launchMode="singleTop"
    override fun onNewIntent(intent: Intent?) {
        super.onNewIntent(intent)
        handleDeepLink(intent)
    }

    private fun handleDeepLink(intent: Intent?) {
        val data = intent?.data ?: return
        val path = data.path
        val host = data.host
    }
}
```

## *App Links*
### Implementación
> ⚠️ **Importante**  
> El dominio debe estar **verificado** (ver [acá](../Android/Libs,%20APIs%20&%20Frameworks/Background%20&%20System.md#app-links))

Declarar el _app link_ en el *Manifest* para la *Activity* que corresponda.  
Para que funcione como *App Link* y no como un [*Deep Link*](#deep-links) común, se debe incluir `android:autoVerify="true"`.

```xml
<activity
    android:name=".ui.auth.MainActivity"
    android:exported="true">
    <intent-filter android:autoVerify="true">
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <category android:name="android.intent.category.BROWSABLE" />

        <!-- Acepta URLs que comiencen con https://www.example.com/users -->
        <data 
            android:scheme="https" 
            android:host="www.example.com" 
            android:pathPrefix="/users" />
    </intent-filter>
</activity>
```

### Uso: Invocar *app link*
#### 1. Desde código (App :arrow_right: App)
Exactamente igual que un _Deep Link_, pero usando una URL HTTPS válida:

```kotlin
val intent = Intent(
    Intent.ACTION_VIEW,
    Uri.parse("https://www.example.com/users/42")
)

startActivity(intent)
```

#### 2. Desde navegador (caso principal de uso)
Escribiendo o tocando un link web normal:

```text
https://www.example.com/users/42
```

#### 3. Desde otra app (ej. email, _WhatsApp_, etc.)
Cualquier app que lance un ``ACTION_VIEW`` con esa URL:

```kotlin
Intent(Intent.ACTION_VIEW)
    .setData(Uri.parse("https://www.example.com/users/42"))
```

#### 4. Desde ``adb`` (testing / QA)
Ideal para validar que el _App Link_ esté correctamente verificado (ver también [acá](../Utils%20&%20Miscellaneous/Comandos%20útiles%20-%20ADB.md#iniciar-activity-con-un-deeplink)).

```bash
adb shell am start \
    -a android.intent.action.VIEW \
    -d "https://www.example.com/users/42"
```

#### Recibir y procesar el *App Link* en la *Activity*
Idéntico a un [_Deep Link_](#recibir-y-procesar-el-deep-link-en-la-activity). Desde el punto de vista del código no hay diferencia.

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    val data: Uri? = intent.data

    if (data != null) {
        // https://www.example.com/users/42
        val path = data.path        // /users/42
        val host = data.host        // www.example.com

        // Resolver navegación interna
    }
}
```

## *Shortcuts*
### Implementación & Uso: *Shortcut* dinámico

```kotlin
class MainActivity : AppCompatActivity() {
    private val shortcutManager by lazy {
        getSystemService(ShortcutManager::class.java)
    }
        
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            ExampleTheme {
                // Whatever...
            }
        }
    }

    private fun onProfileButtonClick() {
        val shortcut = ShortcutInfo.Builder(this, "id_perfil")
            .setShortLabel("Ver perfil")
            .setLongLabel("Abrir el perfil del usuario")
            .setIcon(Icon.createWithResource(this, R.drawable.ic_user))
            .setIntent(
                Intent(
                    Intent.ACTION_VIEW,
                    Uri.parse("https://www.example.com/profile")
                )
            )
            .build()

        shortcutManager.addDynamicShortcuts(listOf(shortcut))
        
    // Otras APIs útiles
        // shortcutManager.updateDynamicShortcuts(listOf(updatedShortcut))
        // shortcutManager.removeDynamicShortcuts(listOf("id_perfil"))
        // shortcutManager.removeAllDynamicShortcuts()
    }
}
```

### Implementación: *Shortcut* estático
#### 1. Declarar el *Shortcut*
Buenas prácticas:
- ``shortcutId`` estable (no cambiarlo)
- ``ShortLabel`` ≤ 10 caracteres aproximadamente
- ``LongLabel`` más descriptivo 
- Reutilizar _deeplinks_ internos

```text
res/xml/shortcuts.xml
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<shortcuts xmlns:android="http://schemas.android.com/apk/res/android">

    <shortcut
        android:shortcutId="shortcut_new_user"
        android:enabled="true"
        android:icon="@drawable/ic_user_add"
        android:shortcutShortLabel="Nuevo usuario"
        android:shortcutLongLabel="Crear nuevo usuario">

        <intent
            android:action="android.intent.action.VIEW"
            android:data="example://screen/user/create"
            android:targetPackage="com.tuapp"
            android:targetClass="com.tuapp.MainActivity" />
    </shortcut>

</shortcuts>
```

#### 2. Registrar el *Shortcut* en el *Manifest*
Dentro de la _Activity_ que actuará como punto de entrada:

```xml
<activity
    android:name=".MainActivity"
    android:exported="true">

    <meta-data
        android:name="android.app.shortcuts"
        android:resource="@xml/shortcuts" />

    <!-- Intent-filter de launcher -->
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>

</activity>
```

### Uso: Recibir el *Shortcut* estático en la *Activity*
El _shortcut_ llega como un **_Intent_ normal**.  
Si se usan _deeplinks_, se procesan igual que cualquier otro.  

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)

    handleIntent(intent)
}

// Clave si la Activity usa launchMode="singleTop"
override fun onNewIntent(intent: Intent) {
    super.onNewIntent(intent)
    handleIntent(intent)
}

private fun handleIntent(intent: Intent) {
    val data = intent.data
    if (data != null) {
        // example://screen/user/create
        // Navegar a la pantalla correspondiente
    }
}
```

## *In-App Updates*
### Dependencias y *Plugins*

```kotlin
implementation("com.google.android.play:app-update:<VERSION>")

// En caso de usar KTX
implementation("com.google.android.play:app-update-ktx:<VERSION>")
```

### Implementación & Uso: Verificar y lanzar la actualización
#### Modo Inmediato
Se implementa generalmente en el ``onResume`` de la _Activity_ principal para asegurar que el usuario tenga la última versión.

```kotlin
class MainActivity : AppCompatActivity() {
    private val appUpdateManager by lazy {
        AppUpdateManagerFactory.create(this)
    }

    private val updateResultLauncher =
        registerForActivityResult(ActivityResultContracts.StartIntentSenderForResult()) { result ->
            if (result.resultCode != Activity.RESULT_OK) {
                // El usuario canceló o falló la actualización
                // (decisión de UX: reintentar, ignorar, etc.)
            }
        }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            ExampleTheme {
                // Whatever...
            }
        }
    }

    override fun onResume() {
        super.onResume()

        appUpdateManager.appUpdateInfo.addOnSuccessListener { appUpdateInfo ->
            if (appUpdateInfo.updateAvailability() == UpdateAvailability.UPDATE_AVAILABLE
                && appUpdateInfo.isUpdateTypeAllowed(AppUpdateType.IMMEDIATE)
            ) {
                // Lanza el flujo de actualización inmediata
                appUpdateManager.startUpdateFlowForResult(
                    appUpdateInfo,
                    updateResultLauncher
                )
            }
        }
    }
}
```

#### Modo Flexible con monitoreo de estado
A diferencia del modo inmediato, requiere un `InstallStateUpdatedListener` para detectar cuándo la descarga ha finalizado y solicitar al usuario el reinicio mediante `completeUpdate()`.

```kotlin
class MainActivity : AppCompatActivity() {
    private val appUpdateManager by lazy {
        AppUpdateManagerFactory.create(this)
    }

    private val updateResultLauncher =
        registerForActivityResult(ActivityResultContracts.StartIntentSenderForResult()) { result ->
            if (result.resultCode != Activity.RESULT_OK) {
                // El usuario canceló o falló la actualización
                // (decisión de UX: reintentar, ignorar, etc.)
            }
        }

    // 1. Crear el listener para detectar el fin de la descarga
    val listener = InstallStateUpdatedListener { state ->
        if (state.installStatus() == InstallStatus.DOWNLOADED) {
            // Notificar al usuario (ej. Snackbar) para completar la instalación
            // llamando a: appUpdateManager.completeUpdate()
        }
    }
        
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            ExampleTheme {
                // Whatever...
            }
        }
    }

    override fun onResume() {
        super.onResume()
        // 2. Registrar el listener e iniciar el flujo flexible
        appUpdateManager.registerListener(listener)
        appUpdateManager.appUpdateInfo.addOnSuccessListener { appUpdateInfo ->
            if (appUpdateInfo.updateAvailability() == UpdateAvailability.UPDATE_AVAILABLE
                && appUpdateInfo.isUpdateTypeAllowed(AppUpdateType.FLEXIBLE)
            ) {
                // Lanza el flujo de actualización flexible
                appUpdateManager.startUpdateFlowForResult(
                    appUpdateInfo,
                    updateResultLauncher
                )
            }
        }
    }

    override fun onPause() {
        // 3. Eliminar el listener en onPause 
        appUpdateManager.unregisterListener(listener)
        super.onPause()
    }
}
```

## *In-App Reviews*
### Dependencias y *Plugins*

```kotlin
implementation("com.google.android.play:review:<VERSION>")

// En caso de usar KTX
implementation("com.google.android.play:review-ktx:<VERSION>")
```

### Implementación & Uso: Solicitar y lanzar el flujo de reseña
Se implementa habitualmente en la *Activity* tras un evento de éxito (ej. completar una compra).  
No se debe asociar a un botón de "Opinar", sino que debe ocurrir **de forma natural en el flujo**.

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            ExampleTheme {
                // Whatever...
            }
        }
    }

    private fun onBuyButtonClick() {
        val manager = ReviewManagerFactory.create(this)
        val request = manager.requestReviewFlow()
        
        request.addOnCompleteListener { task ->
            if (task.isSuccessful) {
                // Obtener el objeto de información necesario para lanzar el flujo
                val reviewInfo = task.result
                val flow = manager.launchReviewFlow(this, reviewInfo)
                flow.addOnCompleteListener { _ ->
                    // El flujo ha terminado. Google recomienda no mostrar mensajes 
                    // adicionales (como "Gracias") ya que el diálogo es externo.
                }
            } else {
                // Loggear task.exception
            }
        }
    }
}
```

## *Widgets*
### Implementación
#### 1. Crear el *AppWidgetProvider*
```text
app/src/main/java/com/widgets/MyWidgetProvider.kt
```

```kotlin
class MyWidgetProvider : AppWidgetProvider() {
    // Se puede extender para reaccionar a onEnabled, onDisabled o onReceive si se necesita más lógica
    
    override fun onUpdate(
        context: Context,
        appWidgetManager: AppWidgetManager,
        appWidgetIds: IntArray
    ) {
        // Actualizar cada instancia del Widget presente en la pantalla de inicio
        for (appWidgetId in appWidgetIds) {
            val views = RemoteViews(context.packageName, R.layout.widget_layout)
            views.setTextViewText(R.id.widget_text, "Estado: Actualizado")

            appWidgetManager.updateAppWidget(appWidgetId, views)
        }
    }
}
```

#### 2. Definir configuración del *Widget*
Este archivo define las propiedades físicas y de comportamiento del *Widget*. Se referencia en el *Manifest* dentro del bloque `<meta-data>` (paso 3).  
El atributo `updatePeriodMillis` define la **actualización automática**. Por razones de ahorro de batería, Android impone un mínimo de **30 minutos** (1,800,000 ms) y puede retrasar o agrupar actualizaciones adicionales. Si se necesita una actualización inmediata ante un cambio en la app, se debe usar la **actualización manual** (ver [Uso](#uso-actualizar-widget-manualmente)).

```text
res/xml/my_widget_info.xml
```

```xml
<?xml version="1.0" encoding="utf-8"?>
<appwidget-provider xmlns:android="http://schemas.android.com/apk/res/android"
    android:minWidth="40dp"
    android:minHeight="40dp"
    android:updatePeriodMillis="1800000" 
    android:initialLayout="@layout/widget_layout"
    android:resizeMode="horizontal|vertical"
    android:widgetCategory="home_screen">
</appwidget-provider>
```

#### 3. Declarar receptor en el *Manifest*
Es obligatorio asociar el proveedor a un archivo de configuración XML (``appwidget-provider``).

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <!-- Permiso necesario para escuchar el inicio del sistema -->
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />

    <application>
    <!-- Otros componentes... -->

    <receiver android:name=".MyWidgetProvider" android:exported="false">
        <intent-filter>
            <action android:name="android.appwidget.action.APPWIDGET_UPDATE" />
        </intent-filter>
        <meta-data
            android:name="android.appwidget.provider"
            android:resource="@xml/my_widget_info" />
    </receiver>
</application>

    </manifest>
```

### Uso: Actualizar *Widget* manualmente
Se realiza desde cualquier parte de la app (ej. tras una sincronización de datos exitosa) para forzar la actualización de la UI del *Widget* fuera del intervalo automático.

```kotlin
class MainActivity : AppCompatActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            ExampleTheme {
                // Whatever...
            }
        }
    }

    private fun onSuccessfulSync() {
        val intent = Intent(this, MyWidgetProvider::class.java).apply {
            action = AppWidgetManager.ACTION_APPWIDGET_UPDATE

            // Obtener todos los IDs de las instancias del widget activas en el escritorio
            val ids = AppWidgetManager.getInstance(this@MainActivity)
                .getAppWidgetIds(ComponentName(this@MainActivity, MyWidgetProvider::class.java))

            // "Guard clause" por si no hay widgets activos
            if (ids.isEmpty()) return

            putExtra(AppWidgetManager.EXTRA_APPWIDGET_IDS, ids)
        }
        
        // Enviar el broadcast para que se ejecute el onUpdate del Provider
        sendBroadcast(intent)
    }
}
```
