# Apuntes de Android  

### Contenido general:  
#### [0. ***ViewBinding*** y ***DataBinding***](#viewbinding-y-databinding)
#### [1. Componentes de arquitectura: ***Lifecycle***, ***ViewModel*** y ***LiveData***](#componentes-de-arquitectura-lifecycle-viewmodel-y-livedata)
#### [2. Bases de datos: ***Room*** y ***Realm***](#bases-de-datos-room-y-realm)
#### [3. Arquitecturas: MVP / MVVM + ***Repository***](#arquitecturas-mvp-mvvm-repository)
#### [4. ***Navigation component***](#navigation-component)
#### [5. UI: Interfaz de Usuario](#ui-interfaz-de-usuario)
   - ##### 5.1 ***Styles*** y ***Themes***
   - ##### 5.2 ***Custom Views***
   - ##### 5.3 ***RecyclerView***
   - ##### 5.4 ***Menus***

#### [6. Manejo de asíncronos: ***Coroutines / Flow / Rx***](#manejo-de-asincronos-coroutines-flow-rx)
#### [7. ***Retrofit***](#retrofit)
#### [8. ***SharedPreferences / EncryptedSharedPreferences***](#sharedpreferences-encryptedsharedpreferences)
#### [9. Inyección de dependencias: ***Koin*** y ***Dagger***](#inyeccion-de-dependencias-koin-y-dagger)
#### [10. Testing: ***Junit*** y ***Mockito***](#testing-junit-y-mockito)
#### [11. ***Players: ExoPlayer*** y ***JW Player***](#players-exoplayer-y-jwplayer)
#### [12. ***OAuth: Facebook, Twitter, Google+***](#oauth-facebook-twitter-google)
#### [13. ***Frameworks / SDK: Firebase, Fabric, Sentry, Segment, Facebook***](#frameworks-sdk-firebase-fabric-sentry-segment-facebook)
#### [14. ***Deep linking***](#deep-linking)
#### [15. ***Push notifications***](#push-notifications)
#### [16. ***Jetpack Compose***](#jetpack-compose)
#### [Referencias](#referencias-y-fuentes)

---
---



### ***ViewBinding*** y ***DataBinding***

#### ***ViewBinding***  
Es una forma de **acceder a las vistas** (*xml*) que equilibra el rendimiento y la potencia, **sin necesidad de recurrir a otras alternativas**, como el método ***``findViewById()``*** (que en sí mismo es bastante costoso), ***Butterknife*** o ***Synthetic***. *ViewBinding* es un ‘subconjunto’ de *DataBinding* que evita la sobrecarga de compilación que produce el utilizar *DataBinding*. Se usa si no se necesita añadir código a las vistas (*xml*) ni realizar esa asignación directa entre una variable del código y una vista del *xml* que permite *DataBinding*. A diferencia de otras formas de enlazar las vistas, como por ejemplo *synthetic* de *kotlin extensions*, *ViewBinding* permite que **el compilador conozca la nulidad de la vista**.
La forma de configurarlo depende de la versión de Android Studio. Para Android Studio 4.0 y siguientes, en el *build.gradle* poner lo siguiente **dentro de** ***android{}***:  

```kotlin
buildFeatures {
    viewBinding = true
}
```

Si es anterior al 4.0, se usa lo siguiente:

```kotlin
viewBinding { enabled = true }
```

##### Cómo usar *ViewBinding* en una *Activity*  
Luego de crear la variable ***``binding``*** a nivel de clase con ***``lateinit``***, lo único que se necesita es modificar la forma en que se infla la vista. En vez de llamar a ***``setContentView``*** con el identificador del ***layout***, se hace pasándole la vista que se ha inflado previamente con *ViewBinding*:

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // Llamada a setContentView con el identificador del layout
        // setContentView(R.layout.activity_main)

        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
    }
}
```

***``binding.root``*** contiene la raíz del *layout* que ha sido inflado previamente. Ahora solo se necesita acceder a las *properties* del objeto para usar sus vistas:

```kotlin
binding.button.setOnClickListener {
    // Hacer algo cuando se haga clic en el botón
}
```

##### Cómo usar *ViewBinding* en un *Adapter* de *RecyclerView*  

Aquí hay al menos un par de formas de hacerlo:  
   - Usando el **método** ***``inflate``*** en el ***onCreateViewHolder*** y almacenando el objeto ***binding***
   - Inflando de forma clásica el objeto y usando el **método** ***``bind``*** en el ***ViewHolder*** para tenerlo allí disponible.

Con la segunda forma, no hay que andar preocupándose de pasar el objeto de un punto a otro del *adapter*, sino que se crea justo donde se necesita.

```kotlin
override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): ViewHolder {
    val v = parent.inflate(R.layout.view_media_item, false)
    return ViewHolder(v, listener)
}

