<h1><i>Firebase</i> SDKs</h1>

***Index***:
<!-- TOC -->
  * [Dependencias & *Plugins*](#dependencias--plugins)
    * [Dependencias](#dependencias)
    * [*Plugin* de ``google-services``](#plugin-de-google-services)
  * [Inicialización de *Firebase*](#inicialización-de-firebase)
    * [Implementación: ``FirebaseInit``](#implementación-firebaseinit)
    * [Uso: Inicialización en ``Application``](#uso-inicialización-en-application)
  * [*Firebase Authentication* (*Auth*)](#firebase-authentication-auth)
    * [1. Implementación: ``CredentialParser``](#1-implementación-credentialparser)
    * [2. Implementación: ``FirebaseGoogleAuth``](#2-implementación-firebasegoogleauth)
    * [3. Implementación: ``CredentialProvider``](#3-implementación-credentialprovider)
    * [4. Implementación: ``CredentialManagerProvider``](#4-implementación-credentialmanagerprovider)
    * [5. Implementación: ``FirebaseAuthRepository``](#5-implementación-firebaseauthrepository)
    * [Uso: Desde ``LoginActivity``](#uso-desde-loginactivity)
  * [*Firebase Cloud Messaging* (FCM)](#firebase-cloud-messaging-fcm)
    * [1. Mostrar notificaciones](#1-mostrar-notificaciones)
      * [Implementación: ``NotificationHelper``](#implementación-notificationhelper)
    * [2. Obtener y refrescar el *token* (*Provider* + *Repository*)](#2-obtener-y-refrescar-el-token-provider--repository)
      * [Implementación: ``FcmTokenProvider``](#implementación-fcmtokenprovider)
      * [Implementación: ``FcmRepository``](#implementación-fcmrepository)
    * [3. Recepción de mensajes (*Foreground* & *Background*)](#3-recepción-de-mensajes-foreground--background)
      * [Implementación: ``MyFirebaseMessagingService``](#implementación-myfirebasemessagingservice)
    * [Ejemplo de uso: Solicitar el *token* desde la capa de UI](#ejemplo-de-uso-solicitar-el-token-desde-la-capa-de-ui)
      * [Uso: Desde el ``ViewModel``](#uso-desde-el-viewmodel)
      * [Uso: Desde la ``Activity``](#uso-desde-la-activity)
  * [*Firebase Remote Config*](#firebase-remote-config)
    * [Implementación: ``RemoteConfigManager``](#implementación-remoteconfigmanager)
    * [Uso: Inicializar *Remote Config* en ``Application``](#uso-inicializar-remote-config-en-application)
    * [Uso: Obtener valores actualizados desde ``Activity``](#uso-obtener-valores-actualizados-desde-activity)
<!-- TOC -->

---

## Dependencias & *Plugins*
### Dependencias
> ℹ️ Nota:  
> Utilizar las dependencias de los **SDKs que se vayan a utilizar**, no es necesario agregar todas.

```kotlin
dependencies {
    // Firebase BOM (maneja versiones)
    implementation(platform("com.google.firebase:firebase-bom:33.5.1"))

    // Core (Analytics recomendado para inicialización)
    implementation("com.google.firebase:firebase-analytics-ktx")

    // Authentication
    implementation("com.google.firebase:firebase-auth-ktx")

    // Firestore
    implementation("com.google.firebase:firebase-firestore-ktx")

    // Realtime Database
    implementation("com.google.firebase:firebase-database-ktx")
    
    // Remote Config
    implementation("com.google.firebase:firebase-config-ktx")

    // Cloud Storage
    implementation("com.google.firebase:firebase-storage-ktx")

    // FCM
    implementation("com.google.firebase:firebase-messaging-ktx")

    // Crashlytics
    implementation("com.google.firebase:firebase-crashlytics-ktx")

    // Play Services Auth (si se usa login con Google)
    implementation("com.google.android.gms:play-services-auth:20.7.0")
}
```

### *Plugin* de ``google-services``
- En el ``build.gradle.kts`` del proyecto (_root_):
```kotlin
buildscript {
    dependencies {
        // Plugin de Google Services
        classpath("com.google.gms:google-services:4.4.0")
    }
}
```

- En el ``build.gradle.kts`` del módulo:
```kotlin
plugins {
    id("com.android.application")
    id("com.google.gms.google-services")
}
```

## Inicialización de *Firebase*
> ℹ️ Nota:  
> **Usar solamente si la app requiere forzar la inicialización** (ver [acá](/Android/Libs%20&%20Frameworks/Networking%20&%20API's.md#sobre-la-inicialización-de-firebase)).

### Implementación: ``FirebaseInit``
```text
app/src/main/java/com/tuapp/firebase/core/FirebaseInit.kt
```

```kotlin
import android.content.Context
import com.google.firebase.FirebaseApp

object FirebaseInit {
    /**
     * Inicializa Firebase si aún no fue inicializado.
     * Debe llamarse una sola vez, idealmente en Application.onCreate().
     */
    fun init(context: Context) {
        if (FirebaseApp.getApps(context).isEmpty()) {
            FirebaseApp.initializeApp(context)
        }
    }
}
```

### Uso: Inicialización en ``Application``
```text
app/src/main/java/com/tuapp/App.kt
```

```kotlin
import android.app.Application
import com.tuapp.firebase.core.FirebaseInit

class App : Application() {
    override fun onCreate() {
        super.onCreate()

        // Inicializa Firebase UNA sola vez
        FirebaseInit.init(this)
    }
}
```

Y también declararla en el ``Manifest``:
```xml
<application
    android:name=".App"
    ... >
```

## *Firebase Authentication* (*Auth*)
### 1. Implementación: ``CredentialParser``
```text
app/src/main/java/com/tuapp/firebase/auth/CredentialParser.kt
```

```kotlin
import androidx.credentials.CustomCredential
import com.google.android.libraries.identity.googleid.GoogleIdTokenCredential

object CredentialParser {
    fun getGoogleIdToken(credential: Any): String {
        if (credential is CustomCredential &&
            credential.type == GoogleIdTokenCredential.TYPE_GOOGLE_ID_TOKEN_CREDENTIAL
        ) {
            val googleCred = GoogleIdTokenCredential.createFrom(credential.data)
            return googleCred.idToken
        }

        throw IllegalArgumentException("Credential no es un Google ID Token válido")
    }
}
```

### 2. Implementación: ``FirebaseGoogleAuth``
```text
app/src/main/java/com/tuapp/firebase/auth/FirebaseGoogleAuth.kt
```

```kotlin
import android.app.Activity
import android.content.Context
import androidx.credentials.CredentialManager
import androidx.credentials.GetCredentialRequest
import androidx.credentials.GetCredentialResponse
import com.google.android.libraries.identity.googleid.GetGoogleIdOption
import com.google.firebase.auth.GoogleAuthProvider
import com.google.firebase.auth.FirebaseAuth
import kotlinx.coroutines.tasks.await

object FirebaseGoogleAuth {
    /**
     * Crea el intent/solicitud para obtener un Google ID Token mediante Credential Manager.
     */
    fun buildGoogleCredentialRequest(context: Context): GetCredentialRequest {
        val googleIdOption = GetGoogleIdOption.Builder()
            .setFilterByAuthorizedAccounts(false)
            // ⚠️ OIDC Web Client ID (generado en Google Cloud Console, no en Firebase Console)
            .setServerClientId("TU_WEB_CLIENT_ID")
            .build()

        return GetCredentialRequest.Builder()
            .addCredentialOption(googleIdOption)
            .build()
    }

    /**
     * Procesa la respuesta del Credential Manager y devuelve un ID Token.
     */
    fun extractGoogleIdToken(response: GetCredentialResponse): String {
        val credential = response.credential
        return CredentialParser.getGoogleIdToken(credential)
    }

    /**
     * Intercambia el ID Token de Google contra Firebase Auth.
     */
    suspend fun signInWithGoogleIdToken(idToken: String): FirebaseAuthResult {
        val credential = GoogleAuthProvider.getCredential(idToken, null)
        return try {
            val result = FirebaseAuth.getInstance().signInWithCredential(credential).await()
            FirebaseAuthResult.Success(result.user)
        } catch (e: Exception) {
            FirebaseAuthResult.Error(e)
        }
    }
}

/**
 * Resultado de autenticación
 */
sealed class FirebaseAuthResult {
    data class Success(val user: com.google.firebase.auth.FirebaseUser?) : FirebaseAuthResult()
    data class Error(val exception: Exception) : FirebaseAuthResult()
}
```

### 3. Implementación: ``CredentialProvider``
```text
app/src/main/java/com/tuapp/firebase/auth/CredentialProvider.kt
```

```kotlin
import androidx.credentials.GetCredentialRequest
import androidx.credentials.GetCredentialResponse
import android.content.Context

/**
 * Abstracción para obtener credenciales (Google ID Token, Email, etc.).
 * Permite desacoplar FirebaseAuthRepository de CredentialManager.
 */
interface CredentialProvider {
    suspend fun getCredential(context: Context, request: GetCredentialRequest): GetCredentialResponse
}
```

### 4. Implementación: ``CredentialManagerProvider``
```text
app/src/main/java/com/tuapp/firebase/auth/CredentialManagerProvider.kt
```

```kotlin
import android.content.Context
import androidx.credentials.CredentialManager
import androidx.credentials.GetCredentialRequest
import androidx.credentials.GetCredentialResponse
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.withContext

/**
 * Implementación de CredentialProvider usando CredentialManager de Android.
 * La llamada se ejecuta en Dispatchers.IO para no bloquear el hilo principal.
 */
class CredentialManagerProvider(
    private val credentialManager: CredentialManager
) : CredentialProvider {
    override suspend fun getCredential(
        context: Context,
        request: GetCredentialRequest
    ): GetCredentialResponse {
        return withContext(Dispatchers.IO) {
            credentialManager.getCredential(context, request)
        }
    }

    companion object {
        fun create(context: Context): CredentialManagerProvider {
            val cm = CredentialManager.create(context)
            return CredentialManagerProvider(cm)
        }
    }
}
```

### 5. Implementación: ``FirebaseAuthRepository``
```text
app/src/main/java/com/tuapp/firebase/auth/FirebaseAuthRepository.kt
```

```kotlin
import com.google.firebase.auth.FirebaseAuth
import com.google.firebase.auth.FirebaseUser
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow

class FirebaseAuthRepository(
    private val credentialProvider: CredentialProvider
) {
    private val auth = FirebaseAuth.getInstance()

    /** Cualquier suscriptor recibirá el último valor inmediatamente */
    private val _currentUserFlow = MutableStateFlow<FirebaseUser?>(auth.currentUser)
    val currentUserFlow: StateFlow<FirebaseUser?> = _currentUserFlow

    /** 
     * Listener de cambios en el estado de autenticación de Firebase, removible para evitar leaks.
     * Se dispara cada vez que cambia el usuario actual.
     */
    private val authListener = FirebaseAuth.AuthStateListener { _currentUserFlow.value = it.currentUser }

    init {
        auth.addAuthStateListener(authListener)
    }

    /** Remover listener cuando el repository ya no se use */
    fun clear() {
        auth.removeAuthStateListener(authListener)
    }

    suspend fun signInWithGoogle(context: android.content.Context): FirebaseAuthResult {
        val request = FirebaseGoogleAuth.buildGoogleCredentialRequest(context)

        return try {
            // Obtener credencial mediante el provider (ya maneja hilo IO)
            val response = credentialProvider.getCredential(context, request)
            val idToken = FirebaseGoogleAuth.extractGoogleIdToken(response)
            FirebaseGoogleAuth.signInWithGoogleIdToken(idToken)
        } catch (e: Exception) {
            FirebaseAuthResult.Error(e)
        }
    }

    fun signOut() {
        auth.signOut()
    }
}
```

### Uso: Desde ``LoginActivity``
```text
app/src/main/java/com/tuapp/ui/auth/LoginActivity.kt
```

```kotlin
import android.os.Bundle
import android.widget.Button
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity
import androidx.lifecycle.lifecycleScope
import com.tuapp.firebase.auth.FirebaseAuthRepository
import kotlinx.coroutines.launch

class LoginActivity : AppCompatActivity() {
    private lateinit var authRepo: FirebaseAuthRepository

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // Asegurarse que el layout tenga el botón
        setContentView(R.layout.activity_login)

        // Crear un provider desacoplado
        val credentialProvider = CredentialManagerProvider.create(this)
        authRepo = FirebaseAuthRepository(credentialProvider)

        // Observar cambios de usuario. Si la Activity se recrea,
        // el launchWhenStarted volverá a suscribirse automáticamente
        lifecycleScope.launchWhenStarted {
            authRepo.currentUserFlow.collect { user ->
                if (user != null) {
                    Toast.makeText(this@LoginActivity, "Login OK", Toast.LENGTH_SHORT).show()
                    // Navegar a MainActivity, por ejemplo
                } else {
                    // Usuario deslogueado, permanecer en LoginActivity
                }
            }
        }

        // Ejemplo: comenzar login al tocar un botón
        findViewById<Button>(R.id.btnLoginGoogle).setOnClickListener {
            lifecycleScope.launch {
                authRepo.signInWithGoogle(this@LoginActivity)
            }
        }
    }

    override fun onDestroy() {
        super.onDestroy()
        // Limpiar listener para evitar leaks
        authRepo.clear()
    }
}
```

## *Firebase Cloud Messaging* (FCM)
### 1. Mostrar notificaciones
#### Implementación: ``NotificationHelper``
```text
app/src/main/java/com/tuapp/firebase/fcm/NotificationHelper.kt
```

```kotlin
import android.app.NotificationChannel
import android.app.NotificationManager
import android.content.Context
import android.os.Build
import androidx.core.app.NotificationCompat
import androidx.core.app.NotificationManagerCompat
import com.tuapp.R

object NotificationHelper {
    private const val CHANNEL_ID = "default_channel"

    fun showNotification(context: Context, title: String, body: String) {
        createChannel(context)

        val notification = NotificationCompat.Builder(context, CHANNEL_ID)
            .setSmallIcon(R.drawable.ic_notification) // ícono obligatorio
            .setContentTitle(title)
            .setContentText(body)
            .setAutoCancel(true)
            .build()

        NotificationManagerCompat.from(context)
            .notify(System.currentTimeMillis().toInt(), notification)
    }

    private fun createChannel(context: Context) {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
            val channel = NotificationChannel(
                CHANNEL_ID,
                "General",
                NotificationManager.IMPORTANCE_DEFAULT
            )

            val manager = context.getSystemService(NotificationManager::class.java)
            manager.createNotificationChannel(channel)
        }
    }
}
```

### 2. Obtener y refrescar el *token* (*Provider* + *Repository*)
#### Implementación: ``FcmTokenProvider``
```text
app/src/main/java/com/tuapp/firebase/fcm/FcmTokenProvider.kt
```

```kotlin
import com.google.firebase.messaging.FirebaseMessaging
import kotlinx.coroutines.tasks.await

object FcmTokenProvider {
    /**
     * Obtiene el token actual de FCM.
     */
    suspend fun getToken(): String? {
        return try {
            FirebaseMessaging.getInstance().token.await()
        } catch (e: Exception) {
            null
        }
    }
}
```

#### Implementación: ``FcmRepository``
```text
app/src/main/java/com/tuapp/firebase/fcm/FcmRepository.kt
```

```kotlin
import android.util.Log

class FcmRepository(
    private val backendApi: BackendApi
) {
    /**
     * Envía el token FCM al backend para asociarlo con el usuario.
     */
    suspend fun sendTokenToBackend(token: String) {
        try {
            backendApi.registerFcmToken(token)
            Log.d("FCM", "Token enviado al backend")
        } catch (e: Exception) {
            Log.e("FCM", "Error enviando token al backend: $e")
        }
    }
}

/**
 * Representa un backend propio (REST, GraphQL, etc.)
 * Implementar esta interfaz en la capa de networking.
 */
interface BackendApi {
    suspend fun registerFcmToken(token: String)
}
```

### 3. Recepción de mensajes (*Foreground* & *Background*)
#### Implementación: ``MyFirebaseMessagingService``
```text
app/src/main/java/com/tuapp/firebase/fcm/MyFirebaseMessagingService.kt
```

```kotlin
import android.util.Log
import com.google.firebase.messaging.FirebaseMessagingService
import com.google.firebase.messaging.RemoteMessage
import kotlinx.coroutines.CoroutineScope
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.launch

class MyFirebaseMessagingService : FirebaseMessagingService() {
    override fun onNewToken(token: String) {
        super.onNewToken(token)
        val repo = FcmRepository(MyBackendApi())

        CoroutineScope(Dispatchers.IO).launch {
            try {
                repo.sendTokenToBackend(token)
            } catch (e: Exception) {
                Log.e("FCM", "Error enviando token: $e")
            }
        }
    }

    override fun onMessageReceived(message: RemoteMessage) {
        super.onMessageReceived(message)

        val title = message.notification?.title
        val body = message.notification?.body
        val data = message.data

        NotificationHelper.showNotification(
            context = this,
            title = title ?: "Nuevo mensaje",
            body = body ?: "Tenés una nueva actualización"
        )
    }
}
```

Y también declarar el _service_ en el ``Manifest``:
```xml
<service
    android:name=".firebase.fcm.MyFirebaseMessagingService"
    android:exported="false">
    <intent-filter>
        <action android:name="com.google.firebase.MESSAGING_EVENT"/>
    </intent-filter>
</service>
```

### Ejemplo de uso: Solicitar el *token* desde la capa de UI
#### Uso: Desde el ``ViewModel``
```text
app/src/main/java/com/tuapp/ui/main/MainViewModel.kt
```

```kotlin
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.tuapp.firebase.fcm.FcmRepository
import com.tuapp.firebase.fcm.FcmTokenProvider
import com.tuapp.firebase.fcm.MyBackendApi
import kotlinx.coroutines.launch

class MainViewModel : ViewModel() {
    private val repo = FcmRepository(MyBackendApi())

    fun registerFcmToken() {
        viewModelScope.launch {
            // Solicitar token en foreground
            val token = FcmTokenProvider.getToken()
            if (token != null) {
                repo.sendTokenToBackend(token)
            }
        }
    }
}
```

#### Uso: Desde la ``Activity``
```kotlin
class MainActivity : AppCompatActivity() {
    private val vm: MainViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate()

        // Registrar token al iniciar la app
        vm.registerFcmToken()
    }
}
```

## *Firebase Remote Config*
### Implementación: ``RemoteConfigManager``
```text
app/src/main/java/com/tuapp/firebase/remoteconfig/RemoteConfigManager.kt
```

```kotlin
import com.google.firebase.remoteconfig.FirebaseRemoteConfig
import com.google.firebase.remoteconfig.ktx.remoteConfig
import com.google.firebase.remoteconfig.ktx.remoteConfigSettings
import com.google.firebase.ktx.Firebase
import kotlinx.coroutines.flow.MutableStateFlow
import kotlinx.coroutines.flow.StateFlow
import android.util.Log
import com.tuapp.BuildConfig

object RemoteConfigManager {
    private const val MAX_ITEMS_FEED = "max_items_feed"
    private const val TEXT_HOME_TITLE = "text_home_title"
    private const val FEATURE_NEW_HOME = "feature_new_home"

    private val _maxItemsFeedFlow = MutableStateFlow(getMaxItemsFeed())
    val maxItemsFeedFlow: StateFlow<Int> = _maxItemsFeedFlow

    fun init() {
        val configSettings = remoteConfigSettings {
            minimumFetchIntervalInSeconds = if (BuildConfig.DEBUG) 0 else 3600
        }
        Firebase.remoteConfig.setConfigSettingsAsync(configSettings)
        Firebase.remoteConfig.setDefaultsAsync(mapOf(
            MAX_ITEMS_FEED to 10L,
            TEXT_HOME_TITLE to "Bienvenido",
            FEATURE_NEW_HOME to false
        ))
    }

    /** Manejo de errores con try/catch */
    suspend fun fetchAndActivate(): Boolean {
        return try {
            val success = Firebase.remoteConfig.fetchAndActivate().await()
            if (success) {
                _maxItemsFeedFlow.value = getMaxItemsFeed()
            }
            success
        } catch (e: Exception) {
            Log.e("RemoteConfig", "Error fetching config: $e")
            false
        }
    }

    /** Métodos helpers para obtener valores */
    fun isNewHomeEnabled() = Firebase.remoteConfig.getBoolean(FEATURE_NEW_HOME)
    fun getHomeTitle() = Firebase.remoteConfig.getString(TEXT_HOME_TITLE)
    fun getMaxItemsFeed() = Firebase.remoteConfig.getLong(MAX_ITEMS_FEED).toInt()
}
```

### Uso: Inicializar *Remote Config* en ``Application``
```text
app/src/main/java/com/tuapp/App.kt
```

```kotlin
package com.tuapp

import android.app.Application
import com.tuapp.firebase.FirebaseInit
import com.tuapp.firebase.remoteconfig.RemoteConfigManager

class App : Application() {
    override fun onCreate() {
        super.onCreate()

        // Inicializar Remote Config
        RemoteConfigManager.init()
    }
}
```

### Uso: Obtener valores actualizados desde ``Activity``
```text
app/src/main/java/com/tuapp/ui/main/MainActivity.kt
```

```kotlin
import android.os.Bundle
import android.widget.TextView
import androidx.appcompat.app.AppCompatActivity
import androidx.lifecycle.lifecycleScope
import com.tuapp.R
import com.tuapp.firebase.remoteconfig.RemoteConfigManager
import kotlinx.coroutines.flow.combine
import kotlinx.coroutines.flow.flow
import kotlinx.coroutines.launch

class MainActivity : AppCompatActivity() {
    private lateinit var titleTextView: TextView
    // Ejemplo para isNewHomeEnabled()
    private lateinit var newHomeTextView: TextView

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        titleTextView = findViewById(R.id.titleTextView)
        // Opcional, para mostrar el feature flag
        newHomeTextView = findViewById(R.id.newHomeTextView)

        // Lanzar el fetch inicial seguro
        lifecycleScope.launch {
            try {
                RemoteConfigManager.fetchAndActivate()
            } catch (e: Exception) {
                // Manejo de error opcional (log, mostrar fallback, etc.)
                e.printStackTrace()
            }
        }

        // Combinar flows de Remote Config
        lifecycleScope.launchWhenStarted {
            combine(
                RemoteConfigManager.maxItemsFeedFlow,
                flow { emit(RemoteConfigManager.isNewHomeEnabled()) },
                flow { emit(RemoteConfigManager.getHomeTitle()) }
            ) { maxItems, isNewHome, title ->
                Triple(maxItems, isNewHome, title)
            }.collect { (maxItems, isNewHome, title) ->
                // Actualizar UI
                titleTextView.text = title
                newHomeTextView.text = if (isNewHome) "Nueva Home Habilitada" else "Home Clásica"

                // Usar maxItems según lógica del feed
                // Ejemplo: recyclerView.adapter?.submitList(data.take(maxItems))
            }
        }
    }
}
```
