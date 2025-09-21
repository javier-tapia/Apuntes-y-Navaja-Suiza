## Persistencia de datos

- [*SharedPreferences y EncryptedSharedPreferences*](#sharedpreferences-y-encryptedsharedpreferences)
- [*DataStore*](#datastore)
- [Bases de datos: *Room y Realm*](#bases-de-datos-room-y-realm)
    - [*Room*](#room)
    - [*Realm*](#realm)

---

### *SharedPreferences y EncryptedSharedPreferences*
TODO...

### *DataStore*
TODO...

### Bases de datos: *Room y Realm*

#### *Room*
Android soporta *SQLite* desde el comienzo. Sin embargo, para hacerlo funcionar, siempre fue necesario escribir mucho código *boilerplate*. Además, *SQLIte* no guardaba *POJO’s* (*plain-old Java objects* u objetos planos de Java), y no comprobaba consultas (*queries*) en tiempo de compilación. *Room* vino a solucionar estos problemas. Es una librería de mapeo *SQLite*, capaz de persistir *POJO’s* de Java, convertir consultas directamente a objetos, comprobar errores en tiempo de compilación y **producir observables ***LiveData*** de los resultados de las consultas**. *Room* es una librería ORM (*Object Relational Mapping* o mapeo objeto-relacional) con algunos agregados de Android.  
Para crear una base de datos con *Room*, se necesita una **Entidad** (***Entity***) para persistir, que puede ser cualquier *POJO* Java (en Kotlin, se utilizan las ***data classes*** para este propósito) y representa una tabla dentro de la base de datos; una ***interface Dao*** (*data acces object* u objeto de acceso a datos) para hacer consultas y operaciones de entrada/salida; y también **una clase abstracta ***Database*** que debe extender a** ***RoomDatabase***.  
*Room* genera todo el código necesario para actualizar el objeto *LiveData* cuando se actualiza una base de datos. El código generado **ejecuta la consulta de manera asíncrona** en un subproceso en segundo plano cuando es necesario. Este patrón es útil para mostrar los datos en una UI sincronizada con los datos almacenados en una base de datos.  
Para usar *Room* en una *app*, agregar las siguientes **dependencias** al archivo *build.gradle*:

```kotlin
    def room_version = "<VERSION>"
    
    implementation "androidx.room:room-runtime:$room_version"
    kapt "androidx.room:room-compiler:$room_version"
    
    // optional - Kotlin Extensions and Coroutines support for Room
    implementation "androidx.room:room-ktx:$room_version"
    
    // optional - Test helpers
    testImplementation "androidx.room:room-testing:$room_version"
```

La *app* usa la base de datos de *Room* para obtener los objetos de acceso a los datos (*DAO*) asociados con esa base de datos. Luego, la *app* usa cada *DAO* para obtener entidades de la base de datos y guardar los cambios realizados en esas entidades en la base de datos. Por último, la *app* usa una entidad para obtener y configurar valores que corresponden a columnas de tabla dentro de la base de datos.  
En el siguiente fragmento de código, se muestra una configuración de base de datos de ejemplo con una entidad y un *DAO*:

```kotlin
    @Entity
    data class User(
        @PrimaryKey val uid: Int,
        @ColumnInfo(name = "first_name") val firstName: String?,
        @ColumnInfo(name = "last_name") val lastName: String?
    )
    
    @Dao
    interface UserDao {
        @Query("SELECT * FROM user")
        fun getAll(): List<User>
    
        @Query("SELECT * FROM user WHERE uid IN (:userIds)")
        fun loadAllByIds(userIds: IntArray): List<User>
    
        @Query(
            "SELECT * FROM user WHERE first_name LIKE :first AND " +
                    "last_name LIKE :last LIMIT 1"
        )
        fun findByName(first: String, last: String): User
    
        @Insert
        fun insertAll(vararg users: User)
    
        @Delete
        fun delete(user: User)
    
        @Update
        fun updateUsers(vararg users: User)
    }
    
    @Database(entities = arrayOf(User::class), version = 1)
    abstract class AppDatabase : RoomDatabase() {
        abstract fun userDao(): UserDao
    }
```

Después de crear los archivos anteriores, se puede obtener una instancia de la base de datos creada con el siguiente código:

```kotlin
    import kotlin.jvm.java
    
    val db = Room.databaseBuilder(
        applicationContext,
        AppDatabase::class.java, "database-name"
    ).build()
```

En *Room 2.2* y versiones posteriores, se puede asegurar de que la UI de la *app* se mantenga actualizada mediante los ***Flows*** de Kotlin. Para que se actualice automáticamente la UI cuando cambien los datos subyacentes, se escriben métodos de consulta que muestren objetos *Flow*:

```kotlin
    @Query("SELECT * FROM User")
    fun getAllUsers(): Flow<List<User>>
```

Cada vez que cambia alguno de los datos de la tabla, el objeto *Flow* que se muestra vuelve a activar la consulta y vuelve a emitir el conjunto de resultados completo.  
También se puede agregar la palabra reservada ***``suspend``*** a los métodos del *DAO* para que sean asíncronos mediante la funcionalidad de **corrutinas** de Kotlin. De esta manera, se asegura de que no se puedan ejecutar en el subproceso principal.

```kotlin
    @Dao
    interface MyDao {
        @Insert(onConflict = OnConflictStrategy.REPLACE)
        suspend fun insertUsers(vararg users: User)
    
        @Update
        suspend fun updateUsers(vararg users: User)
    
        @Delete
        suspend fun deleteUsers(vararg users: User)
    
        @Query("SELECT * FROM user")
        suspend fun loadAllUsers(): Array<User>
    }
```

#### *Realm*
Es una base de datos móvil que se ejecuta directamente en teléfonos, *tablets* o dispositivos portátiles. Los datos se exponen **directamente como objetos** y se pueden **consultar mediante código**, lo que elimina la necesidad de usar ORM, que puede traer problemas de rendimiento y mantenimiento.
