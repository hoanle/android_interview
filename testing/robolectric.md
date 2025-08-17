
---

# 🤖 Robolectric in Android Testing

**Robolectric** is a **unit testing framework for Android** that allows you to run **Android framework code on the JVM** instead of a real device or emulator. It simulates Android behavior, enabling **fast and reliable tests** without needing an instrumented environment.

---

## 1️⃣ Purpose of Robolectric

* **Run Android code locally on JVM:** Test Activities, Fragments, Views, and other framework components without an emulator.
* **Fast execution:** Avoid the overhead of launching emulators or devices.
* **Isolated testing:** Allows testing Android code in **unit-test style** rather than full instrumented tests.
* **Integration with existing tools:** Works with **JUnit4/5, Mockito, and other testing libraries**.

---

## 2️⃣ When to Use Robolectric

| Use Case                          | Description                                                          |
| --------------------------------- | -------------------------------------------------------------------- |
| **Testing UI logic**              | Verify Activity or Fragment behavior without an emulator.            |
| **Testing lifecycle methods**     | Test `onCreate`, `onResume`, or `onPause` interactions.              |
| **Testing Views and Layouts**     | Check visibility, text, and interaction of XML views.                |
| **Repository or Service Testing** | Test Android-dependent services like SharedPreferences or Resources. |
| **Faster Feedback**               | When instrumented tests are too slow for CI or local development.    |

> Note: Robolectric is **not a replacement for instrumented UI tests**. It cannot fully replicate real device behaviors like hardware sensors, multi-window mode, or Compose-specific rendering.

---

## 3️⃣ Example: Testing an Activity

### Setup in `build.gradle`

```gradle
testImplementation "org.robolectric:robolectric:4.10.3"
testImplementation "androidx.test:core:1.5.0"
```

### Test Example

```kotlin
@RunWith(RobolectricTestRunner::class)
class MainActivityTest {

    private lateinit var activity: MainActivity

    @Before
    fun setup() {
        activity = Robolectric.buildActivity(MainActivity::class.java)
            .create()
            .start()
            .resume()
            .get()
    }

    @Test
    fun `textView displays correct text`() {
        val textView = activity.findViewById<TextView>(R.id.textView)
        assertEquals("Hello World!", textView.text.toString())
    }

    @Test
    fun `button click updates text`() {
        val button = activity.findViewById<Button>(R.id.button)
        val textView = activity.findViewById<TextView>(R.id.textView)

        button.performClick()
        assertEquals("Button Clicked!", textView.text.toString())
    }
}
```

### Key Points

* Uses **`RobolectricTestRunner`** to run tests on JVM.
* Supports **Activity lifecycle simulation** (`create`, `start`, `resume`, etc.).
* Allows testing **UI interactions** without launching a device.

---

## 4️⃣ Benefits of Robolectric

* **Fast local tests** without needing devices.
* **Access to Android APIs** like Resources, Context, and Views.
* **Easy integration** with JUnit, Mockito, and other JVM-based testing tools.
* Helps catch bugs **earlier in CI/CD** pipelines.

---

## 5️⃣ Limitations

* **Not 100% accurate** for all Android behaviors; some device-specific issues may be missed.
* Limited support for **advanced hardware features** (camera, sensors, multi-window).
* For **Jetpack Compose**, Robolectric has limited support compared to ComposeTestRule.

---

## 6️⃣ Summary

* **Robolectric** is a JVM-based Android testing framework for **unit-testing Activities, Fragments, and Views**.
* Useful when you need **fast feedback** on Android code without launching emulators.
* Complementary to **instrumented UI tests**; use Robolectric for **logic and lifecycle tests**, and Espresso/ComposeTestRule for **full UI integration tests**.

---