class ViewHolder(view: View, private val listener: MediaListener) :
    RecyclerView.ViewHolder(view) {

    private val binding = ViewMediaItemBinding.bind(view)

    fun bind(mediaItem: MediaItem) = with(binding) {
        mediaTitle.text = mediaItem.title
        mediaThumb.loadUrl(mediaItem.url)
        root.setOnClickListener {
            android.widget.Toast.makeText(this, "mediaItem.title", Toast.LENGTH_SHORT).show()
        }

        mediaVideoIndicator.visibility = when (mediaItem.type) {
            MediaItem.Type.PHOTO -> View.GONE
            MediaItem.Type.VIDEO -> View.VISIBLE
        }
    }
}
```

A partir de ese punto, ya se puede usar exactamente igual que como en la *Activity*.

#### ***DataBinding***  
Para usar *DataBinding* dentro de una *activity* o un *fragment*, se puede hacer lo mismo que en el caso de *ViewBinding*. Sin embargo, el fuerte de *DataBinding* es que permite **escribir código en el mismo** ***xml*** para **asignarle variables del** ***ViewModel***, así cuando se modifican en el *ViewModel*, **la vista se modifica directamente, sin pasar por la** ***activity*** **o** ***fragment***. Para configurarlo, en el ***build.gradle*** se debe agregar el plugin de ***Kotlin kapt*** para que *DataBinding* pueda generar código a través de *procesamiento de anotaciones* (***``apply plugin: ‘kotlin-kapt’``***) y, además, se agrega **dentro de** ***android{}***:

```kotlin
dataBinding { enabled = true }
```

Para Android Studio 4.0 y siguientes, sería similar a *ViewBinding*:

```kotlin
buildFeatures {
    dataBinding = true
}
```

Luego, en el *xml*, se debe convertir el *layout* a *DataBinding* (posicionando el cursor sobre la vista raíz (*root*) y oprimiendo ***Alt + Enter -> convert to DataBinding Layout***). Esto genera una etiqueta ***``<data>``*** fuera de la vista raíz del diseño, en la que se pueden **incluir variables a usar dentro del** ***xml***, con un ***``name``*** y un ***``type``***, como también ***imports*** con clases necesarias. No tener *DataBinding* automáticamente para todas las *activities* representa una ventaja, ya que así no se generan todas las clases que se precisan para usar la herramienta, de tal manera que se puede elegir en qué lugares se debe generar ese código extra que además tiene un costo.  
Con *DataBinding*, entonces, se puede agregar una variable al *xml* que sea de **tipo del** ***ViewModel***, lo cual permite que el *xml* pueda acceder directamente al *ViewModel*, sin pasar por la *activity/fragment*, **llamando a los objetos** ***LiveData*** **que el** ***viewModel*** **contiene**. Como *DataBinding* ‘entiende’ a los objetos *LiveData*, cuando uno de estos valores del *ViewModel* cambian, la vista cambia automáticamente. Esto **evita tener que observar los** ***LiveData*** **en la** ***activity/fragment***, en cuyo caso estaría más cercano al patrón MVP que al MVVM.  
La expresión se escribe ***``“@{ CÓDIGO }”``***, y **permite acceder tanto a una variable como a un método** (ver el apartado de *Arquitecturas: ViewModel*), con una sintaxis similar a un *lambda*, por ejemplo:  

```java
“@{ () -> viewModel.onButtonPressed ( name.getText().toString()) }”
```

```xml
<data>
<import type="android.view.View" />
<variable
    name="viewModel"
    type="com.ejemplo.(...).EjemploViewModel" />
</data>

<LinearLayout>
android:layout_width="match_parent"
android:layout_height="match_parent"

<TextView
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:text="@{ viewModel.nombre }" />

</LinearLayout>
```

En la *activity/fragment*, indicarle al objeto *binding* cuál es la variable *viewModel* usada en el *layout* (la instancia del *ViewModel*). Y además, para que se pueda bindear con los *LiveData* que tiene el *ViewModel*, se le pasa el propietario del ciclo de vida (el controlador de la UI, es decir, la *activity/fragment*):

```kotlin
binding.viewModel = viewModel
binding.lifecycleOwner = this
```

#### ***@BindingAdapter***  
Cuando en el *xml* se coloca el atributo que indica el *BindingAdapter* (en el ejemplo, “visible”), se le indica que debe ejecutar la función escrita en el *BindingAdapter*. Se puede utilizar como una **función de extensión** para proveer un comportamiento *custom* cuando cambian los datos de un tipo determinado. La propia vista (***ProgressBar*** en el ejemplo) accede al valor de la variable del *ViewModel* (***progressVisibility*** en el ejemplo), y ese valor se lo pasa a la función del *BindingAdapter*.

###### En el archivo *.kt*:

```kotlin
@BindingAdapter("visible")
fun View.bindVisible (visible: Boolean?) {
    visibility = if (visible == true) View.VISIBLE else View.GONE
}
```

###### Y en el *xml*:

```xml
<ProgressBar
android:id="@+d/progress"
android:layout_width="wrap_content"
android:layout_height="wrap_content"
app:visible="@ { viewModel.progressVisibility }" />
```

Otro ejemplo útil sería usar un *BindingAdapter* para cargar una imagen en un ***ImageView*** desde una URL con ***Glide***, por ejemplo:

###### En el archivo *.kt*:

```kotlin
@BindingAdapter("url")
fun ImageView.loadUrl(url: String) {
    Glide.with(this).load(url).into(this)
}
```

###### Y en el *xml*:

```xml
<ImageView
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    app:url="@{ viewModel.url }" />
```



---



### Componentes de arquitectura: ***Lifecycle***, ***ViewModel*** y ***LiveData***

#### ***Lifecycle***  
Los componentes concientes del ciclo de vida (***lifecycle-aware***), **ajustan sus comportamientos** en base al ciclo de vida de *activities* y *fragments*. **Evitan poner acciones** de los componentes y/o librerías dependientes **en los controladores del ciclo de vida** (***``onResume()``***, ***``onPause()``***, ***``onStop()``***, etc.). Este aislamiento de esas acciones, ayuda a crear código liviano, conciso y organizado, lo que se traduce en **mayor facilidad de mantenimiento** de la *app*. Para esto, se utiliza el modelo de observación propuesto en *Android Jetpack* para componentes conscientes de ciclos de vida de la librería ***androix.lifecycle***. Básicamente, se debe:  
1. Implementar la escucha ***LifecycleOwner*** sobre el componente que tiene el ciclo de vida
2. Implementar ***LifecycleObserver*** sobre el componente que realizará acciones basado en el propietario del ciclo de vida
3. Vincular ambos objetos con ***``addObserver()``***

La **Clase** ***Lifecycle*** mantiene la información sobre el estado del ciclo de vida de un componente (como una *activity* o un *fragment*) y permite que otros objetos tengan este estado.
Sus **enumeraciones** ***Event*** y ***State*** permiten **rastrear los flujos posibles del ciclo de vida** como se ve la siguiente gráfica:

<img src="Lifecycle - Events & States.png" width="500">

La **Interfaz** ***LifecycleObserver*** representa al observador encargado de monitorear el estado del ciclo de vida de un componente a través de la anotación ***``@OnLifecycleEvent``***. El método ***``addObserver()``*** se usa para vincular el dueño del ciclo de vida al observador.

```kotlin
class MyObserver : LifecycleObserver {

