<h1>Fragment handling</h1>

***Index***:
<!-- TOC -->
  * [🚀 *Fragment Operations Cheatsheet*](#-fragment-operations-cheatsheet)
    * [``FragmentManager``](#fragmentmanager)
    * [``FragmentTransaction``](#fragmenttransaction)
    * [``commit`` vs ``commitNow``](#commit-vs-commitnow)
    * [``remove`` vs ``replace`` vs ``add`` vs ``hide/show``](#remove-vs-replace-vs-add-vs-hideshow)
  * [``SampleFragment``](#samplefragment)
  * [*XML view for the ``SampleActivity`` with its ``FragmentContainerView``*](#xml-view-for-the-sampleactivity-with-its-fragmentcontainerview)
  * [``SampleActivity``](#sampleactivity)
    * [*Back press handling*](#back-press-handling)
    * [*Fragment setup / reuse by tag*](#fragment-setup--reuse-by-tag)
    * [*Add fragment (Single Active Fragment pattern)*](#add-fragment-single-active-fragment-pattern)
    * [*Show fragment (reuse without recreating)*](#show-fragment-reuse-without-recreating)
    * [*Clean back stack*](#clean-back-stack)
  * [``AnotherFragment`` :arrow_right: *Child Fragment handling*](#anotherfragment-arrow_right-child-fragment-handling)
<!-- TOC -->

---

## 🚀 *Fragment Operations Cheatsheet*
### ``FragmentManager``
| **Operación**               | **Qué hace**                                                       | **Cuándo usarla**                 |
|-----------------------------|--------------------------------------------------------------------|-----------------------------------|
| `findFragmentById(id)`      | Busca un _Fragment_ por `containerViewId`                          | Recuperar _Fragment_ activo       |
| `findFragmentByTag(tag)`    | Busca un _Fragment_ por `tag`                                      | Reutilización / navegación manual |
| `beginTransaction()`        | Inicia una transacción explícita                                   | API clásica (Java / bajo nivel)   |
| `commit {}`                 | Crea y ejecuta una transacción (`beginTransaction()` + `commit()`) | API moderna (KTX, recomendada)    |
| `popBackStack()`            | Revierte la última transacción del _back stack_                    | _Back_ navigation                 |
| `popBackStack(name, flags)` | Limpia el _back stack_ hasta un punto                              | _Reset_ / _logout_                |
| `backStackEntryCount`       | Cantidad de transacciones en el _back stack_                       | Decidir comportamiento de _back_  |
| `fragments`                 | Lista de _Fragments_ conocidos por el _manager_                    | Control manual de visibilidad     |


### ``FragmentTransaction``
| **Operación**                         | **Qué hace**                             | **Impacto**                  |
|---------------------------------------|------------------------------------------|------------------------------|
| `add(containerId, fragment, tag)`     | Agrega un _Fragment_ al _container_      | Crea instancia               |
| `remove(fragment)`                    | Elimina un _Fragment_                    | Destruye instancia           |
| `replace(containerId, fragment, tag)` | `remove(all)` + `add(new)`               | Recrea todo                  |
| `show(fragment)`                      | Hace visible un _Fragment_ existente     | Reusa instancia              |
| `hide(fragment)`                      | Oculta un _Fragment_ existente           | Mantiene estado              |
| `addToBackStack(name)`                | Guarda la transacción en el _back stack_ | Habilita _back_              |
| `setMaxLifecycle(fragment, state)`    | Controla estado de _lifecycle_           | _Performance_ / control fino |
| `setCustomAnimations(...)`            | Define animaciones de la transacción     | UX                           |
| `commit()`                            | Ejecuta la transacción (_async_)         | Navegación normal            |
| `commitAllowingStateLoss()`           | Ejecuta ignorando estado guardado        | Casos excepcionales          |
| `commitNow()`                         | Ejecuta inmediatamente (_sync_)          | Inicialización interna       |


### ``commit`` vs ``commitNow``
> ℹ️ **Nota:**  
> Usar ``commit`` **siempre** (es el mecanismo estándar de navegación).  
> ``commitNow`` se usa solo cuando se requiere que el _Fragment_ exista y esté activo **inmediatamente**, y no forme parte del _back stack_.

| **Aspecto**                                               | ``commit``                                                                                                                                                            | ``commitNow``            |
|-----------------------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------|
| Tipo de ejecución                                         | **Asíncrona**                                                                                                                                                         | **Síncrona**             |
| Momento de ejecución                                      | Próximo _frame_ del _main thread_                                                                                                                                     | Inmediato                |
| Bloquea el _main thread_                                  | ❌                                                                                                                                                                     | ✅                        |
| Permite ``addToBackStack``                                | ✅                                                                                                                                                                     | ❌                        |
| Participa del _back stack_                                | ✅                                                                                                                                                                     | ❌                        |
| Orden garantizado de ejecución                            | ❌                                                                                                                                                                     | ✅                        |
| _Lifecycle_ actualizado al retornar                       | ❌                                                                                                                                                                     | ✅                        |
| Uso en navegación                                         | ✅                                                                                                                                                                     | ❌                        |
| Uso recomendado                                           | General                                                                                                                                                               | Inicialización interna   |
| Ejecución tras `onSaveInstanceState` (estado ya guardado) | ❌ (sin `allowStateLoss` :arrow_right: Lanza `IllegalStateException`)<br/>✅ (con `allowStateLoss` :arrow_right: Puede ejecutarse, pero el estado no se va a restaurar) | ❌                        |
| 📌 **Uso típico**                                         | **UI principal, navegación**                                                                                                                                          | **_Fragments_ internos** |

### ``remove`` vs ``replace`` vs ``add`` vs ``hide/show``
> ℹ️ **Notas:**  
> **Equivalencias mentales**
>   - ``replace`` :arrow_right: ``remove(all)`` + ``add(new)`` (destruye y recrea)
>   - ``hide``/``show`` :arrow_right: Mantener instancias vivas
>
> **Reglas rápidas**
>   - ``replace`` para **flujos simples**
>   - ``add``/``remove`` (+ ``hide``/``show``) para **_performance_ y reutilización**.

| **Aspecto**            | ``remove``               | ``replace``                                                | ``add`` + ``hide/show`` |
|------------------------|--------------------------|------------------------------------------------------------|-------------------------|
| Qué elimina            | Un _Fragment_ específico | Todos los _fragments_ actualmente asociados al _container_ | Ninguno                 |
| Agrega _Fragment_      | ❌                        | ✅                                                          | ✅                       |
| Preserva instancia     | ❌                        | ❌                                                          | ✅                       |
| Preserva estado        | ❌                        | ❌                                                          | ✅                       |
| Reutiliza instancias   | ❌                        | ❌                                                          | ✅                       |
| Control de _lifecycle_ | Medio                    | Bajo                                                       | Alto                    |
| _Performance_          | Media                    | Baja                                                       | Alta                    |
| Complejidad            | Media                    | Baja                                                       | Alta                    |
| 📌 **Uso típico**      | **Control manual**       | **Navegación simple**                                      | **Navegación compleja** |

## ``SampleFragment``
```kotlin
// onCreate() and whatever...

companion object {
    private const val ARG_1 = "arg_1"
    private const val ARG_2 = "arg_2"

    @JvmStatic
    fun newInstance(arg1: String, arg2: String) =
        SampleFragment().apply {
            arguments = bundleOf(
                ARG_1 to arg1,
                ARG_2 to arg2
            )
        }
}
```

## *XML view for the ``SampleActivity`` with its ``FragmentContainerView``*
```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/main"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".SampleActivity">

    <androidx.fragment.app.FragmentContainerView
        android:id="@+id/fragment_container_view"
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

</androidx.constraintlayout.widget.ConstraintLayout>
```

## ``SampleActivity``
### *Back press handling*
```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
    // Other configurations
    interceptOnBackPressed()
    setupObservers()
}

private fun interceptOnBackPressed() {
    val callback = object : OnBackPressedCallback(true) {
        override fun handleOnBackPressed() {
            handleOnBackPressedInternal()
        }
    }
    onBackPressedDispatcher.addCallback(this, callback)
}

// Back navigation is based on back stack entries, not fragment visibility
private fun handleOnBackPressedInternal() {
    val fragmentManager = supportFragmentManager

    if (fragmentManager.backStackEntryCount > 1) {
        fragmentManager.popBackStack()
    } else {
        finish()
    }
}
```

### *Fragment setup / reuse by tag*
```kotlin
// ============================================================
// Called from an observer
// ============================================================
private fun setupSampleFragment() {
    val fragmentTag = SampleFragment::class.java.simpleName

    if (isFragmentLoaded(fragmentTag)) {
        showFragment(fragmentTag)
    } else {
        val fragment = SampleFragment.newInstance(
            arg1 = someJson.encodeUrl(),
            arg2 = otherJson.encodeUrl()
        )
        addFragment(fragment, fragmentTag)
    }
}

private fun isFragmentLoaded(fragmentTag: String): Boolean =
    supportFragmentManager.findFragmentByTag(fragmentTag) != null
```

### *Add fragment (Single Active Fragment pattern)*
```kotlin
private fun addFragment(
    fragment: Fragment,
    fragmentTag: String,
    addToBackStack: Boolean = true,
    allowStateLoss: Boolean = false
) {
    if (isFinishing || isDestroyed) return

    try {
        supportFragmentManager.commit(allowStateLoss) {

            supportFragmentManager.fragments.forEach {
                hide(it)
                setMaxLifecycle(it, Lifecycle.State.STARTED)
            }

            setCustomAnimations(
                R.anim.slide_in,
                R.anim.slide_out,
                R.anim.fade_in,
                R.anim.fade_out
            )

            add(R.id.fragment_container_view, fragment, fragmentTag)
            setMaxLifecycle(fragment, Lifecycle.State.RESUMED)

            if (addToBackStack) {
                addToBackStack(fragmentTag)
            }
        }
    } catch (e: IllegalStateException) {
        if (!allowStateLoss) {
            // ``allowStateLoss`` should only be used when losing state is acceptable
            // (example: logout, splash, hard reset)
            addFragment(fragment, fragmentTag, addToBackStack, true)
        }
    }
}
```

### *Show fragment (reuse without recreating)*
```kotlin
private fun showFragment(fragmentTag: String) {
    if (isFinishing || isDestroyed) return

    val fragment = supportFragmentManager.findFragmentByTag(fragmentTag) ?: return
    if (fragment.isVisible) return

    try {
        supportFragmentManager.commit {

            supportFragmentManager.fragments.forEach {
                if (it.isVisible) {
                    hide(it)
                    setMaxLifecycle(it, Lifecycle.State.STARTED)
                }
            }

            show(fragment)
            setMaxLifecycle(fragment, Lifecycle.State.RESUMED)
        }
    } catch (e: IllegalStateException) {
        // Ignored
        return
    }
}
```

### *Clean back stack*
```kotlin
// Clears navigation history but does NOT remove currently added fragments
private fun cleanBackStack() {
    supportFragmentManager.popBackStack(
        null,
        FragmentManager.POP_BACK_STACK_INCLUSIVE
    )
}
```

## ``AnotherFragment`` :arrow_right: *Child Fragment handling*
> ℹ️ **Nota:**  
> Este _snippet_ NO pertenece al flujo de navegación de la _Activity_.

```kotlin
// ============================================================
// Fragment hosting another Fragment using childFragmentManager
// ============================================================
private fun setupResults() = binding?.fragmentContainerView?.apply {
    SampleFragment().also { fragment ->
        // getChildFragmentManager() -> Return a private FragmentManager for placing and managing Fragments inside of this Fragment.
        childFragmentManager.beginTransaction()
            // getId() -> This view's ID
            .replace(id, fragment)
            .commitNow()
    }
}
```
