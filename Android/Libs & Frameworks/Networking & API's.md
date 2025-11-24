<h1><i>Networking & APIs</i></h1>

***Index***:
<!-- TOC -->
  * [*Retrofit*](#retrofit)
    * [🚀 Cheatsheet Retrofit](#-cheatsheet-retrofit)
    * [Dependencias y permisos](#dependencias-y-permisos)
    * [Modelo de Respuesta](#modelo-de-respuesta)
    * [*Interface API service*](#interface-api-service)
    * [Cliente OkHttp](#cliente-okhttp)
    * [Interceptores de OkHttp: diferencias y usos recomendados](#interceptores-de-okhttp-diferencias-y-usos-recomendados)
      * [🛂 *Application Interceptor*](#-application-interceptor)
      * [🌐 *Network Interceptor*](#-network-interceptor)
    * [Instancia de Retrofit](#instancia-de-retrofit)
    * [Manejo de respuestas y errores](#manejo-de-respuestas-y-errores)
  * [*Ktor* (cliente)](#ktor-cliente)
    * [🚀 Cheatsheet Ktor Client](#-cheatsheet-ktor-client)
    * [Dependencias y permisos](#dependencias-y-permisos-1)
    * [Modelo de respuesta con KotlinX Serialization](#modelo-de-respuesta-con-kotlinx-serialization)
    * [Configuración del cliente HTTP](#configuración-del-cliente-http)
    * [Realizar solicitudes](#realizar-solicitudes)
    * [Manejo de respuestas y errores](#manejo-de-respuestas-y-errores-1)
  * [*OAuth: Facebook, Twitter, Google+*](#oauth-facebook-twitter-google)
  * [*Frameworks y SDK's: Firebase, Fabric, Sentry, Segment, Facebook*](#frameworks-y-sdks-firebase-fabric-sentry-segment-facebook)
<!-- TOC -->

---

## *Retrofit*
> 🔍 Referencias:  
> https://square.github.io/retrofit/  
> https://github.com/square/retrofit  
> https://square.github.io/okhttp/  
> https://johncodeos.com/how-to-make-post-get-put-and-delete-requests-with-retrofit-using-kotlin/

Es una librería con **seguridad de tipo** (_type-safe_) para **_realizar solicitudes HTTP_** y **_mapear las respuestas_** a objetos previamente modelados (con _data class_ en Kotlin).  
No tiene injerencia sobre _cache_, _retries_ ni _logging_. Estas responsabilidades recaen completamente en [OkHttp](#cliente-okhttp), no en Retrofit.

### 🚀 Cheatsheet Retrofit
1. **Definir el modelo de datos**

```kotlin
data class UserDto(
    val id: String,
    val name: String
)
```

2. **Definir interfaz del servicio**

```kotlin
interface UserApi {
    @GET("users/{id}")
    suspend fun fetchUser(
        @Path("id") id: String
    ): Response<UserDto>
}
```

3. **Crear instancia de Retrofit**

```kotlin
val retrofit = Retrofit.Builder()
    .baseUrl("https://api.example.com/")
    .addConverterFactory(GsonConverterFactory.create())
    .build()
```

4. **Crear implementación del servicio (una por cada interfaz en caso de haber más)**

```kotlin
val api = retrofit.create(UserApi::class.java)
```

5. **Ejecutar _request_ + Manejo de respuesta y errores**

```kotlin
suspend fun getUser(id: String): Result<UserDto> {
    return try {
        // Solicitud a la red
        val response = api.fetchUser(id)

        // Otras operaciones
    } catch (e: Exception) {
        // Gestionar errores
    }
}
```

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
data class UserDto(
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
Crear una interfaz que declare los métodos para realizar las solicitudes HTTP y el tipo de retorno, el cual puede estar encapsulado en un ``Response<T>`` (ver [Manejo de respuestas y errores](#manejo-de-respuestas-y-errores)). Esto es opcional y se podría usar un tipo definido por el desarrollador directamente, pero ``Response`` sirve para leer _headers_, verificar si la respuesta fue exitosa con ``isSuccessful``, acceder al código de respuesta con ``code()`` y manejar errores de forma más controlada.  
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

| Característica                | 🛂 **Application Interceptor**                                                                            | 🌐 **Network Interceptor**                                                                                       |
|-------------------------------|-----------------------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------------|
| **Cuándo se ejecuta**         | Antes de cualquier acceso a red                                                                           | Justo antes y después de tocar la red (socket)                                                                   |
| **Redirecciones**             | ❌ Se ejecuta **una sola vez**                                                                             | ✔️ Se ejecuta **por cada redirección**                                                                           |
| **Cache**                     | ✔️ Puede ver respuestas de cache                                                                          | ❌ **No** se ejecuta **cuando OkHttp responde desde el cache** (**sin tocar la red**)                             |
| **Logging**                   | Logging "lógico" (lo que el código envía)                                                                 | Logging "real" (lo que realmente se envió/recibió en red)                                                        |
| **Modificación de request**   | ✔️ Ideal para agregar headers o reescribir requests                                                       | ✔️ Puede modificar request pero ya "casi finalizada"                                                             |
| **Modificación de respuesta** | ✔️ Puede modificarla (incluyendo respuestas cacheadas), pero no es lo habitual                            | ✔️ Puede modificarla (solo cuando viene de red)                                                                  |
| **Casos ideales**             | - Headers globales<br>- Auth tokens<br>- Retries lógicos<br>- Logging de negocio<br>- Reescritura general | - TLS / Certificado<br>- Logging de red real<br>- Inspección de proxies/servidor<br>- Manejo específico de cache |
| **Acceso al socket**          | ❌ No                                                                                                      | ✔️ Sí                                                                                                            |
| **Uso más común**             | Interceptor general de la app                                                                             | Interceptor para debugging, inspección y validaciones profundas de red                                           |
| **Ejecutado sobre**           | La **request original**                                                                                   | La **request final** (después de compresión, headers automáticos, etc.)                                          |

#### 🛂 *Application Interceptor*
Se ejecuta **_antes de que la request llegue a la red_**, actuando en la capa más externa del OkHttpClient. En resumen: es el **_interceptor a usar para lógica de la aplicación_**, sin preocuparte por detalles de transporte o red.

Es ideal para:
- Agregar **_headers_ globales** 
- Manejar **autenticación** (_Tokens_, API _Keys_, _Bearer_, etc.)
- **_Logging_ general** que no dependa de la red
- **_Retries_ personalizados** que se quieran controlar manualmente
- Reescritura de **_requests_ y respuestas** a nivel de aplicación

Comportamiento clave:
- Se ejecuta **una sola vez por _request_**, incluso si hay _redirects_ o _retries_ internos de OkHttp.
- **Ve respuestas provenientes del _cache_**, porque OkHttp puede resolver una _request_ desde disco antes de tocar la red. 
- **No puede modificar la política del _cache_** (qué se guarda, cuándo expira, cómo se revalida), solo puede ver el resultado final. 
- No ve la versión final de la _request_ tal como OkHttp la enviaría por red, porque no participa en las transformaciones de bajo nivel (compresión, _headers_ automáticos, etc.).

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
Se ejecuta dos veces por ciclo de red: **_una al enviar la request al servidor_** y **_otra al recibir la respuesta desde la red_**. En resumen: es el **_interceptor para lógica estrictamente de red_**, no para lógica de aplicación.

Es ideal para:
- **_Logging_ real de red** (lo que realmente se envió y lo que realmente llegó)
- Inspeccionar **_headers_ generados por el servidor, _proxies_ o _gateways_** 
- Manipular _headers_ relacionados con **_cache_** (_Cache-Control_, _ETag_, _If-Modified-Since_)
- Operaciones que requieren acceso directo a la **conexión** (certificados, TLS, tamaño real de payload, etc.)

Comportamiento clave:
- Se ejecuta **en cada redirección**, porque cada salto reenvía la _request_ al servidor.
- **No se ejecuta cuando la respuesta proviene del _cache_** :arrow_right: Solo corre cuando hay un acceso real a la red.
- Ve la _request_ **después** de que OkHttp aplicó todas las transformaciones finales (como compresión o _headers_ automáticos).
- Puede modificar la respuesta **antes de que llegue a la capa superior**, lo cual es útil para casos muy específicos (no recomendado para lógica general).

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
    .create(SampleApiService::class.java) // Implementación del servicio
```

### Manejo de respuestas y errores
> ⚠️ Importante:  
> Retrofit **NO lanza excepción** en errores HTTP (4xx/5xx) al usar ``Response<T>``. Solo lanza ``HttpException`` en errores HTTP si el método NO devuelve ``Response<T>``.  
> Retrofit **SÍ lanza excepción** en errores de red (_timeout_, DNS, desconexión, SSL) o serialización (JSON mal formado).

El manejo completo implica distinguir tres niveles:

1. **Excepciones de red o serialización (_throw_)**
Ocurre antes de recibir respuesta (conectividad, timeout, SSL…).

```kotlin
return try {
    val response = api.fetchUser()
    // Pasa al punto 2
    handleResponse(response)
} catch (e: IOException) {
    // Errores de red
    Result.Error("Network error: ${e.localizedMessage}")
} catch (e: MalformedJsonException) {
    // Errores de JSON
    Result.Error("Serialization error: ${e.localizedMessage}")
} catch (e: Exception) {
    // Errores inesperados
    Result.Error("Unexpected error: ${e.localizedMessage}")
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
Se puede estandarizar con un _wrapper_ propio, como puede ser ``Result``. Revisar también el uso de [``Either``](../../Code%20Snippets%20with%20Kotlin/JSON%20operations%20&%20Error%20handling%20with%20Either.md#sealed-class-either-left-and-right)

```kotlin
sealed class Result<out T> {
    data class Success<T>(val data: T) : Result<T>()
    data class Error(val message: String) : Result<Nothing>()
}
```

## *Ktor* (cliente)
> 🔍 Referencias:  
> https://ktor.io/  
> https://www.slf4j.org/  
> https://logback.qos.ch/  
> https://logging.apache.org/log4j/2.x/index.html

Ktor es un _framework_ para crear aplicaciones asincrónicas **del lado del servidor y del lado del cliente** con facilidad.  
Incluye un cliente HTTP asincrónico multiplataforma, que permite realizar solicitudes, manejar respuestas y ampliar su funcionalidad con _plugins_, como autenticación, serialización JSON y más.  
A diferencia de Retrofit, Ktor **no usa anotaciones ni interfaces**: se trabaja directamente con un cliente configurado y se realiza cada solicitud mediante la función `client.request{}`.

Para utilizar el cliente HTTP de Ktor en un proyecto Android, se deben configurar los repositorios y agregar las dependencias mandatorias y opcionales en caso de requerirlas.

### 🚀 Cheatsheet Ktor Client
1. **Definir el modelo de datos**

```kotlin
@kotlinx.serialization.Serializable
data class UserDto(
    val id: String,
    val name: String
)
```

2. **Crear instancia de Ktor Client**

```kotlin
val client = HttpClient {
    install(ContentNegotiation) {
        json() // kotlinx.serialization
    }
    install(HttpTimeout) {
        requestTimeoutMillis = 15_000
    }
    install(DefaultRequest) {
        url("https://myapi.com/")
        headers.appendIfNameAbsent("X-Custom-Header", "Hello")
    }
}
```

3. **Crear “servicio” (una clase por cada conjunto de _endpoints_)**

```kotlin
class UserApi(private val client: HttpClient) {
    suspend fun fetchUser(id: String): HttpResponse {
        return client.get("users/$id")
    }
}
```

4. **Instanciar el servicio**

```kotlin
val api = UserApi(client)
```

5. **Ejecutar _request_ + Manejo de respuesta y errores**

```kotlin
suspend fun getUser(id: String): Result<UserDto> {
    return try {
        val response = api.fetchUser(id)

        if (response.status.isSuccess()) {
            val body = response.body<UserDto>()
            Result.success(body)
        } else {
            Result.failure(
                Exception("HTTP ${response.status.value}: ${response.status.description}")
            )
        }

    } catch (e: Exception) {
        Result.failure(Exception("Network/serialization error: ${e.localizedMessage}"))
    }
}
```

### Dependencias y permisos
Luego de asegurarse que está agregado el repositorio ``mavenCentral()``, se pueden agregar las dependencias en el archivo ``libs.versions.toml``:

```toml
[versions]
ktor = "{VERSION}"
slf4j = "{VERSION}"

[libraries]
ktor-client-core = { module = "io.ktor:ktor-client-core", version.ref = "ktor" }
ktor-client-okhttp = { module = "io.ktor:ktor-client-okhttp", version.ref = "ktor" }
ktor-client-logging = { module = "io.ktor:ktor-client-logging", version.ref = "ktor" }
ktor-client-content-negotiation = { module = "io.ktor:ktor-client-content-negotiation", version.ref = "ktor" }
ktor-serialization-kotlinx-json = { module = "io.ktor:ktor-serialization-kotlinx-json", version.ref = "ktor" }
slf4j-android = { module = "org.slf4j:slf4j-android", version.ref = "slf4j" }
```

**A tener en cuenta**:

- La funcionalidad principal del cliente está disponible en el artefacto ``ktor-client-core``.
- Un **motor** (**_engine_**) se encarga de **procesar las solicitudes de red**. Existen diferentes motores de cliente disponibles para diversas plataformas, como Apache, CIO, Android, iOS, etc.
- Muchas aplicaciones requieren **funciones comunes que escapan a la lógica de la aplicación**. Estas pueden ser funciones como el _logging_, la serialización o la autorización. Todas estas funciones se proporcionan en Ktor mediante **_plugins_**.
- En JVM, Ktor utiliza **_Simple Logging Facade for Java_** (**_SLF4J_**) como una capa de abstracción para el _logging_. SLF4J desacopla la API de _logging_ de la implementación de _logging_ subyacente, lo que permite integrar el _framework_ de _logging_ que mejor se adapte a los requisitos de la aplicación. Las opciones más comunes incluyen **_Logback_** o **_Log4j_**. Si no se proporciona ningún _framework_, SLF4J utilizará por defecto una implementación sin operación (NOP), que básicamente deshabilita el _logging_.

Implementar las dependencias en el archivo ``build.gradle.kts`` del módulo que corresponda:

```kotlin
// Ktor
implementation(libs.ktor.client.core)
implementation(libs.ktor.client.okhttp)
implementation(libs.ktor.client.logging)
implementation(libs.ktor.client.content.negotiation)
implementation(libs.ktor.serialization.kotlinx.json)
implementation(libs.slf4j.android)
```

Agregar el permiso de internet en el ``Manifest``:

```xml
<uses-permission android:name="android.permission.INTERNET" />
```

### Modelo de respuesta con KotlinX Serialization
Ktor sí depende de KotlinX Serialization para el manejo de JSONs, por lo cual se requiere anotar los modelos con ``@Serializable``.

```kotlin
@Serializable
data class UserDto(
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

### Configuración del cliente HTTP
> ⚠️ Importante:  A diferencia de Retrofit, el cliente de Ktor sí debe cerrarse cuando ya no va a utilizarse.  
> Para eso, se llama a ``client.close()``.
> Si se usa DI esto no hace falta, ya que el cliente **se cierra automáticamente** al liberar el contenedor.

Ktor requiere configurar un **_engine_**: para Android/JVM, el más común es **OkHttp**. En KMP, se suele usar CIO.  
Además, toda la funcionalidad extra se incorpora mediante **_plugins_**, los cuales se agregan con ``install``:

- ``ContentNegotiation`` :arrow_right: JSON via _KotlinX Serialization_
- ``Logging`` :arrow_right: _Logging_ configurable (dependiendo del backend SLF4J)
- ``DefaultRequest`` :arrow_right: URL base, _headers_ comunes, etc.

Ejemplo:

```kotlin
val client = HttpClient(engineFactory = OkHttp) {
    install(plugin = ContentNegotiation) {
        json(
            Json {
                ignoreUnknownKeys = true
                isLenient = false
            }
        )
    }

    if (BuildConfig.DEBUG) {
        install(Logging) {
            logger = Logger.DEFAULT
            level = LogLevel.BODY
        }
    }

    install(plugin = DefaultRequest) {
        url("https://api.miservicio.com/")
        header("User-Agent", "My-App/1.0")
        headers.appendIfNameAbsent("X-Custom-Header", "Hello")
    }
}
```

### Realizar solicitudes
Ktor no utiliza interfaces como Retrofit: se usa la función ``client.request``.

La clase ``HttpRequestBuilder`` ofrece:

- Método HTTP (``method = HttpMethod.Get``)
- URL (``url("users/1")``)
- Headers (``headers.append``)
- Body (``setBody()``)

Ejemplo:

```kotlin
suspend fun fetchUser(client: HttpClient): SampleResponse {
    val response: HttpResponse = client.request {
        method = HttpMethod.Get
        url("users/1")
        header("Journey-Id", "12345")
    }

    return response.body()
}
```

### Manejo de respuestas y errores
> ⚠️ Importante:  
> Ktor **SÍ lanza excepción** en errores HTTP por defecto (``ClientRequestException`` para los 4xx, ``ServerResponseException`` para los 5xx). Se puede configurar manualmente el ``HttpResponseValidator`` para que no lance excepciones.  
> Ktor **SÍ lanza excepción** en errores de red (_timeout_, DNS, desconexión, SSL) o serialización.

El tipo de respuesta que devuelve es un ``HttpResponse``.

Ejemplo:

```kotlin
suspend fun safeCall(client: HttpClient): Result<SampleResponse> {
    return try {
        val response: HttpResponse = client.request {
            url("users/1")
        }

        if (response.status.isSuccess()) {
            Result.Success(response.body())
        } else {
            Result.Error("HTTP ${response.status.value}: ${response.bodyAsText()}")
        }

    } catch (e: Exception) {
        Result.Error("Network error: ${e.localizedMessage}")
    }
}
```

## *OAuth: Facebook, Twitter, Google+*
TODO...

## *Frameworks y SDK's: Firebase, Fabric, Sentry, Segment, Facebook*
TODO...