    @OnLifecycleEvent(Lifecycle.Event.ON_RESUME)
    fun connectListener() {
        ...
    }

    @OnLifecycleEvent(Lifecycle.Event.ON_PAUSE)
    fun disconnectListener() {
        ...
    }
}

// myLifecycleOwner implementa la interfaz LifecycleOwner
myLifecycleOwner.getLifecycle().addObserver(MyObserver())
```

La **interfaz** ***LifecycleOwner*** indica que la clase tiene un ***Lifecycle***. Su único método, ***``getLifecycle()``***, permite a los observadores estar al tanto del ciclo de vida. Tanto *AppCompatActivity* como *Fragment* ya vienen con esta **interfaz implementada**, por lo que no hay que preocuparse por hacerlo. Por otro lado, si se intenta administrar el ciclo de vida de **todo el proceso** de una aplicación, se debe utilizar ***ProcessLifecycleOwner***. 
Un observador (en el ejemplo que sigue, el objeto de tipo ***MyLifecycleObserver***) puede implementar la interfaz ***LifecycleObserver*** y luego inicializarlo con el ***Lifecycle*** de la *activity* (que implementa ***LifecycleOwner*** por defecto) en el método ***``onCreate()``***. Esta acción permite que la clase ***MyLifecycleObserver*** sea autosuficiente, lo que significa que la lógica para reaccionar ante los cambios en el estado del ciclo de vida se declara en ***MyLifecycleObserver***, no en la *activity*. El hecho de que los componentes individuales almacenen su propia lógica permite que la lógica de las *activities* y los *fragments* sea más fácil de administrar.

```kotlin
class MyActivity : AppCompatActivity() {
    private lateinit var MyLifecycleObserver: MyLifecycleObserver

    override fun onCreate(...) {
        observer = MyLifecycleObserver(lifecyle)
    }
}
```

Para **implementar un** ***LifecycleOwner*** **personalizado**, se utiliza la clase ***LifecycleRegistry***, y se deben **desviar los eventos** a esa clase.

```kotlin
class MyActivity : Activity(), LifecycleOwner {

