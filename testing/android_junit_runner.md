
---

# 🏃 AndroidJUnitRunner

**AndroidJUnitRunner** is the **test runner provided by Android** to execute **instrumented tests** on real devices or emulators. It extends **JUnit’s capabilities** to work in the Android environment.

---

## 1️⃣ Role of AndroidJUnitRunner

* **Runs Instrumented Tests:** Executes tests that require access to **Android framework APIs**.
* **Provides Android Context:** Supplies a **`Context` and `Instrumentation`** object for interacting with app resources.
* **Integrates with Test Frameworks:** Supports **JUnit4**, Espresso, and Compose testing.
* **Handles Test Lifecycle:** Manages **setup, execution, and teardown** for each test on Android devices.
* **Custom Runner Support:** Allows extending its behavior with custom test runners if needed.

---

## 2️⃣ Instrumented Tests vs Local Unit Tests

| Feature                   | Local Unit Test | Instrumented Test (AndroidJUnitRunner)  |
| ------------------------- | --------------- | --------------------------------------- |
| Runs on JVM               | ✅               | ❌                                       |
| Runs on Device / Emulator | ❌               | ✅                                       |
| Access to Android APIs    | ❌               | ✅                                       |
| Speed                     | Fast            | Slower                                  |
| Tools                     | JUnit, Mockito  | Espresso, ComposeTestRule, UI Automator |

**Key Point:** Instrumented tests are **slower but necessary** for testing **UI, Room, and framework interactions**.

---

## 3️⃣ How to Use AndroidJUnitRunner

### Step 1: Configure in `build.gradle`

```gradle
android {
    defaultConfig {
        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }
}
```

### Step 2: Write an Instrumented Test

```kotlin
@RunWith(AndroidJUnit4::class)
class ExampleInstrumentedTest {

    @Test
    fun useAppContext() {
        // Context of the app under test
        val appContext = InstrumentationRegistry.getInstrumentation().targetContext
        assertEquals("com.example.myapp", appContext.packageName)
    }
}
```

### Step 3: Run the Test

* Use **Android Studio Run/Debug** configuration
* Or run via **Gradle**:

```bash
./gradlew connectedAndroidTest
```

---

## 4️⃣ Features for Testing

1. **Instrumentation Object**

```kotlin
val instrumentation = InstrumentationRegistry.getInstrumentation()
```

* Provides APIs for **sending key events, interacting with activities, or waiting for idle state**.

2. **UI Testing Integration**

* Works with **Espresso** and **ComposeTestRule**.
* Ensures **synchronization with UI threads** during tests.

3. **Custom Test Runner**

* You can **extend AndroidJUnitRunner** for custom behavior:

```kotlin
class MyTestRunner : AndroidJUnitRunner() {
    override fun onStart() {
        // Custom setup
        super.onStart()
    }
}
```

* Useful for **dependency injection setup**, **mocking services**, or **initializing global test state**.

---

## 5️⃣ Summary

* **AndroidJUnitRunner** is the **core test runner for instrumented tests**.
* Provides **Android context, instrumentation, and lifecycle management**.
* Supports **UI testing, Room testing, and framework integration**.
* Configured in `build.gradle` and works with **JUnit4, Espresso, and Compose testing**.
* Can be extended for **custom test setups**.

**Key Takeaway:** If you need **tests running on a real device with Android APIs**, AndroidJUnitRunner is essential.

---
