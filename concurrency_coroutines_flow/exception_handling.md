
---

# ⚠️ Exception Handling in Kotlin Coroutines

Handling exceptions in coroutines requires understanding their **structured concurrency**, the **coroutine scope**, and the **type of coroutine builder** used. Kotlin provides multiple mechanisms: `try-catch`, `CoroutineExceptionHandler`, and **supervision**.

---

## 1️⃣ Using `try-catch` in Coroutines

* The simplest way to catch exceptions is **inside the coroutine** using `try-catch`.
* Works for **suspend functions** and **child coroutines** launched with `launch`.

```kotlin
viewModelScope.launch {
    try {
        val user = api.getUser() // suspend function
        updateUI(user)
    } catch (e: IOException) {
        showError("Network error: ${e.message}")
    }
}
```

**Notes:**

* `launch` propagates exceptions **immediately to the parent scope** if not caught.
* `async` exceptions are **deferred** and only thrown when `await()` is called.

```kotlin
val deferred = async { api.getUser() }
try {
    val user = deferred.await() // exception thrown here
} catch (e: IOException) {
    showError("Network error: ${e.message}")
}
```

---

## 2️⃣ CoroutineExceptionHandler

* A **global handler** for uncaught exceptions in a coroutine.
* Only works with **`launch` coroutines**, not `async` (since `async` exceptions are deferred until `await`).

```kotlin
val handler = CoroutineExceptionHandler { _, exception ->
    Log.e("CoroutineException", "Caught $exception")
}

viewModelScope.launch(handler) {
    throw IOException("Network failed") // caught by handler
}
```

**Notes:**

* Can be combined with structured concurrency to handle errors globally.
* Useful for logging or reporting exceptions from background coroutines.

---

## 3️⃣ Supervision: SupervisorJob & SupervisorScope

* **Parent-child relationships** by default propagate exceptions from child to parent, cancelling all children.
* **SupervisorJob** / `supervisorScope` allows **children to fail independently** without cancelling siblings.

```kotlin
val supervisor = SupervisorJob()
val scope = CoroutineScope(Dispatchers.IO + supervisor)

scope.launch {
    launch { throw IOException("Child 1 fails") }  // does NOT cancel sibling
    launch { delay(1000); println("Child 2 completes") }  // continues
}
```

* `supervisorScope { ... }` works similarly inside structured concurrency:

```kotlin
viewModelScope.launch {
    supervisorScope {
        launch { throw IOException("Fail") }  // only this child fails
        launch { println("Other child continues") } 
    }
}
```

**Use Case:**

* Ideal for **concurrent tasks where failure of one should not cancel others** (e.g., loading multiple independent network requests).

---

## 4️⃣ Summary of Exception Handling Strategies

| Strategy                            | Use Case                     | Notes                                             |
| ----------------------------------- | ---------------------------- | ------------------------------------------------- |
| **try-catch**                       | Local, per coroutine         | Works for both `launch` and `async`               |
| **CoroutineExceptionHandler**       | Global, uncaught exceptions  | Only works with `launch`                          |
| **SupervisorJob / supervisorScope** | Independent child coroutines | Prevents one failing child from cancelling others |

---

## 5️⃣ Best Practices

1. Use **try-catch** for network or DB operations inside coroutines.
2. Use **`async` + `await()`** carefully; handle exceptions when awaiting.
3. Use **SupervisorJob** for **independent tasks** that must not cancel each other.
4. Combine **CoroutineExceptionHandler** for **global logging** or crash reporting.
5. Avoid **GlobalScope** for error handling; prefer **lifecycle-aware scopes** (`viewModelScope`, `lifecycleScope`).

---

