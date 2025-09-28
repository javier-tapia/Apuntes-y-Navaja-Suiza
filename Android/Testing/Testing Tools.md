<h1><i>Testing Tools</i></h1>

***Index***:
<!-- TOC -->
  * [*JUnit*](#junit)
    * [*JUnit* + TDD + *Hamcrest*](#junit--tdd--hamcrest)
  * [*MockK*](#mockk)
  * [*Mockito*](#mockito)
  * [*Robolectric*](#robolectric)
    * [Versiones Robolectric y nivel de API](#versiones-robolectric-y-nivel-de-api)
    * [*Tests* para clases que heredan de `FragmentDialog`](#tests-para-clases-que-heredan-de-fragmentdialog)
  * [*Espresso*](#espresso)
    * [Anotación `@LargeTest`](#anotación-largetest)
    * [Uso de `ActivityScenarioRule`](#uso-de-activityscenariorule)
    * [Pruebas básicas (ver https://developer.android.com/training/testing/espresso/basics#components)](#pruebas-básicas-ver-httpsdeveloperandroidcomtrainingtestingespressobasicscomponents)
    * [Modificar contenido de componentes visuales](#modificar-contenido-de-componentes-visuales)
    * [ID's repetidos en las vistas + *Hamcrest*](#ids-repetidos-en-las-vistas--hamcrest)
    * [*Record Espresso Test*](#record-espresso-test)
    * [Acceder a un `RecyclerView`](#acceder-a-un-recyclerview)
    * [Recursos dinámicos](#recursos-dinámicos)
    * [Aserción `fail()`](#aserción-fail)
  * [*Testing* con Corrutinas](#testing-con-corrutinas)
    * [Dependencia para *Testing* con Corrutinas](#dependencia-para-testing-con-corrutinas)
    * [Uso de `runBlocking{}`](#uso-de-runblocking)
    * [Uso de la clase `MainCoroutineRule`](#uso-de-la-clase-maincoroutinerule)
    * [Uso de `LiveDataTestUtil`](#uso-de-livedatatestutil)
  * [Complementos de *testing*](#complementos-de-testing)
<!-- TOC -->

---

## *JUnit*
Se puede crear una **clase de reglas** (clases extra que se pueden incluir en las clases de prueba) heredando de la clase `TestRule`, y luego aplicar esa regla en las clases de *test*:

```kotlin
  @get:Rule
  val <NAME> = <RULE CLASS>()
```

Para pruebas unitarias a las que se les debe pasar muchos datos, se puede utilizar el *runner* `Parameterized`. De la documentación oficial: "_Cuando se ejecuta una clase de prueba parametrizada, se crean instancias para el producto cruzado de los métodos de prueba y los elementos de datos de prueba._"

```kotlin
  @RunWith(value = Parameterized::class)
  class ParameterizedTest(var currentValue: Boolean, var currentUser: User) {}
```

### *JUnit* + TDD + *Hamcrest*
Dentro de las pruebas unitarias, hay múltiples formas de captar y analizar una excepción para que pueda ser procesada correctamente.

- Se le puede indicar al *test* que se espera una excepción determinada:

```kotlin
  @Test(expected = NullPointerException::class)
  fun loginUser_nullEmail_returnsException() {}
```

- Se le puede indicar al *test* que se espera una excepción personalizada:

```kotlin
  @Test(expected = CustomException::class)
  fun loginUser_nullEmail_returnsCustomException() {}
```

- También se puede utilizar el método `assertThrows()`:

```kotlin
  @Test
  fun loginUser_nullPassword_returnsCustomException() {
    assertThrows(CustomException::class.java) { }
  }
```

- Se pueden utilizar bloques ***try-catch***:

```kotlin
  @Test
  fun loginUser_nullEmailAndPassword_returnsCustomException() {
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

## *MockK*
TODO...

## *Mockito*
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

- Uso del complemento ***Mockito Inline*** para los casos que se quiere *mockear* un ``object``:

```kotlin
    testImplementation("org.mockito:mockito-inline:<VERSION")
```

- Uso de `when`-`thenReturn` para realizar ***stubs*** (comportamiento de retornar algo *hardcodeado*):

```kotlin
    `when`(< MOCKED OBJECT FUNCTION >).thenReturn(< SIMULATED RESULT >)
```

- Uso de un ***spy*** (permite que una clase, en vez de estar *mockeada*, funcione totalmente y se pueda monitorear el comportamiento):

> 🔍 Reference:   
> https://stackoverflow.com/questions/28295625/mockito-spy-vs-mock
> 

```kotlin
  @Spy
  lateinit var mockedSomething: Something
```

## *Robolectric*
Es recomendable utilizar el conjunto de librerías de *Jetpack* ***AndroidX Test***. De la documentación oficial: "_AndroidX Test proporciona reglas JUnit4 para iniciar actividades e interactuar con ellas en las pruebas JUnit4. También contiene marcos de prueba de UI como Espresso, UI Automator y el simulador Robolectric._"  
Las dependencias se agregan con **``testImplementation``** en vez de **``androidTestImplementation``** para poder acceder a la clase en la anotación `@RunWith(AndroidJUnit4::class)`:

```kotlin
  testImplementation 'androidx.test:core-ktx:<VERSION>'
  testImplementation 'androidx.test.ext:junit-ktx:<VERSION>'
```

Para resolver cuestiones referentes a ***LiveData***, se debe agregar una dependencia que permite incluir una regla para lograrlo:

```kotlin
  testImplementation 'androidx.arch.core:core-testing:<VERSION>'
  
----------------------------------------------------------
  
  @RunWith(AndroidJUnit4::class)
  class AddViewModelTest {
    @get:Rule
    var instantExcecutorRule = InstantTaskExecutorRule()
  
    // All the tests
  }
```

Si surge el *warning* sobre que no se encuentra el ***Manifest***, se debe agregar lo siguiente el ***build.gradle(:app)***:

```kotlin
  testOptions {
    unitTests.includeAndroidResources = true
  }
```

Los ***ViewModel*** que heredan de ***AndroidViewModel***, requieren de una instancia de ***`Application`*** por constructor.  
Para eso, en un *test* se puede hacer de la siguiente manera, pasando el ***`Context`*** de la aplicación:

```kotlin
    val mainViewModel = MainViewModel(ApplicationProvider.getApplicationContext())
```

### Versiones Robolectric y nivel de API
Cuando sale una nueva versión de Android, puede que *Robolectric* no la soporte de inmediato. Para eso, hay dos formas de solucionarlo, con la anotación ***`@Config`***:

1. `@Config(sdk = [21, 22, 30])` -> Corre los *tests* para las versiones configuradas.
2. `@Config(maxSdk = 30)` -> Corre los *tests* para todas las versiones hasta la configurada inclusive.

### *Tests* para clases que heredan de `FragmentDialog`
Antes que nada, se debe tener agregada la dependencia para testear *fragments* en el ***build.gradle(:app)***:

```kotlin
  debugImplementation 'androidx.fragment:fragment-testing:1.4.0'
```

Posteriormente, en el *test*, se puede usar un concepto llamado ***`Scenario`***, en donde primero se crea el *fragment*, luego se configura el estado del ciclo de vida al cual se moverá, y por último se accede a él.  
Por ejemplo:

```kotlin
  val scenario = launchFragment<AddProductFragment>(themeResId = R.style.Theme_Example)
  scenario.moveToState(Lifecycle.State.RESUMED)
  scenario.onFragment { fragment ->
    (fragment.dialog as? AlertDialog)?.let {
      it.getButton(DialogInterface.BUTTON_NEGATIVE).performClick()
      assertThat(fragment.dialog, `is`(nullValue()))
    }
  }
```

## *Espresso*
> 📝 Las animaciones se pueden desactivar desde **Opciones del desarrollador** del dispositivo para que no interfieran con el resultado de los *tests* con *Espresso*.

Los *tests* con *Espresso* se van a guardar en `src/androidTest/`, en vez de `src/test/`, ya que son pruebas instrumentadas (o de instrumentación).

### Anotación `@LargeTest`
Sirve para indicar que las pruebas podrían demorar, que el acceso a la UI puede requerir más tiempo y que es necesario configurar un entorno seguro.

```kotlin
  @RunWith(AndroidJUnit4::class)
  @LargeTest
  class MainActivityTest {}
```

### Uso de `ActivityScenarioRule`
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

### Pruebas básicas (ver https://developer.android.com/training/testing/espresso/basics#components)
Existen varios componentes para realizar pruebas con *Espresso*, como por ejemplo `onView`, `withId`, `check`, `matches`, `withText`, `perform`, `click`.

### Modificar contenido de componentes visuales

```kotlin
    onView(withId(R.id.editTextExample)).perform(ViewActions.replaceText("12"))
```

### ID's repetidos en las vistas + *Hamcrest*
Para los casos en donde una jerarquía de vistas tiene identificadores repetidos, como por ejemplo cuando tiene un *layout* incluido (usando el *tag* `<include/>`) que contiene recursos con el mismo identificador, se utilizan las funciones ***`allOf`*** (*matcher* que permite abarcar todas las coincidencias posibles) y ***`not`*** (sería la negación de `allOf`), pertenecientes a *Hamcrest*.

```kotlin
  import java.util.regex.Pattern.matches
  
  @Test
  fun checkTextField_startQuantity() {
    onView(allOf(withId(R.id.etNewQuantity), withContentDescription("cantidad")))
      .check(matches(withText("1")))
  
    onView(allOf(withId(R.id.etNewQuantity), not(withContentDescription("cantidad alterna"))))
      .check(matches(withText("1")))
  
    onView(allOf(withId(R.id.etNewQuantity), withContentDescription("cantidad alterna")))
      .check(matches(withText("5")))
  }
```

### *Record Espresso Test*
***Run*** ▶️ ***Record Espresso Test*** ▶️ Interactuar con la *app* y/o agregar aserciones con el botón ***Add Assertion***

### Acceder a un `RecyclerView`
Para esto, se utiliza la siguiente dependencia:

```kotlin
    androidTestImplementation 'androidx.test.espresso:espresso-contrib:<VERSION>'
```

Luego, se puede crear una clase que retorne un ***`ViewAction`*** personalizado para pasarle como argumento a las funciones que lo requieren, por ejemplo, `actionOnItemAtPosition()`, que accede a los hijos de un `RecyclerView`.  
También existe una función de *Espresso* que permite abrir el menú de opciones (menú *overflow*):

```kotlin
    openActionBarOverflowOrOptionsMenu(ApplicationProvider.getApplicationContext())
```

### Recursos dinámicos
Para los casos en que los recursos pueden cambiar (por ejemplo, por cambios de idioma del dispositivo), es preferible utilizar `ActivityScenario` para hacer que el texto bucado sea dinámico:

```kotlin
  import java.util.regex.Pattern.matches
  
  var snackbarMsg = ""
  activityScenarioRule.scenario.onActivity { activity ->
    snackbarMsg = activity.resources.getString(R.string.main_msg_go_history)
  }
  
  onView(withId(com.google.android.material.R.id.snackbar_text))
    .check(matches(withText(snackbarMsg)))
```

### Aserción `fail()`
Esta función de *JUnit* lanza una excepción. Se utiliza por ejemplo dentro de un *try-catch* **para definir que el *test* no debería haber llegado hasta esa línea**.

## *Testing* con Corrutinas

### Dependencia para *Testing* con Corrutinas

```kotlin
    org.jetbrains.kotlinx:kotlinx-coroutines-test:<VERSION>
```

### Uso de `runBlocking{}`
Se usa para ***testear funciones `suspend`***. Esta función fue ***remplazada por `runTest{}`*** (ver https://github.com/Kotlin/kotlinx.coroutines/blob/master/kotlinx-coroutines-test/MIGRATION.md#replace-runblocking-with-runtest)

### Uso de la clase `MainCoroutineRule`
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

### Uso de `LiveDataTestUtil`
Este archivo provisto por Google, **provee de una función de extensión para poder testear objetos `LievData`**. Ver [ejemplo de Google](https://github.com/google/iosched/blob/main/androidTest-shared/src/main/java/com/google/samples/apps/iosched/androidtest/util/LiveDataTestUtil.kt).

## Complementos de *testing*
Perteneciente a ***Square*** (*Retrofit*, *OkHttp*), ***`MockWebServer`*** es usada para pruebas realacionadas con la respuesta del servidor.  
A su vez, `MockWebServer` tiene una herramienta llamada ***`MockResponse`***, que sirve para *mockear* la respuesta en formato JSON que provendría del servidor, pero de forma local.
