# Apuntes de Android  

### Contenido general:  
#### [0. ***ViewBinding*** y ***DataBinding***](#viewbinding-y-databinding)
#### [1. Componentes de arquitectura: ***Lifecycle***, ***ViewModel*** y ***LiveData***](#componentes-de-arquitectura-lifecycle-viewmodel-y-livedata)
#### [2. Bases de datos: ***Room*** y ***Realm***](#bases-de-datos-room-y-realm)
#### [3. Arquitecturas: MVP y MVVM con ***Repository***](#arquitecturas-mvp-y-mvvm-con-repository)
#### [4. ***Navigation component***](#navigation-component)
#### [5. UI: Interfaz de Usuario](#ui-interfaz-de-usuario)
   - ##### [5.1 ***Styles*** y ***Themes***](#styles-y-themes)
   - ##### [5.2 ***Custom Views***](#custom-views)
   - ##### [5.3 ***RecyclerView***](#recyclerview)
   - ##### [5.4 ***Menus***](#menus)

#### [6. Manejo de asíncronos: ***Coroutines, Flow, Rx***](#manejo-de-asíncronos-coroutines-flow-rx)
#### [7. ***Retrofit***](#retrofit)
#### [8. ***SharedPreferences y EncryptedSharedPreferences***](#sharedpreferences-y-encryptedsharedpreferences)
#### [9. Inyección de dependencias: ***Koin*** y ***Dagger***](#inyección-de-dependencias-koin-y-dagger)
#### [10. Testing](#testing)
#### [11. ***Players: ExoPlayer*** y ***JW Player***](#players-exoplayer-y-jw-player)
#### [12. ***OAuth: Facebook, Twitter, Google+***](#oauth-facebook-twitter-google)
#### [13. ***Frameworks y SDK's: Firebase, Fabric, Sentry, Segment, Facebook***](#frameworks-y-sdks-firebase-fabric-sentry-segment-facebook)
#### [14. ***Deep linking***](#deep-linking)
#### [15. ***Push notifications***](#push-notifications)
#### [16. ***Jetpack Compose***](#jetpack-compose)
#### [Referencias - Fuentes](#referencias-y-fuentes)

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



### Bases de datos: ***Room y Realm***

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
Es una base de datos móvil que se ejecuta directamente en teléfonos, *tablets* o dispositivos portátiles. Los datos se exponen **directamente como objetos** y se pueden **consultar mediante código**, lo que elimina la necesidad de usar ORM, que puede traer problemas de rendimiento y mantenimiento.



---



### Arquitecturas: MVP y MVVM con ***Repository***

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
Para solventar esta situación, se puede recurrir al uso de **Kotlin** ***Coroutines Flow*** (ver apartado *Manejo de Asíncronos*).



---



### ***Navigation component***

El **componente** ***Navigation*** consta de tres partes clave que se describen a continuación:
- **Gráfico de navegación** (***Navigation graph***): Es un recurso XML que contiene **toda la información relacionada con la navegación** en una ubicación centralizada. Esto incluye todas las áreas de contenido individuales dentro de la *app*, llamadas **destinos** (***destinations***), así como las posibles rutas que un usuario puede tomar a través de la *app*, llamadas **acciones** (***actions***).
- ***NavHost***: Es un **contenedor vacío que muestra los destinos del gráfico de navegación**. El componente *Navigation* contiene una implementación *NavHost* predeterminada, ***NavHostFragment***, que muestra destinos de *fragments*.
- ***NavController***: Es un **objeto que administra la navegación de la** ***app*** dentro de un *NavHost*. *NavController* orquesta el intercambio de contenido de destino en el objeto *NavHost* a medida que los usuarios se mueven a través de la *app*.  
A su vez, como principios de navegación generales (no se aplican solo al usar el componente *Navigation*), cabe destacar lo siguiente: toda *app* debe tener un **destino de inicio fijo**; los botones de **Atrás** (***Back***) y **Arriba** (***Up***), funcionan de la misma manera dentro de la *app*, con la diferencia de que **el botón ***Up*** nunca sale de la aplicación**; en caso de utilizar **vínculos directos** (***deep linking***) para navegar a una pantalla de la *app*, se debe crear una **pila** (***back stack***) **artificial**, como si se hubiera llegado a esa pantalla de forma normal.  
Para usar el componente *Navigation* con Kotlin, se agregan las dependencias en el *build.gradle*:

```kotlin
implementation "androidx.navigation:navigation-fragment-ktx:$nav_version"
implementation "androidx.navigation:navigation-ui-ktx:$nav_version"
```

Para agrear un nuevo gráfico de navegación, se crea un nuevo ***resource file*** de tipo *Navigation*. Para agregar un *NavHostFragment*, se agrega en el *xml* de esta forma:

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <androidx.appcompat.widget.Toolbar
        .../>
    
    <!-- Se agrega un NavHostFragment -->
    <androidx.fragment.app.FragmentContainerView
        android:id="@+id/nav_host_fragment"
        android:name="androidx.navigation.fragment.NavHostFragment"
        android:layout_width="0dp"
        android:layout_height="0dp"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toBottomOf="parent"

        app:defaultNavHost="true"
        app:navGraph="@navigation/nav_graph" />

    <com.google.android.material.bottomnavigation.BottomNavigationView
        .../>

</androidx.constraintlayout.widget.ConstraintLayout>
```

Para establecer un destino como el **destino de inicio**, se hace clic derecho sobre el destino y luego ***Set as Start Destination***. En el editor de navegación, **las acciones se representan con flechas**, aunque para navegar realmente entre los destinos **hace falta escribir el código** correspondiente. Estas acciones, en la vista *xml* del gráfico, se representan con elementos ***``<action>``***. Como mínimo, una acción contiene su propio ID y el ID del destino (*fragment* o *activity*) al que se debería dirigir a un usuario.

```xml
<?xml version="1.0" encoding="utf-8"?>
<navigation xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    xmlns:android="http://schemas.android.com/apk/res/android"
    app:startDestination="@id/blankFragment">
    <fragment
        android:id="@+id/blankFragment"
        android:name="com.example.cashdog.cashdog.BlankFragment"
        android:label="fragment_blank"
        tools:layout="@layout/fragment_blank" >
        <action
            android:id="@+id/action_blankFragment_to_blankFragment2"
            app:destination="@id/blankFragment2" />
    </fragment>
    <fragment
        android:id="@+id/blankFragment2"
        android:name="com.example.cashdog.cashdog.BlankFragment2"
        android:label="fragment_blank_fragment2"
        tools:layout="@layout/fragment_blank_fragment2" />
</navigation>
```

Para recuperar un ***NavController*** (el objeto que administra la navegación de una app dentro de un elemento ***NavHost***), se puede usar uno de los siguientes métodos (en Kotlin):  
- ***``Fragment.findNavController()``***  
- ***``View.findNavController()``***  
- ***``Activity.findNavController(viewId: Int)``***

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var navController: NavController

    override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)
            setContentView(R.layout.activity_main)

            // Otras configuraciones
    
            navController = findNavController(R.id.nav_host_fragment)
    }
}
```

#### Navegar entre destinos
Para navegar entre destinos, Android recomienda utilizar el *plugin* de *Gradle* llamado ***Safe Args***, que permite navegar con **seguridad de tipo** y **pasar argumentos** entre los destinos. Para agregar *Safe Args* al proyecto, se incluye la siguiente *classpath* en el archivo *build.gradle* de nivel superior (a nivel de proyecto):

```kotlin
buildscript {
    repositories {
        google()
    }
    dependencies {
        def nav_version = "2.3.0"
        classpath "androidx.navigation:navigation-safe-args-gradle-plugin:$nav_version"
    }
}
```

También se agrega el complemento correspondiente:

```kotlin
apply plugin: "androidx.navigation.safeargs.kotlin"
```

Después de habilitar *Safe Args*, el complemento genera código que contiene clases y métodos para cada acción definida. En cada acción, *Safe Args* también genera una clase **para cada destino de origen**, que es el destino desde el que se origina la acción. El nombre de clase generada es una combinación del nombre de la clase del destino de origen y la palabra ***Directions***. Por ejemplo, si el destino se llama ***SpecifyAmountFragment***, la clase generada se llama ***SpecifyAmountFragmentDirections***. La clase generada contiene **un método estático para cada acción definida** en el destino de origen. Este método toma cualquier parámetro de acción definido como argumento y retorna un objeto ***NavDirections*** que se puede pasar al **método** ***``navigate()``***.

```kotlin
override fun onClick(view: View) {
    val action =
        SpecifyAmountFragmentDirections
            .actionSpecifyAmountFragmentToConfirmationFragment()
    view.findNavController().navigate(action)
}
```

Si bien *Safe Args* es lo recomendado, también se puede navegar entre destinos **a través de ID’s** o con ***DeepLinkRequest***. En el primer caso, ***``navigate(int)``*** toma el ID ya sea de una acción o directamente de un destino (se recomienda usar acciones siempre que sea posible, ya que se puede remplazar por operaciones con *SafeArgs*, brindan información adicional y también permiten animar las transiciones).

```kotlin
viewTransactionsButton.setOnClickListener { view ->
   view.findNavController().navigate(R.id.viewTransactionsAction)
}
```

En el segundo caso, se puede usar ***``navigate(NavDeepLinkRequest)``*** para navegar directamente a un destino de vínculo directo implícito. 

