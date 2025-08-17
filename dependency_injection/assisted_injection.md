
---

# 🤝 Assisted Injection in Hilt

**Assisted Injection** is a technique used in **dependency injection** when a class requires **both Hilt-provided dependencies** and **runtime parameters** that are only known at instantiation.

---

## 1️⃣ Why Assisted Injection?

* Some objects (like **ViewModels with runtime arguments**, or classes with dynamic IDs) cannot have all dependencies injected automatically because **Hilt cannot know runtime values ahead of time**.
* **Assisted Injection** solves this problem by letting Hilt **inject some dependencies**, while the caller provides the remaining parameters.

---

## 2️⃣ Core Annotations

| Annotation           | Purpose                                                                                                       |
| -------------------- | ------------------------------------------------------------------------------------------------------------- |
| **@AssistedInject**  | Marks a constructor where Hilt will inject some dependencies and accept **runtime parameters** for the rest.  |
| **@Assisted**        | Marks the parameters that **must be provided at runtime** (not injected by Hilt).                             |
| **@AssistedFactory** | Defines a **factory interface** to create instances of the assisted class. Hilt generates the implementation. |

---

## 3️⃣ Example: Assisted Injection in a Class

Suppose we have a `UserViewModel` that requires a repository (injected) and a user ID (runtime):

```kotlin
class UserRepository @Inject constructor() {
    fun getUser(userId: String) = "User $userId"
}
```

### Step 1: Annotate constructor with `@AssistedInject`

```kotlin
class UserViewModel @AssistedInject constructor(
    private val repository: UserRepository,       // Hilt-provided
    @Assisted private val userId: String          // runtime-provided
) {
    fun getUserName() = repository.getUser(userId)
}
```

### Step 2: Create an Assisted Factory

```kotlin
@AssistedFactory
interface UserViewModelFactory {
    fun create(userId: String): UserViewModel
}
```

### Step 3: Inject the Factory and create instances

```kotlin
@AndroidEntryPoint
class MainActivity : AppCompatActivity() {

    @Inject
    lateinit var factory: UserViewModelFactory

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)

        val userViewModel = factory.create(userId = "123")
        println(userViewModel.getUserName()) // prints "User 123"
    }
}
```

---

## 4️⃣ Key Points

* Hilt **injects dependencies** (like `UserRepository`) automatically.
* Parameters marked with `@Assisted` are **provided at runtime**.
* `@AssistedFactory` allows you to **create instances with dynamic parameters** without manually writing factories.
* Often used for **ViewModels with runtime arguments**, dialog controllers, or any class that needs a mix of injected and runtime values.

---

## 5️⃣ Benefits of Assisted Injection

| Benefit                                    | Description                                                               |
| ------------------------------------------ | ------------------------------------------------------------------------- |
| **Simplifies runtime parameter injection** | No need for manual factory implementation.                                |
| **Combines DI and dynamic creation**       | Use Hilt for dependencies and provide runtime values seamlessly.          |
| **Reduces boilerplate**                    | Hilt generates the factory automatically.                                 |
| **Lifecycle-aware injection**              | Can be used with components like ViewModels while respecting Hilt scopes. |

---

## 6️⃣ Summary

* **Assisted Injection** = partial Hilt injection + runtime parameters.
* Use **`@AssistedInject`** for constructors, **`@Assisted`** for runtime parameters, and **`@AssistedFactory`** for generating instances.
* Common use cases: **ViewModels with runtime arguments, controllers, or dynamically-configured objects**.

---

