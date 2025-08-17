
---

# 🔑 Hilt Entry Points: Accessing Dependencies from Unsupported Classes

Hilt automatically injects dependencies into **Activities, Fragments, ViewModels, Services, and other Android classes**.
However, sometimes you need dependencies in **classes not directly supported by Hilt** (e.g., **custom utility classes, third-party libraries, broadcast receivers, or plain Kotlin objects**).

Hilt provides **Entry Points** to access dependencies in such cases.

---

## 1️⃣ @EntryPoint

* `@EntryPoint` defines an **interface that exposes Hilt dependencies**.
* You use it to **retrieve dependencies manually** from Hilt’s component.

### Example

Suppose we want to inject `UserRepository` into a **plain Kotlin class**:

```kotlin
@EntryPoint
@InstallIn(SingletonComponent::class)
interface UserRepositoryEntryPoint {
    fun userRepository(): UserRepository
}

class UserHelper(context: Context) {

    private val userRepository: UserRepository

    init {
        val appContext = context.applicationContext as Application
        val hiltEntryPoint = EntryPointAccessors.fromApplication(
            appContext,
            UserRepositoryEntryPoint::class.java
        )
        userRepository = hiltEntryPoint.userRepository()
    }

    fun printUser(id: String) {
        println(userRepository.getUser(id))
    }
}
```

**Key Points:**

* `EntryPointAccessors.fromApplication()` fetches the **Hilt component**.
* Dependencies can now be accessed in classes **not directly supported by Hilt**.

---

## 2️⃣ @InstallIn

* `@InstallIn` specifies **which Hilt component the entry point belongs to**.
* This determines the **scope and lifetime** of the dependencies.

### Example Components:

| Component          | Lifetime / Scope            |
| ------------------ | --------------------------- |
| SingletonComponent | Application-wide singleton  |
| ActivityComponent  | Tied to Activity lifecycle  |
| ViewModelComponent | Tied to ViewModel lifecycle |
| FragmentComponent  | Tied to Fragment lifecycle  |

* Choosing the right component ensures **dependencies respect the intended lifecycle**.

---

## 3️⃣ Use Cases for Entry Points

* Access Hilt dependencies in **classes not supported by `@AndroidEntryPoint`**:

  * Plain Kotlin classes or utility classes
  * Third-party libraries that you cannot modify
  * BroadcastReceivers, ContentProviders, or Services outside standard DI scope
* When you need **manual control** over dependency access without refactoring the class to support injection.

---

## 4️⃣ Example: Using in a BroadcastReceiver

```kotlin
class MyReceiver : BroadcastReceiver() {
    override fun onReceive(context: Context, intent: Intent) {
        val entryPoint = EntryPointAccessors.fromApplication(
            context.applicationContext,
            UserRepositoryEntryPoint::class.java
        )
        val userRepository = entryPoint.userRepository()
        println(userRepository.getUser("123"))
    }
}
```

* Hilt automatically **resolves and provides dependencies** even though BroadcastReceiver cannot use `@AndroidEntryPoint`.

---

## 5️⃣ Summary

* **`@EntryPoint`**: Exposes dependencies in classes **Hilt cannot inject directly**.
* **`@InstallIn`**: Specifies **which Hilt component the entry point belongs to**, controlling lifecycle.
* **Workflow**:

  1. Define an `@EntryPoint` interface exposing the dependency.
  2. Annotate it with `@InstallIn` for the correct component.
  3. Access dependencies manually via `EntryPointAccessors`.

**Use Case**: Access Hilt-managed dependencies in **plain Kotlin classes, utilities, or third-party classes** that cannot be annotated with `@AndroidEntryPoint`.

---