```kotlin
val request = NavDeepLinkRequest.Builder
    .fromUri("android-app://androidx.navigation.app/profile".toUri())
    .build()
findNavController().navigate(request)
```

Además de Uri, ***NavDeepLinkRequest*** también admite vínculos directos con acciones y tipos de MIME. Para agregar una acción a la solicitud, se usa ***``fromAction()``*** o ***``setAction()``***. Para agregar un tipo de MIME a una solicitud, se usa ***``fromMimeType()``*** o ***``setMimeType()``***. Hay un apartado específico sobre *DeepLinking* (apartado 15).

#### Pasar datos entre destinos
*Navigation* permite adjuntar datos a una operación de navegación si se definen los argumentos de un destino. Por ejemplo, el destino de un perfil de usuario podría tomar como argumento el ID de un usuario para determinar a quién mostrar el contenido. En general, se debe optar por pasar **sólo la cantidad mínima de datos entre destinos**. Por ejemplo, pasar una clave a fin de recuperar un objeto en lugar de pasar el objeto en sí, ya que el espacio total para los estados guardados en Android es limitado.  
Si se necesita pasar **grandes cantidades de datos**, es preferible utilizar un ***ViewModel*** **compartido**: dos *fragments* pueden compartir un *ViewModel* usando su ámbito de actividad (*activity scope*) para manejar la comunicación.

```kotlin
class SharedViewModel : ViewModel() {
    val selected = MutableLiveData<Item>()

    fun select(item: Item) {
        selected.value = item
    }
}

class MasterFragment : Fragment() {

    private lateinit var itemSelector: Selector

    // Use the 'by activityViewModels()' Kotlin property delegate
    // from the fragment-ktx artifact
    private val model: SharedViewModel by activityViewModels()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        itemSelector.setOnClickListener { item ->
            // Update the UI
        }
    }
}

class DetailFragment : Fragment() {

    // Use the 'by activityViewModels()' Kotlin property delegate
    // from the fragment-ktx artifact
    private val model: SharedViewModel by activityViewModels()

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        model.selected.observe(viewLifecycleOwner, Observer<Item> { item ->
            // Update the UI
        })
    }
}
```

Para pasar datos entre destinos, se define el argumento agregándolo al destino que lo recibe (desde el panel ***Attributes***).

```xml
 <fragment android:id="@+id/myFragment" >
     <argument
         android:name="myArg"
         app:argType="integer"
         android:defaultValue="0" />
 </fragment>
```

Todas las acciones que navegan al destino usan valores predeterminados y argumentos a nivel de destino. Si es necesario, **se puede sobreescribir el valor predeterminado de un argumento** (o definir uno si todavía no existe). **Para eso, se define un argumento a nivel de la acción**. Este argumento debe tener el mismo nombre y tipo que el declarado en el destino. El código XML que se muestra a continuación declara una acción con un argumento que sobreescribe el argumento a nivel de destino del ejemplo anterior:

```xml
<action android:id="@+id/startMyFragment"
    app:destination="@+id/myFragment">
    <argument
        android:name="myArg"
        app:argType="integer"
        android:defaultValue="1" />
</action>
```

Como ya se dijo, lo recomendado para navegar y pasar datos entre destinos es ***Safe Args***, ya que garantiza **seguridad de tipo** (***type-safety***). Así como crea una clase por cada destino donde se origina una acción (el nombre es el destino de origen más la palabra *Directions*), también se crea una clase para el destino de recepción. El nombre de esta clase es el nombre del destino, unido a la palabra ***Args***. Por ejemplo, si el *fragment* de destino se llama ***ConfirmationFragment***, la clase generada se llamará ***ConfirmationFragmentArgs***. Y también se crea una clase interna para cada acción que se usa a fin de pasar el argumento, cuyo nombre se basa en la acción. Por ejemplo, si la acción se llama ***confirmationAction***, la clase se llamará ***ConfirmationAction***. Si la acción contiene argumentos sin un ***defaultValue***, se debe usar la clase de acción asociada para configurar el valor de los argumentos.  
En el siguiente ejemplo, se muestra cómo utilizar estos métodos para configurar un argumento y pasarlo al **método** ***``navigate()``***:

```kotlin
override fun onClick(v: View) {
   val amountTv: EditText = view!!.findViewById(R.id.editTextAmount)
   val amount = amountTv.text.toString().toInt()
   val action = SpecifyAmountFragmentDirections.confirmationAction(amount)
   v.findNavController().navigate(action)
}
```

En el código del destino de recepción, se usa el **método** ***``getArguments()``*** a fin de recuperar el *bundle* y usar su contenido. Cuando se usan las **dependencias** ***-ktx***, con Kotlin también se puede usar el **delegado de propiedades** ***``by navArgs()``*** para acceder a los argumentos:

```kotlin
val args: ConfirmationFragmentArgs by navArgs()

override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
    val tv: TextView = view.findViewById(R.id.textViewAmount)
    val amount = args.amount
    tv.text = amount.toString()
}
```

En algunos casos, por ejemplo, **si no se usa** ***Gradle***, no se puede usar el complemento *Safe Args*. En esas situaciones, se pueden utilizar objetos ***Bundle*** para pasar datos de forma directa. Se crea un objeto *Bundle* y se lo pasa al destino usando ***``navigate()``***.

```kotlin
val bundle = bundleOf("amount" to amount)
view.findNavController().navigate(R.id.confirmationAction, bundle)
```

Y en el código del destino de recepción, se usa el **método** ***``getArguments()``*** para recuperar el *Bundle*:

```kotlin
val tv = view.findViewById<TextView>(R.id.textViewAmount)
tv.text = arguments?.getString("amount")
```

#### ***NavigationUI***
El componente *Navigation* incluye una **clase** ***NavigationUI***. Esta clase contiene métodos estáticos que administran la navegación con la barra superior de la *app* (***top bar***), el panel lateral de navegación (***navigation drawer***) y la navegación inferior (***bottom navigation***). *NavigationUI* también proporciona asistentes para **vincular destinos a componentes de UI controlados por el menú**: contiene un método asistente, ***``onNavDestinationSelected()``***, que toma un elemento ***MenuItem*** junto con el elemento ***NavController*** que aloja el destino asociado. Si el elemento ***``id``*** de *MenuItem* **coincide** con el elemento ***``id``*** del destino, *NavController* puede entonces navegar a ese destino.  
- ***Top bar***: *NavigationUI* contiene métodos que actualizan automáticamente el contenido de la barra superior a medida que los usuarios navegan por la *app*. Por ejemplo, **se pueden usar las etiquetas (***labels***) del destino del gráfico de navegación para mantener actualizado el título de la barra superior** de la *app*, usando el **método** ***``setupWithNavController``***. Cuando se usa *NavigationUI* con alguna de las implementaciones de la barra superior (***Toolbar***, ***CollapsingToolbarLayout*** o ***ActionBar***), la etiqueta que se adjunta a los destinos puede ser automáticamente populada a partir de los argumentos proporcionados al destino, usando la sintaxis ***``{argName}``*** en la etiqueta. *NavigationUI* usa un objeto ***AppBarConfiguration*** para **administrar el comportamiento del botón** ***Navigation*** en la esquina superior izquierda del área de visualización de la *app*. El comportamiento del botón *Navigation* cambia si el usuario se encuentra en un **destino de nivel superior** (la raíz, o el destino de nivel más alto, en un conjunto de destinos relacionados de manera jerárquica). Los destinos de nivel superior no muestran un botón ***Arriba*** en la barra superior de la *app*, ya que no existe un destino superior a este. De forma predeterminada, el destino de inicio de la *app* es el único destino de nivel superior.  
- ***Navigation drawer***: El ícono del panel lateral (***drawer icon***) se muestra en todos los destinos de nivel superior que usan un ***DrawerLayout***. Para agregar un panel lateral de navegación, primero se declara un *DrawerLayout* como vista raíz. Dentro del *DrawerLayout*, se agrega un diseño para el contenido principal de la UI y otra vista que tenga el contenido del panel lateral de navegación (***NavigationView***). Luego, se conecta el *DrawerLayout* al gráfico de navegación pasándoselo a ***AppBarConfiguration***. Y por último, en el método *``onCreate()``* de la *activity* se llama a ***``setupWithNavController()``***.  
- ***Bottom Navigation***: *NavigationUI* también puede controlar la navegación inferior. Cuando un usuario selecciona un elemento de menú, *NavController* llama a ***``onNavDestinationSelected()``*** y actualiza automáticamente el elemento seleccionado en la barra de navegación inferior. Para crear una barra de navegación inferior, primero se define en el XML de la *activity* principal; luego, se llama a ***``setupWithNavController()``*** desde el método *``onCreate()``* de la *activity* principal.  
La interacción con *NavController* es el método principal para navegar entre destinos. El *NavController* es responsable de reemplazar el contenido del *NavHost* con el destino nuevo. En muchos casos, los elementos de la UI (por ejemplo, una barra superior u otros controles de navegación persistentes como *BottomNavigationBar*) permanecen fuera del *NavHost* y se deben actualizar a medida que se navega entre destinos. *NavController* ofrece la **interface** ***OnDestinationChangedListener*** que se llama cuando **el destino actual o los argumentos de ***NavController*** cambian**. Es posible registrar un nuevo objeto de escucha mediante el **método** ***``addOnDestinationChangedListener()``***.  
*NavigationUI* usa *OnDestinationChangedListener* para hacer que estos componentes comunes de la UI reconozcan la navegación. Sin embargo, hay que tener en cuenta que **también se puede usar el elemento ***OnDestinationChangedListener*** por sí solo, con el fin de que cualquier UI personalizada o lógica de negocios esté al tanto de los eventos de navegación**. Por ejemplo, tal vez se tengan elementos de UI comunes que se quieran mostrar en algunas áreas de la *app* y ocultar en otras. Con un *OnDestinationChangedListener* propio, se pueden ocultar o mostrar selectivamente estos elementos de la UI en función del destino objetivo, como se muestra en el siguiente ejemplo:

