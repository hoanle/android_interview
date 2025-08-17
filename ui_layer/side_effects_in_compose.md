
---

# ⚡ Side-Effect Handlers in Jetpack Compose

## 📌 What are Side Effects?

In Jetpack Compose, **side effects** are operations that go **beyond rendering UI** — things like launching coroutines, logging, making network calls, or subscribing/unsubscribing to external data sources.

Because composables can **recompose many times**, Compose provides **side-effect APIs** to safely perform these operations without duplicating work or causing leaks.

---

## 🔑 Main Side-Effect Handlers

### 1. **`LaunchedEffect`**

* Runs a coroutine tied to the **composition lifecycle**.
* Cancels automatically when the key changes or when the composable leaves the composition.

```kotlin
@Composable
fun UserProfile(userId: String) {
    var user by remember { mutableStateOf<User?>(null) }

    LaunchedEffect(userId) {
        user = fetchUser(userId) // suspending network call
    }

    Text(user?.name ?: "Loading...")
}
```

✅ Use when:

* You need to **launch coroutines** (network calls, animations, data loading) when entering the composition or when a key changes.

---

### 2. **`rememberCoroutineScope`**

* Provides a **stable `CoroutineScope`** tied to the composable’s lifecycle.
* Unlike `LaunchedEffect`, it doesn’t auto-launch — you decide when to start coroutines (e.g., on button clicks).

```kotlin
@Composable
fun DownloadButton() {
    val scope = rememberCoroutineScope()

    Button(onClick = {
        scope.launch {
            downloadFile()
        }
    }) {
        Text("Download")
    }
}
```

✅ Use when:

* You need a **coroutine scope on-demand** for user interactions.

---

### 3. **`DisposableEffect`**

* Runs side effects that need **cleanup** when the composable leaves composition or when keys change.

```kotlin
@Composable
fun LocationTracker() {
    DisposableEffect(Unit) {
        startLocationUpdates()

        onDispose {
            stopLocationUpdates() // cleanup
        }
    }
}
```

✅ Use when:

* You need to **register/unregister listeners** (e.g., sensors, callbacks, receivers).
* You want deterministic cleanup logic.

---

### 4. **`SideEffect`**

* Runs a block of code **after every successful recomposition**.
* Always executes on the **main thread**.

```kotlin
@Composable
fun AnalyticsScreen(screenName: String) {
    SideEffect {
        logScreenView(screenName) // logs every recomposition
    }

    Text("Welcome to $screenName")
}
```

✅ Use when:

* You need to **synchronize Compose state with external systems** (logging, analytics, non-Compose states).

---

### 5. **`produceState`**

* Bridges **suspending functions / async data sources** into Compose state.
* Creates a `State<T>` that updates automatically.

```kotlin
@Composable
fun UserProfile(userId: String): State<User?> {
    return produceState<User?>(initialValue = null, userId) {
        value = fetchUser(userId) // suspending function
    }
}
```

✅ Use when:

* You want to **expose async data as Compose `State`** without managing `mutableStateOf` + `LaunchedEffect`.
* Great for integrating APIs, databases, or Flows.

---

## 📊 Summary

| Handler                    | Purpose                                                        | Best Use Case                              |
| -------------------------- | -------------------------------------------------------------- | ------------------------------------------ |
| **LaunchedEffect**         | Launch coroutine when entering composition or when key changes | Network requests, animations               |
| **rememberCoroutineScope** | Provides a coroutine scope for event-driven actions            | Run async work on button click             |
| **DisposableEffect**       | Run effect with cleanup when composable leaves                 | Register/unregister listeners, resources   |
| **SideEffect**             | Run code after every recomposition                             | Logging, analytics, non-Compose state sync |
| **produceState**           | Convert async data into `State<T>`                             | Expose suspend data as state               |

---

✅ **Key Idea:**

* Use **`LaunchedEffect`** for one-off jobs on enter/change.
* Use **`rememberCoroutineScope`** for manual coroutine launching.
* Use **`DisposableEffect`** when cleanup is required.
* Use **`SideEffect`** to notify external systems of recompositions.
* Use **`produceState`** to turn async data into observable state.

---