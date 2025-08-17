
---

# 🔄 Kotlin Flow Operators: flatMap and Combining Flows

Kotlin Flow provides **powerful operators** to **transform, merge, and combine streams of data**. Understanding operators like `flatMapConcat`, `flatMapMerge`, `flatMapLatest`, `combine`, and `zip` is essential for building reactive, asynchronous pipelines.

---

## 1️⃣ flatMap Operators

**`flatMap` operators** are used when a **Flow emits another Flow** (nested flows). They **flatten** the stream into a single Flow.

| Operator          | Behavior                                                                                                                |
| ----------------- | ----------------------------------------------------------------------------------------------------------------------- |
| **flatMapConcat** | Concatenates inner flows sequentially. **Waits for each inner flow to complete** before starting the next.              |
| **flatMapMerge**  | Merges inner flows concurrently. Emissions from all inner flows are interleaved.                                        |
| **flatMapLatest** | Cancels the **previous inner flow** if a new value is emitted by the outer flow. Only keeps the **latest active flow**. |

---

### a. Example: flatMapConcat

```kotlin
flowOf(1, 2, 3)
    .flatMapConcat { number ->
        flow {
            emit("$number-A")
            emit("$number-B")
        }
    }
    .collect { println(it) }
```

**Output (sequential):**

```
1-A
1-B
2-A
2-B
3-A
3-B
```

* Each inner flow completes before starting the next.

---

### b. Example: flatMapMerge

```kotlin
flowOf(1, 2, 3)
    .flatMapMerge { number ->
        flow {
            emit("$number-A")
            delay(100)
            emit("$number-B")
        }
    }
    .collect { println(it) }
```

**Output (interleaved):**

```
1-A
2-A
3-A
1-B
2-B
3-B
```

* Inner flows **run concurrently**, so emissions may interleave.

---

### c. Example: flatMapLatest

```kotlin
flowOf(1, 2, 3)
    .flatMapLatest { number ->
        flow {
            emit("$number-A")
            delay(100)
            emit("$number-B")
        }
    }
    .collect { println(it) }
```

**Output (only latest active flow):**

```
1-A
2-A
3-A
3-B
```

* Each time a **new outer emission arrives**, the previous inner flow is **cancelled**.

**Use Case for flatMapLatest:**

* Handling **search-as-you-type** in a search bar:

  * Each query triggers a network request (inner flow).
  * Cancel previous request if the user types a new query.

---

## 2️⃣ Combining Flows: combine vs zip

### a. combine

* Combines **latest values** from multiple flows.
* Emits a value whenever **any of the flows emit**.

```kotlin
val flow1 = flowOf(1, 2)
val flow2 = flowOf("A", "B")

flow1.combine(flow2) { a, b -> "$a$b" }
    .collect { println(it) }
```

**Output:**

```
1A
2A
2B
```

* Useful for **state synchronization**, e.g., combining user input flows from multiple fields.

---

### b. zip

* Combines values **pairwise**, **matching emissions by index**.
* Emits only when **both flows emit a value**.

```kotlin
flow1.zip(flow2) { a, b -> "$a$b" }
    .collect { println(it) }
```

**Output:**

```
1A
2B
```

* Useful for **pairing related streams** like coordinates `(xFlow, yFlow)` or responses with IDs.

---

## 3️⃣ Summary Table

| Operator      | Behavior                       | Use Case                                    |
| ------------- | ------------------------------ | ------------------------------------------- |
| flatMapConcat | Sequential flattening          | Sequential API calls                        |
| flatMapMerge  | Concurrent flattening          | Parallel API calls                          |
| flatMapLatest | Cancels previous, keeps latest | Search-as-you-type, auto-complete           |
| combine       | Combine latest values          | Merge state from multiple sources           |
| zip           | Pair emissions by index        | Combine two streams with 1:1 correspondence |

---

## ✅ Key Takeaways

* **flatMap** operators handle **nested flows** differently based on concurrency and cancellation rules.
* **flatMapLatest** is ideal for **UI-driven events** where only the latest value matters.
* **combine** and **zip** are essential for **merging multiple flows**, each serving **different semantics**.

---
