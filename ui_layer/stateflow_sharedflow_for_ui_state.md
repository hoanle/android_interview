
---

# 🔄 StateFlow vs SharedFlow in Jetpack Compose

When exposing **UI state** from a `ViewModel` to composables, you can use **`StateFlow`** or **`SharedFlow`** depending on the nature of the data and how it should be observed.

---

## 📌 StateFlow

### Characteristics

* **Holds a single, current state value**.
* Always **replays the latest value** to new collectors.
* Hot stream: updates are **active even without collectors**.
* Designed for **state that represents the UI at any point in time**.

### Use Case

* **UI state** that needs to be displayed immediately when a composable subscribes.
* Examples:

  * Counter value
  * Form input state
  * Current screen selection

### Example

```kotlin
class CounterViewModel : ViewModel() {
    private val _count = MutableStateFlow(0)
    val count: StateFlow<Int> = _count

    fun increment() {
        _count.value++
    }
}

@Composable
fun CounterScreen(viewModel: CounterViewModel = viewModel()) {
    val count by viewModel.count.collectAsState()
    Text("Count: $count")
}
```

✅ **Key point:** StateFlow always has a **single source of truth**, and the UI sees the latest value immediately.

---

## 📌 SharedFlow

### Characteristics

* **Does not hold state** by default (no initial value).
* Can **replay** a configurable number of past events to new collectors.
* Designed for **one-time events or actions**, not persistent UI state.
* Hot stream: active even without collectors.

### Use Case

* **Transient events** such as:

  * Navigation events
  * Toast/snackbar messages
  * Single-use triggers
* Avoids **UI replaying the same event on recomposition** unless explicitly configured.

### Example

```kotlin
class EventViewModel : ViewModel() {
    private val _events = MutableSharedFlow<String>()
    val events: SharedFlow<String> = _events

    fun showToast(message: String) {
        viewModelScope.launch {
            _events.emit(message)
        }
    }
}

@Composable
fun EventScreen(viewModel: EventViewModel = viewModel()) {
    val context = LocalContext.current

    LaunchedEffect(Unit) {
        viewModel.events.collect { message ->
            Toast.makeText(context, message, Toast.LENGTH_SHORT).show()
        }
    }
}
```

✅ **Key point:** SharedFlow is ideal for **fire-and-forget events** that should **not persist as state**.

---

## 📊 Comparison

| Feature             | StateFlow                     | SharedFlow                                            |
| ------------------- | ----------------------------- | ----------------------------------------------------- |
| Holds current value | ✅ Yes                         | ❌ No (unless replay > 0)                              |
| Default replay      | 1 (latest)                    | Configurable (0 by default)                           |
| Best for            | Persistent UI state           | One-time events / actions                             |
| Collects new values | Always receives latest state  | Receives only emitted events (based on replay buffer) |
| Example             | Form data, counter, selection | Toast, snackbar, navigation, alerts                   |

---

## ✅ Summary

* **Use StateFlow** when exposing **UI state** that represents the current state of the screen.
* **Use SharedFlow** for **events or actions** that should be **handled once** by the UI.
* Both integrate seamlessly with Compose via `collectAsState()` (StateFlow) or `LaunchedEffect` (SharedFlow).

---
