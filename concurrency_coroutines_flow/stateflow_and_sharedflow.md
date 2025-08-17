
---

# 🔹 StateFlow and SharedFlow in Kotlin

Kotlin **StateFlow** and **SharedFlow** are **hot flows** designed for **state management** and **event broadcasting** in reactive applications, especially with **Jetpack Compose** and **ViewModel**.

---

## 1️⃣ StateFlow

**StateFlow** is a **hot, observable state holder** that always has a **current value**.

### Key Characteristics:

| Feature                      | Description                                                                                    |
| ---------------------------- | ---------------------------------------------------------------------------------------------- |
| **Initial value**            | Must be provided when creating a `MutableStateFlow`.                                           |
| **Distinct values**          | Emits values only if the new value is **different from the current value**.                    |
| **Hot flow**                 | Continuously holds the **latest state**; new collectors immediately receive the current value. |
| **Thread-safe**              | Can be updated from multiple coroutines safely.                                                |
| **Integration with Compose** | Works seamlessly with `collectAsState()` for UI updates.                                       |

### Example:

```kotlin
val _state = MutableStateFlow(0)      // initial value
val state: StateFlow<Int> = _state

// Update state
_state.value = 1

// Collect state
state.collect { value ->
    println("Current value: $value")
}
```

* Each new collector receives the **latest value** immediately.
* Helps model **UI state** (loading, success, error) in a **single source of truth** manner.

---

## 2️⃣ SharedFlow

**SharedFlow** is a **hot flow** that can **broadcast values to multiple collectors**. Unlike StateFlow, it **does not require an initial value** and is ideal for **events**.

### Key Characteristics:

| Feature                       | Description                                                                               |
| ----------------------------- | ----------------------------------------------------------------------------------------- |
| **No initial value required** | Can emit any number of events.                                                            |
| **Multiple collectors**       | Every subscriber receives emitted values depending on configuration.                      |
| **Replay**                    | Number of past values to replay for new subscribers (`replay = n`).                       |
| **Buffer capacity**           | Defines how many uncollected emissions to store (`extraBufferCapacity`).                  |
| **One-time events**           | Useful for navigation events, toasts, or signals that shouldn’t be persisted in UI state. |

### Example:

```kotlin
val _events = MutableSharedFlow<String>(
    replay = 0,                 // no replay
    extraBufferCapacity = 1      // allow buffering
)
val events: SharedFlow<String> = _events

// Emit an event
_events.emit("ShowToast")

// Collect events
events.collect { event ->
    println("Received event: $event")
}
```

* `replay = 0`: new collectors do not get old events.
* `replay = 1` or more: new collectors receive the **last n emitted events**.
* Supports **broadcasting to multiple subscribers simultaneously**.

---

## 3️⃣ Use Cases

| Flow Type      | Use Case                                                                                                    |
| -------------- | ----------------------------------------------------------------------------------------------------------- |
| **StateFlow**  | Modeling **UI state** like form values, selected item, or loading status. Always reflects the latest state. |
| **SharedFlow** | Handling **one-time events** like navigation, showing a snackbar, or sending signals across components.     |

---

## 4️⃣ Summary

* **StateFlow**

  * Holds a **single, latest value**.
  * Always emits **distinct values**.
  * Ideal for **stateful UI**.
* **SharedFlow**

  * Broadcasts **events** to multiple subscribers.
  * Configurable **replay** and **buffer**.
  * Ideal for **one-time events and signals**.
* Both are **hot flows**, coroutine-native, and integrate seamlessly with modern Android architectures and Jetpack Compose.

---