    private lateinit var lifecycleRegistry: LifecycleRegistry

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        lifecycleRegistry = LifecycleRegistry(this)
        lifecycleRegistry.markState(Lifecycle.State.CREATED)
    }

    public override fun onStart() {
        super.onStart()
        lifecycleRegistry.markState(Lifecycle.State.STARTED)
    }

    override fun getLifecycle(): Lifecycle {
        return lifecycleRegistry
    }
}
```

#### ***ViewModel***  
Se diseñó la clase *ViewModel* a fin de **almacenar y administrar datos relacionados con la UI** de manera **optimizada para los ciclos de vida** (se asegura de que sobrevivan a cambios de configuración de la *activity/fragment*, por ej.: rotación del dispositivo o realizar cambios en el idioma del dispositivo). Un *ViewModel* siempre **se crea en asociación con un ámbito** (***scope***), es decir, un *fragment* o una *activity*, y será retenido tanto como el ámbito esté vivo. La *activity*/*fragment* es capaz de observar los cambios en el *ViewModel*, el cual usualmente **expone esta información** vía ***LiveData*** (se observan desde la *activity*/*fragment*) y, dependiendo del caso, también con ***DataBinding*** (el *xml* accede directamente a los *LiveData*), y sólo puede ser accedido después de que la *activity* es adjuntada a la aplicación. La única responsabilidad del *ViewModel* es manejar los datos para la UI. **Nunca debería acceder** a la jerarquía de vistas **ni tener una referencia** de vuelta a la *activity*/*fragment*. Aunque hay una excepción a esta regla, ya que a veces se puede necesitar un ***Application context*** (en contraposición a un ***Activity context***), para usarlo con cosas como servicios del sistema. **Almacenar un contexto de aplicación en un** ***ViewModel*** **está bien**, ya que está sujeto al ciclo de vida de la aplicación. Es diferente al contexto de una *activity*, que está sujeto al ciclo de vida de la *activity*. De hecho, si se necesita un *Application context*, se puede extender ***AndroidViewModel***, que es simplemente **un** ***ViewModel*** **que incluye una referencia a la aplicación**.  
Como se dijo, la **responsabilidad de conseguir y almacenar los datos** recae en el *ViewModel*, no en la *Activity*. Lo más importante es que con el uso de *ViewModel* se sigue el principio de **separación de responsabilidades** (***separation of concerns***). Esta arquitectura permite separar el código de forma más inteligente, así la *activity* sólo será responsable de pintar los datos. De esta manera, el *ViewModel* es capaz de comunicarse con el ***Repository*** para obtener sus *LiveData* (existe una **mejor alternativa** usando ***Flow***: ver apartado *Repositories* en *Arquitecturas*), y a su vez los deja disponibles para ser observados por la vista.  
Hay tres pasos para configurar y usar un *ViewModel*:  
1.  Separar los datos del controlador de la UI (*activity* o *fragment*), creando una **clase que extienda a** ***ViewModel***. En general, se crea una clase *ViewModel* para cada pantalla de la aplicación.

```kotlin
class MyViewModel : ViewModel() {
    private val users: MutableLiveData<List<User>> by lazy {
        MutableLiveData().also {
            loadUsers()
        }
    }

    fun getUsers(): LiveData<List<User>> {
        return users
    }

    private fun loadUsers() {
        // Do an asynchronous operation to fetch users.
    }
}
```

2.  Configurar la **comunicación entre el** ***ViewModel*** **y el controlador de la UI, creando una instancia del** ***ViewModel***.  
3.  Usar el *ViewModel* en el controlador de la UI.

Los *ViewModel* **se instancian** a través del constructor de la **clase** ***ViewModelProvider***; o con una **función de extensión** específica para esto, parte de **Android KTX**: ***``by viewModels()``***, ***``by activityViewModels()``*** o ***``by navGraphViewModels(R.id.some_nav_graph)``***, siendo preferible crear el *ViewModel* como una propiedad ***val*** **a nivel de clase**. Estas funciones **retornan un** ***property delegate*** que internamente utiliza *ViewModelProvider* y define el ámbito (*scope*) del *ViewModel* al *Fragment/Activity* (como se dijo antes, el ciclo de vida del *ViewModel* estará vinculado a la *activity/fragment* que utiliza la función de extensión). De esta forma, el *ViewModel* sólo será instanciado si aún no existe en el mismo ámbito; **si ya existe, la librería va a retornar la misma instancia que ya estaba usando**. Cuando la *activity* (o *fragment*) que creó el *ViewModel* finaliza (o se desadjunta), se llama al **método** ***``onCleared()``*** de *ViewModel*.

Como la librería es la responsable de crear el *ViewModel*, **no se puede llamar a su constructor**, ya que la librería lo hace internamente; y, por defecto, **siempre llamará al constructor vacío**, haciendo imposible pasarle datos. Para empeorar las cosas, se producirá una excepción fatal en tiempo de ejecución si *ViewModel* no tiene un constructor vacío. Para solventar esto, ambas formas (*ViewModelProviders* y las funciones de extensión) permiten pasar un ***ViewModelProvider.Factory*** **por parámetro**, que será usado para construir la instancia del *ViewModel* en los casos en que se debe inyectar alguna dependencia. Un *factory* sería, por ejemplo, así:

```kotlin
class ViewModelFactory(private val repo: Repo): ViewModelProvider.Factory {
    override fun <T : ViewModel?> create(modelClass: Class<T>): T {
        return modelClass.getConstructor(Repo::class.java).newInstance(repo)
    }
}
```

***``by viewModels()``***: se usa cuando la *activity/fragment* que instancia el *ViewModel*, es la única *activity/fragment* que accede a los datos de dicho ViewModel.
***``by activityViewModels()``***: se usa en *fragments* que comparten datos y se comunican con otra *activity/fragment*. Es útil para compartir información entre vistas (ver en apartado 5, *Pasar datos entre destinos*).
***``by navGraphViewModels(R.id.some_nav_graph)``***: se usa en *fragments* en los que el *ViewModel* se beneficia al estar en el ámbito de un gráfico de navegación específico o un gráfico anidado. Es útil para manejar la lógica de negocios de características organizadas en un gráfico anidado, para garantizar que los datos no se filtren a otras partes de la aplicación y que el ciclo de vida se maneje correctamente.

```kotlin
val viewModel = ViewModelProvider(this).get(MyViewModel::class.java)
```

O, con la función de extensión de Kotlin:

```kotlin
// Get a reference to the ViewModel scoped to this Fragment
val viewModel by viewModels<MyViewModel>()

