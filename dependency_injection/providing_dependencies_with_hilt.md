
---

# 🧩 Providing Dependencies in Hilt: Interfaces and External Classes

In Hilt, sometimes you need to **inject interfaces or external classes** that you cannot annotate with `@Inject`. This is where **Hilt modules** and **`@Provides` functions** come into play.

---

## 1️⃣ Providing an Instance of an Interface

If a class depends on an interface, Hilt needs to know **which concrete implementation to use**. You provide this using a **module** and a **`@Provides` function**.

### Example

```kotlin
// 1. Define the interface
interface AnalyticsService {
    fun trackEvent(event: String)
}

// 2. Implement the interface
class FirebaseAnalyticsService @Inject constructor() : AnalyticsService {
    override fun trackEvent(event: String) {
        println("Tracking event: $event")
    }
}

// 3. Provide the interface in a Hilt Module
@Module
@InstallIn(SingletonComponent::class)
object AnalyticsModule {

    @Singleton
    @Provides
    fun provideAnalyticsService(
        firebaseAnalyticsService: FirebaseAnalyticsService
    ): AnalyticsService = firebaseAnalyticsService
}

// 4. Inject in a ViewModel
@HiltViewModel
class MainViewModel @Inject constructor(
    private val analyticsService: AnalyticsService
) : ViewModel() {
    fun logEvent() {
        analyticsService.trackEvent("button_clicked")
    }
}
```

**Key Points:**

* `AnalyticsService` is an **interface**.
* Hilt needs a **concrete implementation**.
* `@Provides` function tells Hilt how to provide the interface.

---

## 2️⃣ Providing an External Class

For classes **you cannot annotate with `@Inject`** (like Retrofit, OkHttp, or third-party libraries), Hilt modules are used to **manually construct the instance**.

### Example: Providing Retrofit

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object NetworkModule {

    @Singleton
    @Provides
    fun provideRetrofit(): Retrofit {
        return Retrofit.Builder()
            .baseUrl("https://api.example.com")
            .addConverterFactory(GsonConverterFactory.create())
            .build()
    }

    @Singleton
    @Provides
    fun provideApiService(retrofit: Retrofit): ApiService {
        return retrofit.create(ApiService::class.java)
    }
}
```

**Key Points:**

* You **cannot modify Retrofit** to add `@Inject`.
* Hilt **uses the module** to provide an instance.
* The provided dependency can then be **injected into ViewModels or Activities**.

---

## 3️⃣ Tips and Best Practices

1. **Prefer constructor injection** whenever possible.
2. **Use modules for:**

   * Interfaces with multiple implementations.
   * External libraries without `@Inject`.
3. **Use `@Singleton` or appropriate scope** to manage lifecycle.
4. **Combine multiple dependencies** in a single `@Provides` function if needed.

---

## 4️⃣ Summary

* **Interfaces** → Provide a concrete implementation via `@Module` + `@Provides`.
* **External classes** → Use Hilt modules to manually construct and provide instances.
* This approach allows Hilt to manage **lifecycle, scoping, and injection** for dependencies that cannot be injected automatically.

---
