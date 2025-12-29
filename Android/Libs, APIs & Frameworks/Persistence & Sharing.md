<h1><i>Persistence & Sharing</i></h1>

***Index***:
<!-- TOC -->
  * [*Preferences* & *Key-Value pairs*](#preferences--key-value-pairs)
    * [*SharedPreferences*](#sharedpreferences)
      * [Características principales](#características-principales)
      * [1. Obtener una instancia de `SharedPreferences`](#1-obtener-una-instancia-de-sharedpreferences)
      * [2. Escritura de datos](#2-escritura-de-datos)
      * [3. Lectura de datos](#3-lectura-de-datos)
      * [4. Eliminación de datos](#4-eliminación-de-datos)
      * [Cuándo usar `SharedPreferences`](#cuándo-usar-sharedpreferences)
    * [*EncryptedSharedPreferences*](#encryptedsharedpreferences)
      * [1. Agregar dependencias](#1-agregar-dependencias)
      * [2. Crear la `MasterKey`](#2-crear-la-masterkey)
      * [3. Crear una instancia de `EncryptedSharedPreferences`](#3-crear-una-instancia-de-encryptedsharedpreferences)
      * [4. Uso (lectura y escritura)](#4-uso-lectura-y-escritura)
      * [Consideraciones importantes](#consideraciones-importantes)
      * [`SharedPreferences` vs `EncryptedSharedPreferences`](#sharedpreferences-vs-encryptedsharedpreferences)
    * [*DataStore*](#datastore)
      * [Configuraciones de *DataStore*](#configuraciones-de-datastore)
      * [*Preferences DataStore*](#preferences-datastore)
      * [Persistencia de clases personalizadas](#persistencia-de-clases-personalizadas)
      * [Otras estrategias de serialización](#otras-estrategias-de-serialización)
      * [Resumen comparativo y Conclusión](#resumen-comparativo-y-conclusión)
    * [Conceptos avanzados / relacionados](#conceptos-avanzados--relacionados)
  * [Bases de datos](#bases-de-datos)
    * [*Room*](#room)
      * [1. Agregar dependencias y *plugins*](#1-agregar-dependencias-y-plugins)
      * [2. Crear Entidades](#2-crear-entidades)
      * [3. Crear DAOs](#3-crear-daos)
      * [4. Crear la Base de Datos](#4-crear-la-base-de-datos)
      * [5. Instanciar y configurar la Base de Datos](#5-instanciar-y-configurar-la-base-de-datos)
      * [6. Usar la Base de Datos](#6-usar-la-base-de-datos)
    * [*Realm*](#realm)
      * [1. Agregar dependencias](#1-agregar-dependencias-1)
      * [2. Definir modelos de datos](#2-definir-modelos-de-datos)
      * [3. Configurar e inicializar *Realm*](#3-configurar-e-inicializar-realm)
      * [4. Leer y escribir datos con *Realm*](#4-leer-y-escribir-datos-con-realm)
      * [5. Uso típico en una arquitectura Android (*Repository* + *ViewModel* + UI)](#5-uso-típico-en-una-arquitectura-android-repository--viewmodel--ui)
    * [Conceptos avanzados / relacionados](#conceptos-avanzados--relacionados-1)
  * [Almacenamiento de Archivos](#almacenamiento-de-archivos)
    * [Almacenamiento específico de la app](#almacenamiento-específico-de-la-app)
      * [Almacenamiento interno](#almacenamiento-interno)
      * [Almacenamiento externo específico de la app](#almacenamiento-externo-específico-de-la-app)
    * [Almacenamiento compartido](#almacenamiento-compartido)
      * [Qué es *Scoped Storage*](#qué-es-scoped-storage)
      * [Acceso mediante ``MediaStore``](#acceso-mediante-mediastore)
      * [Acceso mediante *Storage Access Framework* (SAF)](#acceso-mediante-storage-access-framework-saf)
    * [Cuándo usar cada tipo de almacenamiento](#cuándo-usar-cada-tipo-de-almacenamiento)
    * [Conceptos avanzados / relacionados](#conceptos-avanzados--relacionados-2)
  * [Compartir datos](#compartir-datos)
    * [*Intents* y extras](#intents-y-extras)
      * [*Intent* explícito](#intent-explícito)
      * [*Intent* implícito](#intent-implícito)
    * [IPC (*Inter-Process Communication*)](#ipc-inter-process-communication)
    * [Límites y buenas prácticas](#límites-y-buenas-prácticas)
    * [Conceptos avanzados / relacionados](#conceptos-avanzados--relacionados-3)
  * [Compartir archivos](#compartir-archivos)
    * [URI y acceso indirecto](#uri-y-acceso-indirecto)
    * [`FileProvider`](#fileprovider)
    * [Compartir archivos mediante `Intent`](#compartir-archivos-mediante-intent)
    * [Impacto de *Scoped Storage*](#impacto-de-scoped-storage)
    * [Casos de uso típicos](#casos-de-uso-típicos)
    * [Límites y buenas prácticas](#límites-y-buenas-prácticas-1)
    * [Conceptos avanzados / relacionados](#conceptos-avanzados--relacionados-4)
  * [*Content Providers*](#content-providers)
    * [Modelo basado en URIs](#modelo-basado-en-uris)
    * [Acceso mediante `ContentResolver`](#acceso-mediante-contentresolver)
    * [*Providers* del sistema](#providers-del-sistema)
    * [Seguridad y permisos](#seguridad-y-permisos)
    * [Cuándo usar *Content Providers*](#cuándo-usar-content-providers)
    * [Conceptos avanzados / relacionados](#conceptos-avanzados--relacionados-5)
<!-- TOC -->

---

## *Preferences* & *Key-Value pairs*
> 🔍 Referencias:  
> https://developer.android.com/training/data-storage/shared-preferences  
> https://developer.android.com/reference/androidx/security/crypto/EncryptedSharedPreferences  
> https://developer.android.com/topic/libraries/architecture/datastore

### *SharedPreferences*
En Android, **`SharedPreferences`** es un mecanismo de persistencia **ligero** basado en pares **_clave–valor_**, orientado a almacenar **datos simples y de pequeño tamaño**, como:
- _Flags_ de configuración
- Preferencias del usuario
- Estados simples (ej. *onboarding completado*, *tema seleccionado*)
- _Tokens_ no sensibles (en su versión no cifrada)

Los datos se almacenan internamente en un archivo XML (no debe considerarse estable ni manipulable directamente) privado de la aplicación y **persisten entre ejecuciones**, incluso si la app se cierra o el proceso es destruido.

Este mecanismo **no está diseñado para grandes volúmenes de datos ni estructuras complejas**, y **no reemplaza** a una base de datos como *Room* o *Realm*.

#### Características principales
- **Modelo _Key–Value_** :arrow_right: Claves de tipo `String` asociadas a valores primitivos
- **Almacenamiento persistente** :arrow_right: Los datos sobreviven a reinicios de la app
- **Acceso síncrono** :arrow_right: Lecturas inmediatas; escrituras mediante `apply` o `commit`
- **_Scope_ privado por defecto** :arrow_right: Los datos solo son accesibles por la app
- **Bajo _overhead_** :arrow_right: Ideal para configuraciones y estados simples

Tipos de datos soportados:
- `String`
- `Int`
- `Long`
- `Float`
- `Boolean`
- `Set<String>`

#### 1. Obtener una instancia de `SharedPreferences`
Para acceder a `SharedPreferences`, se utiliza el método `getSharedPreferences`, indicando:
- El **nombre del archivo**
- El **modo de acceso** (usualmente `Context.MODE_PRIVATE`)

```kotlin
val sharedPreferences = context.getSharedPreferences(
    "app_prefs",
    Context.MODE_PRIVATE
)
```

También existe una variante específica para **preferencias por defecto**:

```kotlin
val sharedPreferences =
    PreferenceManager.getDefaultSharedPreferences(context)
```

#### 2. Escritura de datos
Las escrituras se realizan mediante un objeto `SharedPreferences.Editor`.

Existen dos formas de persistir los cambios:
- `apply()` :arrow_right: **Asíncrono**; **no bloquea** el hilo llamante (recomendado)
- `commit()` :arrow_right: **Síncrono**; devuelve `Boolean` indicando éxito

```kotlin
sharedPreferences.edit()
    .putString("username", "juan.perez")
    .putBoolean("is_logged_in", true)
    .putInt("launch_count", 5)
    .apply()
```

> ℹ️ **Nota:**  
> `apply()` escribe los datos en memoria inmediatamente y los persiste en disco de forma asíncrona.  
> En la mayoría de los casos es preferible a `commit()` para evitar bloqueos del hilo principal.

#### 3. Lectura de datos
Para leer valores, se utilizan los métodos `getX`, indicando:
- La **clave**
- Un **valor por defecto**, utilizado si la clave no existe

```kotlin
val username: String? =
    sharedPreferences.getString("username", null)

val isLoggedIn: Boolean =
    sharedPreferences.getBoolean("is_logged_in", false)

val launchCount: Int =
    sharedPreferences.getInt("launch_count", 0)
```

#### 4. Eliminación de datos
Se pueden eliminar claves individuales o limpiar todo el archivo.

```kotlin
// Eliminar una clave
sharedPreferences.edit()
    .remove("username")
    .apply()

// Limpiar todas las preferencias
sharedPreferences.edit()
    .clear()
    .apply()
```

#### Cuándo usar `SharedPreferences`
Uso recomendado:
- Preferencias simples del usuario
- _Flags_ de estado
- Configuración de UI
- Datos pequeños y no estructurados

Evitar usarlo para:
- Grandes volúmenes de datos
- Datos relacionales
- Listas complejas
- Información sensible sin cifrar

### *EncryptedSharedPreferences*
`EncryptedSharedPreferences` es una **extensión segura** de `SharedPreferences` provista por **AndroidX Security**, que cifra automáticamente:
- **Las claves**
- **Los valores**

Utiliza:
- **AES-256** para cifrado de valores
- **_MasterKey_** almacenada en el **Android _Keystore_**

Esto permite almacenar de forma segura:
- _Tokens_ de autenticación
- Credenciales
- Información sensible de sesión

#### 1. Agregar dependencias
Para utilizar `EncryptedSharedPreferences`, se debe agregar la dependencia correspondiente:

```kotlin
dependencies {
    implementation("androidx.security:security-crypto:<VERSION>")
}
```

#### 2. Crear la `MasterKey`
La `MasterKey` define cómo se gestionará la clave criptográfica utilizada para cifrar los datos.

```kotlin
val masterKey = MasterKey.Builder(context)
    .setKeyScheme(MasterKey.KeyScheme.AES256_GCM)
    .build()
```

#### 3. Crear una instancia de `EncryptedSharedPreferences`
La API es similar a `SharedPreferences`, pero requiere:
- La `MasterKey`
- Esquemas de cifrado para claves y valores

```kotlin
val encryptedSharedPreferences =
    EncryptedSharedPreferences.create(
        context,
        "secure_prefs",
        masterKey,
        EncryptedSharedPreferences.PrefKeyEncryptionScheme.AES256_SIV,
        EncryptedSharedPreferences.PrefValueEncryptionScheme.AES256_GCM
    )
```

#### 4. Uso (lectura y escritura)
La API de uso es **idéntica** a `SharedPreferences`.

```kotlin
encryptedSharedPreferences.edit()
    .putString("auth_token", "abc123")
    .apply()

val token: String? =
    encryptedSharedPreferences.getString("auth_token", null)
```

#### Consideraciones importantes
- El cifrado y descifrado se realiza **automáticamente**
- Existe un **ligero _overhead_** (irrelevante para volúmenes pequeños) respecto a `SharedPreferences` normal
- Es adecuado para **datos sensibles de pequeño tamaño**
- No reemplaza soluciones de almacenamiento seguro de mayor complejidad

#### `SharedPreferences` vs `EncryptedSharedPreferences`

| Característica  | SharedPreferences | EncryptedSharedPreferences |
|-----------------|-------------------|----------------------------|
| Persistencia    | ✅                 | ✅                          |
| Cifrado         | ❌                 | ✅                          |
| Simplicidad     | ✅                 | ⚠️                         |
| Rendimiento     | Más rápido        | Ligeramente menor          |
| Datos sensibles | ❌                 | ✅                          |

📌 **Resumen**
- `SharedPreferences` es ideal para **configuración y estado simple**
- `EncryptedSharedPreferences` debe usarse cuando los datos **requieren confidencialidad**
- Para datos complejos o reactivos, considerar alternativas como **_DataStore_**, **_Room_** o **_Realm_**

### *DataStore*
Es una API moderna de Android para la persistencia de datos locales, diseñada para **reemplazar a `SharedPreferences`** y resolver sus principales limitaciones.

Está basada en **Kotlin Coroutines** y **Flow**, lo que permite:
- Acceso **asíncrono y no bloqueante**
- Operaciones **atómicas y seguras ante concurrencia**
- Observación reactiva de cambios
- Manejo explícito de errores

A diferencia de `SharedPreferences`, *DataStore* evita condiciones de carrera, bloqueos del hilo principal y estados inconsistentes.

#### Configuraciones de *DataStore*
La documentación oficial define **dos grandes configuraciones conceptuales** para utilizar *DataStore*, según el modelo de datos que se quiera persistir:

1. **_Preferences DataStore_** :arrow_right: Persistencia *key–value*, sin esquema predefinido y sin tipado fuerte.
2. **Persistencia de clases personalizadas** :arrow_right: Persistencia de objetos estructurados mediante un esquema y un `Serializer`.

Ambas configuraciones utilizan la misma API base (`DataStore<T>`), pero difieren en:
- El tipo de datos persistidos (``<T>``)
- La necesidad (o no) de un esquema
- El nivel de seguridad de tipos

#### *Preferences DataStore*
Esta configuración está orientada a **pares _key–value_**, similar a `SharedPreferences`, pero sin sus desventajas.

Características principales:
- No requiere esquema
- API similar a `SharedPreferences`
- Acceso asíncrono
- Ideal para flags, preferencias y configuraciones simples

1. **Agregar dependencia**:

```kotlin
dependencies {
    implementation("androidx.datastore:datastore-preferences:<VERSION>")
}
```

2. **Crear la instancia**:

Se recomienda declarar la instancia como una **extensión de `Context`**, garantizando una única instancia por archivo.

```kotlin
val Context.dataStore by preferencesDataStore(
    name = "app_preferences"
)
```

3. **Definir claves de preferencia**:

Las claves se definen mediante funciones helper tipadas.

```kotlin
object PreferencesKeys {
    val USERNAME = stringPreferencesKey("username")
    val IS_LOGGED_IN = booleanPreferencesKey("is_logged_in")
    val LAUNCH_COUNT = intPreferencesKey("launch_count")
}
```

4. **Escritura de datos**:

```kotlin
suspend fun savePreferences(context: Context) {
    context.dataStore.edit { prefs ->
        prefs[PreferencesKeys.USERNAME] = "juan.perez"
        prefs[PreferencesKeys.IS_LOGGED_IN] = true
        prefs[PreferencesKeys.LAUNCH_COUNT] = 1
    }
}
```

5. **Lectura de datos**:

```kotlin
val usernameFlow: Flow<String?> =
    context.dataStore.data
        .catch { exception ->
            if (exception is IOException) {
                // Fallback
                emit(emptyPreferences())
            } else {
                throw exception
            }
        }
        .map { prefs ->
            prefs[PreferencesKeys.USERNAME]
        }
```

#### Persistencia de clases personalizadas
Además del modelo *key–value*, *DataStore* permite persistir **clases personalizadas**.

En este enfoque:
- Se define un **esquema de datos**
- Se provee un **_Serializer_**
- El tipo almacenado es **fuertemente tipado**

Es posible utilizar:
- **_Protocol Buffers_** (ver documentación sobre [_Protobuf_](https://protobuf.dev/))
- **JSON**
- **Cualquier otra estrategia de serialización**

_Protocol Buffers_ es la opción **más común y recomendada** por Android, debido a:
- Alto rendimiento
- Bajo tamaño en disco
- Esquema versionable

1. **Agregar dependencias (ejemplo con _Protocol Buffers_)**:

```kotlin
dependencies {
    // Dependencia Base
    implementation("androidx.datastore:datastore:<VERSION>")
    
    // Dependencia Protobuf
    implementation("com.google.protobuf:protobuf-javalite:<VERSION>")
}
```

2. **Definir el esquema (`.proto`)**:

```proto
syntax = "proto3";

option java_package = "com.example.datastore";
option java_multiple_files = true;

message UserPreferences {
  string username = 1;
  bool is_logged_in = 2;
  int32 launch_count = 3;
}
```

3. **Crear el _Serializer_**:

```kotlin
object UserPreferencesSerializer : Serializer<UserPreferences> {

    override val defaultValue: UserPreferences =
        UserPreferences.getDefaultInstance()

    override suspend fun readFrom(input: InputStream): UserPreferences =
        try {
            UserPreferences.parseFrom(input)
        } catch (exception: InvalidProtocolBufferException) {
            defaultValue
        }

    override suspend fun writeTo(
        t: UserPreferences,
        output: OutputStream
    ) {
        t.writeTo(output)
    }
}
```

4. **Crear la instancia de ``DataStore``**:

```kotlin
val Context.userPreferencesDataStore: DataStore<UserPreferences> by dataStore(
    fileName = "user_prefs.pb",
    serializer = UserPreferencesSerializer
)
```

#### Otras estrategias de serialización
Aunque *Protocol Buffers* es el enfoque más utilizado, *DataStore* **no impone una tecnología específica**.

Es posible implementar un `Serializer<T>` basado en:
- JSON (por ejemplo, **_Moshi_** o **_Kotlinx Serialization_**)
- XML
- Formatos binarios personalizados

La responsabilidad de la consistencia, validación y versionado del esquema recae en el `Serializer`.

#### Resumen comparativo y Conclusión

| Configuración         | Esquema | Tipado fuerte | Caso de uso          |
|-----------------------|---------|---------------|----------------------|
| Preferences DataStore | ❌       | ❌             | Preferencias simples |
| Clases personalizadas | ✅       | ✅             | Estados complejos    |

📌 **Conclusión**
- *DataStore* es la alternativa moderna a `SharedPreferences`
- La API define **dos configuraciones conceptuales claras**
- `Preferences DataStore` es simple y directo
- La persistencia de clases personalizadas ofrece mayor robustez
- _Protocol Buffers_ es recomendado, pero no obligatorio

### Conceptos avanzados / relacionados
- **Migración desde `SharedPreferences`** (ver [acá](https://developer.android.com/topic/libraries/architecture/datastore#migrate-preferences))  
  *DataStore* permite migrar datos existentes desde `SharedPreferences`, evitando pérdidas al adoptar la nueva API.

- **Manejo de errores de lectura (`IOException`)** (ver [acá](https://developer.android.com/topic/libraries/architecture/datastore#handle-corruption))  
  El flujo `data` puede fallar por errores de I/O o datos corruptos; es responsabilidad del consumidor definir un *fallback* apropiado (por ejemplo, preferencias vacías).

- **_Testing_ con _DataStore_**  
  *DataStore* puede testearse utilizando archivos temporales y *coroutine scopes* controlados para simular escenarios de lectura y escritura.

- **_Backup_ y _Auto Backup_** (ver [acá](https://developer.android.com/guide/topics/data/autobackup))  
  Las preferencias pueden incluirse o excluirse del sistema de *Auto Backup* de Android, afectando su restauración al reinstalar la app.

- **Alcance del almacenamiento**  
  *DataStore* siempre es **privado a la aplicación** y no está diseñado para compartir datos entre procesos o aplicaciones.

## Bases de datos
### *Room*
> 🔍 Referencia:  
> https://developer.android.com/training/data-storage/room

Android soporta *SQLite* desde sus primeras versiones. Sin embargo, su uso directo requiere escribir una cantidad considerable de código *boilerplate* y trabajar con APIs de bajo nivel. Además, *SQLite* no ofrece soporte nativo para el mapeo de objetos (*POJO’s*, *plain-old Java objects*) ni valida las consultas (*queries*) en tiempo de compilación.  
*Room* surge para resolver estas limitaciones, proporcionando una **librería ORM** (*Object Relational Mapping* o mapeo objeto–relacional) integrada al ecosistema Android. Permite **mapear bases de datos *SQLite* a objetos**, convertir automáticamente los resultados de las consultas en instancias de clases, **validar las consultas en tiempo de compilación** y **exponer los resultados mediante tipos observables como ``LiveData`` o ``Flow`` de Kotlin**.

Para crear una base de datos con *Room*, se requiere la definición de los siguientes componentes:
- Una **Entidad** (``Entity``) :arrow_right: **Representa una tabla dentro de la base de datos** y se define como un *POJO*. En Kotlin, se utilizan habitualmente ***data classes*** para este propósito.
- Una ***interface*** ``Dao`` (***Data Access Object*** u Objeto de Acceso a Datos) :arrow_right: Define las **consultas y operaciones de lectura y escritura** sobre la base de datos.
- Una **clase abstracta** ``Database`` :arrow_right: Actúa como **punto de acceso principal a la base de datos** y debe extender a ``RoomDatabase``.

*Room* genera automáticamente el código necesario para gestionar el acceso a los datos y previene la ejecución de operaciones bloqueantes en el hilo principal, facilitando su ejecución asíncrona. Las consultas pueden exponerse como **tipos observables** (``LiveData`` o ``Flow``), permitiendo que la UI se mantenga sincronizada con los datos persistidos.  
Cuando un método del *DAO* retorna un **objeto concreto** (por ejemplo, una entidad o una lista de entidades), puede declararse con la palabra reservada ``suspend`` para integrarse con **corrutinas de Kotlin** y ejecutarse de forma asíncrona. En cambio, los métodos que retornan un ``Flow`` **no deben declararse como ``suspend``**, ya que el propio flujo gestiona la ejecución asíncrona y la emisión de actualizaciones.

#### 1. Agregar dependencias y *plugins*
_Room_ utiliza procesamiento de anotaciones para generar el código necesario en tiempo de compilación. En proyectos Kotlin modernos, se recomienda utilizar **KSP** (**_Kotlin Symbol Processing_**), que ofrece mejor rendimiento y tiempos de compilación más rápidos en comparación con ``kapt``.  
En ese caso, debe estar agregado el _plugin_ correspondiente en el archivo ``build.gradle.kts`` **del proyecto raíz**:

```kotlin
plugins {
    id("com.google.devtools.ksp") version "<KSP_VERSION>" apply false
}
```

Luego, en el archivo ``build.gradle.kts`` **del módulo en el que se utilice _Room_**:

```kotlin
plugins {
    id("com.android.application") // O com.android.library, según corresponda
    kotlin("android")
    id("com.google.devtools.ksp")
}

dependencies {
    implementation("androidx.room:room-runtime:$room_version")

    // Opcional - Kotlin extensions y soporte para corrutinas y Flow
    implementation("androidx.room:room-ktx:$room_version")

    // Procesador de anotaciones (KSP)
    ksp("androidx.room:room-compiler:$room_version")

    // Si se usa kapt en lugar de KSP (alternativa legacy)
    // kapt("androidx.room:room-compiler:$room_version")

    // Opcional - Test helpers
    testImplementation("androidx.room:room-testing:$room_version")
}
```

#### 2. Crear Entidades
Las **Entidades** definen la estructura de las tablas dentro de la base de datos y representan los datos persistidos como objetos. Cada entidad se declara mediante la anotación ``@Entity`` y puede configurarse con un nombre de tabla explícito mediante el parámetro ``tableName``.

Las propiedades de la clase representan las columnas de la tabla y pueden personalizarse utilizando anotaciones como ``@PrimaryKey`` (que **puede configurarse como autogenerada**) y ``@ColumnInfo``. En Kotlin, las entidades suelen definirse como ***data classes***, ya que proveen automáticamente métodos utilitarios como ``equals``, ``hashCode`` y ``toString``.  
Además, la nulabilidad de las propiedades se refleja directamente en el esquema de la base de datos.

> ℹ️ **Nota:**  
> Cuando el ``PrimaryKey`` es autogenerado, el valor por defecto (``0``) actúa únicamente como un **marcador temporal en memoria**.  
> La generación real del identificador queda a cargo de _SQLite_ al momento de persistir el registro, garantizando unicidad y consistencia.

```kotlin
@Entity(tableName = "user")
data class User(
    // Room detecta el valor 0 como no-asignado y delega la generación a SQLite
    @PrimaryKey(autoGenerate = true) val uid: Int = 0,
    @ColumnInfo(name = "first_name") val firstName: String?,
    @ColumnInfo(name = "last_name") val lastName: String?
)

// Asumiendo una entidad Post relacionada
data class UserWithPosts(
    // @Embedded permite incluir los campos de otra entidad dentro de un objeto de resultado
    @Embedded val user: User,

    // @Relation define relaciones entre tablas (1-a-1, 1-a-N) sin escribir SQL manual
    @Relation(
        parentColumn = "uid",
        entityColumn = "userId"
    )
    val posts: List<Post>
)
```

#### 3. Crear DAOs
Los ***DAOs*** definen la interfaz entre la *app* y la base de datos. Se declaran mediante la anotación ``@Dao`` y contienen los métodos que permiten realizar **consultas**, **inserciones**, **actualizaciones** y **eliminaciones** de datos.

Los métodos del *DAO* pueden declararse como funciones de suspensión (``suspend``) **cuando retornan un objeto concreto**.  
En cambio, cuando el método retorna un ``Flow``, _Room_ reejecuta automáticamente la consulta ante cambios en la tabla, por lo cual no debe declararse como ``suspend``.

```kotlin
@Dao
interface UserDao {
    @Query("SELECT * FROM user")
    suspend fun getAll(): List<User>

    // Al usar Flow, la función no debe ser de suspensión
    @Query("SELECT * FROM user")
    fun getAllUsers(): Flow<List<User>>

    // @Transaction garantiza que todas las operaciones de lectura asociadas se ejecuten de forma atómica, 
    // evitando estados inconsistentes cuando hay múltiples consultas relacionadas
    @Transaction
    @Query("SELECT * FROM user")
    fun getUsersWithPosts(): Flow<List<UserWithPosts>>

    @Query("SELECT * FROM user WHERE uid IN (:userIds)")
    suspend fun loadAllByIds(userIds: IntArray): List<User>

    @Query(
        "SELECT * FROM user WHERE first_name LIKE :first AND " +
                "last_name LIKE :last LIMIT 1"
    )
    suspend fun findByName(first: String, last: String): User?

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insertAll(vararg users: User)

    @Delete
    suspend fun delete(user: User)

    @Update
    suspend fun updateUsers(vararg users: User)
}
```

#### 4. Crear la Base de Datos
La clase de base de datos define la configuración principal de *Room* y actúa como el punto de acceso a los *DAOs*. Debe declararse como una **clase abstracta** anotada con ``@Database`` y extender a ``RoomDatabase``.

La anotación ``@Database`` permite especificar las entidades que forman parte del esquema y la versión de la base de datos.

```kotlin
@Database(entities = arrayOf(User::class), version = 1)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao
}
```

#### 5. Instanciar y configurar la Base de Datos
Para utilizar la base de datos, se debe crear una instancia de ``RoomDatabase`` mediante ``Room.databaseBuilder``.  
Esta instancia **debe inicializarse una única vez** y reutilizarse a lo largo del ciclo de vida de la *app*, ya que la creación de múltiples instancias puede generar problemas de rendimiento y consistencia.

En aplicaciones reales, **se recomienda utilizar Inyección de Dependencias (DI)** para proveer la instancia de la base de datos como una dependencia *singleton*. De esta forma, se centraliza su creación, se evita el acoplamiento entre capas y se facilita el testeo.  
_Frameworks_ como **_Hilt_** o **_Dagger_** son comúnmente utilizados para este propósito en Android.

El nombre de la base de datos identifica el archivo físico donde se almacenan los datos persistidos.

```kotlin
val db = Room.databaseBuilder(
    applicationContext,
    AppDatabase::class.java,
    "database-name"
).build()
```

Cuando la base de datos ya existe en dispositivos de usuarios y **se realizan cambios en el esquema** (por ejemplo, **agregar columnas o modificar tablas**), es necesario definir **migraciones**.  
Las migraciones se registran mediante el método ``addMigrations`` y se ejecutan **una sola vez**, únicamente cuando la versión almacenada en el dispositivo es anterior a la versión actual declarada en ``@Database``.

```kotlin
val MIGRATION_1_2 = object : Migration(1, 2) {
    override fun migrate(database: SupportSQLiteDatabase) {
        // Se ejecuta una sola vez al actualizar de la versión 1 a la 2
        database.execSQL("ALTER TABLE user ADD COLUMN age INTEGER")
    }
}
```

La instancia de la base de datos puede configurarse incluyendo las migraciones necesarias. _Room_ se encargará de ejecutar únicamente las migraciones correspondientes según la versión desde la cual se actualice la aplicación.

> ℹ️ **Nota:**  
> Cada salto de versión requiere su propia migración (``1→2``, ``2→3``, etc.).

```kotlin
val db = Room.databaseBuilder(
    applicationContext,
    AppDatabase::class.java,
    "database-name"
)
    .addMigrations(MIGRATION_1_2)
    .build()
```

#### 6. Usar la Base de Datos
Una vez creada la instancia de la base de datos, el acceso a los datos se realiza a través de los **DAOs** expuestos por la clase ``RoomDatabase``.  
En una arquitectura común (por ejemplo, **MVVM**), **estas llamadas suelen realizarse desde un _Repository_**, que actúa como intermediario entre la capa de datos (*Room*) y el resto de la aplicación, manteniendo una separación clara de responsabilidades.

El flujo de uso típico es:

1. Obtener el DAO desde la instancia de la base de datos
2. Invocar los métodos definidos en el DAO
3. Ejecutar las operaciones dentro de corrutinas o consumir los ``Flow`` expuestos

📌 Ejemplo:

```kotlin
// Módulo DI
@Module
@InstallIn(SingletonComponent::class)
object RoomModule {
    @Provides
    @Singleton
    fun provideDatabase(@ApplicationContext context: Context): AppDatabase =
        Room.databaseBuilder(
            context,
            AppDatabase::class.java,
            "app_database"
        ).build()

    @Provides
    @Singleton
    fun provideUserDao(db: AppDatabase): UserDao =
        db.userDao()
}

// Repository - `userDao` se provee con DI
class UserRepository @Inject constructor(
    private val userDao: UserDao
) {
    suspend fun insert(user: User) {
        userDao.insertAll(user)
    }

    suspend fun getAll(): List<User> =
        userDao.getAll()

    fun getAllUsers(): Flow<List<User>> =
        userDao.getAllUsers()
}

// ViewModel
class UserViewModel(
    private val repository: UserRepository
) : ViewModel() {
    fun insertUser(user: User) {
        viewModelScope.launch {
            repository.insert(user)
        }
    }

    val usersFlow: Flow<List<User>> =
        repository.getAllUsers()
}

// Fragment
class UserFragment : Fragment(R.layout.fragment_user) {
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        // Acción de UI - Insertar usuario
        saveButton.setOnClickListener {
            val user = User(
                firstName = firstNameEditText.text.toString(),
                lastName = lastNameEditText.text.toString()
            )

            viewModel.insertUser(user)
        }

        // Observación reactiva del estado con Flow
        viewLifecycleOwner.lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.usersFlow.collect { userList ->
                    // Actualizar la UI con los datos emitidos
                    // Ej: adapter.submitList(userList)
                }
            }
        }
    }
}
```

### *Realm*
> 🔍 Referencia:  
> https://github.com/realm/realm-kotlin

> :warning: A partir de *Realm Kotlin 3.0.0+*, la librería **no incluye funcionalidades de sincronización (_Sync_)** ni dependencias asociadas a *MongoDB Atlas*.  
> La base de datos funciona exclusivamente como **almacenamiento local**, lo cual es adecuado para la mayoría de los casos de uso en aplicaciones Android.

*Realm* es una **base de datos orientada a objetos**, multiplataforma y diseñada para aplicaciones móviles, que permite **persistir datos directamente como objetos**, sin necesidad de mapearlos a tablas relacionales ni escribir consultas SQL tradicionales.

A diferencia de *SQLite* (y por extensión *Room*), *Realm* **no utiliza un modelo relacional**, sino que almacena los datos en un formato propio optimizado para acceso rápido y bajo consumo de memoria. Esto permite trabajar con los datos de forma más natural desde el código, reduciendo significativamente el *boilerplate*.

Entre sus principales características se destacan:

- **Persistencia orientada a objetos** :arrow_right: Los modelos se definen como clases y se persisten directamente, sin necesidad de un **ORM** relacional tradicional (ver [Room](#room)).
- **Consultas reactivas** :arrow_right: Los resultados pueden observarse y reaccionar automáticamente ante cambios en los datos.
- **Sin SQL** :arrow_right: Las consultas se realizan mediante una API basada en predicados y expresiones del lenguaje.
- **Alto rendimiento** :arrow_right: Optimizado para dispositivos móviles, incluso con grandes volúmenes de datos.
- **Soporte multiplataforma** :arrow_right: Android, iOS, Kotlin Multiplatform, entre otros.

En el ecosistema Android, *Realm* suele utilizarse como **alternativa a *Room*** cuando se busca:

- Un modelo de datos más orientado a objetos
- Menor cantidad de código repetitivo
- Observación reactiva de datos sin configuración adicional
- Evitar el manejo de migraciones SQL, utilizando migraciones basadas en código

Sin embargo, este enfoque implica **adoptar APIs y conceptos propios**, diferentes a los estándares basados en *SQLite*.

Para utilizar *Realm* en una aplicación Android, es necesario definir:

- **Modelos de datos** :arrow_right: Clases que representan los objetos persistidos.
- **Configuración de la base de datos** :arrow_right: Inicialización y versionado del esquema.
- **Operaciones de lectura y escritura** :arrow_right: Realizadas dentro de transacciones controladas por *Realm*.

#### 1. Agregar dependencias
Para utilizar *Realm* en Android se debe agregar la librería **Realm Kotlin** como dependencia del módulo.  
Desde la versión **3.x**, *Realm Kotlin* se distribuye como una librería estándar y **no requiere procesamiento de anotaciones (*KSP* o *kapt*) ni _plugins_ especiales** (a diferencia de *Room*).

Esto simplifica la configuración inicial y reduce la complejidad del *build*.

```kotlin
dependencies {
    implementation("io.realm.kotlin:library-base:<REALM_VERSION>")
}
```

#### 2. Definir modelos de datos
En *Realm*, los datos persistidos se representan mediante **clases orientadas a objetos**, no mediante tablas relacionales como en *SQLite*/*Room*.

Cada modelo debe:
- Implementar (o extender) `RealmObject`
- Definir sus propiedades como `var`
- Utilizar tipos soportados por *Realm*

A diferencia de *Room*:
- No se utilizan anotaciones de mapeo relacional como `@Entity` o `@ColumnInfo`
- No existe un esquema SQL explícito
- El esquema de la base de datos se deriva directamente de las clases del modelo

Claves importantes sobre los modelos:
- **Clases abiertas y propiedades mutables**:  
  _Realm_ gestiona internamente el ciclo de vida de los objetos, por lo que las propiedades deben ser `var`.
- **Clave primaria opcional pero recomendada**:  
  La anotación `@PrimaryKey` (propia de *Realm*) permite identificar de forma única cada objeto y facilita operaciones de actualización y reemplazo.
- **Inicialización sin constructor primario**:  
  Los modelos de _Realm_ suelen definirse sin `data class` ni constructor con parámetros, ya que _Realm_ instancia y administra los objetos.
- **Objetos gestionados vs no gestionados**:  
  Un objeto puede existir:
    - Como objeto normal de Kotlin (no gestionado)
    - Como objeto gestionado por _Realm_, cuando se encuentra dentro de la base de datos y una transacción

```kotlin
import io.realm.kotlin.types.RealmObject
import io.realm.kotlin.types.annotations.PrimaryKey

class User : RealmObject {
    @PrimaryKey
    var id: Long = 0

    var firstName: String = ""
    var lastName: String = ""
}
```

#### 3. Configurar e inicializar *Realm*
Antes de poder leer o escribir datos, es necesario **configurar e inicializar una instancia de _Realm_**.  
Esta configuración define **qué modelos forman parte del esquema**, la **versión del esquema** y, opcionalmente, cómo manejar **cambios estructurales** mediante migraciones.

A diferencia de *Room*:
- No existe una clase equivalente a `RoomDatabase`
- No se definen *DAOs*
- El acceso a los datos se realiza directamente a través de una instancia de `Realm`

La configuración se realiza mediante un objeto `RealmConfiguration`, en el cual se indican:
- Las clases de modelo que forman parte del esquema
- La versión del esquema (`schemaVersion`)
- Opcionalmente, la lógica de migración

El esquema de la base de datos se deriva automáticamente de las clases de modelo indicadas, sin necesidad de definir tablas o consultas SQL.

```kotlin
import io.realm.kotlin.Realm
import io.realm.kotlin.RealmConfiguration

val config = RealmConfiguration.Builder(
    schema = setOf(User::class)
)
    .schemaVersion(1)
    .build()
```

Una vez definida la configuración, se obtiene una instancia de `Realm` mediante `Realm.open`.  
Esta instancia representa la conexión activa a la base de datos y es el **punto de acceso** para realizar operaciones de lectura y escritura.

En aplicaciones reales, **se recomienda inicializar y proveer esta instancia mediante Inyección de Dependencias**, de forma similar a lo que se hace con *Room*, para:
- Reutilizar una única instancia
- Evitar acoplamiento entre capas
- Facilitar el testeo

```kotlin
val realm: Realm = Realm.open(config)
```

La instancia de `Realm` mantiene recursos abiertos y es **_thread-confined_**, por lo que:
- No puede utilizarse desde otro hilo distinto al que fue creada (en caso de necesitar usar _Realm_ en otro hilo, se debe abrir otra instancia)
- Debe cerrarse explícitamente **cuando su ciclo de vida es acotado** (por ejemplo, si está asociada a un `ViewModel` o a un _scope_ específico)

En configuraciones donde `Realm` se provee como una instancia **_Singleton_** (por ejemplo, desde un contenedor de Inyección de Dependencias a nivel de aplicación), **no suele ser necesario cerrarla manualmente**, ya que se libera junto con el proceso de la app. Pero en caso de tener que hacerlo (ciclo de vida acotado), se utiliza:

```kotlin
realm.close()
```

#### 4. Leer y escribir datos con *Realm*
Una vez inicializada la instancia de `Realm`, las operaciones de acceso a datos se realizan **directamente sobre dicha instancia**, sin necesidad de *DAOs* ni capas intermedias obligatorias.

En *Realm* existen dos conceptos clave:
- **Transacciones de escritura** :arrow_right: Necesarias para insertar, actualizar o eliminar datos
- **Consultas reactivas** :arrow_right: Permiten observar cambios automáticamente

Las operaciones de escritura deben ejecutarse dentro de un bloque `write {}`, el cual garantiza consistencia y atomicidad.

```kotlin
realm.write {
    val user = User().apply {
        id = 1
        firstName = "Juan"
        lastName = "Pérez"
    }

    copyToRealm(user)
}
```

Durante una transacción de escritura, *Realm* gestiona automáticamente:
- La inserción del objeto
- La persistencia en disco
- La notificación a observadores activos

Si el objeto tiene una clave primaria definida, `copyToRealm` puede reemplazar registros existentes cuando se utiliza el modo adecuado.

```kotlin
realm.write {
    copyToRealm(
        User().apply {
            id = 1
            firstName = "Juan"
            lastName = "Actualizado"
        },
        // Permite reemplazar todos los campos del objeto existente
        // Si no hay @PrimaryKey, UpdatePolicy no tiene efecto
        updatePolicy = UpdatePolicy.ALL
    )
}
```

Para realizar lecturas, se utiliza el método `query`, el cual devuelve resultados **vivos** (***live objects***).  
Esto significa que los objetos obtenidos se actualizan automáticamente cuando cambian los datos subyacentes.

```kotlin
val users = realm.query<User>().find()
```

Las consultas pueden incluir filtros utilizando expresiones basadas en cadenas, similares a predicados.  
No se utiliza SQL, sino un lenguaje de consulta propio de *Realm*.

```kotlin
val usersNamedJuan = realm
    .query<User>("firstName == $0", "Juan")
    .find()
```

Uno de los principales beneficios de *Realm* es la **observación reactiva de datos**.  
Los resultados de una consulta pueden observarse como un `Flow`, emitiendo actualizaciones automáticamente cuando los datos cambian.

```kotlin
val usersFlow: Flow<ResultsChange<User>> =
    realm.query<User>().asFlow()
```

Este enfoque elimina la necesidad de invalidar manualmente consultas o refrescar la UI, ya que *Realm* se encarga de emitir los cambios de forma automática.

En arquitecturas comunes (por ejemplo, **MVVM**), estas operaciones suelen encapsularse dentro de un *Repository*, manteniendo la UI desacoplada de la lógica de persistencia.

#### 5. Uso típico en una arquitectura Android (*Repository* + *ViewModel* + UI)
Aunque *Realm* permite acceder directamente a la base de datos desde cualquier parte del código, **no es recomendable hacerlo desde la UI**.  
En aplicaciones Android modernas, lo habitual es encapsular el acceso a datos dentro de un **_Repository_**, exponiendo un API claro hacia la capa de presentación.

Este enfoque permite:
- Separar responsabilidades
- Facilitar el testeo
- Centralizar el acceso a *Realm*
- Evitar dependencias directas entre la UI y la base de datos

El *Repository* expone:
- Métodos de escritura encapsulados en transacciones
- Flujos reactivos (`Flow`) para la observación de datos

La capa de presentación no necesita conocer detalles internos de *Realm*, como transacciones o políticas de actualización.

El *ViewModel* actúa como intermediario entre la UI y el *Repository*, gestionando:
- Corrutinas
- Exposición de estado observable
- Persistencia frente a cambios de configuración

📌 Ejemplo:

```kotlin
// Módulo DI
@Module
@InstallIn(SingletonComponent::class)
object RealmModule {
    @Provides
    @Singleton
    fun provideRealmConfiguration(): RealmConfiguration =
        RealmConfiguration.Builder(
            schema = setOf(User::class)
        )
            .schemaVersion(1)
            .build()

    @Provides
    @Singleton
    fun provideRealm(
        config: RealmConfiguration
    ): Realm = Realm.open(config)
}

// Repository - `realm` y `config` se proveen con DI
class UserRepository @Inject constructor(
    private val realm: Realm,
    private val config: RealmConfiguration
) {
    fun observeUsers(): Flow<List<User>> =
        realm.query<User>()
            .asFlow()
            .map { resultsChange ->
                resultsChange.list
            }

    suspend fun insertUser(firstName: String, lastName: String) {
        realm.write {
            copyToRealm(
                User().apply {
                    this.firstName = firstName
                    this.lastName = lastName
                }
            )
        }
    }

    suspend fun insertUserInBackground(firstName: String, lastName: String) {
        withContext(Dispatchers.IO) {
            val realm = Realm.open(config) // Nueva instancia en el hilo IO
            realm.write {
                copyToRealm(
                    User().apply {
                        this.firstName = firstName
                        this.lastName = lastName
                    }
                )
            }
            realm.close() // Cerrar la instancia del hilo IO
        }
    }
}

// ViewModel
class UserViewModel(
    private val repository: UserRepository
) : ViewModel() {
    val usersFlow: Flow<List<User>> =
        repository.observeUsers()

    fun insertUser(firstName: String, lastName: String) {
        viewModelScope.launch {
            repository.insertUser(firstName, lastName)
        }
    }
}

// Fragment
class UserFragment : Fragment(R.layout.fragment_user) {
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        saveButton.setOnClickListener {
            viewModel.insertUser(
                firstName = firstNameEditText.text.toString(),
                lastName = lastNameEditText.text.toString()
            )
        }

        viewLifecycleOwner.lifecycleScope.launch {
            repeatOnLifecycle(Lifecycle.State.STARTED) {
                viewModel.usersFlow.collect { users ->
                    // Actualizar la UI
                    // Ej: adapter.submitList(users)
                }
            }
        }
    }
}
```

### Conceptos avanzados / relacionados
- **Migraciones de esquema**  
  Tanto *Room* como *Realm* requieren definir cómo evolucionan los datos cuando cambia la estructura del modelo persistido.

- **Relaciones entre entidades** (ver [acá](https://developer.android.com/training/data-storage/room/relationships))  
  *Room* soporta relaciones mediante anotaciones como `@Relation` y `@Embedded`, evitando la necesidad de escribir *joins* manuales.

- **Transacciones** (ver [acá](https://developer.android.com/training/data-storage/room/transactions))  
  Permiten agrupar múltiples operaciones de lectura o escritura garantizando consistencia y atomicidad.

- **Concurrencia y _threading_**  
  *Room* integra corrutinas y `Flow`; *Realm* utiliza instancias *thread-confined*, lo que impacta cómo se accede a los datos desde distintos hilos.

- **Cifrado de bases de datos (``SQLCipher``)** (ver [acá](https://developer.android.com/topic/security/data#encrypt-database))  
  Es posible cifrar bases de datos _SQLite_ cuando se requiere protección adicional de los datos en reposo.

- **Observación reactiva de datos**  
  Ambas soluciones permiten mantener la UI sincronizada con el estado persistido sin lógica de refresco manual.

- **_Testing_ de bases de datos**  
  Es posible utilizar bases de datos en memoria, configuraciones de test y *fake repositories* para pruebas unitarias y de integración.

## Almacenamiento de Archivos
> 🔍 Referencias:  
> https://developer.android.com/training/data-storage/app-specific  
> https://developer.android.com/training/data-storage/shared

Además de preferencias y bases de datos, Android permite persistir **archivos** directamente en el sistema de almacenamiento del dispositivo.  
Este enfoque es adecuado para:
- Archivos binarios (imágenes, videos, audio)
- Documentos (PDF, CSV, JSON)
- _Cache_ de recursos descargados
- Exportación o importación de datos

La documentación oficial distingue **dos grandes categorías** de almacenamiento de archivos:
1. **Almacenamiento específico de la app**
2. **Almacenamiento compartido**

La elección depende de:
- Quién debe poder acceder al archivo
- El ciclo de vida esperado de los datos
- Requerimientos de privacidad y permisos

### Almacenamiento específico de la app
El **almacenamiento específico de la app** (*app-specific storage*) está reservado exclusivamente para la aplicación que lo crea.

Características principales:
- Los archivos **solo son accesibles por la app**
- **No requiere permisos de almacenamiento**
- Los archivos se **eliminan automáticamente al desinstalar la app**
- Es la opción recomendada para datos internos de la aplicación

Android provee dos ubicaciones principales:
- **Almacenamiento interno**
- **Almacenamiento externo específico de la app**

#### Almacenamiento interno
El almacenamiento interno reside dentro del [*sandbox*](https://source.android.com/docs/security/app-sandbox) privado de la aplicación.

Características:
- Máximo nivel de privacidad
- No accesible por otras apps ni por el usuario directamente
- Ideal para datos sensibles o críticos

```kotlin
// Directorio raíz del almacenamiento interno
val internalDir: File = context.filesDir

// Crear un archivo
val file = File(internalDir, "user_data.txt")

file.writeText("Contenido interno")
```

También existe un directorio específico para **_cache_ interna**, cuyo contenido puede ser eliminado por el sistema si se requiere espacio.

```kotlin
val cacheFile = File(context.cacheDir, "temp_cache.txt")
```

#### Almacenamiento externo específico de la app
Este almacenamiento se encuentra en el almacenamiento externo del dispositivo, pero **aislado por aplicación**.

Características:
- Visible para el usuario (por ejemplo, vía USB)
- No requiere permisos de almacenamiento
- Se elimina automáticamente al desinstalar la app
- Útil para archivos grandes que no deben ocupar espacio interno

```kotlin
context.getExternalFilesDir(null)?.let { dir ->
    val file = File(dir, "exported_data.json")
}
```

Opcionalmente (no es un requisito para el correcto funcionamiento del almacenamiento externo específico de la app), pueden utilizarse subdirectorios convencionales definidos por el sistema para organizar los archivos por tipo.  
Estas constantes no afectan el modelo de acceso ni los permisos, y se utilizan únicamente con fines de clasificación.

```kotlin
val picturesDir = context.getExternalFilesDir(
    // Otros ejemplos son: Environment.DIRECTORY_DOCUMENTS o Environment.DIRECTORY_MOVIES
    Environment.DIRECTORY_PICTURES
)

val file = File(picturesDir, "image.jpg")
```

### Almacenamiento compartido
El **almacenamiento compartido** permite que los archivos sean accesibles:
- Por otras aplicaciones
- Por el usuario
- Por el sistema

Este modelo es común para:
- Imágenes y videos del usuario
- Documentos descargados
- Archivos exportados por la app

Desde Android 10 (API 29), este acceso está regulado por **_Scoped Storage_**.

#### Qué es *Scoped Storage*
Es un **modelo de acceso al almacenamiento** introducido en Android 10 (API 29) cuyo objetivo es **restringir y aislar el acceso de las aplicaciones al sistema de archivos compartido**, mejorando la privacidad del usuario. Dicho de otra forma, no se trata de una API específica ni de una librería que deba configurarse manualmente, sino de un **cambio de comportamiento del sistema operativo** que redefine cómo las aplicaciones pueden interactuar con el almacenamiento externo.

Bajo este modelo, cada aplicación:
- Tiene acceso directo únicamente a su propio espacio de almacenamiento
- No puede recorrer libremente el _filesystem_ compartido
- No puede leer ni modificar archivos de otras apps sin una mediación explícita

En versiones antiguas de Android, una app con permisos de almacenamiento podía acceder a rutas absolutas del almacenamiento externo (``/sdcard/...``) y manipular archivos de forma directa. Con *Scoped Storage*, este enfoque deja de ser el recomendado y, en Android moderno, deja de estar permitido para la mayoría de los casos.

En la práctica, *Scoped Storage* está **habilitado por defecto** y su comportamiento depende del `targetSdkVersion` de la aplicación. Para apps con `targetSdkVersion >= 30` (Android 11 o superior), este modelo es **obligatorio y no puede desactivarse**.  
Como consecuencia, el acceso al almacenamiento compartido debe realizarse mediante **APIs controladas por el sistema**, principalmente:
- ``MediaStore`` :arrow_right: Para imágenes, audio y video que forman parte de colecciones públicas
- **_Storage Access Framework_ (SAF)** :arrow_right: Para cuando el usuario debe seleccionar explícitamente archivos o carpetas

Estas APIs permiten que el usuario mantenga el control sobre sus datos y evitan el uso de permisos amplios de almacenamiento, alineándose con el modelo de privacidad de Android moderno.

#### Acceso mediante ``MediaStore``
Es la API utilizada para archivos multimedia (imágenes, audio, video).

Este enfoque:
- No requiere permisos especiales para escritura básica (para Android 10+ usando ``RELATIVE_PATH``)
- Inserta archivos en colecciones públicas del sistema
- Hace visibles los archivos para otras apps

```kotlin
val values = ContentValues().apply {
    put(MediaStore.Images.Media.DISPLAY_NAME, "imagen.jpg")
    put(MediaStore.Images.Media.MIME_TYPE, "image/jpeg")

    // RELATIVE_PATH permite definir la carpeta pública visible al usuario donde se almacenará el archivo, 
    // sin usar rutas absolutas ni permisos amplios (Android 10+)
    put(
        MediaStore.Images.Media.RELATIVE_PATH,
        "Pictures/MyApp"
    )
}

val uri = context.contentResolver.insert(
    MediaStore.Images.Media.EXTERNAL_CONTENT_URI,
    values
)
```

#### Acceso mediante *Storage Access Framework* (SAF)
Permite que el usuario seleccione explícitamente:
- Archivos
- Carpetas
- Ubicaciones de almacenamiento

La app obtiene acceso **limitado y mediado por el sistema** únicamente a los recursos seleccionados por el usuario. SAF prioriza la privacidad del usuario sobre la conveniencia de la app, por lo que todo acceso al almacenamiento compartido debe ser explícito y reversible.

> ℹ️ **Nota sobre permisos persistentes**  
> Cuando se utiliza ``ActivityResultContracts.OpenDocument`` (o `ACTION_OPEN_DOCUMENT` en caso de usar ``Intents``), Android otorga permisos temporales de lectura y/o escritura sobre la URI seleccionada.  
> Si la app necesita acceder a ese archivo en ejecuciones futuras, debe llamar explícitamente a  
> [`takePersistableUriPermission`](https://developer.android.com/reference/kotlin/android/content/ContentResolver#takepersistableuripermission), persistiendo el permiso de lectura (y/o escritura) más allá del ciclo de vida inmediato.  
> Este mecanismo es **exclusivo del _Storage Access Framework_**.

📌 Ejemplo con ``startActivityForResult`` (deprecado):

```kotlin
val intent = Intent(Intent.ACTION_OPEN_DOCUMENT).apply {
    type = "*/*"
}

startActivityForResult(intent, REQUEST_CODE_OPEN_FILE)
```

📌 Ejemplo con **_Activity Result API_**:

```kotlin
// Registrar el launcher en la Activity o Fragment
private val openDocumentLauncher =
    registerForActivityResult(ActivityResultContracts.OpenDocument()) { uri: Uri? ->
        if (uri != null) {
            // Persistir permiso de lectura para usos futuros.
            // En caso de necesitar modificar el archivo, debe persistirse también Intent.FLAG_GRANT_WRITE_URI_PERMISSION.
            contentResolver.takePersistableUriPermission(
                uri,
                Intent.FLAG_GRANT_READ_URI_PERMISSION
            )

            // El usuario seleccionó un archivo.
            // Como se persistió el permiso, ahora la URI puede usarse incluso después de reiniciar la app
            contentResolver.openInputStream(uri)?.use { input ->
                // Procesar archivo
            }
        }
    }

// Lanzar el selector de archivos del SAF
openDocumentLauncher.launch(arrayOf("*/*"))

// Alternativamente, también se pueden filtrar tipos
// openDocumentLauncher.launch(arrayOf("application/pdf", "image/*"))
```

Este enfoque es ideal cuando:
- El usuario debe elegir el archivo
- Se requiere acceso a documentos arbitrarios
- Se desea máxima compatibilidad y privacidad

### Cuándo usar cada tipo de almacenamiento

| Tipo de almacenamiento      | Privacidad             | Permisos                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               | Caso de uso típico           |
|-----------------------------|------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|------------------------------|
| Interno                     | Muy alta               | - No requiere ningún permiso<br>- Acceso exclusivo de la app                                                                                                                                                                                                                                                                                                                                                                                                                                                                                           | Datos internos sensibles     |
| Externo específico          | Alta                   | - No requiere ningún permiso<br>- Aunque esté en “_external storage_”, sigue estando aislado                                                                                                                                                                                                                                                                                                                                                                                                                                                           | Archivos grandes propios     |
| Compartido (``MediaStore``) | Media                  | - **Escritura**:<br/>&nbsp;&nbsp;&nbsp;&nbsp;- Generalmente no requiere permisos especiales al insertar en colecciones públicas<br/>- **Lectura**:<br/>&nbsp;&nbsp;&nbsp;&nbsp;- Requiere permisos específicos según tipo de archivo y API level<br/>&nbsp;&nbsp;&nbsp;&nbsp;- En Android moderno:<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- Ya no existe un permiso único “amplio”<br/>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;- Ejemplos de permisos granulares (específicos): ``READ_MEDIA_IMAGES``, ``READ_MEDIA_VIDEO``, etc. | Multimedia del usuario       |
| Compartido (SAF)            | Controlada por usuario | - No se usan permisos de almacenamiento<br/>- El acceso está mediado por una acción explícita del usuario<br/>- El permiso es implícito y limitado al recurso elegido                                                                                                                                                                                                                                                                                                                                                                                  | Importar / exportar archivos |

📌 **Conclusión**
- Usar **almacenamiento específico de la app** siempre que sea posible
- Preferir ``MediaStore`` o **SAF** para archivos compartidos
- Evitar acceso directo al _filesystem_ compartido
- **_Scoped Storage_** es el modelo recomendado y obligatorio en Android moderno

### Conceptos avanzados / relacionados
- **_Persisted URI permissions_ (SAF)** (ver [acá](https://developer.android.com/guide/topics/providers/document-provider))  
  El acceso a archivos seleccionados por el usuario puede persistirse entre reinicios de la app mediante permisos de URI persistentes.

- **Actualización y borrado en `MediaStore`** (ver [acá](https://developer.android.com/training/data-storage/shared/media))  
  Los archivos multimedia compartidos pueden modificarse o eliminarse utilizando URIs y [`ContentResolver`](#acceso-mediante-contentresolver), respetando el modelo de *Scoped Storage*.

- **Diferencias por API _level_** (ver [acá](https://developer.android.com/about/versions/10/privacy/changes#scoped-storage))  
  El comportamiento del almacenamiento compartido y los permisos varía significativamente entre Android 9, 10, 11 y versiones posteriores.

- **Estrategias de _cache_ y eviction** (ver [acá](https://developer.android.com/topic/performance/storage#cache))  
  Los directorios de *cache* pueden ser limpiados por el sistema; la app debe tolerar su eliminación y definir políticas de regeneración.

- **_Backup_ y restauración de archivos** (ver [acá](https://developer.android.com/guide/topics/data/autobackup))  
  Los archivos pueden incluirse o excluirse del sistema de *Auto Backup*, afectando su persistencia tras reinstalaciones o cambios de dispositivo.

- **Cifrado de archivos (_Encrypted File APIs_)** (ver [acá](https://developer.android.com/topic/security/data#encrypt-files))  
  Android provee APIs para cifrar archivos individuales cuando se requiere mayor protección de datos almacenados en disco.

- **Separación entre persistencia y compartición**  
  Guardar un archivo no implica automáticamente que deba ser accesible por otras apps o por el usuario.

## Compartir datos
> 🔍 Referencias:  
> https://developer.android.com/guide/components/intents-filters  
> https://developer.android.com/guide/components/aidl  
> https://developer.android.com/training/sharing

En Android, **compartir datos** se refiere a los mecanismos mediante los cuales una app permite que **otros componentes, procesos o aplicaciones** accedan a información que la app expone o genera.

A diferencia de la **persistencia**, el foco no está en **cómo se almacenan** los datos, sino en **cómo se exponen, transfieren o consultan**, generalmente de forma **temporal, controlada y acotada**.

El mecanismo a utilizar depende principalmente de:
- El tamaño y complejidad de los datos
- Si la comunicación es _intra-app_ o entre apps
- Si ocurre dentro del mismo proceso o entre procesos distintos (IPC)

### *Intents* y extras
Los **_Intents_** son el mecanismo de más alto nivel para compartir datos en Android.

Características:
- Adecuados para **datos pequeños**
- Comunicación **puntual y efímera**
- No garantizan persistencia ni disponibilidad del receptor
- Orientados a **acciones**, no a APIs de datos

Los datos se envían típicamente como:
- `String`
- Tipos primitivos (`Int`, `Boolean`, etc.)
- `Bundle`
- `Uri`

Son ideales para:
- Compartir texto o contenido simple
- Lanzar acciones en otras apps
- Comunicación básica entre componentes

#### *Intent* explícito
Indica exactamente **qué componente de la misma app debe manejarlo**.  
Puede pasar datos adicionales mediante ``extras``.

Características:
- Comunicación **_intra-app_** 
- El receptor está completamente definido 
- Uso típico para navegación entre pantallas 
- Los datos se reciben en la ``Activity`` destino mediante ``intent.getIntExtra(...)``

Casos de uso típicos:
- Navegar a una pantalla de detalle pasando un identificador 
- Pasar _flags_ o parámetros simples entre pantallas

📌 Ejemplo:

```kotlin
val intent = Intent(context, DetailActivity::class.java).apply {
    putExtra("user_id", 42)
}

startActivity(intent)
```

#### *Intent* implícito
Sirve para **compartir contenido con otras apps**.  
No especifica un componente concreto, sino una acción (por ejemplo, ``ACTION_SEND``) que otras apps pueden manejar.

Características:
- Comunicación **_inter-app_** 
- Basado en acciones y filtros declarados en el ``AndroidManifest`` 
- El usuario decide qué app recibe los datos 
- No hay control directo sobre qué aplicación concreta será el receptor

Casos de uso típicos:
- Compartir texto, _links_ o contenido simple 
- Integrarse con apps externas (_WhatsApp_, _Gmail_, _Drive_, etc.)

📌 Ejemplo:

```kotlin
// El sistema buscará todas las apps instaladas que puedan manejar ACTION_SEND con text/plain
val intent = Intent(Intent.ACTION_SEND).apply {
    type = "text/plain"
    putExtra(Intent.EXTRA_TEXT, "Texto a compartir")
}

// createChooser fuerza la aparición del selector, evita que una app se abra automáticamente por defecto
// y mejora la experiencia y el control del usuario
startActivity(Intent.createChooser(intent, "Compartir con"))
```

### IPC (*Inter-Process Communication*)
Cuando se requiere una comunicación **más estructurada y continua** entre procesos, Android utiliza **IPC**.

Esto es necesario cuando:
- Los componentes se ejecutan en procesos distintos
- Se necesita una **API bien definida**
- El intercambio no es simplemente disparar una acción

Android provee mecanismos como:
- `Binder`
- **AIDL** (***Android Interface Definition Language***)

Estos mecanismos operan a un nivel más bajo que los ``Intents`` y permiten:
- Comunicación directa entre procesos
- Control estricto de acceso
- Mayor eficiencia y escalabilidad

> ℹ️ **Nota:**  
> AIDL se define en archivos `.aidl` y genera código `Binder` automáticamente.  
> No se utiliza como un `Intent`, sino como una interfaz de servicio (``Service``).

### Límites y buenas prácticas
- No usar `Intents` para datos grandes o complejos
- No asumir que el receptor siempre estará disponible
- No usar IPC directo si no es estrictamente necesario
- Para compartir datos de forma estructurada, persistente y reutilizable entre apps, considerar **_Content Providers_** (ver [acá](#content-providers))

📌 **Resumen**
- Compartir datos trata sobre **acceso y comunicación**, no sobre almacenamiento
- *Intents* :arrow_right: Simples, desacoplados y temporales
- IPC :arrow_right: Comunicación estructurada entre procesos
- *Content Providers* :arrow_right: Modelo estándar para exponer datos entre apps

### Conceptos avanzados / relacionados
- **_Bound Services_** (ver [acá](https://developer.android.com/guide/components/bound-services))  
  Permiten que una app exponga una **API** a otros componentes o aplicaciones mediante un `Service` enlazado (*bound*).  
  Se apoyan en `Binder` (o AIDL) y son adecuados cuando se requiere **comunicación directa, estructurada y continua**.

- **_Messenger_ (IPC simplificado)** (ver [acá](https://developer.android.com/guide/components/bound-services#Messenger))  
  Alternativa a AIDL para IPC basado en **mensajes** (`Message` + `Handler`).  
  Es más simple de implementar, pero menos flexible y potente que AIDL.

- **Permisos y visibilidad de componentes** (ver [acá](https://developer.android.com/guide/topics/manifest/manifest-intro))  
  Android permite controlar **quién puede acceder** a `Activities`, `Services` y otros componentes compartidos mediante:
    - Permisos declarados en el `AndroidManifest`
    - Atributos como `exported`
    - Restricciones a nivel de firma o usuario  
  Son clave para compartir datos de forma **segura**.

- **_App sandboxing_ y aislamiento de procesos** (ver [acá](https://source.android.com/docs/security/app-sandbox))  
  Cada app se ejecuta en su propio **_sandbox_**, con un UID y espacio de memoria aislados.  
  Este modelo de seguridad es la razón por la cual, cuando se necesita compartir datos entre apps o procesos, es obligatorio recurrir a **IPC** u otros mecanismos controlados por el sistema.

## Compartir archivos
> 🔍 Referencias:  
> https://developer.android.com/training/secure-file-sharing  
> https://developer.android.com/about/versions/11/privacy/storage

En Android, **compartir archivos** implica permitir que **otras apps accedan a archivos físicos** (imágenes, PDFs, videos, etc.) que pertenecen a nuestra aplicación.

A diferencia de *Compartir datos*:
- No se trata solo de pasar valores en memoria
- Involucra **archivos reales en el sistema**
- Requiere considerar **permisos, seguridad y control de acceso**

Desde Android 7 (API 24) y, especialmente, con *Scoped Storage* (Android 10+), **no se deben compartir rutas de archivos (`File`) directamente**, sino **URIs controladas por el sistema**.

### URI y acceso indirecto
El concepto central para compartir archivos en Android es el **URI (_Uniform Resource Identifier_)**.

Un ``Uri``:
- Representa una **referencia abstracta** a un archivo
- No expone la ruta real en el sistema
- Permite que el sistema controle permisos y acceso

Las apps receptoras:
- No pueden acceder al archivo libremente
- Solo pueden leer/escribir **mientras el permiso esté concedido**
- No conocen la ubicación real del archivo

### `FileProvider`
El mecanismo recomendado para compartir archivos privados de la app es [**`FileProvider`**](https://developer.android.com/reference/kotlin/androidx/core/content/FileProvider).

`FileProvider`:
- Es un tipo especial de `ContentProvider` (ver [acá](#content-providers))
- Convierte un `File` interno en un `content://Uri`
- Otorga permisos **temporales y específicos**

Ventajas:
- Evita exponer rutas (`file://`)
- Cumple con las restricciones de seguridad modernas
- Funciona con *Scoped Storage*

📌 Ejemplo:

```kotlin
// Obtener un Uri seguro a partir de un File usando FileProvider
val file = File(context.filesDir, "document.pdf")

val uri = FileProvider.getUriForFile(
    context,
    "com.example.app.fileprovider",
    file
)
```

### Compartir archivos mediante `Intent`
Una vez obtenido el `Uri`, se utiliza un `Intent` implícito para compartir el archivo.

Buenas prácticas:
- Usar `ACTION_SEND`
- Especificar correctamente el `MIME type`
- Conceder permisos temporales con _flags_

📌 Ejemplo:

```kotlin
val shareIntent = Intent(Intent.ACTION_SEND).apply {
    type = "application/pdf"
    putExtra(Intent.EXTRA_STREAM, uri)
    addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION)
}

startActivity(Intent.createChooser(shareIntent, "Compartir archivo"))
```

### Impacto de *Scoped Storage*
Desde Android 10 (API 29), el sistema introduce **_Scoped Storage_** (ver [acá](#qué-es-scoped-storage)), que redefine cómo las apps acceden a archivos.

Principios clave:
- Cada app tiene acceso directo solo a su propio espacio
- El acceso a archivos compartidos está mediado por el sistema
- Se priorizan `Uri`, `MediaStore` y `FileProvider`

Impacto en compartir archivos:
- No se pueden compartir _paths_ absolutos
- Los permisos son siempre explícitos y temporales
- El sistema revoca accesos automáticamente

### Casos de uso típicos
- Compartir una imagen generada por la app
- Enviar un PDF por _mail_ o mensajería
- Exportar un archivo para que otra app lo procese
- Adjuntar archivos a un ``Intent`` externo

### Límites y buenas prácticas
- ❌ No usar `file://` URIs
- ❌ No compartir rutas internas del sistema
- ✅ Usar siempre `content://` URIs
- ✅ Conceder permisos solo cuando sea necesario
- ✅ Definir correctamente el `MIME type`

📌 **Resumen**
- Compartir archivos implica acceso a recursos físicos
- `Uri` es la abstracción central
- `FileProvider` es el mecanismo recomendado
- *Scoped Storage* define los límites de acceso modernos

### Conceptos avanzados / relacionados
- **`MediaStore`** (ver [acá](https://developer.android.com/training/data-storage/shared/media))  
  Permite compartir archivos multimedia (imágenes, audio, video) en colecciones públicas del sistema  

- **Permisos temporales (`FLAG_GRANT_*`)** (ver [acá](https://developer.android.com/reference/kotlin/android/content/Intent#flag_grant_read_uri_permission))  
  El sistema permite conceder permisos de lectura/escritura limitados en el tiempo y alcance   

- **Seguridad y exposición mínima**  
  Compartir archivos debe hacerse siguiendo el principio de [*least privilege*](../../Glosary%20&%20Core%20Concepts/Software%20in%20general.md#principle-of-least-privilege-polp)

## *Content Providers*
> 🔍 Referencias:  
> https://developer.android.com/guide/topics/providers/content-providers  

Los **_Content Providers_** son el mecanismo estándar de Android para **exponer datos de forma estructurada y segura** a otros componentes o aplicaciones, **dentro o fuera del proceso**.

A diferencia de los [*Intents*](#intents-y-extras) o del [IPC](#ipc-inter-process-communication) directo, un *Content Provider* define un **modelo de acceso a datos**, no un intercambio puntual.  
Su propósito principal es permitir **consultar, insertar, actualizar o borrar datos** mediante una **API uniforme**, basada en URIs.

Son especialmente útiles cuando:
- Los datos deben ser **compartidos entre apps**
- Se requiere **acceso controlado y reutilizable**
- Los datos tienen una **estructura persistente** (tablas, filas, colecciones)

### Modelo basado en URIs
El acceso a un *Content Provider* se realiza siempre mediante **URIs**, con el esquema ``content://authority/path``

Donde:
- `authority` identifica al *Content Provider*
- `path` identifica el recurso (tabla, colección, ítem, sub-recurso)

Ejemplos comunes:
- `content://contacts/people`
- `content://media/external/images/media`

Las operaciones disponibles son:
- `query()` :arrow_right: Leer datos
- `insert()` :arrow_right: Insertar datos
- `update()` :arrow_right: Modificar datos
- `delete()` :arrow_right: Borrar datos

Todas se ejecutan a través de `ContentResolver`.

### Acceso mediante `ContentResolver`
Desde el lado consumidor, Android provee la clase `ContentResolver`, que actúa como **puerta de entrada** a cualquier *Content Provider*, propio o externo.

Este modelo:
- Desacopla completamente al consumidor del origen de datos
- Permite que el proveedor controle permisos y validaciones
- Es consistente en todo el sistema Android

📌 Ejemplo de consulta:

```kotlin
val cursor = contentResolver.query(
    uri,
    arrayOf("name", "age"),
    "age > ?",
    arrayOf("18"),
    null
)

cursor?.use {
    while (it.moveToNext()) {
        val name = it.getString(0)
        val age = it.getInt(1)
    }
}
```

### *Providers* del sistema
Android incluye varios *Content Providers* listos para usar, entre ellos:
- **``ContactsProvider``** (contactos)
- **``MediaStore``** (archivos multimedia)
- **``CalendarProvider``**
- **``SettingsProvider``**

Estos providers permiten a las apps interactuar con datos del sistema **sin acceso directo al almacenamiento ni a bases internas**.

📌 Ejemplo:

```kotlin
val uri = MediaStore.Images.Media.EXTERNAL_CONTENT_URI
```

### Seguridad y permisos
Los *Content Providers* ofrecen un modelo de seguridad más fino que otros mecanismos de compartición:
- Permisos declarados en el `AndroidManifest`
- _Providers_ no exportados por defecto
- Permisos a nivel de lectura y escritura
- Concesión de permisos temporales mediante URIs (`FLAG_GRANT_READ_URI_PERMISSION`)

Esto permite:
- Exponer solo los datos necesarios
- Evitar acceso no autorizado
- Cumplir con el [PoLP](../../Glosary%20&%20Core%20Concepts/Software%20in%20general.md#principle-of-least-privilege-polp)

### Cuándo usar *Content Providers*
✅ Usar un *Content Provider* es recomendable cuando:
- Los datos deben ser accesibles por **múltiples apps**
- Se necesita una **API estable y bien definida**
- Los datos son **persistentes y estructurados**
- Se requiere **control de permisos y visibilidad**

❌ No son ideales cuando:
- El intercambio es puntual y simple → *Intents*
- La comunicación es directa y privada → IPC / *Bound Services*
- Los datos no deben salir del [_sandbox_](https://source.android.com/docs/security/app-sandbox) de la app

📌 **Resumen**
- *Content Providers* exponen **datos**, no acciones
- Usan URIs y `ContentResolver` como contrato
- Son el mecanismo estándar para compartir datos persistentes
- Proveen un modelo sólido de seguridad y control de acceso

### Conceptos avanzados / relacionados
- **`FileProvider`** (ver [acá](https://developer.android.com/training/secure-file-sharing))  
  Implementación especializada de *Content Provider* orientada exclusivamente a **compartir archivos**.  
  Traduce archivos privados (`File`) en URIs `content://` con **permisos temporales**, y es el mecanismo recomendado para compartir archivos entre apps desde Android 7+.

- **`DocumentsProvider` y _Storage Access Framework_ (SAF)** (ver [acá](https://developer.android.com/guide/topics/providers/document-provider))  
  Variante avanzada de *Content Provider* utilizada por el sistema para exponer documentos y árboles de archivos.  
  Permite a las apps integrarse con el **selector de archivos del sistema** y ofrecer acceso persistente mediante URIs (`ACTION_OPEN_DOCUMENT`).

- **Permisos persistentes sobre URIs (`takePersistableUriPermission`)** (ver [acá](https://developer.android.com/reference/kotlin/android/content/ContentResolver#takepersistableuripermission))  
  En el contexto de SAF, una app puede solicitar permisos de lectura/escritura **persistentes** sobre un `Uri`, que sobreviven reinicios de la app y del sistema.  
  Esto habilita acceso controlado a documentos elegidos por el usuario sin requerir permisos amplios de almacenamiento.

- **Notificaciones de cambio (`ContentObserver`)** (ver [acá](https://developer.android.com/reference/kotlin/android/database/ContentObserver))  
  Permiten a una app **observar cambios** en los datos expuestos por un *Content Provider*.  
  Son útiles cuando se necesita reaccionar a modificaciones en datos compartidos (por ejemplo, cambios en contactos o medios).

- **`Room` + `ContentProvider` (exposición controlada)**  
  Aunque `Room` es una solución de persistencia interna, puede usarse como _backend_ de un *Content Provider* para exponer **solo una vista controlada** de los datos, manteniendo encapsulamiento y seguridad.