// Get a reference to the ViewModel scoped to its Activity
val viewModel by activityViewModels<MyViewModel>()
```

#### ***LiveData***  
Es una clase de **almacenamiento de datos** que permite que estos sean **observados**. Se **diferencia de un** ***Observable*** en que es **consciente del ciclo de vida** de la *activity* (o *fragment*) evitando, por ejemplo, mandarle datos cuando dicha *activity* (o *fragment*) no está en primer plano, ya que *LiveData* considera que un observador, que está representado por la clase ***Observer***, está en estado activo si su ciclo de vida está en ***STARTED*** o ***RESUMED***. *LiveData* **solo notifica a los observadores activos** sobre las actualizaciones. Los observadores inactivos registrados para ver objetos *LiveData* no reciben notificaciones sobre los cambios.  
Se siguen estos pasos para trabajar con objetos *LiveData*:  
1.  Crear una **instancia de** ***LiveData*** para contener un tipo de datos determinado. Por lo general, esto se hace **dentro de la clase** ***ViewModel***.  
2.  Crear un objeto ***Observer*** que defina el **método** ***``onChanged()``*** (en Kotlin, el método queda definido en el *lambda*, ya que es el **único método abstracto** de la interface *Observer*), el cual controla lo que sucede cuando cambian los datos retenidos del objeto *LiveData*. Por lo general, se crea un objeto *Observer* **en un controlador de UI**, como una *activity* o un *fragment*.  
3.  Conectar el objeto *Observer* al objeto *LiveData* con el **método** ***``observe()``***. Dicho método toma un objeto *LifecycleOwner*. Esto suscribe el objeto *Observer* al objeto *LiveData* para que se le notifiquen los cambios.  
*LiveData* es un *wrapper* que se puede usar con cualquier dato, incluidos los objetos que implementan *Collections*, como *List*. Por lo general, un objeto *LiveData* se almacena dentro de un objeto *ViewModel*, y se puede acceder a este a través de un *getter*.  

```kotlin
class NameViewModel : ViewModel() {
    // Create a LiveData with a String
    val currentName: MutableLiveData<String> by lazy {
        MutableLiveData<String>()
    }
    // Rest of the ViewModel...
}
```

En la mayoría de los casos, el método ``onCreate()`` de un componente de la aplicación es el lugar adecuado para comenzar a observar un objeto *LiveData*.  
```kotlin
class NameActivity : AppCompatActivity() {
    // Use the 'by viewModels()' Kotlin property delegate
    // from the activity-ktx artifact
    private val model: NameViewModel by viewModels()

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        // Other code to setup the activity...

        // Create the observer which updates the UI.
        val nameObserver = Observer<String> { newName ->
            // Update the UI, in this case, a TextView.
            nameTextView.text = newName
        }

