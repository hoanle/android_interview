
---

# 🌀 Kotlin Features for Android Development

---

## 1️⃣ Null Safety in Kotlin

Kotlin provides **built-in null safety** to prevent **NullPointerExceptions**.

### Key Operators

| Operator | Description                                  | Example                                                      |
| -------- | -------------------------------------------- | ------------------------------------------------------------ |
| `?`      | Nullable type declaration                    | `val name: String? = null`                                   |
| `!!`     | Assert non-null (throws NPE if null)         | `val len = name!!.length`                                    |
| `?.`     | Safe call operator (calls only if non-null)  | `val len = name?.length`                                     |
| `?:`     | Elvis operator (default value if null)       | `val len = name?.length ?: 0`                                |
| `let`    | Executes a block if value is non-null        | `name?.let { println(it) }`                                  |
| `run`    | Executes a block and returns last expression | `val result = name?.run { length * 2 }`                      |
| `apply`  | Runs block on object, returns object         | `val builder = StringBuilder().apply { append("Hello") }`    |
| `also`   | Runs block on object, returns object         | `val builder = StringBuilder().also { println("Building") }` |

**Example:**

```kotlin
val name: String? = null
val length = name?.length ?: 0   // Safe default value
name?.let { println("Name is $it") }  // Executes only if name is non-null
```

---

## 2️⃣ Extension Functions

**Extension functions** let you **add new functionality to existing classes** without inheritance.

**Example:**

```kotlin
fun String.isEmail(): Boolean {
    return this.contains("@") && this.contains(".")
}

val email = "test@example.com"
println(email.isEmail())  // true
```

**Use Case:** Simplifies utility functions for classes like `String`, `View`, or `RecyclerView`.

---

## 3️⃣ Sealed Classes

**Sealed classes** represent **restricted class hierarchies**, useful for **type-safe handling of states**.

**Example: Representing UI states**

```kotlin
sealed class UiState {
    object Loading : UiState()
    data class Success(val data: List<String>) : UiState()
    data class Error(val message: String) : UiState()
}

fun handleState(state: UiState) {
    when(state) {
        is UiState.Loading -> println("Loading")
        is UiState.Success -> println("Data size: ${state.data.size}")
        is UiState.Error -> println("Error: ${state.message}")
    }
}
```

**Benefit:** Compiler ensures **exhaustive `when` statements**.

---

## 4️⃣ Data Classes

**Data classes** automatically provide:

* `equals()`, `hashCode()`, `toString()`, and `copy()` functions.
* Ideal for **holding immutable data**.

**Example:**

```kotlin
data class User(val id: Int, val name: String)

val user1 = User(1, "Alice")
val user2 = user1.copy(name = "Bob")

println(user1)  // User(id=1, name=Alice)
println(user2)  // User(id=1, name=Bob)
```

**Use Case:** Representing API responses, database entities, or UI models.

---

## 5️⃣ Higher-Order Functions and Lambdas

**Higher-order functions** accept **functions as parameters** or **return functions**.

**Example:**

```kotlin
fun calculate(a: Int, b: Int, operation: (Int, Int) -> Int): Int {
    return operation(a, b)
}

val sum = calculate(3, 5) { x, y -> x + y }
println(sum)  // 8
```

**Benefit:** Enables **functional programming patterns** and **reusable logic**.

---

## 6️⃣ Scope Functions

Kotlin has five **scope functions** to **simplify object configuration and null handling**:

| Function | Returns       | Typical Use Case                                        |
| -------- | ------------- | ------------------------------------------------------- |
| `let`    | Lambda result | Execute code on non-null objects                        |
| `run`    | Lambda result | Object configuration or return value                    |
| `with`   | Lambda result | Perform operations on object without extension call     |
| `apply`  | Object itself | Initialize or configure an object                       |
| `also`   | Object itself | Perform additional actions like logging or side-effects |

**Examples:**

```kotlin
// let
val email: String? = "test@example.com"
email?.let { println("Email is $it") }

// run
val length = email?.run { length * 2 }

// apply
val builder = StringBuilder().apply { append("Hello") }

// also
builder.also { println("Builder contains: $it") }

// with
val user = User(1, "Alice")
val description = with(user) { "User: $name, ID: $id" }
println(description)
```

**Benefit:** Reduces boilerplate, improves readability, and handles nullability elegantly.

---

## ✅ Summary

* **Null Safety:** `?`, `!!`, `?.`, `?:`, `let`, `run`, `apply`, `also` prevent crashes.
* **Extension Functions:** Add reusable behavior to existing classes.
* **Sealed Classes:** Type-safe representation of state or hierarchy.
* **Data Classes:** Auto-generated `equals`, `hashCode`, `toString`, and `copy`.
* **Higher-Order Functions:** Pass functions as parameters for flexible logic.
* **Scope Functions:** Simplify object configuration, initialization, and null handling.

---
