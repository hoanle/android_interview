
---

# 🗂️ State Management in Jetpack Compose

## 📌 Why State Matters

In **Jetpack Compose**, UI is **declarative** → the UI is a function of state.
To build responsive UIs, you must manage state carefully and decide **where it lives** (inside a composable, higher up, or in a `ViewModel`).

---

## 🔑 Ways to Manage State

### 1. **`remember`**

* Stores a single value **across recompositions**, but not across configuration changes (e.g., screen rotation).
* Best for **ephemeral UI state** (temporary, local).

```kotlin
var counter by remember { mutableStateOf(0) }

Button(onClick = { counter++ }) {
    Text("Clicked $counter times")
}
```

✅ Use when:

* State is **local to the composable**.
* Doesn’t need to survive activity/fragment recreation.

---

### 2. **`mutableStateOf`**

* A special observable type provided by Compose.
* Triggers **recomposition** when value changes.

```kotlin
val name = mutableStateOf("Alice")
name.value = "Bob" // UI updates automatically
```

✅ Use when:

* You need **reactive state** inside a composable or `ViewModel`.
* Typically wrapped inside `remember` in composables:

  ```kotlin
  var text by remember { mutableStateOf("") }
  ```

---

### 3. **`rememberSaveable`**

* Like `remember`, but survives **process death and configuration changes** by saving state in a `Bundle`.
* Works with types that are `Parcelable`/`Serializable`.

```kotlin
var username by rememberSaveable { mutableStateOf("") }
```

✅ Use when:

* State should persist across screen rotations, process death, or system-initiated recreation.
* Examples: text input, selected tab, scroll position.

---

### 4. **`ViewModel`**

* Lifecycle-aware class that **outlives configuration changes**.
* Holds UI-related state and business logic.
* Works seamlessly with Compose since Compose can observe `LiveData`, `Flow`, or `State`.

```kotlin
class UserViewModel : ViewModel() {
    var username by mutableStateOf("Alice")
        private set

    fun updateName(newName: String) {
        username = newName
    }
}

@Composable
fun UserScreen(viewModel: UserViewModel = viewModel()) {
    Text("Hello, ${viewModel.username}")
}
```

✅ Use when:

* State is **shared across multiple composables**.
* State should survive configuration changes and be tied to the screen’s lifecycle.

---

## 🧺 Collections in Compose

### **`SnapshotStateList`**

* A `MutableList` implementation that notifies Compose when items are added, removed, or changed.
* Designed for **observable lists** in Compose.

```kotlin
val items = remember { mutableStateListOf("A", "B", "C") }
items.add("D") // UI recomposes automatically
```

✅ Use when:

* You need a **dynamic list** that triggers recomposition on changes.
* E.g., chat messages, todo lists, shopping cart items.

---

### **`SnapshotStateMap`**

* A `MutableMap` that works like `SnapshotStateList`, but for key-value pairs.

```kotlin
val userScores = remember { mutableStateMapOf("Alice" to 10, "Bob" to 20) }
userScores["Charlie"] = 15 // UI recomposes
```

✅ Use when:

* You need a **key-value store** that updates UI when modified.
* E.g., caching user data, mapping IDs to objects.

---

## 📊 When to Use What?

| State Tool                  | Survives Recomposition         | Survives Config Change                           | Survives Process Death              | Scope                       |
| --------------------------- | ------------------------------ | ------------------------------------------------ | ----------------------------------- | --------------------------- |
| `remember`                  | ✅                              | ❌                                                | ❌                                   | Local UI state              |
| `mutableStateOf`            | ✅ (when wrapped in `remember`) | ❌                                                | ❌                                   | Basic observable state      |
| `rememberSaveable`          | ✅                              | ✅                                                | ✅                                   | Small, saveable UI state    |
| `ViewModel`                 | ✅                              | ✅                                                | ❌ (unless using `SavedStateHandle`) | Screen-wide or shared state |
| `SnapshotStateList` / `Map` | ✅                              | Depends (usually with `remember` or `ViewModel`) | ❌                                   | Dynamic collections         |

---

## ✅ Key Takeaways

* Use **`remember`** for **temporary UI state**.
* Use **`rememberSaveable`** when state should persist through **rotation & process death**.
* Use **`ViewModel`** for **business logic and shared state across composables**.
* Use **`SnapshotStateList` / `Map`** when working with **observable collections** that should recompose the UI on updates.

---