        // Observe the LiveData, passing in this activity as the LifecycleOwner and the observer.
        model.currentName.observe(this, nameObserver)
    }
}
```

*LiveData* no tiene métodos disponibles públicamente para actualizar los datos almacenados. La clase ***MutableLiveData*** expone los **métodos** ***``setValue(T)``*** y ***``postValue(T)``*** de forma pública; se deben usar si se necesita editar el valor almacenado en un objeto *LiveData*. El comportamiento de estos métodos es bastante similar; sin embargo, *setValue* establecerá directamente **un nuevo valor** y solo se puede llamar **desde el hilo principal**, mientras que *postValue* crea **una nueva tarea** en el hilo principal para establecer el nuevo valor y se puede llamar **desde un hilo secundario**. Por lo general, ***MutableLiveData*** **se usa en** ***ViewModel*** **y, luego,** ***ViewModel*** **solo expone objetos** ***LiveData*** **inmutables a los observadores**.

```kotlin
private val _progressVisibility = MutableLiveData<Boolean>()
val progressVisibility: LiveData<Boolean> get() = _progressVisibility
```

Después de configurar la relación del observador, se puede actualizar el valor del objeto *LiveData*, como se muestra en el siguiente ejemplo, que activa todos los observadores cuando el usuario presiona un botón:

```kotlin
button.setOnClickListener {
    val anotherName = "John Doe"
    model.currentName.setValue(anotherName)
}
```

Eventualmente, se deberá cambiar un *LiveData* y propagar su nuevo valor a su observador. O tal vez se necesite crear una reacción en cadena entre dos objetos *LiveData*, haciendo que uno reaccione a los cambios en otro. Para lidiar con ambas situaciones, se puede usar la **clase** ***Transformations***.  
***``Transformations.map``*** proporciona una forma de **realizar manipulaciones de datos en el** ***LiveData*** **de origen y devolver el resultado** de esa transformación. Este método toma la fuente (***source***) *LiveData* y una **función** ***lambda*** (que manipula ese *LiveData* de origen) como parámetros. La función pasada a *``Transformation.map()``* **se ejecuta en el hilo principal** (*main thread*), por lo que **no se deben incluir tareas de larga duración**.

```kotlin
val userLiveData: LiveData<User> = UserLiveData()
val userName: LiveData<String> = Transformations.map(userLiveData) {
        user -> "${user.name} ${user.lastName}"
}
```

***``Transformations.switchMap``*** es bastante similar a *``Transformations.map``*, pero como resultado debe **devolver otro objeto** ***LiveData***.

```kotlin
private fun getUser(id: String): LiveData<User> {
    ...
}
val userId: LiveData<String> = ...
val user = Transformations.switchMap(userId) { id -> getUser(id) }
```

***MediatorLiveData*** es un tipo más avanzado de *LiveData*. Tiene capacidades muy similares a las de la clase *Transformations*: puede reaccionar a otros objetos *LiveData*, llamando a una función cuando los datos observados cambian. Sin embargo, tiene muchas ventajas en comparación con *Transformations*, ya que **no necesita ejecutarse en el hilo principal y puede observar múltiples** ***LiveDatas*** **a la vez**. Los observadores de los objetos *MediatorLiveData* se activan cada vez que cambia alguno de los objetos de origen de *LiveData*.  
Por ejemplo, si se tiene un objeto *LiveData* en la UI que se puede actualizar desde una base de datos local o una red, se puede agregar las siguientes fuentes al objeto *MediatorLiveData*:
•   Un objeto *LiveData* asociado con los datos almacenados en la base de datos.
•   Un objeto *LiveData* asociado con los datos a los que se accede desde la red.
La *activity* sólo necesita observar el objeto *MediatorLiveData* para recibir actualizaciones de ambas fuentes.



---



### Bases de datos: ***Room - Realm***

#### ***Room***  
Android soporta *SQLite* desde el comienzo. Sin embargo, para hacerlo funcionar, siempre fue necesario escribir mucho código *boilerplate*. Además, *SQLIte* no guardaba *POJO’s* (*plain-old Java objects* u objetos planos de Java), y no comprobaba consultas (*queries*) en tiempo de compilación. *Room* viene a solucionar estos problemas. Es una librería de mapeo *SQLite*, capaz de persistir *POJO’s* de Java, convertir consultas directamente a objetos, comprobar errores en tiempo de compilación y **producir observables ***LiveData*** de los resultados de las consultas**. *Room* es una librería ORM (*Object Relational Mapping* o mapeo objeto-relacional) con algunos agregados de Android.  
Para crear una base de datos con *Room*, se necesita una **Entidad** (***Entity***) para persistir, que puede ser cualquier *POJO* Java (en Kotlin, se utilizan las ***data classes*** para este propósito) y representa una tabla dentro de la base de datos; una ***interface Dao*** (*data acces object* u objeto de acceso a datos) para hacer consultas y operaciones de entrada/salida; y también **una clase abstracta ***Database*** que debe extender a** ***RoomDatabase***.  
*Room* genera todo el código necesario para actualizar el objeto *LiveData* cuando se actualiza una base de datos. El código generado **ejecuta la consulta de manera asíncrona** en un subproceso en segundo plano cuando es necesario. Este patrón es útil para mostrar los datos en una UI sincronizada con los datos almacenados en una base de datos.  
Para usar *Room* en una *app*, agregar las siguientes **dependencias** al archivo *build.gradle*:

```kotlin
def room_version = "2.2.5"

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

    @Query("SELECT * FROM user WHERE first_name LIKE :first AND " +
            "last_name LIKE :last LIMIT 1")
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
val db = Room.databaseBuilder(
    applicationContext,
    AppDatabase::class.java, "database-name"
).build()
```

En *Room 2.2* y versiones posteriores, se puede asegurar de que la UI de la *app* se mantenga actualizada mediante la funcionalidad ***Flow*** de Kotlin. Para que se actualice automáticamente la UI cuando cambien los datos subyacentes, se escriben métodos de consulta que muestren objetos *Flow*:

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

#### ***Realm***  
Es una base de datos móvil que se ejecuta directamente en teléfonos, *tablets* o dispositivos portátiles. Los datos se exponen **directamente como objetos** y se pueden c**onsultar mediante código**, lo que elimina la necesidad de usar ORM, que puede traer problemas de rendimiento y mantenimiento.



---



### Arquitecturas: MVP / MVVM + ***Repository***

#### ***MVP (Model View Presenter)***  
Es una derivación del patrón MVC (*Model View Controller*) y es un patrón arquitectónico de interfaz de usuario diseñado principalmente para facilitar las pruebas unitarias automatizadas. En MVP, el presentador asume la funcionalidad del “hombre medio”. Toda **la lógica de presentación se envía al presentador** y toda **la lógica de negocio al modelo**.  
Interacción entre componentes:  
- ***Model***: Es una interfaz que define la lógica de negocio y los datos que se mostrarán en la interfaz de usuario. Está separado de la vista.
- ***View***: Es una interfaz pasiva que muestra los datos que recibe del presentador y enruta los comandos (eventos) del usuario al presentador. Está separada del modelo.
- ***Presenter***: Es una interfaz que contiene toda la lógica de presentación de la vista. Envía comandos al modelo y notificaciones a la vista (*activity/fragment*) de los cambios ocurridos en el modelo, **haciendo uso de** ***interfaces***. El presentador es el mediador entre la vista y el modelo.

<img src="MVP.png" width="600">

Un ejemplo del patrón MVP sería algo así:  

###### En la *Activity*:

```kotlin
class MainActivityPresenter : AppCompatActivity(), MainPresenter.View {

