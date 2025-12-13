<h1>Security</h1>

***Index***: 
<!-- TOC -->
  * [Integración con Google - OAuth + PKCE](#integración-con-google---oauth--pkce)
    * [1. Dependencias](#1-dependencias)
    * [2. Valores de configuración](#2-valores-de-configuración)
    * [3. Agregar *deeplink* en el ``Manifest``](#3-agregar-deeplink-en-el-manifest)
    * [4. PKCE *utils* (generar ``state``, ``code_challenge`` y ``code_verifier``)](#4-pkce-utils-generar-state-code_challenge-y-code_verifier)
    * [5. Almacenamiento seguro con ``EncryptedSharedPreferences``](#5-almacenamiento-seguro-con-encryptedsharedpreferences)
    * [6. Ktor *client* + Kotlinx.serialization *setup*](#6-ktor-client--kotlinxserialization-setup)
    * [7. Construir la URL de autorización y abrir *Custom Tab*](#7-construir-la-url-de-autorización-y-abrir-custom-tab)
      * [Implementación: *Start Auth*](#implementación-start-auth)
      * [Uso: *Login screen*](#uso-login-screen)
    * [8. *Callback Activity*](#8-callback-activity)
    * [9. Intercambio del ``code`` por los *tokens*](#9-intercambio-del-code-por-los-tokens)
    * [10. Guardar *tokens* de forma segura](#10-guardar-tokens-de-forma-segura)
    * [11. Obtener información del usuario](#11-obtener-información-del-usuario)
      * [A. Usar ``id_token`` (OIDC) — Decodificar ``claims``](#a-usar-id_token-oidc--decodificar-claims)
      * [B. Usar ``/userinfo`` (recomendado para datos estándar)](#b-usar-userinfo-recomendado-para-datos-estándar)
      * [C. Usar *Google People API*](#c-usar-google-people-api)
      * [Uso: *User Profile screen*](#uso-user-profile-screen)
    * [12. Verificar firma del ``id_token`` con Nimbus](#12-verificar-firma-del-id_token-con-nimbus)
    * [13. Revocación + *Logout*](#13-revocación--logout)
      * [Implementación: *Revoke* & *Logout*](#implementación-revoke--logout)
      * [Uso: *Settings screen*](#uso-settings-screen)
<!-- TOC -->

---

## Integración con Google - OAuth + PKCE
### 1. Dependencias
```kotlin
dependencies {
    // Ktor client + OkHttp engine
    implementation("io.ktor:ktor-client-core:2.3.8")
    implementation("io.ktor:ktor-client-okhttp:2.3.8")
    implementation("io.ktor:ktor-client-logging:2.3.8")

    // Kotlinx Serialization (JSON)
    implementation("org.jetbrains.kotlinx:kotlinx-serialization-json:1.6.0")
    implementation("io.ktor:ktor-serialization-kotlinx-json:2.3.8")

    // Chrome Custom Tabs
    implementation("androidx.browser:browser:1.6.0")

    // EncryptedSharedPreferences
    implementation("androidx.security:security-crypto:1.1.0")

    // Nimbus (opcional, para verificar firmas de id_token)
    implementation("com.nimbusds:nimbus-jose-jwt:9.31.2")
}
```

### 2. Valores de configuración
```text
app/
 └─ src/main/java/com/yourapp/config/
       OAuthConfig.kt
```

```kotlin
object OAuthConfig {
    // ⚠️ REEMPLAZAR estos valores con los del proyecto en Google Cloud Console
    const val CLIENT_ID = "<YOUR_CLIENT_ID>.apps.googleusercontent.com"
    const val REDIRECT_URI_GOOGLE = "com.googleusercontent.apps.<YOUR_CLIENT_ID>:/oauth2redirect"

    // Endpoints oficiales: no reemplazar
    const val AUTHORIZATION_ENDPOINT = "https://accounts.google.com/o/oauth2/v2/auth"
    const val TOKEN_ENDPOINT = "https://oauth2.googleapis.com/token"
    const val USERINFO_ENDPOINT = "https://openidconnect.googleapis.com/v1/userinfo"
    const val PEOPLE_API_ENDPOINT = "https://people.googleapis.com/v1/people/me"
    const val REVOCATION_ENDPOINT = "https://oauth2.googleapis.com/revoke"
}
```

### 3. Agregar *deeplink* en el ``Manifest``
> ℹ️ Notas:  
> El ``scheme`` debe coincidir exactamente con la parte antes de ``:/oauth2redirect`` en ``REDIRECT_URI_GOOGLE``.  
> Si se registró un _redirect_ con ``path``, se debe añadir ``android:path`` con el ``path`` exacto.

```xml
<activity
    android:name=".ui.auth.GoogleRedirectActivity"
    android:exported="true">
    <intent-filter>
        <action android:name="android.intent.action.VIEW"/>
        <category android:name="android.intent.category.DEFAULT"/>
        <category android:name="android.intent.category.BROWSABLE"/>

        <data
            android:scheme="com.googleusercontent.apps.<YOUR_CLIENT_ID>"
            android:host="oauth2redirect" />
    </intent-filter>
</activity>
```

### 4. PKCE *utils* (generar ``state``, ``code_challenge`` y ``code_verifier``)

```text
app/src/main/java/com/yourapp/auth/
   OAuthPKCE.kt
```

```kotlin
import java.security.MessageDigest
import java.security.SecureRandom
import java.util.UUID
import android.util.Base64

object OAuthPKCE {
    fun generateState(): String = UUID.randomUUID().toString()

    // RFC 7636: code_verifier length 43..128 chars; we produce ~88 chars
    fun generateCodeVerifier(): String {
        val bytes = ByteArray(64)
        SecureRandom().nextBytes(bytes)
        return Base64.encodeToString(bytes, Base64.URL_SAFE or Base64.NO_WRAP or Base64.NO_PADDING)
    }

    fun generateCodeChallenge(verifier: String): String {
        val bytes = verifier.toByteArray(Charsets.UTF_8)
        val digest = MessageDigest.getInstance("SHA-256").digest(bytes)
        return Base64.encodeToString(digest, Base64.URL_SAFE or Base64.NO_WRAP or Base64.NO_PADDING)
    }
}
```

### 5. Almacenamiento seguro con ``EncryptedSharedPreferences``
> ℹ️ Notas:  
> **Guardar temporalmente**: ``state``, ``code_verifier``. 
> **Guardar a largo plazo**: ``access_token``, ``refresh_token``, ``id_token``.

```text
app/src/main/java/com/yourapp/auth/
   OAuthStorage.kt
```

```kotlin
import androidx.security.crypto.EncryptedSharedPreferences
import androidx.security.crypto.MasterKey

class OAuthStorage(context: Context) {
    private val masterKey = MasterKey.Builder(context)
        .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
        .build()

    private val prefs = EncryptedSharedPreferences.create(
        context,
        "oauth_prefs",
        masterKey,
        EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
        EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
    )

    fun save(key: String, value: String?) {
        prefs.edit().putString(key, value).apply()
    }

    fun read(key: String): String? = prefs.getString(key, null)

    fun remove(key: String) = prefs.edit().remove(key).apply()

    fun clearAll() = prefs.edit().clear().apply()
}
```

### 6. Ktor *client* + Kotlinx.serialization *setup*

```text
app/src/main/java/com/yourapp/network/
   KtorClient.kt
```

```kotlin
import io.ktor.client.*
import io.ktor.client.engine.okhttp.*
import io.ktor.client.plugins.contentnegotiation.*
import io.ktor.serialization.kotlinx.json.*
import kotlinx.serialization.json.Json

val ktorClient = HttpClient(OkHttp) {
    engine {
        // configure OkHttp if needed (timeouts, interceptors)
    }
    install(ContentNegotiation) {
        json(Json { ignoreUnknownKeys = true })
    }
}
```

### 7. Construir la URL de autorización y abrir *Custom Tab*
#### Implementación: *Start Auth*
```text
app/src/main/java/com/yourapp/auth/
   AuthLauncher.kt
```

```kotlin
import androidx.browser.customtabs.CustomTabsIntent
import android.net.Uri

object AuthLauncher {
    fun startAuth(activity: Activity, storage: OAuthStorage) {
        val state = OAuthPKCE.generateState()
        val codeVerifier = OAuthPKCE.generateCodeVerifier()
        val codeChallenge = OAuthPKCE.generateCodeChallenge(codeVerifier)

        val scopes = listOf(
            "openid",
            "email",
            "profile",
            "https://www.googleapis.com/auth/userinfo.profile", // UserInfo
            "https://www.googleapis.com/auth/userinfo.email", // UserInfo
            "https://www.googleapis.com/auth/user.birthday.read", // People API
            "https://www.googleapis.com/auth/user.phonenumbers.read", // People API
            "https://www.googleapis.com/auth/user.addresses.read" // People API
        )

        storage.save("state", state)
        storage.save("code_verifier", codeVerifier)

        val uri = Uri.parse(OAuthConfig.AUTHORIZATION_ENDPOINT)
            .buildUpon()
            .appendQueryParameter("client_id", OAuthConfig.CLIENT_ID)
            .appendQueryParameter("redirect_uri", OAuthConfig.REDIRECT_URI_GOOGLE)
            .appendQueryParameter("response_type", "code")
            .appendQueryParameter("scope", scopes.joinToString(" "))
            .appendQueryParameter("state", state)
            .appendQueryParameter("code_challenge", codeChallenge)
            .appendQueryParameter("code_challenge_method", "S256")
            .appendQueryParameter("access_type", "offline") // request refresh_token
            .appendQueryParameter("prompt", "consent") // ensure refresh_token is returned
            .build()

        val customTabsIntent = CustomTabsIntent.Builder().build()
        customTabsIntent.launchUrl(activity, uri)
    }
}
```

#### Uso: *Login screen*

```text
app/src/main/java/com/yourapp/ui/auth/
   LoginActivity.kt
```

```kotlin
import android.os.Bundle
import android.widget.Button
import androidx.activity.ComponentActivity

class LoginActivity : ComponentActivity() {
    private val storage by lazy { OAuthStorage(applicationContext) }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_login)

        val loginButton: Button = findViewById(R.id.loginGoogleButton)
        loginButton.setOnClickListener {
            // Lanza Custom Tab para iniciar OAuth + PKCE
            AuthLauncher.startAuth(this, storage)
        }
    }
}
```

### 8. *Callback Activity*
> ℹ️ Notas:  
> Recibe el ``code``, valida el ``state`` y ejecuta el intercambio por los _tokens_ (ver implementación en el paso siguiente).  
> También maneja los posibles errores.

```kotlin
class GoogleRedirectActivity : ComponentActivity() {
    private val storage by lazy { OAuthStorage(applicationContext) }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val uri = intent?.data
        if (uri == null) { finish(); return }

        val error = uri.getQueryParameter("error")
        val code = uri.getQueryParameter("code")
        val returnedState = uri.getQueryParameter("state")
        val originalState = storage.read("state")
        storage.remove("state") // avoid reuse

        when {
            error != null -> {
                // e.g. access_denied
                showError("Auth error: $error"); finish(); return
            }
            returnedState == null || returnedState != originalState -> {
                showError("State mismatch"); finish(); return
            }
            code == null -> {
                showError("Authorization code missing"); finish(); return
            }
            else -> {
                // Exchange code for tokens immediately
                lifecycleScope.launch {
                    val verifier = storage.read("code_verifier")
                    if (verifier == null) {
                        showError("Missing code_verifier; retry auth")
                        finish(); return@launch
                    }
                    try {
                        val token = exchangeCodeForToken(code, verifier)
                        
                        // ============================================================
                        // 1️⃣ Opción sin Nimbus: Validaciones lógicas, no criptográficas
                        // ============================================================
                        // Decodificar payload del id_token
                        val claims = decodeIdTokenClaims(token.id_token!!)
                        val aud = claims["aud"]
                        val iss = claims["iss"]
                        val exp = claims["exp"]?.toLongOrNull()
                        val now = System.currentTimeMillis() / 1000

                        // Validaciones mínimas
                        if (!aud.contains(OAuthConfig.CLIENT_ID)) {
                            showError("Token con 'aud' inválido")
                            return
                        }

                        if (iss != "https://accounts.google.com" && iss != "accounts.google.com") {
                            showError("Token con 'iss' inválido")
                            return
                        }

                        if (exp == null || exp < now) {
                            showError("Token expirado")
                            return
                        }

                        // ============================================================
                        // 2️⃣ Opción con Nimbus: Validaciones criptográficas sobre la firma del id_token
                        // ============================================================
                        val isValid = verifyIdTokenWithNimbus(token.id_token!!)
                        if (!isValid) {
                            showError("Invalid ID token")
                            return
                        }

                        // Todo OK → guardar tokens
                        saveTokens(token, storage)
                        storage.remove("code_verifier")

                        startActivity(Intent(this@GoogleRedirectActivity, ProfileActivity::class.java))
                    } catch (e: OAuthExchangeException) {
                        showError("Token exchange failed: ${e.error} - ${e.description}")
                    } catch (e: Exception) {
                        showError("Unexpected: ${e.message}")
                    } finally {
                        finish()
                    }
                }
            }
        }
    }

    private fun showError(msg: String) {
        Toast.makeText(this, msg, Toast.LENGTH_LONG).show()
    }
}
```

### 9. Intercambio del ``code`` por los *tokens*
> ℹ️ Notas:  
> Errores concretos que se podrían recibir:
> - ``invalid_grant`` :arrow_right: ``code`` expired / ``code_verifier`` mismatch / ``code`` reused 
> - ``invalid_client`` :arrow_right: misconfig (``client_id``)
> - ``invalid_scope`` :arrow_right: ``scopes`` incorrectos

```text
app/src/main/java/com/yourapp/auth/
   TokenExchange.kt
```

```kotlin
import io.ktor.client.request.*
import io.ktor.client.statement.*
import io.ktor.http.*
import kotlinx.serialization.Serializable
import kotlinx.serialization.json.Json
import kotlinx.coroutines.Dispatchers
import kotlinx.coroutines.withContext

@Serializable
data class TokenResponse(
    val access_token: String? = null,
    val refresh_token: String? = null,
    val expires_in: Long? = null,
    val id_token: String? = null,
    val token_type: String? = null,
    val scope: String? = null
)

@Serializable
data class OAuthError(
    val error: String? = null,
    val error_description: String? = null
)

class OAuthExchangeException(val error: String, val description: String, val httpCode: Int) : Exception("$error: $description")

suspend fun exchangeCodeForToken(code: String, codeVerifier: String): TokenResponse = withContext(Dispatchers.IO) {
    val response: HttpResponse = ktorClient.post(OAuthConfig.TOKEN_ENDPOINT) {
        contentType(ContentType.Application.FormUrlEncoded)
        setBody(listOf(
            "grant_type" to "authorization_code",
            "client_id" to OAuthConfig.CLIENT_ID,
            "code" to code,
            "code_verifier" to codeVerifier,
            "redirect_uri" to OAuthConfig.REDIRECT_URI_GOOGLE
        ).formUrlEncode())
    }

    val body = response.bodyAsText()
    return@withContext if (response.status.isSuccess()) {
        Json { ignoreUnknownKeys = true }.decodeFromString(TokenResponse.serializer(), body)
    } else {
        // try parse standard OAuth error
        val oauthErr = try {
            Json { ignoreUnknownKeys = true }.decodeFromString(OAuthError.serializer(), body)
        } catch (e: Exception) { null }
        val err = oauthErr?.error ?: "unknown_error"
        val desc = oauthErr?.error_description ?: body
        throw OAuthExchangeException(err, desc, response.status.value)
    }
}
```

### 10. Guardar *tokens* de forma segura
> ℹ️ Nota:  
> Para ``refresh_token`` a largo plazo, es preferible gestionarlo en _backend_ si es posible.

```text
app/src/main/java/com/yourapp/auth/
   TokenRepository.kt
```

```kotlin
object TokenRepository {
    fun saveTokens(token: TokenResponse, storage: OAuthStorage) {
        token.access_token?.let { storage.save("access_token", it) }
        token.refresh_token?.let { storage.save("refresh_token", it) }
        token.id_token?.let { storage.save("id_token", it) }
        token.expires_in?.let { storage.save("expires_in", it.toString()) }
    }
}
```

### 11. Obtener información del usuario
#### A. Usar ``id_token`` (OIDC) — Decodificar ``claims``
> ℹ️ Notas:  
> Validaciones mínimas recomendadas:
> - ``iss`` debería ser ``https://accounts.google.com`` o ``accounts.google.com``
> - ``aud`` debe ser igual al ``CLIENT_ID`` (o contenerlo)
> - ``exp`` debe estar en el futuro 
> 
> **Pero esto NO verifica la firma**. Para seguridad completa, verificar la firma (ver más abajo con [Nimbus](#12-verificar-firma-del-id_token-con-nimbus)).

```text
app/src/main/java/com/yourapp/auth/
    IdTokenUtils.kt
```

```kotlin
import android.util.Base64
import kotlinx.serialization.json.Json
import kotlinx.serialization.json.jsonObject

object IdTokenUtils {
    fun decodeIdTokenClaims(idToken: String): Map<String, String> {
        val parts = idToken.split(".")
        require(parts.size == 3)
        val payload = parts[1]
        val decoded = Base64.decode(payload, Base64.URL_SAFE or Base64.NO_PADDING or Base64.NO_WRAP)
        val json = String(decoded, Charsets.UTF_8)
        val obj = Json.parseToJsonElement(json).jsonObject
        return obj.mapValues { it.value.toString().trim('"') }
    }
}
```

#### B. Usar ``/userinfo`` (recomendado para datos estándar)

```text
app/src/main/java/com/yourapp/auth/api/
    GoogleApiService.kt
```

```kotlin
@Serializable
data class GoogleUser(
    val sub: String,
    val email: String? = null,
    val name: String? = null,
    val picture: String? = null
)

suspend fun fetchUserInfo(accessToken: String): GoogleUser = withContext(Dispatchers.IO) {
    val resp = ktorClient.get(OAuthConfig.USERINFO_ENDPOINT) {
        header("Authorization", "Bearer $accessToken")
    }
    val body = resp.bodyAsText()
    if (resp.status.isSuccess()) {
        Json { ignoreUnknownKeys = true }.decodeFromString(GoogleUser.serializer(), body)
    } else {
        throw Exception("UserInfo error: $body")
    }
}
```

#### C. Usar *Google People API*
> ℹ️ Nota:  
> Si se necesitan más campos, requiere otros ``scopes`` (ver [acá](#7-construir-la-url-de-autorización-y-abrir-custom-tab))

```text
app/src/main/java/com/yourapp/network/
   GoogleApiService.kt
```

```kotlin
// En lugar de un String, también se podría deserializar a objeto para mayor consistencia con /userinfo
suspend fun fetchPeopleApi(accessToken: String): String = withContext(Dispatchers.IO) {
    // Example: request names, emailAddresses, photos
    val url = "${OAuthConfig.PEOPLE_API_ENDPOINT}?personFields=names,emailAddresses,photos"
    val resp = ktorClient.get(url) {
        header("Authorization", "Bearer $accessToken")
    }
    val body = resp.bodyAsText()
    if (resp.status.isSuccess()) body
    else throw Exception("People API error: $body")
}
```

#### Uso: *User Profile screen*
```text
app/src/main/java/com/yourapp/ui/profile/
   ProfileActivity.kt
```

```kotlin
import android.os.Bundle
import android.widget.TextView
import androidx.activity.ComponentActivity
import androidx.lifecycle.lifecycleScope
import com.yourapp.auth.OAuthStorage
import com.yourapp.auth.api.fetchUserInfo
import com.yourapp.network.fetchPeopleApi
import kotlinx.coroutines.launch

class ProfileActivity : ComponentActivity() {
    private val storage by lazy { OAuthStorage(applicationContext) }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_profile)

        val nameText: TextView = findViewById(R.id.nameText)
        val emailText: TextView = findViewById(R.id.emailText)

        lifecycleScope.launch {
            val accessToken = storage.read("access_token") ?: return@launch

            // 1️⃣ Llamada a /userinfo
            val userInfo = fetchUserInfo(accessToken)
            nameText.text = userInfo.name
            emailText.text = userInfo.email

            // 2️⃣ Llamada opcional a Google People API
            val peopleJson = fetchPeopleApi(accessToken)
            // procesar JSON según se necesite
        }
    }
}
```

### 12. Verificar firma del ``id_token`` con Nimbus
> ℹ️ Notas:  
> - Es un **paso opcional** realizado típicamente después de realizar el intercambio del ``code`` por los _tokens_ y antes de guardar dichos _tokens_
> - **Por qué**: decodificar claims NO garantiza que el _token_ fue emitido por Google. Verificar firma JWKs asegura autenticidad
> - **IMPORTANTE**: En producción se deberían cachear las JWKs y manejar errores de red; no consultar JWKs en cada verificación

```text
app/src/main/java/com/yourapp/auth/
    IdTokenUtils.kt
```

```kotlin
import com.nimbusds.jose.JWSHeader
import com.nimbusds.jose.JWSVerifier
import com.nimbusds.jose.crypto.RSASSAVerifier
import com.nimbusds.jose.jwk.JWKMatcher
import com.nimbusds.jose.jwk.JWKSelector
import com.nimbusds.jose.jwk.RSAKey
import com.nimbusds.jose.jwk.source.RemoteJWKSet
import com.nimbusds.jwt.SignedJWT
import com.nimbusds.jose.proc.SecurityContext
import java.net.URL
import java.util.Date

object IdTokenUtils {
    suspend fun verifyIdTokenWithNimbus(idToken: String): Boolean {
        // 1) parse token
        val jwt = SignedJWT.parse(idToken)
        val kid = jwt.header.keyID

        // 2) fetch Google's JWK set (cache in prod!)
        val jwkSetUrl = URL("https://www.googleapis.com/oauth2/v3/certs")
        val jwkSource = RemoteJWKSet<SecurityContext>(jwkSetUrl)

        // 3) find key and verify
        val keys = jwkSource.get(JWKSelector(JWKMatcher.Builder().keyID(kid).build()), null)
        if (keys.isEmpty()) return false
        val jwk = keys[0] as RSAKey
        val pubKey = jwk.toRSAPublicKey()

        val verifier = RSASSAVerifier(pubKey)
        val signatureValid = jwt.verify(verifier)
        if (!signatureValid) return false

        // 4) check claims (iss, aud, exp)
        val claims = jwt.jwtClaimsSet
        val iss = claims.issuer
        val aud = claims.audience
        val exp = claims.expirationTime

        if (iss != "https://accounts.google.com" && iss != "accounts.google.com") return false
        if (!aud.contains(OAuthConfig.CLIENT_ID)) return false
        if (exp == null || exp.before(Date())) return false

        return true
    }
}
```

### 13. Revocación + *Logout*
#### Implementación: *Revoke* & *Logout*

```text
app/src/main/java/com/yourapp/auth/
    SessionManager.kt
```

```kotlin
object SessionManager {
    suspend fun revokeToken(token: String) {
        val resp = ktorClient.post(OAuthConfig.REVOCATION_ENDPOINT) {
            contentType(ContentType.Application.FormUrlEncoded)
            setBody(listOf("token" to token).formUrlEncode())
        }
        if (resp.status.value == 200 || resp.status.value == 400) {
            // success or already invalid
            return
        } else {
            throw Exception("Revoke failed: HTTP ${resp.status.value} - ${resp.bodyAsText()}")
        }
    }

    suspend fun logoutComplete(storage: OAuthStorage) {
        storage.read("refresh_token")?.let {
            try {
                revokeToken(it)
            } catch (_: Exception) { /* continue */
            }
        }
        storage.read("access_token")?.let {
            try {
                revokeToken(it)
            } catch (_: Exception) { /* continue */
            }
        }
        storage.clearAll()
        // Optionally clear browser cookies:
        CookieManager.getInstance().removeAllCookies(null)
        CookieManager.getInstance().flush()
    }
}
```

#### Uso: *Settings screen*
```text
app/src/main/java/com/yourapp/ui/settings/
   AuthLauncher.kt
```

```kotlin
import android.content.Intent
import android.os.Bundle
import android.widget.Button
import androidx.activity.ComponentActivity
import androidx.lifecycle.lifecycleScope
import com.yourapp.auth.OAuthStorage
import com.yourapp.auth.SessionManager
import com.yourapp.ui.auth.LoginActivity
import kotlinx.coroutines.launch

class SettingsActivity : ComponentActivity() {
    private val storage by lazy { OAuthStorage(applicationContext) }

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_settings)

        val logoutButton: Button = findViewById(R.id.logoutButton)
        logoutButton.setOnClickListener {
            lifecycleScope.launch {
                // Revoke tokens
//                storage.read("refresh_token")?.let { SessionManager.revokeToken(it) }
//                storage.read("access_token")?.let { SessionManager.revokeToken(it) }

                // Revoke tokens y limpiar storage seguro
                SessionManager.logoutComplete(storage)

                // Redirigir a LoginActivity
                startActivity(Intent(this@SettingsActivity, LoginActivity::class.java))
                finish()
            }
        }
    }
}
```
