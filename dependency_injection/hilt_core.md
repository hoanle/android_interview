
---

# 🧩 Core Hilt Annotations and Scopes

**Hilt** is a dependency injection (DI) library built on Dagger that simplifies DI in Android apps. It uses **annotations** to define components, modules, and injection points, while managing the **lifecycle of dependencies** automatically.

---

## 1️⃣ Core Hilt Annotations

| Annotation             | Purpose                                                                                                                              | Example                                                                        |
| ---------------------- | ------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------ |
| **@HiltAndroidApp**    | Marks the Application class to trigger **Hilt code generation** and set up the DI container.                                         | `kotlin @HiltAndroidApp class MyApp : Application()`                           |
| **@AndroidEntryPoint** | Marks Android classes (Activity, Fragment, View, Service, BroadcastReceiver) where **dependencies will be injected**.                | `kotlin @AndroidEntryPoint class MainActivity : AppCompatActivity()`           |
| **@Inject**            | Requests a dependency to be provided by Hilt. Can be used on **constructors, fields, or methods**.                                   | `kotlin class UserRepository @Inject constructor(private val api: ApiService)` |
| **@Module**            | Defines a **Hilt module** which provides dependencies that Hilt cannot create automatically (e.g., interfaces, third-party classes). | `kotlin @Module @InstallIn(SingletonComponent::class) object NetworkModule`    |
| **@Provides**          | Tells Hilt **how to create a dependency** within a Module.                                                                           | `kotlin @Provides fun provideApiService(): ApiService = ApiService()`          |
| **@InstallIn**         | Specifies **which Hilt component the module belongs to**, defining the **lifecycle scope** of its dependencies.                      | `kotlin @InstallIn(SingletonComponent::class)`                                 |

---

## 2️⃣ Hilt Component Scopes

Hilt provides **predefined scopes** to manage the **lifecycle of dependencies**. Each scope is tied to a Hilt **component**.

| Scope                       | Lifecycle                                     | Use Case                                                                   |
| --------------------------- | --------------------------------------------- | -------------------------------------------------------------------------- |
| **@Singleton**              | Entire application                            | Shared dependencies across the app (e.g., Retrofit, Database)              |
| **@ActivityRetainedScoped** | Survives configuration changes in an Activity | Dependencies needed by **ViewModels** that should persist across rotations |
| **@ViewModelScoped**        | Lifetime of a ViewModel                       | Dependencies exclusive to a single ViewModel instance                      |
| **@ActivityScoped**         | Lifetime of an Activity                       | Dependencies tied to an Activity but reset on destruction                  |
| **@FragmentScoped**         | Lifetime of a Fragment                        | Dependencies tied to a Fragment                                            |

---

### 3️⃣ Example: Hilt Module with Scopes

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {

    @Singleton
    @Provides
    fun provideApiService(): ApiService = ApiService()
}

@HiltAndroidApp
class MyApp : Application()

@AndroidEntryPoint
class MainActivity : AppCompatActivity() {

    @Inject
    lateinit var apiService: ApiService

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        println(apiService) // injected automatically
    }
}
```

* `@Singleton` ensures only **one instance** of `ApiService` exists for the entire app.
* Dependencies are **automatically injected** into the annotated Activity via `@AndroidEntryPoint`.

---

### 4️⃣ Lifecycle Management

* Hilt **maps component scopes to Android lifecycles**.
* When a component is destroyed, **all dependencies with that scope are also cleared**.
* Example:

  * **@Singleton** → survives as long as the application exists.
  * **@ViewModelScoped** → tied to ViewModel lifecycle, destroyed when ViewModel is cleared.
  * **@ActivityScoped** → destroyed when Activity is destroyed.

---

## 5️⃣ Summary

* **Hilt simplifies DI** with annotations for injection points, modules, and lifecycle-aware scopes.
* **Annotations:**

  * `@HiltAndroidApp` → initializes Hilt
  * `@AndroidEntryPoint` → marks Android components for injection
  * `@Inject` → injects dependencies
  * `@Module` + `@Provides` → defines how to create dependencies
  * `@InstallIn` → binds module to a component
* **Scopes** control **dependency lifetime**:

  * `@Singleton` → app-wide
  * `@ActivityRetainedScoped` → survives config changes
  * `@ViewModelScoped` → tied to ViewModel
  * `@ActivityScoped` → tied to Activity
  * `@FragmentScoped` → tied to Fragment

---
