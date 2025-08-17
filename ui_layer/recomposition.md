
---

# 🔄 Recomposition in Jetpack Compose

## 📌 What is Recomposition?

In Jetpack Compose, **recomposition** is the process where the framework **re-executes composable functions** to update the UI when the underlying **state changes**.

Instead of manually updating views (`textView.setText()`), Compose **reacts automatically**:

* State changes → Compose detects it → Composable functions re-run → UI reflects new state.
* It’s **selective**: not the entire UI redraws, only the parts affected by the change.

---

## ⚙️ How Recomposition Works

1. You declare UI with **composable functions**.
2. UI depends on **state** (`remember`, `State`, `Flow`, `LiveData`).
3. When the state changes:

   * Compose checks which composables read that state.
   * Those functions are **re-invoked** (recomposed).
   * UI tree is efficiently updated (not redrawn from scratch).

Example:

```kotlin
@Composable
fun Greeting(name: String) {
    Text("Hello, $name")
}

@Composable
fun GreetingScreen() {
    var name by remember { mutableStateOf("Alice") }

    Column {
        Greeting(name)
        Button(onClick = { name = "Bob" }) {
            Text("Change Name")
        }
    }
}
```

* When the button is clicked, `name` changes → only `Greeting(name)` recomposes.

---

## 🚀 Strategies to Optimize Recomposition

### 1. **Stability of Parameters**

* Compose tracks **stable types** (e.g., `String`, `Int`, `data class with stable fields`).
* If a parameter is stable and unchanged, Compose **skips recomposition**.
* Use `@Immutable` or `@Stable` annotations for custom classes to help Compose understand stability.

```kotlin
@Immutable
data class User(val id: Int, val name: String)
```

---

### 2. **`remember`**

* Keeps objects in memory **across recompositions**.
* Prevents expensive recalculations or re-allocations.

```kotlin
val painter = remember { loadPainterFromResources(R.drawable.avatar) }
```

* Without `remember`, the painter reloads on every recomposition.

---

### 3. **`remember` with Keys**

* Use keys to re-initialize only when dependencies change.

```kotlin
val userState = remember(userId) { loadUser(userId) }
```

* `loadUser()` runs only when `userId` changes, not on every recomposition.

---

### 4. **`derivedStateOf`**

* Optimizes derived calculations that depend on other states.
* Ensures recomposition happens **only when the result changes**, not every time inputs update.

```kotlin
val items = remember { mutableStateListOf(1, 2, 3) }
val itemCount by remember {
    derivedStateOf { items.size }
}
```

* `itemCount` recomposes **only if list size changes**, not every time list mutates.

---

### 5. **Hoisting State**

* Keep state **at the right level** in the hierarchy.
* If you place state too high, it can trigger unnecessary recompositions of parent composables.
* Hoist state closer to where it’s used.

---

### 6. **Use `LaunchedEffect` / `SideEffect`**

* For side effects that shouldn’t run on every recomposition.
* Example: network calls, logging, analytics.

```kotlin
LaunchedEffect(userId) {
    fetchUser(userId)
}
```

---

## ✅ Key Takeaways

* **Recomposition = UI update mechanism** when state changes.
* Compose avoids unnecessary work with **smart skipping** (stable params, structural equality).
* Use tools like **`remember`**, **`derivedStateOf`**, and **`stable/immutable types`** to minimize overhead.

---