    private lateinit var binding: ActivityMainBinding
    private val presenter = MainPresenter(this, lifecycleScope)

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        with(binding){
            button.setOnClickListener {
                presenter.onButtonClicked(mail.text.toString())
            }
        }
    }

    override fun setProgressVisible(visible: Boolean) {
        binding.progress.visibility = if (visible) View.VISIBLE else View.GONE
    }

    override fun setMessage(message: String) {
        binding.message.text = message
    }
}
```

###### En el *Presenter*:

```kotlin
// Recibe por constructor una vista de una interface View y un scope de corrutina
// para poder hacer la petición en segundo plano.
class MainPresenter(private val view: View, private val scope: CoroutineScope) {

    interface View {
        fun setProgressVisible(boolean: Boolean)
        fun setMessage(message: String)
    }

    // Se llama cuando se hace click en el botón.
    // Recibe el texto de mail por parámetro.
    fun onButtonClicked(mail: String) {
        scope.launch {
            view.setProgressVisible(true)
            view.setMessage(withContext(Dispatchers.IO) {
                Thread.sleep(2000)
                if (mail.isNotEmpty()) "Success" else "Failure"
            })
            view.setProgressVisible(false)
        }
    }
}
```

#### ***MVVM (Model View ViewModel)***  
Este patrón facilita la separación de la lógica de la interfaz gráfica de usuario y la lógica de negocio o modelo de datos de la aplicación. En MVVM, el *ViewModel* tiene la responsabilidad de convertir los objetos de datos del modelo en un formato que permita manejar y presentar fácilmente. En este sentido, **el ***ViewModel*** contiene toda la lógica de presentación de la vista**.  
Interacción entre componentes
- ***Model***: Es una interfaz que **define la lógica de negocio** y los datos que se mostrarán en la interfaz de usuario. Contiene todas las *data classes*, las clases de base de datos (*database*), API y repositorio. Al igual que en el patrón *Model View Controller (MVC)*, una de las estrategias de implementación que se recomienda para desacoplar completamente esta capa del *ViewModel*, es exponer los datos a través de observables, es decir el **patrón** ***Observer***. De esta forma, cualquier *ViewModel* que necesite utilizar el modelo puede subscribir un observador.  
- ***View***: Al igual que en los patrones *Model View Controller (MVC)* y *Model View Presenter (MVP)*, es una interfaz pasiva que representa el estado actual de la información que es visible para el usuario y que recibe la interacción del usuario con la vista (eventos), para luego enviarlos al *ViewModel* a través de ***DataBinding\****, que **se define para vincular la vista y el** ***ViewModel***.  
- ***ViewModel***: Es una interfaz que **contiene toda la lógica de prensentación** de la vista. Envía comandos al modelo y notificaciones a la vista (*activity/fragment*) de los cambios ocurridos en el modelo, **haciendo uso de objetos observables**. Por eso mismo, una de las estrategias de implementación de esta capa es desacoplarla de la vista, es decir, el *ViewModel* no debe ser consciente de la vista con la que interactúa (es decir, **no debe contener una instancia de la vista**).  
***\* Binder***: En el *stack* de soluciones de Microsoft (MVVM fue inventado por dos arquitectos de Microsoft), el *binder* es un lenguaje de marcado llamado XAML. El *binder* libera al desarrollador de escribir lógica *boilerplate* para sincronizar el modelo con la vista. Si se desea utilizar este patrón fuera del *stack* de Microsoft, la presencia de una tecnología de enlace de datos declarativo es imprescindible. **Sin un ***binder***, no llega a ser un MVVM real**. Android Jetpack incorpora entre sus componentes la **librería** ***DataBinding***, que permite a los desarrolladores vincular de manera declarativa los datos observables a los elementos de la interfaz de usuario. De esta forma se disminuye considerablemente mucho código *boilerplate* en la lógica de presentación, lo que hace que la interfaz de usuario sea más simple y fácil de mantener.  
**La principal diferencia** entre el *ViewModel* del MVVM y el *Presenter* del MVP, es que **el ***Presenter*** tiene una referencia a una vista**, mientras que **el ***ViewModel*** no**. En otras palabras, **MVP se basa en un paradigma imperativo** (la vista comunica eventos al presenter, el presenter solicita acciones a la vista) mientras que **MVVM se basa en la observación** (el *ViewModel* expone datos observables, la vista se suscribe y escucha dichos datos).

<img src="MVVM.png" width="500">

El mismo ejemplo, pasado de MVP a MVVM (sin usar *DataBinding*):  

###### En la *Activity*:  

```kotlin
class MainActivityVM : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding
    private lateinit var viewModel: MainViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        viewModel = ViewModelProvider(this)[MainViewModel::class.java]
        // Para suscribirse a los LiveData del ViewModel
        viewModel.progressVisibility.observe(this, Observer {
            binding.progress.visibility = if (it) View.VISIBLE else View.GONE
        })
        viewModel.message.observe(this, Observer {
            binding.message.text = it
        })

        with(binding){
            button.setOnClickListener {
                viewModel.onButtonClicked(mail.text.toString())
            }
        }
    }
}
```

###### En el *ViewModel*:  

```kotlin
// No recibe el scope por parámetro porque ViewModel
// tiene su propio scope (viewModelScope)
class MainViewModel : ViewModel() {

