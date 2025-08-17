
---

# 🧪 Testing with Hilt in Android

Hilt simplifies dependency injection in production code, but testing requires additional setup to **replace dependencies** with **test doubles** or mocks. Hilt provides annotations and utilities to facilitate **instrumented and unit tests**.

---

## 1️⃣ @HiltAndroidTest

* **`@HiltAndroidTest`** is used to annotate **instrumented Android tests** that require Hilt injection.
* It **sets up the Hilt components for testing**, enabling injection into Activities, Fragments, or ViewModels.

### Example:

```kotlin
@HiltAndroidTest
@RunWith(AndroidJUnit4::class)
class MainActivityTest {

    @get:Rule
    var hiltRule = HiltAndroidRule(this)

    @Before
    fun init() {
        hiltRule.inject() // inject dependencies before tests
    }

    @Test
    fun testViewModelFetchUsers() {
        val scenario = launchActivity<MainActivity>()
        // test behavior that depends on injected dependencies
    }
}
```

**Key Points:**

* `HiltAndroidRule` is required for all Hilt tests.
* `hiltRule.inject()` ensures dependencies are injected before running test methods.

---

## 2️⃣ Providing Test Doubles

* You can replace **real dependencies** with **mocks, fakes, or stubs** in tests.
* This allows testing in **isolation**, avoiding network calls or database operations.

### Example: Replacing Repository with a Fake

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object TestRepositoryModule {

    @Provides
    @Singleton
    fun provideFakeRepository(): UserRepository = FakeUserRepository()
}
```

* `FakeUserRepository` implements the same interface as the real repository but returns **predefined test data**.

---

## 3️⃣ Replacing Modules in Tests

* Use **`@TestInstallIn`** to replace production modules with test modules.

### Example:

```kotlin
@Module
@TestInstallIn(
    components = [SingletonComponent::class],
    replaces = [NetworkModule::class]
)
object TestNetworkModule {

    @Provides
    @Singleton
    fun provideFakeApiService(): ApiService = FakeApiService()
}
```

* Replaces the `NetworkModule` in tests with `TestNetworkModule`.
* Allows injecting **mock or fake API services** without modifying production code.

---

## 4️⃣ Benefits of Hilt Testing

| Benefit                    | Description                                                        |
| -------------------------- | ------------------------------------------------------------------ |
| **Dependency Replacement** | Easily inject fakes or mocks for tests.                            |
| **Lifecycle Management**   | Hilt test components respect the Android lifecycle.                |
| **Reduced Boilerplate**    | No need to manually set up factories for ViewModels or Activities. |
| **Isolated Testing**       | Test components in isolation from production dependencies.         |
| **Integration Testing**    | Supports instrumented tests that require real Android components.  |

---

## 5️⃣ Tips for Hilt Testing

1. Always include **`HiltAndroidRule`** in instrumented tests.
2. Use **`@TestInstallIn`** to replace modules for testing.
3. Use **fakes or mocks** for network/database layers.
4. Combine Hilt with **Mockito, MockK, or Truth** for assertions.
5. For **unit tests of ViewModels**, consider using `@HiltViewModel` with **`HiltTestApplication`** or manually injecting dependencies without Android components.

---

## 6️⃣ Summary

* **`@HiltAndroidTest`** enables instrumented tests with Hilt DI.
* **Test doubles** (mocks/fakes) replace production dependencies for isolated testing.
* **`@TestInstallIn`** allows module replacement specifically for test environments.
* Hilt testing provides **clean, lifecycle-aware, and maintainable tests** for Activities, Fragments, and ViewModels.

---
