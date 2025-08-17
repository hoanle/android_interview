
---

# 🌊 Kotlin Flow

**Kotlin Flow** is a **cold asynchronous data stream** that emits values sequentially. It is part of **Kotlin Coroutines** and provides a **reactive programming model** for handling streams of data in a non-blocking, lifecycle-aware way.

---

## 1️⃣ Kotlin Flow vs LiveData vs RxJava Observable

| Feature                 | Kotlin Flow                 | LiveData             | RxJava Observable       |
| ----------------------- | --------------------------- | -------------------- | ----------------------- |
| **Nature**              | Cold by default             | Hot, lifecycle-aware | Can be hot or cold      |
| **Coroutines support**  | Native, suspend-friendly    | Limited              | Uses its own threading  |
| **Backpressure**        | Supports via operators      | Not built-in         | Supports via Flowable   |
| **Threading**           | Dispatchers, suspend        | Main thread only     | Schedulers              |
| **Lifecycle awareness** | No automatic awareness      | Lifecycle-aware      | Needs manual management |
| **Composition**         | Operators (`map`, `filter`) | Limited              | Rich operators          |

**Key Takeaway:** Flow is more **lightweight and coroutine-friendly** than RxJava, and more flexible than LiveData when handling streams of data.

---

## 2️⃣ Cold vs Hot Flows

### a. **Cold Flow**

* Created with `flow {}`.
* **Does not emit until collected**.
* Each collector triggers **a new execution**.

```kotlin
val coldFlow = flow {
    println("Flow started")
    emit(1)
    emit(2)
}

coldFlow.collect { println(it) }
// Output:
// Flow started
// 1
// 2
```

* Each `collect` will **restart the flow**.

---

### b. **Hot Flow**

* Emits values **regardless of collectors**.
* Examples: `StateFlow`, `SharedFlow`.
* **StateFlow:** holds **latest value**; always emits to new collectors.
* **SharedFlow:** **broadcasts values** to multiple collectors; can replay values.

```kotlin
val stateFlow = MutableStateFlow(0)
stateFlow.value = 1
stateFlow.collect { println(it) } // prints 1 (latest value)
```

**Key Difference:**

* **Cold:** lazily computed per collector.
* **Hot:** continuously emits, independent of collectors.

---

## 3️⃣ Flow Operators

### a. Terminal Operators

These **consume the flow**.

| Operator  | Description                               |
| --------- | ----------------------------------------- |
| `collect` | Collects all emitted values               |
| `first`   | Returns the first emitted value           |
| `toList`  | Collects all values into a list           |
| `single`  | Collects a single value or throws if more |

**Example:**

```kotlin
val list = flowOf(1, 2, 3).toList() // [1,2,3]
val first = flowOf(1, 2, 3).first() // 1
```

---

### b. Intermediate Operators

These **transform or manipulate the flow** and return a new flow.

| Operator               | Description                                     |
| ---------------------- | ----------------------------------------------- |
| `map`                  | Transform emitted values                        |
| `filter`               | Emit values matching a condition                |
| `transform`            | Emit multiple values per emission, custom logic |
| `onEach`               | Side effects per emission                       |
| `distinctUntilChanged` | Suppress consecutive duplicates                 |
| `take(n)`              | Take only first n values                        |

**Example:**

```kotlin
flowOf(1, 2, 3, 4)
    .filter { it % 2 == 0 }
    .map { it * 2 }
    .onEach { println("Emitting $it") }
    .collect()
```

**Output:**

```
Emitting 4
Emitting 8
```

---

## 4️⃣ Advantages of Flow

1. **Cold by default** → efficient resource usage.
2. **Coroutines integration** → non-blocking, sequential style.
3. **Rich operators** → map, filter, combine, transform, zip, flatMap.
4. **Lifecycle-agnostic** → can be used in any layer (UI, Domain, Data).
5. **Hot flows** → StateFlow/SharedFlow for **state management and event broadcasting**.

---

## 5️⃣ Summary

* **Flow:** cold asynchronous stream, coroutine-native, composable.
* **Cold Flow:** created with `flow {}`, executes per collector.
* **Hot Flow:** StateFlow / SharedFlow, continuously emits, retains state.
* **Operators:**

  * **Terminal:** `collect`, `first`, `toList`
  * **Intermediate:** `map`, `filter`, `transform`, `onEach`
* Ideal for **reactive, non-blocking, composable data streams** in modern Android apps.

---

