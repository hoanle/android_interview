
---

# 🧩 Dependency Injection (DI) in Android

**Dependency Injection (DI)** is a **software design pattern** in which a class receives its **dependencies from an external source** rather than creating them internally. This promotes **loose coupling**, **modularity**, and **testability** in your application.

---

## 1️⃣ Core Concept

* **Dependency**: Any object a class requires to function.
* **Injection**: Providing the required dependencies from the outside (constructor, setter, or method parameter).
* **Goal**: Classes should **not instantiate their own dependencies**. They should only use what is provided to them.

### Example Without DI:

```kotlin
class UserRepository {
    private val apiService = ApiService() // tightly coupled
}
```

### Example With DI:

```kotlin
class UserRepository(private val apiService: ApiService) {
    // ApiService is injected from outside
}
```

* `UserRepository` no longer creates `ApiService` itself.
* You can easily replace `ApiService` with a **mock** during testing.

---

## 2️⃣ Benefits of Dependency Injection

| Benefit             | Description                                                                           |
| ------------------- | ------------------------------------------------------------------------------------- |
| **Testability**     | You can inject **mock dependencies** to unit test classes in isolation.               |
| **Decoupling**      | Classes don’t depend on concrete implementations; only on interfaces or abstractions. |
| **Reusability**     | Classes become **more modular** and can be reused in different contexts.              |
| **Maintainability** | Changing a dependency’s implementation doesn’t require changes in dependent classes.  |
| **Scalability**     | Large applications become easier to manage as dependencies are centralized.           |

---

## 3️⃣ Types of Dependency Injection

1. **Constructor Injection (Recommended)**

   * Dependencies are provided via the class constructor.

   ```kotlin
   class UserRepository(private val apiService: ApiService)
   ```

2. **Setter Injection**

   * Dependencies are provided via a setter method.

   ```kotlin
   class UserRepository {
       lateinit var apiService: ApiService
   }
   ```

3. **Interface Injection**

   * Dependency provides an injector method that the client must implement.
   * Less common in Kotlin/Android.

---

## 4️⃣ DI in Android

* **Manual DI**

  * Manually pass dependencies through constructors or setters.
* **DI Frameworks**

  * **Hilt (built on Dagger)** → Recommended for modern Android.
  * **Dagger** → Powerful, compile-time DI.
  * **Koin** → Kotlin DSL-based runtime DI.

### Example with Hilt:

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {
    @Provides
    fun provideApiService(): ApiService = ApiService()
}

@HiltViewModel
class UserViewModel @Inject constructor(
    private val repository: UserRepository
) : ViewModel()
```

* Hilt automatically **injects `UserRepository`** into `UserViewModel`.
* Dependencies are **centralized** and **easily testable**.

---

## 5️⃣ Summary

* **Dependency Injection** = providing dependencies externally instead of creating them internally.
* **Benefits**:

  * Easier **unit testing** (mocking dependencies)
  * Better **decoupling** and modularity
  * Improved **reusability** and maintainability
  * Scalable architecture for large applications
* **Implementation** can be **manual** or via **DI frameworks** like Hilt, Dagger, or Koin.

---