```kotlin
navController.addOnDestinationChangedListener { _, destination, _ ->
   if(destination.id == R.id.full_screen_destination) {
       toolbar.visibility = View.GONE
       bottomNavigationView.visibility = View.GONE
   } else {
       toolbar.visibility = View.VISIBLE
       bottomNavigationView.visibility = View.VISIBLE
   }
}
```

#### ***Nested graphs***
Por lo general, los flujos de acceso, los asistentes (*wizards*) y otros subflujos de la *app* se representan mejor como **gráficos de navegación anidados** (***nested graphs***). Si se anidan flujos de subnavegación autónomos de esta manera, el flujo principal de la UI de la *app* será más fácil de comprender y administrar. Además, los gráficos anidados son reutilizables. También proporcionan un nivel de encapsulación, es decir, que los destinos fuera del gráfico anidado no tienen acceso directo a ninguno de los destinos dentro del gráfico anidado. En su lugar, deberían tener un elemento *``navigate()``* que los dirija al propio gráfico anidado, donde la lógica interna puede cambiar sin afectar el resto del gráfico. Esto es útil, por ejemplo, para verificar si hay un usuario registrado. Si el usuario no está registrado, se lo puede dirigir a la pantalla de registro dentro del gráfico anidado. Esto es lo que se conoce como **navegación condicional**.  
Para agrupar destinos en un gráfico anidado, se mantiene presionada la **tecla** ***Shift*** y se hace clic en los destinos que se desean incluir en el gráfico anidado. Luego, se hace clic derecho para abrir el menú contextual y se selecciona ***Move to Nested Graph > New Graph***.  
Además, dentro de un gráfico de navegación se puede hacer referencia a otros gráficos mediante ***``include``***. Si bien esto es funcionalmente lo mismo que usar un gráfico anidado, *``include``* permite usar gráficos de otros módulos del proyecto o de proyectos de biblioteca.

#### Acciones globales
Se puede usar una **acción general o global** (***global action***) para crear una acción común que varios destinos puedan utilizar. Por ejemplo, quizás se desee agregar botones en distintos destinos para navegar a la misma pantalla principal de la *app*. En el Editor de *Navigation*, una acción general se representa mediante **una flecha pequeña que apunta al destino asociado**. A fin de usar una acción general en el código, se pasa el ID de recurso de la acción general al **método** ***``navigate()``*** para cada elemento de la UI, como se muestra en el siguiente ejemplo:

```kotlin
viewTransactionButton.setOnClickListener { view ->
    view.findNavController().navigate(R.id.action_global_mainFragment)
}
```



---



### UI: Interfaz de Usuario

#### ***Styles y Themes***
La principal **diferencia entre estilos** (***styles***) y **temas** (***themes***), es que **un tema se aplica a toda una jerarquía de vistas, una ***activity*** o una** ***app***, mientras que **un estilo sólo afecta a la vista en la que se aplica**. En otras palabras, **un tema es un estilo que se propaga de padres a hijos**. Los temas contienen atributos o configuraciones que aplican a todos los elementos de la UI. Mientras que los temas tienen unos **atributos genéricos**, cada vista puede tener una serie de estilos **específicos** que hagan que esa vista se muestre de una forma u otra. Por ejemplo, el *style* por defecto de un *TextView*, es ***Widget.AppCompat.TextView***.  
Además, en caso de definir un nuevo estilo, sólo el elemento al que se le agrega el **atributo** ***``style``*** recibe los atributos del estilo definido; cualquier vista secundaria no aplica los estilos. Si se desea que las vistas secundarias hereden estilos, se aplica el estilo con el **atributo** ***``android:theme``***.  
Para crear un **tema** ***custom***, éste **debe heredar de un tema de** ***AppCompat***. Además, se agrega en el *Manifest* a la *activity* que corresponda.

###### En ***Styles.xml***:

```xml
<resources>
    <!-- Base application theme. -->
    <style name="AppTheme" parent="Theme.AppCompat.Light.DarkActionBar">
        <!-- Customize your theme here. -->
        <item name="colorPrimary">@color/colorPrimary</item>
        <item name="colorPrimaryDark">@color/colorPrimaryDark</item>
        <item name="colorAccent">@color/colorAccent</item>
    </style>

    <style name="AppTheme.MyActivityTheme" >
        <item name="colorPrimary">@color/colorAccent</item>
    </style>
</resources>
```

###### En ***AndroidManifest.xml***:

```xml
<application
    android:allowBackup="true"
    android:icon="@mipmap/ic_launcher"
    android:label="@string/app_name"
    android:roundIcon="@mipmap/ic_launcher_round"
    android:supportsRtl="true"
    android:theme="@style/AppTheme">
    <activity
        android:name=".MainActivity"
        android:theme="@style/AppTheme.MyActivityTheme">
        <intent-filter>
            <action android:name="android.intent.action.MAIN" />

            <category android:name="android.intent.category.LAUNCHER" />
        </intent-filter>
    </activity>
</application>
```

#### ***Custom Views***
Se puede crear una clase para crear vistas personalizadas y luego utilizarla en el *xml*. Por ejemplo, si se quiere crear un *ImageView* que contenga imágenes para *covers* de películas con un *ratio* personalizado, se puede crear una clase que herede de *ImageView* (la variante de *AppCompat*: *AppCompatImageView*) y agregar los constructores usando ***``@JvmOverloads``***. Después, se define el *aspect ratio* a utilizar y se sobreescribe la **función** ***``onMeasure()``*** para que la vista se adecue al tamaño que le corresponde en función de la vista padre, para luego modificarle la altura respecto al *aspect ratio* y el ancho, y también el ancho respecto al *aspect ratio* y el alto. Finalmente, se le indica a la clase padre que se han modificado las medidas llamando al **método** ***``setMeasureDimension()``***. Y en el *xml*, se agrega la vista personalizada. El ejemplo quedaría así:  

###### En la clase ***custom***:

```kotlin
class ApectRatioImageView @JvmOverloads constructor(
    context: Context, attrs: AttributeSet? = null, defStyleAttr: Int = 0
) : AppCompatImageView(context, attrs, defStyleAttr) {

    // Se deja pública para que se pueda modificar desde afuera
    var ratio = 1f

    override fun onMeasure(widthMeasureSpec: Int, heightMeasureSpec: Int) {
        super.onMeasure(widthMeasureSpec, heightMeasureSpec)

        // Obtener las medidas de la vista
        var width = measuredWidth
        var heigth = measuredHeight

        // Si la vista aún no se ha medido, que retorne
        if (width == 0 && heigth == 0) return

        // Si alguna de las medidas no es 0, comprobar cuál es, para calcular la que falta
        if (width > 0) {
            heigth = (width * ratio).toInt()
        } else if (heigth > 0) {
            width = (heigth / ratio).toInt()
        }

        setMeasuredDimension(width, heigth)
    }
}
```

###### En el ***xml***:

```xml
<com.example.navajasuiza.ApectRatioImageView
    android:id="@+id/cover"
    android:layout_height="200dp"
    android:layout_width="wrap_content">
</com.example.navajasuiza.ApectRatioImageView>
```

Si se quiere modificar el *ratio*, se puede hacer desde el código de esta manera:

```kotlin
binding.cover.ratio = 1.5f
```

Sin embargo, es recomendable añadir ese atributo directamente en el *xml*. Se crea un nuevo ***resource file*** dentro de ***values.xml*** (al que habitualmente se lo llama ***attrs.xml***), en el cual se pueden añadir atributos que definan el estilo de la vista *custom*. Para eso, se usa la etiqueta ***``<declare-styleable``***, se elige el nombre de la imagen, y dentro del bloque se especifican los atributos deseados (cuando se crea uno nuevo, se agrega el formato):

```xml
<resources>
    <declare-styleable name="ApectRatioImageView">
        <attr name="ratio" format="float" />
    </declare-styleable>
</resources>
```

Una vez hecho esto, se puede agregar el atributo en el *xml*:

```xml
app:ratio="1.5"
```

Y además, en la clase *custom* dentro de un bloque *``init``*, se recupera el componente de la siguiente manera:

```kotlin
init {
    // Para acceder a los atributos desde el código, se deben obtener los atributos mediante el método obtainStyledAttributes()
    // Se pasan por parámetro los atributos que vienen por constructor y
    // el styleable custom creado antes.
    val a = context.obtainStyledAttributes(attrs, R.styleable.ApectRatioImageView)
    // Recuperar el atributo ratio
    ratio = a.getFloat(R.styleable.ApectRatioImageView_ratio, 1f)
    // Se recicla la variable
    a.recycle()
}
```

