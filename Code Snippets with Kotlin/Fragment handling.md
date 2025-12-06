<h1>Fragment handling</h1>

***Index***:
<!-- TOC -->
  * [SampleActivity](#sampleactivity)
  * [XML view for the SampleActivity with its FragmentContainerView](#xml-view-for-the-sampleactivity-with-its-fragmentcontainerview)
  * [SampleFragment](#samplefragment)
<!-- TOC -->

---

## SampleActivity

```kotlin
override fun onCreate(savedInstanceState: Bundle?) {
		// Other configurations
		interceptOnBackPressed()
		setupObservers()
}

private fun interceptOnBackPressed() {
    val callback = object : OnBackPressedCallback(true) {
        override fun handleOnBackPressed() {
            handlerOnBackPressed()
        }
    }
    this.onBackPressedDispatcher.addCallback(this, callback)
}

private fun handlerOnBackPressed() {
    val fragmentManager = supportFragmentManager

    if (fragmentManager.backStackEntryCount > 1) {
        fragmentManager.popBackStack()
    } else {
        finish()
    }
}

// Called from an observer
private fun setupSampleFragment() {
    val fragmentTag = SampleFragment::class.java.simpleName
    val isSampleFragmentLoaded = isFragmentLoaded(fragmentTag)
    if (isSampleFragmentLoaded) {
        showFragment(fragmentTag)
    } else {
        val sampleFragment = SampleFragment.newInstance()
        val bundle = Bundle().apply {
            putString("some_query_key", someJson.encodeUrl())
            putString("other_query_key", otherJson.encodeUrl())
        }
        sampleFragment.arguments = bundle
        addFragment(sampleFragment, fragmentTag)
    }
}

private fun isFragmentLoaded(fragmentTag: String): Boolean =
    supportFragmentManager.findFragmentByTag(fragmentTag) != null


private fun addFragment(
    fragment: Fragment,
    fragmentTag: String,
    addToBackStack: Boolean = true,
    allowStateLoss: Boolean = false
) {
    if (isFinishing || isDestroyed) return

    try {
        supportFragmentManager.commit(allowStateLoss) {
            supportFragmentManager.fragments.iterator().forEach {
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
            // Retry with state loss allowed
            return addFragment(fragment, fragmentTag, addToBackStack, true)
        }
        return
    }
}

private fun showFragment(fragmentTag: String) {
    if (isFinishing || isDestroyed) return

    val fragment = supportFragmentManager.findFragmentByTag(fragmentTag) ?: return
    if (fragment.isVisible) return

    try {
        supportFragmentManager.commit {
            supportFragmentManager.fragments.iterator().forEach {
                if (it.isVisible) {
                    hide(it)
                    setMaxLifecycle(it, Lifecycle.State.STARTED)
                }
            }

            show(fragment)
            setMaxLifecycle(fragment, Lifecycle.State.RESUMED)
        }
    } catch (e: IllegalStateException) {
        return
    }
}

private fun removePreviousFragments() {
    supportFragmentManager.apply {
        for (fragment in fragments) {
            beginTransaction().remove(fragment).commit()
        }
        popBackStack(null, POP_BACK_STACK_INCLUSIVE)
    }
}
```

## XML view for the SampleActivity with its FragmentContainerView

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

## SampleFragment

```kotlin
// onCreate() and whatever...

companion object {
    const val SOMETHING = "something"

    @JvmStatic
    fun newInstance() = SampleFragment()
}
```
