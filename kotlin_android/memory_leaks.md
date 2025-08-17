
---

# 🧠 Memory Leaks in Android

Memory leaks occur when **objects are no longer needed but cannot be garbage-collected**, causing **increased memory usage, slow performance, or app crashes**.

---

## 1️⃣ Common Causes of Memory Leaks

| Cause                               | Description                                                                         | Example                                                          |
| ----------------------------------- | ----------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| **Leaking Context**                 | Holding a reference to an `Activity` or `Fragment` beyond its lifecycle.            | Static variable: `static Activity activity;`                     |
| **Non-static inner classes**        | Inner classes hold an **implicit reference** to their outer class (e.g., Activity). | `Handler` or `Runnable` inside an Activity.                      |
| **Listeners / Callbacks**           | Registered listeners not unregistered can hold references to UI components.         | `View.setOnClickListener(this)` without removing in `onDestroy`. |
| **Singletons**                      | Singletons holding references to Activity/Context.                                  | `object MyManager { lateinit var activity: Activity }`           |
| **AsyncTasks, Threads, Coroutines** | Background tasks referencing UI after Activity destruction.                         | `AsyncTask` updating a destroyed Activity.                       |
| **Bitmaps & Drawables**             | Large images not recycled.                                                          | `ImageView.setImageBitmap()` without clearing.                   |
| **Fragments with retained views**   | Accessing destroyed view via `Fragment` after `onDestroyView`.                      | `binding.textView.text = "Hello"` after `onDestroyView`.         |

---

## 2️⃣ How to Identify Memory Leaks

### 2.1 Android Studio Profiler

* **Memory Profiler**: Track memory allocations and see objects retained after Activity destruction.
* Look for **Activity, Context, or Fragment instances** that should be gone.

### 2.2 LeakCanary

* Open-source library to **automatically detect memory leaks**.
* Shows **which objects are leaking and the reference chain**.

```kotlin
dependencies {
    debugImplementation "com.squareup.leakcanary:leakcanary-android:2.x"
}
```

### 2.3 StrictMode

* Detects leaked resources like **file descriptors, SQLite cursors, and network operations** on main thread.

```kotlin
StrictMode.setVmPolicy(
    StrictMode.VmPolicy.Builder()
        .detectLeakedClosableObjects()
        .penaltyLog()
        .build()
)
```

---

## 3️⃣ How to Prevent Memory Leaks

### 3.1 Avoid Leaking Context

* Use **Application context** for objects that outlive Activity:

```kotlin
val prefs = context.applicationContext.getSharedPreferences("prefs", Context.MODE_PRIVATE)
```

### 3.2 Use Static Inner Classes

* Inner classes can be **static** and use **WeakReference** to outer class:

```kotlin
class MyActivity : AppCompatActivity() {
    private val handler = MyHandler(this)

    private class MyHandler(activity: MyActivity) : Handler(Looper.getMainLooper()) {
        private val activityRef = WeakReference(activity)
        override fun handleMessage(msg: Message) {
            val activity = activityRef.get() ?: return
            // Update UI safely
        }
    }
}
```

### 3.3 Remove Listeners and Callbacks

* Unregister listeners in `onDestroy` or `onStop`:

```kotlin
override fun onDestroy() {
    myButton.setOnClickListener(null)
    mySensorManager.unregisterListener(this)
    super.onDestroy()
}
```

### 3.4 Use Lifecycle-aware Components

* **ViewModel**, **LiveData**, **Flow**, and **LifecycleObservers** automatically avoid leaks:

```kotlin
viewModel.liveData.observe(viewLifecycleOwner) { data ->
    textView.text = data
}
```

### 3.5 Clear Views in Fragments

* Nullify binding references after view is destroyed:

```kotlin
private var _binding: FragmentBinding? = null
private val binding get() = _binding!!

override fun onDestroyView() {
    _binding = null
    super.onDestroyView()
}
```

### 3.6 Handle Bitmaps Properly

* Use **Glide or Coil** to manage image memory efficiently.
* Recycle large Bitmaps manually if necessary.

---

## 4️⃣ Best Practices Summary

* **Use Application Context** where possible.
* **Avoid non-static inner classes** holding Activity references.
* **Remove listeners, callbacks, and observers** when not needed.
* **Leverage lifecycle-aware components** (ViewModel, LiveData, Flow).
* **Use memory profiling tools**: Android Profiler, LeakCanary.
* **Manage heavy resources** like Bitmaps with care.

---

Memory leaks are **common but preventable** with careful **lifecycle management** and **lifecycle-aware components**.

---
