
---

# 📱 Activity & Fragment Lifecycles in Android

Understanding **Activity and Fragment lifecycles** is crucial for managing **UI state, resources, and background tasks**. Lifecycle-aware components like **ViewModel** and **LiveData/Flow** rely on this for proper functioning.

---

## 1️⃣ Activity Lifecycle

An **Activity** represents a single screen. Its lifecycle is defined by **callbacks** invoked by the Android system:

### Activity Lifecycle Methods

| Method              | Description                                                                |
| ------------------- | -------------------------------------------------------------------------- |
| `onCreate(Bundle?)` | Activity is **created**. Initialize UI, ViewModels, data binding.          |
| `onStart()`         | Activity becomes **visible** but not interactive yet.                      |
| `onResume()`        | Activity is in the **foreground** and interactive.                         |
| `onPause()`         | Activity is partially obscured. Save **transient state**, stop animations. |
| `onStop()`          | Activity is **no longer visible**. Release heavy resources.                |
| `onDestroy()`       | Activity is **destroyed**. Clean up resources, cancel jobs.                |
| `onRestart()`       | Called after `onStop()` when activity **restarts**.                        |

**Flow Diagram (simplified):**

```
onCreate → onStart → onResume
           ↓
        onPause → onStop → onDestroy
```

**Notes:**

* **Foreground vs. background:** `onResume` → interactive, `onPause/onStop` → background.
* Avoid long-running tasks in `onResume` to prevent UI jank.

---

## 2️⃣ Fragment Lifecycle

A **Fragment** is a reusable UI component within an Activity. Its lifecycle is **tied to the hosting Activity**, but has additional states:

### Fragment Lifecycle Methods

| Method                                              | Description                                                           |
| --------------------------------------------------- | --------------------------------------------------------------------- |
| `onAttach(Context)`                                 | Fragment attached to Activity. Context available.                     |
| `onCreate(Bundle?)`                                 | Fragment is created. Initialize data, ViewModels.                     |
| `onCreateView(LayoutInflater, ViewGroup?, Bundle?)` | Inflate layout. Return the root view.                                 |
| `onViewCreated(View, Bundle?)`                      | View is ready. Bind UI and observe LiveData/Flow.                     |
| `onStart()`                                         | Fragment visible.                                                     |
| `onResume()`                                        | Fragment interactive.                                                 |
| `onPause()`                                         | Fragment paused; save transient UI state.                             |
| `onStop()`                                          | Fragment stopped; release resources.                                  |
| `onDestroyView()`                                   | View destroyed but fragment instance retained (common in navigation). |
| `onDestroy()`                                       | Fragment instance destroyed.                                          |
| `onDetach()`                                        | Fragment detached from Activity.                                      |

**Key Difference from Activity:**

* `onDestroyView()` vs `onDestroy()` allows **retaining fragment instance** while recreating view (important for ViewPager, navigation).

---

## 3️⃣ Interaction with Activity Lifecycle

* Fragment lifecycle is **nested within Activity lifecycle**:

  * Fragment `onAttach` → Activity `onCreate`.
  * Fragment `onDestroy` → Activity `onDestroy`.
* Fragment can outlive its view (`onDestroyView`) while Activity is still running, useful for **ViewModel retention**.

---

## 4️⃣ Lifecycle-aware Components

### 4.1 ViewModel

* Created **per Activity or Fragment** and survives **configuration changes**.
* Activity/Fragment destruction **does not destroy ViewModel** if it’s configuration change.

```kotlin
val viewModel: MyViewModel by viewModels() // Fragment scope
val activityViewModel: MyViewModel by activityViewModels() // Shared across fragments
```

### 4.2 LiveData

* Observes **lifecycle owners** (Activity/Fragment).
* Emits data **only when lifecycle is STARTED or RESUMED**.
* Stops emitting when lifecycle is paused/stopped.

```kotlin
viewModel.liveData.observe(viewLifecycleOwner) { data ->
    // update UI
}
```

### 4.3 Flow / StateFlow

* Use **lifecycle-aware collection** to prevent leaks:

```kotlin
lifecycleScope.launch {
    viewLifecycleOwner.lifecycle.repeatOnLifecycle(Lifecycle.State.STARTED) {
        viewModel.stateFlow.collect { value ->
            // update UI
        }
    }
}
```

* Ensures **collection only occurs when Fragment/Activity is active**.

---

## 5️⃣ Best Practices

* Avoid holding **long-lived references** to UI elements; clean up in `onDestroyView`.
* Use **ViewModel** for **UI state persistence** across rotations.
* Use **LiveData or Flow** with lifecycle-aware collectors to **prevent leaks**.
* Save transient UI state in `onSaveInstanceState`.

---

## 6️⃣ Summary

* **Activity Lifecycle:** onCreate → onStart → onResume → onPause → onStop → onDestroy.
* **Fragment Lifecycle:** Nested in Activity; has additional `onCreateView` / `onDestroyView`.
* **Lifecycle-aware components**:

  * ViewModel → survives configuration changes
  * LiveData / Flow → automatically observes active lifecycle states
* Proper lifecycle management prevents **memory leaks, crashes, and inconsistent UI**.

---

