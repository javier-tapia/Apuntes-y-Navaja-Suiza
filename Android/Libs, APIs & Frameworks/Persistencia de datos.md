<h1>Persistencia de datos</h1>

***Index***:
<!-- TOC -->
  * [*Key-Value* / *Preferences*](#key-value--preferences)
    * [*SharedPreferences y EncryptedSharedPreferences*](#sharedpreferences-y-encryptedsharedpreferences)
    * [*DataStore*](#datastore)
  * [Bases de datos](#bases-de-datos)
    * [*Room*](#room)
      * [1. Agregar dependencias y *plugins*](#1-agregar-dependencias-y-plugins)
      * [2. Crear Entidades](#2-crear-entidades)
      * [3. Crear DAOs](#3-crear-daos)
      * [4. Crear la Base de Datos](#4-crear-la-base-de-datos)
      * [5. Instanciar y configurar la Base de Datos](#5-instanciar-y-configurar-la-base-de-datos)
      * [6. Usar la Base de Datos](#6-usar-la-base-de-datos)
    * [*Realm*](#realm)
      * [1. Agregar dependencias](#1-agregar-dependencias)
      * [2. Definir modelos de datos](#2-definir-modelos-de-datos)
      * [3. Configurar e inicializar *Realm*](#3-configurar-e-inicializar-realm)
      * [4. Leer y escribir datos con *Realm*](#4-leer-y-escribir-datos-con-realm)
      * [5. Uso típico en una arquitectura Android (*Repository* + *ViewModel* + UI)](#5-uso-típico-en-una-arquitectura-android-repository--viewmodel--ui)
<!-- TOC -->

---

## *Key-Value* / *Preferences*
### *SharedPreferences y EncryptedSharedPreferences*
TODO...

### *DataStore*
TODO...

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