También es importante señalar que los atributos pueden tener varios tipos (***format***), y entre ellos cabe destacar a los ***flag attributes*** y a los ***enum attributes***. Los *flags* se usan en los casos en que se quiere **combinar valores múltiples para un mismo atributo** (como puede ser, por ejemplo, aplicar *Bold* e *Italic* a un *custom TextView* en la forma ***``android:textStyle="bold|italic"``***); y hacen uso de **operaciones bit-a-bit** para determinar cómo combinar esos valores (por esta razón, los valores suelen ser **1, 2, 4, 8**, etc.). Por otro lado, los *enum* también pueden definir más de un valor, pero **el atributo sólo puede hacer uso de uno solo**.

###### Por ejemplo, en ***res/values/attr.xml***:

```xml
<!-- declare myenum attribute -->
<attr name="myenum">
    <enum name="zero" value="0" />
    <enum name="one" value="1" />
    <enum name="two" value="2" />
    <enum name="three" value="3" />
</attr>

<!-- declare myflags attribute -->
<attr name="myflags">
    <flag name="one" value="1" />
    <flag name="two" value="2" />
    <flag name="four" value="4" />
    <flag name="eight" value="8" />
</attr>

<!-- declare our custom widget to be styleable by these attributes -->
<declare-styleable name="com.example.MyWidget">
    <attr name="myenum" />
    <attr name="myflags" />
</declare-styleable>
```

###### Y en ***res/layout/mylayout.xml*** ahora se puede hacer:

```xml
<com.example.MyWidget
    myenum="two"
    myflags="one|two"
    ... />
```

#### ***RecyclerView***:
Este *widget* es una versión más avanzada y flexible del *ListView*. Varios componentes diferentes trabajan juntos para mostrar los datos: un **administrador de diseño** (***layout manager***), objetos **contenedores de vistas** (***view holders***) y un **adaptador** (***adapter***). El contenedor general de la UI es un objeto *RecyclerView* que se agrega al diseño. El objeto *RecyclerView* usa un *layout manager* para posicionar los elementos individuales en la pantalla y determinar cuándo volver a usar las vistas de elementos que ya no ve el usuario. Para volver a usar (reciclar) una vista, un *layout manager* puede solicitar al adaptador que reemplace el contenido de la vista por un elemento diferente del conjunto de datos. Android incluye tres administradores de diseño estándar para utilizar (***LinearLayoutManager***, ***GridLayoutManager*** o ***StaggeredGridLayoutManager***), aunque también se puede implementar uno propio extendiendo la clase abstracta ***RecyclerView.LayoutManager***.  
Las vistas incluidas en la lista, que **representan los elementos en un conjunto de datos**, están representadas por objetos *view holder*. Esos objetos son instancias de una clase que se define **extendiendo** ***RecyclerView.ViewHolder***. Cada objeto *view holder* es responsable de mostrar un elemento individual con una vista. Por ejemplo, si la lista muestra una colección de música, cada objeto *view holder* representa un álbum individual. El objeto *RecyclerView* crea solamente la cantidad de objetos *view holder* que sean necesarios para mostrar la parte en pantalla del contenido dinámico, más algunos adicionales. A medida que el usuario se desplaza por la lista, el objeto *RecyclerView* **toma las vistas fuera de pantalla y vuelve a vincularlas con los datos** que se desplazan en la pantalla.  
Un adaptador o *adapter*, que se crea **extendiendo** ***RecyclerView.Adapter***, administra los objetos *view holder*, y consta de tres métodos: ***``onCreateViewHolder()``***, ***``onBindViewHolder()``*** y ***``getItemCount()``***. Primero, el *layout manager* llama al método *``onCreateViewHolder()``* para que construya un objeto *RecyclerView.ViewHolder* y configure la vista que usa para mostrar su contenido. Luego, el *layout manager* vincula el *view holder* con sus datos. Para hacerlo, llama al método *``onBindViewHolder()``* del adaptador y pasa la posición del *view holder* en el *RecyclerView*. Es necesario que el método *``onBindViewHolder()``* busque los datos correspondientes y los use para completar el diseño del *view holder*. En resumen, **el adaptador crea ***view holders***, según sea necesario, y los vincula con sus datos**, asignando el *view holder* a una posición y llamando al método *``onBindViewHolder()``*, que usa la posición del *view holder* para determinar cuál debería ser el contenido, en función de su posición en la lista. El método ***``getItemCount()``*** retorna el tamaño del conjunto de datos.

```kotlin
class MyAdapter(private val myDataset: Array<String>) :
    RecyclerView.Adapter<MyAdapter.MyViewHolder>() {

    // Provide a reference to the views for each data item
    // Complex data items may need more than one view per item, and
    // you provide access to all the views for a data item in a view holder.
    // Each data item is just a string in this case that is shown in a TextView.
    class MyViewHolder(val textView: TextView) : RecyclerView.ViewHolder(textView)

    // Create new views (invoked by the layout manager)
    override fun onCreateViewHolder(parent: ViewGroup,
                                    viewType: Int): MyAdapter.MyViewHolder {
        // create a new view
        val textView = LayoutInflater.from(parent.context)
            .inflate(R.layout.my_text_view, parent, false) as TextView
        // set the view's size, margins, paddings and layout parameters
        ...
        return MyViewHolder(textView)
    }

    // Replace the contents of a view (invoked by the layout manager)
    override fun onBindViewHolder(holder: MyViewHolder, position: Int) {
        // - get element from your dataset at this position
        // - replace the contents of the view with that element
        holder.textView.text = myDataset[position]
    }

    // Return the size of your dataset (invoked by the layout manager)
    override fun getItemCount() = myDataset.size
}
```

También puede suceder que haya cambios en el conjunto de datos. Hay dos tipos diferentes de eventos de cambio en los datos: cambios en los elementos o ***item changes*** (cuando se actualizan los datos de un solo elemento, pero no se producen cambios de posición) y cambios estructurales o ***structural changes*** (cuando se insertan, eliminan o mueven elementos dentro del conjunto de datos). Para la primera clase de eventos se suelen utilizar métodos de notificación en el objeto *RecyclerView.Adapter*, como por ejemplo ***``notifyItemChanged()``***. El *layout manager* vincula nuevamente cualquier *view holder* afectado, lo que permite la actualización de sus datos. Por otro lado, para los cambios estructurales suele usarse ***``notifyDataSetChanged()``***, aunque este método debería usarse como último recurso ya que este evento no especifica qué ha cambiado en el conjunto de datos, lo que obliga a los observadores a asumir que es posible que todos los elementos y la estructura existente ya no sean válidos, en cuyo caso los *layout managers* se verán obligados a volver a vincular y retransmitir por completo todas las vistas visibles.  
Sin embargo, también es posible utilizar otras soluciones que provee *RecyclerView*, como ser *DiffUtil*, *SortedList* o *Paging Library*:  
- ***DiffUtil***: Si el *RecyclerView* muestra una lista que se recupera desde cero para cada actualización (por ejemplo, de la red o de una base de datos), *DiffUtil* puede **calcular la diferencia entre las versiones de la lista**. *DiffUtil* toma ambas listas como entrada y calcula la diferencia, que se puede pasar a *RecyclerView* para activar animaciones y actualizaciones mínimas para mantener el rendimiento de la interfaz de usuario, y las animaciones significativas. Este enfoque requiere que cada lista se represente en la memoria con contenido inmutable y se basa en recibir actualizaciones como nuevas instancias de listas. También es ideal si la capa de interfaz de usuario no implementa un ordenamiento, solo presenta los datos en el orden en que se proporcionan. Hay tres API’s principales para aplicarlo (de mayor a menor nivel de abstracción): ***ListAdapter***, ***AsyncListDiffer*** y ***DiffUtil***. Cada enfoque permite especificar cómo se deben calcular las diferencias en función de los datos.  
- ***SortedList***: Puede mantener los elementos en orden y también **notificar los cambios en la lista** de modo que pueda vincularse a un *RecyclerView.Adapter*. Mantiene los elementos ordenados mediante el método *callback* ***``compare(Object, Object)``*** y utiliza búsqueda binaria para recuperar elementos. Si los criterios de ordenamiento de los elementos pueden cambiar, hay que asegurarse de llamar a los métodos apropiados mientras se editan para evitar inconsistencias de datos. Se puede controlar el orden de los elementos y cambiar las notificaciones a través del **parámetro** ***Callback***. *SortedList* funciona si solo se necesita manejar eventos de inserción y eliminación, y tiene la ventaja de que **solo se necesita tener una única copia de la lista en memoria**.  
- ***Paging Library***: La biblioteca de paginación amplía el enfoque basado en diferencias para admitir, adicionalmente, la carga paginada (cargar y mostrar pequeños fragmentos de datos a la vez). Esta carga de datos parciales a pedido reduce el uso del ancho de banda de la red y los recursos del sistema. Proporciona la **clase** ***androidx.paging.PagedList*** que funciona como una lista de carga automática, proporcionada una fuente de datos como una base de datos o una API de red paginada.

