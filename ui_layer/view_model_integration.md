
---

# 🧩 Jetpack Compose & ViewModel Integration

## 📌 Why ViewModel in Compose?

In Android, a **ViewModel** is designed to hold and manage UI-related data in a lifecycle-conscious way.

* It **survives configuration changes** (like screen rotation).
* It helps keep **UI state separate from UI logic**, enabling testability and reusability.

In a **Compose world**, the **UI is stateless** and driven by state. The `ViewModel` naturally becomes the **source of truth** for state, while composables are responsible only for rendering.

---

## ⚙️ `viewModel()` Composable Function

Jetpack Compose provides the `viewModel()` function to obtain a `ViewModel` instance within composables.

```kotlin
@Composable
fun UserScreen() {
    val userViewModel: UserViewModel = viewModel()

    val username = userViewModel.username
    Text("Hello, $username")
}
```

### 🔑 Key Points:

* `viewModel()` retrieves the **correct instance** of a `ViewModel` tied to the **current `NavBackStackEntry`** or the **Activity/Fragment lifecycle**.
* Ensures **same ViewModel instance is reused** across recompositions.
* You can also inject custom factories:

```kotlin
val userViewModel: UserViewModel = viewModel(factory = UserViewModelFactory())
```

---

## 🔄 How ViewModels Survive Configuration Changes

* A `ViewModel` is scoped to a **lifecycle owner** (Activity, Fragment, or Navigation graph entry).
* When the device rotates or the configuration changes:

  * The **Activity/Fragment is destroyed and recreated**, but
  * The **ViewModel instance is retained** by the `ViewModelStore`.
* Compose recomposes after rotation, but `viewModel()` returns the **same instance**, ensuring **state is preserved**.

---

## 📝 Example with State

```kotlin
class CounterViewModel : ViewModel() {
    var count by mutableStateOf(0)
        private set

    fun increment() {
        count++
    }
}

@Composable
fun CounterScreen(counterViewModel: CounterViewModel = viewModel()) {
    Column {
        Text("Count: ${counterViewModel.count}")
        Button(onClick = { counterViewModel.increment() }) {
            Text("Increment")
        }
    }
}
```

* `count` is stored in the `ViewModel`.
* Even if the screen rotates, `CounterViewModel` survives, so the counter value is preserved.

---

## ✅ Key Takeaways

* **ViewModel holds UI state** → Composables render it.
* **`viewModel()` function** provides lifecycle-aware ViewModels in Compose.
* **Survives configuration changes** via `ViewModelStore`.
* Best practice: keep state/business logic in ViewModel, keep composables stateless.

---
