
---

# 🔄 Unidirectional Data Flow (UDF) in Jetpack Compose + ViewModel

## 📌 What is UDF?

**Unidirectional Data Flow (UDF)** is an architectural pattern where:

1. **State flows down** from a single source of truth (usually a `ViewModel`).
2. **Events flow up** from the UI (composables) to the state owner (ViewModel).

This ensures a **predictable, testable, and maintainable** data flow in Compose applications.

---

## 🧩 UDF in Compose + ViewModel

### 1. **ViewModel → Composable (State down)**

* The **ViewModel exposes state** (e.g., via `State<T>`, `StateFlow`, or `LiveData`).
* The **Composable collects/observes** this state and re-renders automatically when it changes.

Example:

```kotlin
class CounterViewModel : ViewModel() {
    private val _uiState = MutableStateFlow(0)
    val uiState: StateFlow<Int> = _uiState

    fun increment() {
        _uiState.value++
    }
}
```

```kotlin
@Composable
fun CounterScreen(viewModel: CounterViewModel = viewModel()) {
    val count by viewModel.uiState.collectAsState()

    Column {
        Text("Count: $count")
        Button(onClick = { viewModel.increment() }) {
            Text("Increment")
        }
    }
}
```

* `uiState` is exposed as **immutable**.
* `collectAsState()` automatically subscribes and triggers recomposition.

---

### 2. **Composable → ViewModel (Events up)**

* User actions are captured in the **Composable**.
* They are sent to the **ViewModel** via:

  * Direct method calls (`viewModel.increment()`)
  * Or event objects (sealed classes representing actions).

Example with an **event class**:

```kotlin
sealed class CounterEvent {
    object Increment : CounterEvent()
    object Decrement : CounterEvent()
}

class CounterViewModel : ViewModel() {
    private val _uiState = MutableStateFlow(0)
    val uiState: StateFlow<Int> = _uiState

    fun onEvent(event: CounterEvent) {
        when (event) {
            CounterEvent.Increment -> _uiState.value++
            CounterEvent.Decrement -> _uiState.value--
        }
    }
}
```

```kotlin
@Composable
fun CounterScreen(viewModel: CounterViewModel = viewModel()) {
    val count by viewModel.uiState.collectAsState()

    Column {
        Text("Count: $count")
        Row {
            Button(onClick = { viewModel.onEvent(CounterEvent.Decrement) }) {
                Text("-")
            }
            Button(onClick = { viewModel.onEvent(CounterEvent.Increment) }) {
                Text("+")
            }
        }
    }
}
```

---

## 🔄 UDF Flow Summary

1. **ViewModel** holds **state** (`StateFlow`, `LiveData`, `mutableStateOf`).
2. **Composable observes** state → recomposes when it changes.
3. **UI events** (user clicks, input) are sent back to ViewModel.
4. ViewModel **updates state** → Compose recomposes.

```
[ViewModel] → State → [Composable UI]
[Composable UI] → Events → [ViewModel]
```

---

## ✅ Key Benefits of UDF

* **Predictability**: Always know where state lives.
* **Single source of truth**: No duplicate or conflicting states.
* **Reusability & Testability**: Stateless composables are easy to test and reuse.
* **Scalability**: Works well for small UIs and scales to larger apps.

---

