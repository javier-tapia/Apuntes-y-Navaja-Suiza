<h1><i>Networking & APIs</i></h1>

***Index***:
<!-- TOC -->
  * [*Retrofit*](#retrofit)
    * [Dependencias y permisos](#dependencias-y-permisos)
    * [Modelo de Respuesta](#modelo-de-respuesta)
    * [*Interface API service*](#interface-api-service)
    * [Cliente OkHttp](#cliente-okhttp)
    * [Interceptores de OkHttp: diferencias y usos recomendados](#interceptores-de-okhttp-diferencias-y-usos-recomendados)
      * [🛂 *Interceptor (Application Interceptor)*](#-interceptor-application-interceptor)
      * [🌐 *Network Interceptor*](#-network-interceptor)
    * [Instancia de Retrofit](#instancia-de-retrofit)
    * [Manejo de respuestas y errores en Retrofit](#manejo-de-respuestas-y-errores-en-retrofit)
  * [*Ktor*](#ktor)
  * [*OAuth: Facebook, Twitter, Google+*](#oauth-facebook-twitter-google)
  * [*Frameworks y SDK's: Firebase, Fabric, Sentry, Segment, Facebook*](#frameworks-y-sdks-firebase-fabric-sentry-segment-facebook)
<!-- TOC -->

---

## *Retrofit*
> 🔍 Referencias:  
> https://square.github.io/retrofit/  
> https://github.com/square/retrofit  
> https://square.github.io/okhttp/  

Es una librería con **seguridad de tipo** (_type-safe_) para **_realizar solicitudes HTTP_** y **_mapear las respuestas_** a objetos previamente modelados (con _data class_ en Kotlin).  
No tiene injerencia sobre _cache_, _retries_ ni _logging_. Estas responsabilidades recaen completamente en [OkHttp](#cliente-okhttp), no en Retrofit.

### Dependencias y permisos
Agregar las dependencias necesarias en el archivo ``libs.versions.toml``:

```toml
[versions]
kotlinSerialization = "{VERSION}"
retrofit = "{VERSION}"
okhttp = "{VERSION}"

[libraries]
kotlinx-serialization-json = { module = "org.jetbrains.kotlinx:kotlinx-serialization-json", version.ref = "kotlinSerialization" }
retrofit = { module = "com.squareup.retrofit2:retrofit", version.ref = "retrofit" }
retrofit-converter-kotlinx = { module = "com.squareup.retrofit2:converter-kotlinx-serialization", version.ref = "retrofit" }
okhttp-logging = { module = "com.squareup.okhttp3:logging-interceptor", version.ref = "okhttp" }
```

Implementar las dependencias en el archivo ``build.gradle.kts`` del módulo que corresponda:

```kotlin
// Kotlinx Serialization
implementation(libs.kotlinx.serialization.json)

// Retrofit & OkHttp
implementation(libs.retrofit)
implementation(libs.retrofit.converter.kotlinx)
implementation(libs.okhttp.logging)
```

Agregar el permiso de internet en el ``Manifest``:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

### Modelo de Respuesta
> :warning: Es importante destacar que Retrofit no trae soporte nativo para KotlinX Serialization, ya que no es parte de Retrofit, sino un _converter_ externo. Gson, en cambio, sí tiene soporte oficial porque es el _converter_ por defecto recomendado históricamente.  
> Para usar el converter de KotlinX Serialization, se requiere agregar el módulo ``converter-kotlinx-serialization``.

A las propiedades de las _data class_ que modelan la respuesta del servicio, se les puede agregar una _annotation_ (por ejemplo, ``@SerializedName`` para Gson o ``@SerialName`` para KotlinX Serialization) y pasarle el nombre del atributo. Esto permite que el modelo sea agnóstico a los nombres "reales" y se pueda reutilizar más fácilmente. Dichas anotaciones son opcionales si el nombre de la propiedad coincide con el JSON.  
Además, en caso de usar el ``Converter`` propio de KotlinX Serialization, se debe anotar la clase con ``@Serializable``.

```kotlin
@Serializable
data class UserResponse(
    @SerialName("user_id")
    val userId: String,
    @SerialName("name")
    val name: String,
    @SerialName("nickname")
    val nickname: String,
    @SerialName("followers")
    val followers: Int,
    @SerialName("following")
    val following: List<String>,
    @SerialName("user_type")
    val userType: Int,
)
```

### *Interface API service*
Crear una interfaz que declare los métodos para realizar las solicitudes HTTP y el tipo de retorno, el cual puede estar encapsulado en un ``Response<T>`` (ver [Manejo de respuestas y errores en Retrofit](#manejo-de-respuestas-y-errores-en-retrofit)). Esto es opcional y se podría usar un tipo definido por el desarrollador directamente, pero ``Response`` sirve para leer _headers_, verificar si la respuesta fue exitosa con ``isSuccessful``, acceder al código de respuesta con ``code()`` y manejar errores de forma más controlada.  
Se anotan con el verbo de la llamada y, opcionalmente, se pueden pasar parámetros como _headers_, _query params_, _body_ (para los ``POST``, ``PUT`` o ``PATCH``), entre otros.

```kotlin
interface SampleApiService {
    @POST("update_user")
    suspend fun updateUser(
        @Header("Journey-Id") journeyId: String,
        @Query("session_id") sessionId: String?,
        @Body sampleBody: SampleBody?,
    ): Response<SampleUpdateResponse>

    @GET("users/{id}")
    suspend fun fetchUser(
        @Header("Journey-Id") journeyId: String,
        @Query("session_id") sessionId: String?,
        @Path("id") id: String,
    ): Response<SampleFetchResponse>
}
```

### Cliente OkHttp
> ⚠️ Importante:  Para poder utilizar ``BuildConfig``, se debe agregar la _flag_ ``buildConfig = true`` dentro del bloque ``android.buildFeatures`` en el archivo ``build.gradle.kts(:app)``

Retrofit usa OkHttp internamente como cliente HTTP, NO lo reemplaza. Por eso es posible personalizarlo antes de pasárselo a Retrofit.  
Permite configurar el comportamiento real de las conexiones HTTP, incluyendo interceptores, _timeouts_, _logging_, políticas de reintento, _cache_ y _headers_ globales (ver apartado siguiente sobre [interceptores](#interceptores-de-okhttp-diferencias-y-usos-recomendados)).

Ejemplo:

```kotlin
val okHttpClient = OkHttpClient.Builder()
    .connectTimeout(20, TimeUnit.SECONDS)
    .readTimeout(20, TimeUnit.SECONDS)
    .writeTimeout(20, TimeUnit.SECONDS)
    .addInterceptor(HttpLoggingInterceptor().apply {
        level = if (BuildConfig.DEBUG) {
            HttpLoggingInterceptor.Level.BODY
        } else {
            HttpLoggingInterceptor.Level.NONE
        }
    })
    .build()
```

### Interceptores de OkHttp: diferencias y usos recomendados
OkHttp permite agregar dos tipos de interceptores, que se ejecutan en distintos momentos del ciclo de una _request_.

#### 🛂 *Interceptor (Application Interceptor)*

Se ejecuta **_antes de que la request "le pegue" a la red_**.  
Ideal para:

- Agregar _headers_ globales 
- Autenticación (_Tokens_, API _Keys_)
- _Logging_ general 
- _Retries_ personalizados 
- Reescritura de _requests_ o respuestas

Además:

- Se ejecuta **_una sola vez_**, incluso si hay redirecciones. 
- No ve respuestas del _cache_ (solo se vería un “_cache hit_” si el desarrollador implementa una respuesta manual, pero OkHttp no lo hace automáticamente).

Ejemplo:

```kotlin
val okHttpClient = OkHttpClient.Builder()
    .addInterceptor { chain ->
        val newRequest = chain.request()
            .newBuilder()
            .addHeader("User-Agent", "MyApp/1.0")
            .build()

        chain.proceed(newRequest)
    }
```

#### 🌐 *Network Interceptor*

Se ejecuta **_después de que OkHttp "le pega" a la red_**.  
Ideal para:

- _Logging_ de _requests_/_responses_ tal como fueron enviadas/recibidas 
- Inspección de _headers_ agregados por el servidor o _proxies_ 
- Manipular recursos _cacheados_ 
- Verificar certificados u operaciones que requieren acceso al _socket_

Además:

- Se ejecuta **_en cada redirección_**. 
- Se ejecuta **_siempre_**, incluso con _cache_.

Ejemplo:

```kotlin
val okHttpClient = OkHttpClient.Builder()
    .addNetworkInterceptor { chain ->
        val response = chain.proceed(chain.request())
        // Ideal para depurar headers reales enviados/recibidos
        response
    }
```

### Instancia de Retrofit
Para crear una instancia de Retrofit, es necesario llamar al _builder_ y configurar lo que se requiera. Esto puede hacerse en un archivo separado o como parte de un inyector de dependencias.

Consta de algunos elementos comunes:
- ``baseUrl`` :arrow_right: Configura la URL base de la API a consumir. Debe terminar con la ``/``.
- ``addConverterFactory`` :arrow_right: Agrega un _factory_ que creará una instancia del conversor que permitirá serializar y deserializar objetos. El más habitual suele ser **_Gson_**, pero existen varios más, incluyendo el de **_KotlinX Serialization_**.

```kotlin
val json = Json {
    ignoreUnknownKeys = true
    isLenient = true // Solo si es absolutamente necesario (APIs que no cumplen estándares)
}

Retrofit
    .Builder()
    .baseUrl("https://api.miservicio.com/")
    .addConverterFactory(
        json.asConverterFactory("application/json; charset=UTF-8".toMediaType())
        // También es bastante habitual utilizar el Converter de Gson:
        // GsonConverterFactory.create()
    )
    .client(okHttpClient) // Cliente OkHttp creado en el paso previo
    .build()
    .create(SampleApiService::class.java)
```

### Manejo de respuestas y errores en Retrofit
> ⚠️ Importante:  
> Retrofit **NO lanza excepción** en errores HTTP (4xx/5xx).  
> Retrofit **SÍ lanza excepción** en errores de red (_timeout_, DNS, desconexión, SSL) o serialización.

El manejo completo implica distinguir tres niveles:

1. **Excepciones de red o serialización (_throw_)**
Ocurre antes de recibir respuesta (conectividad, timeout, SSL…).

```kotlin
return try {
    val response = api.fetchUser(...)
    // Pasa al punto 2
    handleResponse(response)
} catch (e: Exception) {
    // Error de red o serialización
    Result.Error("Network error: ${e.localizedMessage}")
}
```

2. **Respuesta HTTP exitosa o con error (2xx / 4xx / 5xx)**
Retrofit devuelve un ``Response<T>``.

```kotlin
fun handleResponse(response: Response<SampleFetchResponse>): Result<SampleFetchResponse> {
    if (response.isSuccessful) {
        val body = response.body()
        return if (body != null) {
            Result.Success(body)
        } else {
            Result.Error("Response body is null")
        }
    } else {
        // Error 4xx / 5xx
        val code = response.code()
        val errorMsg = response.errorBody()?.string()

        return Result.Error("HTTP $code: $errorMsg")
    }
}
```

3. **Mapeo final a un modelo de dominio**
Se puede estandarizar. Revisar también el uso de [``Either``](../../Code%20Snippets%20with%20Kotlin/JSON%20operations%20&%20Error%20handling%20with%20Either.md#sealed-class-either-left-and-right)

```kotlin
sealed class Result<out T> {
    data class Success<T>(val data: T) : Result<T>()
    data class Error(val message: String) : Result<Nothing>()
}
```

## *Ktor*
TODO...

## *OAuth: Facebook, Twitter, Google+*
TODO...

## *Frameworks y SDK's: Firebase, Fabric, Sentry, Segment, Facebook*
TODO...