    // Encapsula las variables para que no puedan ser accedidas desde
    // la activity.
    private var _message = MutableLiveData<String>()
    val message: LiveData<String> get() = _message

    private var _progressVisibility = MutableLiveData<Boolean>()
    val progressVisibility: LiveData<Boolean> get() = _progressVisibility


    // Se llama cuando se hace click en el botón.
    // Recibe el texto de mail por parámetro.
    fun onButtonClicked(mail: String) {
        viewModelScope.launch {
            _progressVisibility.value = true
            _message.value = withContext(Dispatchers.IO) {
                Thread.sleep(2000)
                if (mail.isNotEmpty()) "Success" else "Failure"
            }
            _progressVisibility.value = false
        }
    }
}
```

En el caso de que se aplique *DataBinding* al mismo ejemplo con MVVM, quedaría así (el *ViewModel* no se modifica y la *activity* queda mucho más limpia):

###### En la *Activity*:  

```kotlin
class MainActivityVM : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding
    private lateinit var viewModel: MainViewModel

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        viewModel = ViewModelProvider(this)[MainViewModel::class.java]

        binding.viewModel = viewModel
        binding.lifecycleOwner = this
    }
}
```

###### En el *xml*:  

```xml
<data>
    <import type="android.view.View" />
    <variable
        name="viewModel"
        type="com.example.navajasuiza.MainViewModel" />
</data>

<Button
    android:id="@+id/button"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:text="@string/button"
    android:onClick="@{ () -> viewModel.onButtonClicked(mail.getText().toString()) }"/>

<TextView
    android:id="@+id/message"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:gravity="center"
    android:text="@{ viewModel.message }"/>

<ProgressBar
    android:id="@+id/progress"
    style="?android:attr/progressBarStyle"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:visibility="@{ viewModel.progressVisibility ? View.VISIBLE : View.GONE }"/>
```

#### ***Repository***:  
Este patrón arquitectónico se usa en conjunto con los patrones descriptos antes (MVP y MVVM), entre otros, y es responsable de abstraer o **aislar las fuentes de datos del resto de la aplicación**, sea una base de datos o una consulta a una servicio *web*. Para implementar un repositorio, se usa una clase Repositorio que implemente una interfaz previamente creada, para cumplir con el **principio de inversión de dependencias** (depender de abstracciones y no de implementaciones) y que el código quede más desacoplado y fácil de testear. Un módulo de repositorio maneja operaciones de datos y permite usar múltiples *backends*. En una aplicación típica, **el repositorio implementa la lógica para decidir si obtener datos de una red o usar los resultados cacheados en una base de datos local**.

<img src="Repository.png" width="600">

Hay casos en los que se usa *LiveData* para comunicarse entre un repositorio y un *ViewModel*. Usar *LiveData* en este caso parece correcto. Después de todo, está diseñado para almacenar y pasar datos. Pero no se recomienda. El propósito principal de *LiveData* es contener un conjunto de datos que se puedan observar, distinguiéndose además de otros observables en que tiene en cuenta el ciclo de vida. Sin embargo, algo clave a tener en cuenta, es que **los observadores de ***LiveData*** siempre se llaman en el hilo principal** (***main thread***). Cuando se usa *LiveData* en el repositorio para pasar los datos al *ViewModel*, el *ViewModel* debería actuar como un observador, por lo que se necesita invocar la ejecución en el hilo principal. Y no se recomienda que los *ViewModel*s utilicen operaciones en el hilo principal. A menudo también se usan transformaciones en los datos obtenidos de los servidores. Y al usar *LiveData*, hay que hacerlo en el hilo principal, lo cual tampoco se recomienda.  
Para solventar esta situación, se puede recurrir al uso de **Kotlin** ***Coroutines Flow*** (ver apartado *Manejo de Asíncronos*). Como *Flow* está diseñado explícitamente para manejar operaciones asincrónicas complejas de manera efectiva y emitir varias veces según se requiera, es una excelente alternativa a *LiveData*. *Flow* ofrece una funcionalidad similar: *builders*, *cold streams* y auxiliares útiles (por ejemplo, transformación de datos). Y a diferencia de *LiveData*, no están vinculados al ciclo de vida y brindan más control sobre el contexto de ejecución.



---



### ***Navigation component***





---



### UI: Interfaz de Usuario



---



### Manejo de asíncronos: ***Coroutines / Flow / Rx***



---



### ***Retrofit***



---



### ***SharedPreferences / EncryptedSharedPreferences***



---



### Inyección de dependencias: ***Koin*** y ***Dagger***



---



### Testing: ***Junit*** y ***Mockito***



---



### ***Players: ExoPlayer*** y ***JW Player***



---



### ***OAuth: Facebook, Twitter, Google+***



---



### ***Frameworks / SDK: Firebase, Fabric, Sentry, Segment, Facebook***



---



### ***Deep linking***



---



### ***Push notifications***



---



### ***Jetpack Compose***







---
---



### Referencias y Fuentes:

-