#### ***Menus***:
Para todos los tipos de menús, Android proporciona un formato XML estándar que permite definir los elementos de menú. En lugar de incorporar un menú en el código de la *activity*, se debe definir un menú y todos los elementos en un **recurso de menú** (***menu resource***). Luego, se puede inflar el recurso de menú (cargarlo como un objeto ***Menu***) en la *activity* o el *fragment*. Para definir el menú, se crea un archivo XML dentro del directorio ***res/menu/*** del proyecto y se desarrolla el menú con los siguientes elementos: ***``<menu>``*** (define un ***Menu***, que es un contenedor para items de menú. Un elemento *``<menu>``* debe ser el nodo raíz del archivo y puede tener uno o más elementos *``<item>``* y *``<group>``*); ***``<item>``*** (crea un ***MenuItem***, que representa un único item en un menú. Este elemento puede contener un elemento *``<menu>``* anidado para crear un submenú) y ***``<group>``*** (contenedor **opcional e invisible** para elementos *``<item>``*, que permite categorizar los *items* del menú para que compartan propiedades como el estado activo/inactivo y la visibilidad).  
Existen tres tipos fundamentales de presentaciones de menús o acciones en todas las versiones de Android:  
- **Menú de opciones y** ***AppBar***: El menú de opciones es la colección principal de elementos de menú de una *activity*. Es donde se deben colocar las **acciones que tienen un impacto global en la** ***app***, como "Buscar", "Redactar correo electrónico" y "Configuración".

```kotlin
override fun onCreateOptionsMenu(menu: Menu): Boolean {
    val inflater: MenuInflater = menuInflater
    inflater.inflate(R.menu.game_menu, menu)
    return true
}

override fun onOptionsItemSelected(item: MenuItem): Boolean {
    // Handle item selection
    return when (item.itemId) {
        R.id.new_game -> {
            newGame()
            true
        }
        R.id.help -> {
            showHelp()
            true
        }
        else -> super.onOptionsItemSelected(item)
    }
}
```

- **Menú contextual (***context menu***) y modo de acción contextual (***contextual action bar***)**: Un menú contextual es un **menú flotante** que aparece cuando el usuario hace un **clic largo** en un elemento, proporciona acciones que afectan el contenido seleccionado o el marco contextual y se puede realizar una acción contextual **en un elemento por vez**. Por otro lado, el modo de acción contextual es una implementación (por parte del sistema) de ***ActionMode***, que muestra los elementos de acción que afectan al contenido seleccionado en una **barra en la parte superior** de la pantalla y **permite al usuario seleccionar varios elementos a la vez**.

```kotlin
private val actionModeCallback = object : ActionMode.Callback {
    // Called when the action mode is created; startActionMode() was called
    override fun onCreateActionMode(mode: ActionMode, menu: Menu): Boolean {
        // Inflate a menu resource providing context menu items
        val inflater: MenuInflater = mode.menuInflater
        inflater.inflate(R.menu.context_menu, menu)
        return true
    }

    // Called each time the action mode is shown. Always called after onCreateActionMode, but
    // may be called multiple times if the mode is invalidated.
    override fun onPrepareActionMode(mode: ActionMode, menu: Menu): Boolean {
        return false // Return false if nothing is done
    }

    // Called when the user selects a contextual menu item
    override fun onActionItemClicked(mode: ActionMode, item: MenuItem): Boolean {
        return when (item.itemId) {
            R.id.menu_share -> {
                shareCurrentItem()
                mode.finish() // Action picked, so close the CAB
                true
            }
            else -> false
        }
    }

    // Called when the user exits the action mode
    override fun onDestroyActionMode(mode: ActionMode) {
        actionMode = null
    }
}

someView.setOnLongClickListener { view ->
    // Called when the user long-clicks on someView
    when (actionMode) {
        null -> {
            // Start the CAB using the ActionMode.Callback defined above
            actionMode = activity?.startActionMode(actionModeCallback)
            view.isSelected = true
            true
        }
        else -> false
    }
}
```

- **Menú emergente** (***popup menu***): Un menú emergente muestra una lista de elementos en una lista vertical que está anclada a la vista que invocó el menú. Es adecuado para proporcionar una ampliación de acciones relacionadas con contenido específico o para proporcionar opciones en una segunda parte de un comando. Las acciones en un menú emergente **no deben afectar directamente al contenido correspondiente**, ya que para eso están las acciones contextuales. En cambio, el menú emergente es para **acciones extendidas** relacionadas con partes del contenido de la *activity*. Además, puede ser una interfaz útil para activar y desactivar opciones, usar una casilla de verificación (*checkbox*) para opciones independientes o botones de selección (*radio buttons*) para grupos de opciones mutuamente exclusivas. Para eso se agrega un **atributo** ***``android:checkableBehavior``*** que acepta los valores ***single*** (solo se puede activar un elemento –*radio buttons*–), ***all*** (todos los elementos se pueden activar –*checkboxes*–) o ***none*** (no se puede activar ningún elemento).

```kotlin
fun showPopup(v: View) {
    val popup = PopupMenu(this, v)
    val inflater: MenuInflater = popup.menuInflater
    inflater.inflate(R.menu.actions, popup.menu)
    popup.show()
}


fun showMenu(v: View) {
    PopupMenu(this, v).apply {
        // MainActivity implements OnMenuItemClickListener
        setOnMenuItemClickListener(this@MainActivity)
        inflate(R.menu.actions)
        show()
    }
}

override fun onMenuItemClick(item: MenuItem): Boolean {
    return when (item.itemId) {
        R.id.archive -> {
            archive(item)
            true
        }
        R.id.delete -> {
            delete(item)
            true
        }
        else -> false
    }
}
```

###### Para usar elementos de menú que se pueden activar, en el ***xml***:
```xml
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <group android:checkableBehavior="single">
        <item android:id="@+id/red"
              android:title="@string/red" />
        <item android:id="@+id/blue"
              android:title="@string/blue" />
    </group>
</menu>
```

###### Y para comprobar y establecer el estado de activación:
```kotlin
override fun onOptionsItemSelected(item: MenuItem): Boolean {
    return when (item.itemId) {
        R.id.vibrate, R.id.dont_vibrate -> {
            item.isChecked = !item.isChecked
            true
        }
        else -> super.onOptionsItemSelected(item)
    }
}
```



---



### Manejo de asíncronos: ***Coroutines, Flow, Rx***

Para más información sobre corrutinas, ver el apartado [*Corrutinas (Coroutines) en Kotlin*](https://github.com/Ulises-Jota/Apuntes-y-Navaja-Suiza/blob/master/Apuntes-Kotlin.md#4-corrutinas-coroutines-en-kotlin) en *Apuntes-Kotlin.md*
Las corrutinas están diseñadas para ejecutar operaciones asíncronas complejas de forma limpia y secuencialmente. Las funciones de suspensión de las corrutinas **retornan asincrónicamente un solo valor**, pero ¿cómo se pueden retornar múltiples valores computados asincrónicamente? Acá es donde Kotlin ***Flows*** reluce.  
*Flow* está diseñado explícitamente para manejar operaciones asíncronas complejas de forma efectiva y emitir varias veces según se requiera. A diferencia de los canales (*channels*), los flows son *cold streams*, al igual que las secuencias (*sequences*) de Kotlin. El código dentro del constructor de un *flow* (*flow builder*), no se ejecuta hasta que el *flow* es recolectado.  
Entonces, estas habilidades hacen de *Flow* una excelente alternativa a *LiveData* en los repositorios. *Flow* ofrece una funcionalidad similar: *builders*, *cold streams* y auxiliares útiles (por ejemplo, transformación de datos). Y a diferencia de *LiveData*, **no están vinculados al ciclo de vida y brindan más control sobre el contexto de ejecución**.



---



### ***Retrofit***



---



### ***SharedPreferences y EncryptedSharedPreferences***



---



### Inyección de dependencias: ***Koin*** y ***Dagger***

#### Inyección de dependencias
La inyección de dependencias nació para reducir el acoplamiento entre los componentes (las clases) de un sistema; básicamente, es un patrón de diseño en el que **se suministran objetos a una clase en lugar de ser la propia clase la que crea dichos objetos**. Este patrón facilita mucho intentar cumplir uno de los principios SOLID, el de **inversión de dependencias**. Según este principio, **las clases deben depender de abstracciones y no de detalles de implementación**, esto las hace más fuertes frente al cambio e independientes de *frameworks*, además de más fáciles de testear (si por ejemplo se crea una instancia dentro de un método, no se podrá testear dicho método de forma aislada, ya que no se tendrá forma de sustituir el comportamiento de la instancia creada, y cualquier error en el test hará dudar de qué clase es la culpable).  

##### Dependencia fuerte == Alto acoplamiento  
Ocurre cuando se instancia el objeto directamente en la clase, produciendo un efecto de acoplamiento. Esto la hace más difícil de reutilizar y además no podrá ser testeada de forma inependiente a la otra clase.

```kotlin
class Auto {
    val rueda = RuedaMichelin()
}
```

##### Formas básicas para inyectar dependencias  
- **Mediante el constructor**: se pasa la dependencia que la clase necesita a través de su constructor.

```kotlin
class Auto(val rueda: Rueda) { }
```

- **Mediante un método**: suelen ser métodos *setter*.

```kotlin
class Auto {
    private var rueda: Rueda? = null
    fun setRueda(rueda: Rueda) {
        this.rueda = rueda
    }
}
```

- **Mediante una interfaz**: la clase cliente debe implementar una interfaz (un contrato) mediante la cual se le pueda inyectar la dependencia. Por otro lado, se necesita de la clase dedicada a realizar la inyección de la dependencia, a la que se le pasan los clientes que implementen la interfaz. Esta clase será la responsable de inyectar la dependencia a todos los clientes. Además, puede tener métodos que permitan sustituir una dependencia por otra, que cumpla el mismo contrato.

```kotlin
interface RuedaSetter {
    fun setRueda(rueda: Rueda?)
}

class Auto : RuedaSetter {
    private var rueda: Rueda? = null

    override fun setRueda(rueda: Rueda?) {
        this.rueda = rueda
    }
}
```

```kotlin
class RuedaInjector {
    var clients: MutableSet<RuedaSetter>? = null
    
    fun inject(client: RuedaSetter) {
        clients!!.add(client)
        client.setRueda(RuedaMichelin())
    }

    fun cambiarAGoodYear() {
        for (client in clients!!) {
            client.setRueda(RuedaGoodyear())
        }
    }
}
```

#### ***Dagger***
Es un *framework* de inyección de dependencias, que **se divide sobre todo en Componentes y Módulos**. Los **Módulos** son las **clases que se encargan de proveer las dependencias**. Los **Componentes** son **interfaces** a las que están ligados uno o varios módulos; estas interfaces, que serán usadas por Dagger para generar el código, **actúan como puente entre las dependencias que proveen los módulos y las clases donde serán inyectadas**.

En el ***build.gradle*** de la *app* (si se utiliza Kotlin):

```kotlin
apply plugin: 'kotlin-kapt'

implementation 'com.google.dagger:dagger:<VERSION>'
kapt 'com.google.dagger:dagger-compiler:<VERSION>'
```

###### ``@Inject``
Una vez creada la interfaz y la clase que la implementa, se puede utilizar la dependencia creando una instancia de la clase con la implementación concreta, aunque así se hace manualmente.  
Para hacer uso de la inyección de dependencias con Dagger, primero hay que mostrarle **cómo crear instancias de la clase de la que se depende (la que tiene la implementación concreta)**. Para esto, se coloca el tag *``@Inject``* delante del constructor. Cuando se necesite una instancia de la clase, Dagger obtendrá los parámetros requeridos e invocará el constructor.

```kotlin
class ConsolaLog @Inject constructor(): Log {
    override fun log(tag: String, message: String) {
        android.util.Log.i(tag, message)
    }
}
```

###### ``@Component``
Para que Dagger sepa de dónde **obtener las dependencias que necesita**, se debe crear un **grafo** con ellas. Esto se hace en una interfaz anotada con el tag *``@Component``*. Con esto, **Dagger creará automáticamente una implementación de la interfaz**, añadiendo el prefijo Dagger. Dentro de la interfaz, se declaran funciones que devuelvan instancias de la clase que se tiene como dependencia.

```kotlin
@Component
interface ApplicationGraph {
    // Por convención, el nombre de la función suele comenzar con 'provides'
    fun providesConsolaLog(): ConsolaLog
}

// Y donde se implemente, por ejemplo un fragment
private const val TAG = "GalleryFragment"
class GalleryFragment : Fragment() {
    private lateinit var logger: ConsolaLog

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        logger = DaggerApplicationGraph.create().providesConsolaLog()
    }

    override fun onResume() {
        super.onResume()
        logger.log(TAG, "onResume()")
    }
}
```

Sin embargo, esto se parece más a un proveedor de servicios que a inyección de dependencias.  
Para inyectar la dependencia, se debe sustuir el método que provee de un objeto de tipo ``ConsolaLog`` por otro que permita **pasarle como parámetro el cliente**:

```kotlin
@Component
interface ApplicationGraph {
    fun inject(target: GalleryFragment)
}

// Y en el fragment
private const val TAG = "GalleryFragment"
class GalleryFragment : Fragment() {
    // Dagger no soporta el modificador 'private'
    @Inject lateinit var logger: ConsolaLog

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        DaggerApplicationGraph.create().inject(target: this)
    }
    
    override fun onResume() {
        super.onResume()
        logger.log(TAG, "onResume()")
    }
}
```

###### ``@Singleton``
Si no se indica lo contrario, Dagger creará por defecto una nueva instancia cada vez que se quiera asignar una dependencia de un tipo en concreto. También creará una nueva instancia de la implementación de las interfaces *``@Component``*.  
Sin embargo, en muchos casos, esto no es lo deseado. Para solucionarlo, se utiliza el tag *``@Singleton``*:

```kotlin
@Singleton
class ConsolaLog @Inject constructor(): Log {
    override fun log(tag: String, message: String) {
        android.util.Log.i(tag, message)
    }
}

@Singleton
@Component
interface ApplicationGraph {
    fun inject(target: GalleryFragment)
}
```

Para crear una **instancia única a compartir en toda la aplicación**, se crea una propiedad en la que se intancia ``DaggerApplicationGraph``:

```kotlin
class App: Application() {
    val component = DaggerApplicationGraph. create()
}

// Y en el fragment
private const val TAG = "GalleryFragment"
class GalleryFragment : Fragment() {
    // Dagger no soporta el modificador 'private'
    @Inject lateinit var logger: ConsolaLog

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        (requireActivity().application as App).component.inject(target: this)
    }
    
    override fun onResume() {
        super.onResume()
        logger.log(TAG, "onResume()")
    }
}
```

###### ``@Module``, ``@Provides`` y ``@Binds``
La anotación *``@Inject``* es muy útil para inyectar dependencias cuando se puede acceder directamente a la clase de la que se depende. Pero no servirá para interfaces o para clases de terceros.  
Para que Dagger sepa cuál de las **múltiples implementaciones** que la interfaz pueda llegar a tener debe instanciar, se utiliza una clase con el tag *``@Module``*, donde se declara el método que provea la dependencia. La práctica recomendada respecto a este método, es utilizar el tag *``@Binds``* para indicarle a Dagger qué implementación debe usar de una interfaz. Y usar el tag *``@Provides``* para indicarle cómo proporcionar clases de librerías externas.

```kotlin
@Module
// Si se utiliza @Provides
class ApplicationModule () {
    @Provides fun providesLog(log: ConsolaLog): Log = log
}
// Si se utiliza @Binds, la clase y el método deben ser abstractos
abstract class ApplicationModule () {
    @Binds abstract fun providesLog(log: ConsolaLog): Log
}

@Singleton
@Component(modules = arrayOf(ApplicationModule::class))
interface ApplicationGraph {
    fun inject(target: GalleryFragment)
    fun inject(target: SlideshowFragment)
}

// Y en el fragment, el logger pasa a ser del tipo de la interfaz
private const val TAG = "GalleryFragment"
class GalleryFragment : Fragment() {
    @Inject lateinit var logger: Log
    // El resto...
}
```

<img src="Dagger.png" width="700">

El esquema anterior representa una forma de estructurar componentes y módulos para proveer las dependencias. En él destaca un componente, ***AppComponent***, el componente principal al que están ligados los siguientes módulos:  
- ***AppModule***: este módulo se encargará de proporcionar las **dependencias únicas** (***Singleton***) que **pervivirán a lo largo del ciclo de vida de la aplicación**.
- ***ActivityBuilder***: gracias a la anotación ***``@ContributesAndroidInjector``*** se pueden acoplar fácilmente las *Activities* y/o *Fragments* al grafo de dependencias, sin olvidarse de hacer referencia al módulo concreto que proveerá la instancia del *Presenter*.
- ***AndroidSupportInjectionModule***: clase interna incluida en *Dagger* 2.10 que ayuda a **enlazar los tipos propios de Android** (*Activity*, *Fragment*, *Service*…).  
Al entrar en el método *``onCreate()``* de la *app*, se construirá ***AppComponent***, y con él el grafo de dependencias. En tiempo de compilación se habrá puesto en conocimiento de *Dagger* las *Activities* y/o *Fragments* que se desean acoplar al grafo, y se estára listo para acceder a las dependencias a través de la anotación ***``@Inject``*** (ejemplo en Java):

```java
public class MoviesRepositoryImpl implements MoviesRepository {

  private MoviesLocalDataSource localDataSource;
  private MoviesRemoteDataSource remoteDataSource;
  private EntityDataMapper entityDataMapper;

  @Inject
  public MoviesRepositoryImpl(MoviesLocalDataSource localDataSource, 
    MoviesRemoteDataSource remoteDataSource,
    EntityDataMapper entityDataMapper) {
    this.localDataSource = localDataSource;
    this.remoteDataSource = remoteDataSource;
    this.entityDataMapper = entityDataMapper;
  }

  @Override
  public void getMovies(final Handler<List<Movie>> handler) {
    // Code . . .
  }

  @Override
  public void getMovie(int movieId, final Handler<Movie> handler) {
    // Code . . .
  }
}
```

#### ***Koin***
Es un **framework de inyección de dependencias** que busca el mismo objetivo que *Dagger* pero que además es mucho más **fácil de implementar y se integra perfectamente con el ecosistema Android y los ViewModel**.

```kotlin
val appModule = module {

  single<SQLiteOpenHelper> { MoviesDatabaseHelper(androidContext()) }

  single<MovieService> {
    Retrofit.Builder()
      .baseUrl("https://api.themoviedb.org/3/")
      .addConverterFactory(GsonConverterFactory.create())
      .build()
      .create<MovieService>(MovieService::class.java)
  }

  single<MoviesLocalDataSource> { MoviesLocalDataSourceImpl(sqLiteOpenHelper = get()) }

  single<MoviesRemoteDataSource> { MoviesRemoteDataSourceImpl(movieService = get()) }

  single<MoviesRepository> {
    MoviesRepositoryImpl(
      localDataSource = get(),
      remoteDataSource = get(),
      movieMapper = MovieMapper()
    )
  }

  single<InteractorExecutor> {
    AsyncInteractorExecutor(
      runOnBgThread = BackgroundRunner(),
      runOnMainThread = MainRunner()
    )
  }

  single { Formatter() }
}
```

La implementación más común de *Koin* se suele basar en un **módulo principal**, que contiene aquellas dependencias (normalmente ***Singleton***) que se quiere pervivir a lo largo de todo el ciclo de vida de la aplicación, más **un módulo por cada** ***feature***, donde se alojarán las dependencias que afecten de un modo concreto a una determinada funcionalidad.

```kotlin
val listModule = module {
  factory { GetMovies(repository = get()) }
  factory { MovieModelFactory() }
  viewModel {
    MovieListViewModel(
      executor = get(),
      getMovies = get(),
      movieModelFactory = get()
    )
  }
}
```

La declaración de las dependencias queda muy limpia, y además, sus palabras reservadas resultan bastante intuitivas:  
- ***single*** -> crea una instancia *Singleton*.  
- ***factory*** -> crea una instancia nueva cada vez que es requerida.  
- ***viewModel*** -> crea una instancia del *ViewModel*, y abstrae así de la utilización de *ViewModelProvider*.  
- ***get*** -> infiere una dependencia.  
Por último, solo quedaría lanzar en el **método** ***``onCreate()``*** de la clase ***Application*** el **método** ***``startKoin()``*** con los módulos que se quieren inyectar.

```kotlin
class MoviesApp : Application() {

  override fun onCreate() {
    super.onCreate()
    startKoin {
      androidLogger()
      androidContext(this@MoviesApp)
      modules(listOf(
        appModule,
        listModule,
        detailModule
      ))
    }
  }
}
```

Y ya se podría recibir la dependencia o dependencias en la *Activity* o *Fragment* a través de *``inject()``* o *``viewModel()``*.

```kotlin
import org.koin.android.viewmodel.ext.android.viewModel

class MovieListActivity : BaseActivity() {

  private val viewModel: MovieListViewModel by viewModel()

  // Code...
}
```



---



### ***Testing***

##### Algunas consideraciones/notas sobre los *tests*:
  - Las pruebas unitarias **no deberían lidiar con nada del ciclo de vida** de Android, tal como el contexto.
  - ***Mocks***: **sirven para testear "comportamiento"**. Es decir, si una clase se llamó, cuántas veces se llamó, qué argumentos se pasaron, etc.
  - ***Fakes***: **sirven para testear el "estado"**. Es decir, se hace sobre los componentes, sobre esos ***test doubles***, y después se comprueba en qué estado quedó ese *fake*. Suelen ser simplificaciones de las dependencias sobre las que se está trabajando.


##### Fundamentos de *JUnit*:
Se puede crear una **clase de reglas** (clases extra que se pueden incluir en las clases de prueba) heredando de la clase `TestRule`, y luego aplicar esa regla en las clases de *test*:

```kotlin
@get:Rule val <NAME> = <RULE CLASS>()
```

Para pruebas unitarias a las que se les debe pasar muchos datos, se puede utilizar el *runner* `Parameterized`. De la documentación oficial: "Cuando se ejecuta una clase de prueba parametrizada, se crean instancias para el producto cruzado de los métodos de prueba y los elementos de datos de prueba."

```kotlin
@RunWith(value = Parameterized::class)
class ParameterizedTest(var currentValue: Boolean, var currentUser: User) { }
```


##### *JUnit* + TDD + *Hamcrest*:
Dentro de las pruebas unitarias, hay múltiples formas de captar y analizar una excepción para que pueda ser procesada correctamente.  

  - Se le puede indicar al *test* que se espera una excepción determinada:

```kotlin
@Test(expected = NullPointerException::class)
fun loginUser_nullEmail_returnsException() { }
```

  - Se le puede indicar al *test* que se espera una excepción personalizada:

```kotlin
@Test(expected = CustomException::class)
fun loginUser_nullEmail_returnsCustomException() { }
```

  - También se puede utilizar el método `assertThrows()`:  

```kotlin
@Test
fun loginUser_nullPassword_returnsCustomException(){
    assertThrows(CustomException::class.java) { }
}
```

  - Se puede utilizar bloques ***try-catch***:  

```kotlin
@Test
fun loginUser_nullEmailAndPassword_returnsCustomException(){
    try {
        val result = userAuthentication(null, null)
        assertEquals(AuthEvent.NULL_FORM, result)
    } catch (e: Exception) {
        (e as? CustomException)?.let {
            assertEquals(AuthEvent.NULL_FORM, it.authEvent)
        }
    }
}
```

##### *Mockito*
  - Se puede correr los *tests* con el *runner* de *Mockito* anotando la clase con lo siguiente:  

```kotlin
@RunWith(MockitoJUnitRunner::class)
```

  - Hay diferentes anotaciones para *mockear* dependencias del **SUT** (***Subject Under Test***):  

```kotlin
@Mock
@Spy
```

  - Uso de la función `verify()` y `verifyNoInteractions()` para validar que se ha producido tal o cual comportamiento:  

```kotlin
verify(mockedObject).realMethod()
verifyNoInteractions(mockedObject)
```

  - Uso del complemento ***Mockito Inline*** para los casos que se quiere *mockear* un ***Object*** (***final class*** **en Java**):  

```kotlin
testImplementation("org.mockito:mockito-inline:<VERSION")
```

  - Uso de `when`-`thenReturn` para realizar ***stubs*** (comportamiento de retornar algo *hardcodeado*):  

```kotlin
`when`(<MOCKED OBJECT FUNCTION>).thenReturn(<SIMULATED RESULT>)
```

  - Uso de ***spy*** (permite que una clase, en vez de estar *mockeada*, funcione totalmente y se pueda monitorear el comportamiento):  
    Ref.: https://stackoverflow.com/questions/28295625/mockito-spy-vs-mock

```kotlin
@Spy
lateinit var mockedSomething: Something
```

##### *Robolectric*
Es recomendable utilizar el conjunto de librerías de *Jetpack* ***AndroidX Test***. De la documentación oficial: "*AndroidX Test* proporciona reglas *JUnit4* para iniciar actividades e interactuar con ellas en las pruebas *JUnit4*. También contiene marcos de prueba de UI como *Espresso*, *UI Automator* y el simulador *Robolectric*."  
Las dependencias se agregan con ***testImplementation*** en vez de ***androidTestImplementation*** para poder acceder a la clase en la anotación `@RunWith: @RunWith(AndroidJUnit4::class)`:

```kotlin
testImplementation 'androidx.test:core-ktx:<VERSION>'
testImplementation 'androidx.test.ext:junit-ktx:<VERSION>'
```

Para resolver cuestiones referentes a ***LiveData*** (Componente de Arquitectura), se debe agregar una dependencia que permite incluir una regla para lograrlo:  

```kotlin
testImplementation 'androidx.arch.core:core-testing:<VERSION>'
 
----------------------------------------------------------
 
@RunWith(AndroidJUnit4::class)
class AddViewModelTest{
    @get:Rule
    var instantExcecutorRule = InstantTaskExecutorRule()
```

Si surge el *warning* sobre que no se encuentra el ***Manifest***, se debe agregar lo siguiente el ***build.gradle(:app)***:  

```kotlin
testOptions{
    unitTests.includeAndroidResources = true
}
```

Los ***ViewModel*** que heredan de ***AndroidViewModel***, requieren de una instancia de ***`Application`*** por constructor. Para eso, en un *test* se puede hacer de la siguiente manera, pasando el ***`Context`*** de la aplicación:  

```kotlin
val mainViewModel = MainViewModel(ApplicationProvider.getApplicationContext())
```

###### Versiones Robolectric y nivel de API
Cuando sale una nueva versión de Android, puede que *Robolectric* no la soporte de inmediato. Para eso, hay dos formas de solucionarlo, con la anotación ***`@Config`***:  

  1. `@Config(sdk = [21, 22, 30])` -> Corre los *tests* para las versiones configuradas.
  2. `@Config(maxSdk = 30)` -> Corre los *tests* para todas las versiones hasta la configurada inclusive.

###### *Tests* para clases que heredan de ***`FragmentDialog`***
Antes que nada, se debe tener agregada la dependencia para testear *fragments* en el ***build.gradle(:app)***:  

```kotlin
debugImplementation 'androidx.fragment:fragment-testing:1.4.0'
```

Posteriormente, en el *test*, se puede usar un concepto llamado ***`Scenario`***, en donde primero se crea el *fragment*, luego se configura el estado del ciclo de vida al cual se moverá, y por último se accede a él.  
Por ejemplo:  

```kotlin
val scenario = launchFragment<AddProductFragment>(themeResId = R.style.Theme_Example)
scenario.moveToState(Lifecycle.State.RESUMED)
scenario.onFragment{ fragment ->
    (fragment.dialog as? AlertDialog)?.let {
        it.getButton(DialogInterface.BUTTON_NEGATIVE).performClick()
        assertThat(fragment.dialog, `is`(nullValue()))
    }
}
```

##### *Espresso*
**Nota**: las animaciones se pueden desactivar desde **Opciones del desarrollador** del dispositivo para que no interfieran con el resultado de los *tests* con *Espresso*.  

Los *tests* con *Espresso* se van a guardar en `src/androidTest/`, en vez de `src/test/`, ya que son pruebas instrumentadas (o de instrumentación).  

###### Anotación `@LargeTest`
Sirve para indicar que las pruebas podrían demorar, que el acceso a la UI puede requerir más tiempo y que es necesario configurar un entorno seguro.

```kotlin
@RunWith(AndroidJUnit4::class)
@LargeTest
class MainActivityTest { }
```

###### Uso de `ActivityScenarioRule`
Permite tener acceso a la *activity* en caso de requerir una **modificación interna**.  

```kotlin
@get:Rule
val activityRule = ActivityScenarioRule(ExampleActivity::class.java)
```

Por ejemplo, serviría para **limitar un valor** a ingresar dentro de la *activity*:  

```kotlin
val scenario = activityRule.scenario
scenario.moveToState(Lifecycle.State.RESUMED)
scenario.onActivity { activity ->
    activity.selectedProduct.quantity = 1
}
```

###### Pruebas básicas (ver https://developer.android.com/training/testing/espresso/basics#components)
Existen varios componentes para realizar pruebas con *Espresso*, como por ejemplo `onView`, `withId`, `check`, `matches`, `withText`, `perform`, `click`.  

###### Modificar contenido de componentes visuales

```kotlin
onView(withId(R.id.editTextExample)).perform(ViewActions.replaceText("12"))
```

###### ID's repetidos en las vistas + *Hamcrest*
Para los casos en donde una jerarquía de vistas tiene identificadores repetidos, como por ejemplo cuando tiene un *layout* incluido (usando el *tag* `<include/>`) que contiene recursos con el mismo identificador, se utilizan las funciones ***`allOf`*** (*matcher* que permite abarcar todas las coincidencias posibles) y ***`not`*** (sería la negación de `allOf`), pertenecientes a *Hamcrest*.  

```kotlin
@Test
fun checkTextField_startQuantity(){
    onView(allOf(withId(R.id.etNewQuantity), withContentDescription("cantidad")))
        .check(matches(withText("1")))
 
    onView(allOf(withId(R.id.etNewQuantity), not(withContentDescription("cantidad alterna"))))
        .check(matches(withText("1")))
 
    onView(allOf(withId(R.id.etNewQuantity), withContentDescription("cantidad alterna")))
        .check(matches(withText("5")))
}
```

###### *Record Espresso Test*
***Run*** > ***Record Espresso Test*** > Interactuar con la *app* y/o agregar aserciones con el botón ***Add Assertion***

###### Acceder a un `RecyclerView`
Para esto, se utiliza la siguiente dependencia:  

```kotlin
androidTestImplementation 'androidx.test.espresso:espresso-contrib:<VERSION>'
```

Luego, se puede crear una clase que retorne un ***`ViewAction`*** personalizado para pasarle como argumento a las funciones que lo requieren, por ejemplo, `actionOnItemAtPosition()`, que accede a los hijos de un `RecyclerView`.  
También existe una función de *Espresso* que permite abrir el menú de opciones (menú *overflow*):  

```kotlin
openActionBarOverflowOrOptionsMenu(ApplicationProvider.getApplicationContext())
```

###### Recursos dinámicos
Para los casos en que los recursos pueden cambiar (por ejemplo, por cambios de idioma del dispositivo), es preferible utilizar `ActivityScenario` para hacer que el texto bucado sea dinámico:  

```kotlin
var snackbarMsg = ""
activityScenarioRule.scenario.onActivity { activity ->
    snackbarMsg = activity.resources.getString(R.string.main_msg_go_history)
}
 
onView(withId(com.google.android.material.R.id.snackbar_text))
    .check(matches(withText(snackbarMsg)))
```

###### Aserción `fail()`
Esta función de *JUnit* lanza una excepción. Se utiliza por ejemplo dentro de un *try-catch* **para definir que el *test* no debería haber llegado hasta esa línea**.

##### *Retrofit* + Corrutinas

###### Uso de `runBlocking{}`
Se usa para ***testear funciones `suspend`***.  
Esta función fue ***remplazada por `runTest{}`*** (ver https://github.com/Kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-test/MIGRATION.md#replace-runblocking-with-runtest)  

###### Dependencia para *Testing* con Corrutinas

```kotlin
org.jetbrains.kotlinx:kotlinx-coroutines-test:<VERSION>
```

###### Uso de la clase `MainCoroutineRule`
Esta regla se usa para facilitar el *testing* con corrutinas. Ver como ejemplo este [*codelab* de Google](https://github.com/googlecodelabs/kotlin-coroutines/blob/master/coroutines-codelab/finished_code/src/test/java/com/example/android/kotlincoroutines/main/utils/MainCoroutineScopeRule.kt):  

```kotlin
@ExperimentalCoroutinesApi
class MainCoroutineScopeRule(val dispatcher: TestCoroutineDispatcher = TestCoroutineDispatcher()) :
        TestWatcher(),
        TestCoroutineScope by TestCoroutineScope(dispatcher) {
    override fun starting(description: Description?) {
        super.starting(description)
        // If your codebase allows the injection of other dispatchers like
        // Dispatchers.Default and Dispatchers.IO, consider injecting all of them here
        // and renaming this class to `CoroutineScopeRule`
        //
        // All injected dispatchers in a test should point to a single instance of
        // TestCoroutineDispatcher.
        Dispatchers.setMain(dispatcher)
    }

    override fun finished(description: Description?) {
        super.finished(description)
        cleanupTestCoroutines()
        Dispatchers.resetMain()
    }
}
```

###### Uso de `LiveDataTestUtil`
Este archivo provisto por Google, **provee de una función de extensión para poder testear objetos `LievData`**. Ver [ejemplo de Google](https://github.com/google/iosched/blob/main/androidTest-shared/src/main/java/com/google/samples/apps/iosched/androidtest/util/LiveDataTestUtil.kt).  

##### Complementos de *testing*
Perteneciente a ***Square*** (*Retrofit*, *OkHttp*), ***`MockWebServer`*** es usada para pruebas realacionadas con la respuesta del servidor.  
A su vez, `MockWebServer` tiene una herramienta llamada ***`MockResponse`***, que sirve para *mockear* la respuesta en formato JSON que provendría del servidor, pero de forma local.



---



### ***Players: ExoPlayer*** y ***JW Player***



---



### ***OAuth: Facebook, Twitter, Google+***



---



### ***Frameworks y SDK's: Firebase, Fabric, Sentry, Segment, Facebook***



---



### ***Deep linking***



---



### ***Push notifications***



---



### ***Jetpack Compose***







---
---



### Referencias y Fuentes:

- [Android Docs](https://developer.android.com/guide?hl=es_419)
- [DevExperto](https://devexperto.com/)
- [Curso Testing para Android con JUnit, Mockito, Espresso, TDD](https://www.udemy.com/course/curso-testing-para-android-con-junit-mockito-espresso-tdd/)
- [Desarrollo Android: Arquitectura avanzado](https://www.linkedin.com/learning/desarrollo-android-arquitectura-avanzado)
- [Android From Scratch](https://code.tutsplus.com/series/android-from-scratch--cms-996)
- [SGOliver.net](https://www.sgoliver.net/blog/curso-de-programacion-android/indice-de-contenidos/)
- [Sociedad Androide](https://www.youtube.com/c/SociedadAndroide/videos)
- [Suneet Agrawal](https://agrawalsuneet.github.io/publications/)
- [MVVM on Android with the Architecture Components + Koin](https://medium.com/swlh/mvvm-on-android-with-the-architecture-components-koin-f53c3c200363)
- [View Model Creation in Android — Android Architecture Components & Kotlin](https://proandroiddev.com/view-model-creation-in-android-android-architecture-components-kotlin-ce9f6b93a46b)
- [Android Basics — MVVM](https://medium.com/@brandonwever/android-mvvm-basics-5c48556e3ecc)
- [Android by example : MVVM +Data Binding](https://medium.com/@husayn.hakeem/android-by-example-mvvm-data-binding-introduction-part-1-6a7a5f388bf7)
- [No More LiveData in Repositories in Kotlin](https://betterprogramming.pub/no-more-livedata-in-repositories-in-kotlin-85f5a234a8fe)
- [MVVM (Model View ViewModel) + Kotlin + Google Jetpack](https://medium.com/@er.ankitbisht/mvvm-model-view-viewmodel-kotlin-google-jetpack-f02ec7754854)
- [Patrones Arquitectónicos en Android](https://medium.com/@vespasoft/patrones-arquitect%C3%B3nicos-en-android-ded39f7a2c10)
- [Optimizing Android ViewModel with Lifecycle 2.2.0](https://proandroiddev.com/optimizing-viewmodel-with-lifecycle-2-2-0-a2895b5c01fd)
- [Como usar el Android Navigation Component](https://dev.to/gvetri/como-usar-el-android-navigation-component-4hhg)
- [Android Navigation Component](https://proandroiddev.com/android-navigation-component-fc783c03bb8d)
