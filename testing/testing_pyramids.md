
---

# 🧪 Android Testing Pyramid

The **testing pyramid** is a concept that helps structure your tests for **scalability, reliability, and speed**. It emphasizes having **more low-level tests** and **fewer high-level tests**, balancing coverage with maintainability.

---

## 1️⃣ Structure of the Testing Pyramid

```
      UI / End-to-End (E2E) Tests
      ---------------------------
             Integration Tests
      ---------------------------
               Unit Tests
```

* **Base:** Unit Tests → many, fast, isolated
* **Middle:** Integration Tests → moderate, test interactions between components
* **Top:** UI / E2E Tests → few, slow, cover full app workflows

---

## 2️⃣ Unit Tests

### Purpose:

* Test **individual classes or functions in isolation**.
* Ensure **business logic** or algorithms behave as expected.

### Characteristics:

* **Scope:** One class or function
* **Dependencies:** Use **mocks or fakes** for collaborators
* **Speed:** Very fast
* **Tools:** JUnit, Mockito, MockK

### Example:

```kotlin
class Calculator {
    fun add(a: Int, b: Int) = a + b
}

@Test
fun testAdd() {
    val calc = Calculator()
    assertEquals(5, calc.add(2, 3))
}
```

---

## 3️⃣ Integration Tests

### Purpose:

* Test **interaction between multiple components** (e.g., repository + database, network layer + repository).
* Verify **integration points and contracts** work as expected.

### Characteristics:

* **Scope:** Multiple classes or modules
* **Dependencies:** May use **real implementations** or **in-memory substitutes**
* **Speed:** Moderate
* **Tools:** JUnit, Robolectric, Room in-memory database, Retrofit MockWebServer

### Example:

```kotlin
@Test
fun testRepositoryFetch() = runBlocking {
    val repository = UserRepository(FakeApiService())
    val users = repository.getUsers()
    assertTrue(users.isNotEmpty())
}
```

---

## 4️⃣ UI / End-to-End (E2E) Tests

### Purpose:

* Test **the app as a whole** from the user perspective.
* Validate **UI flows, navigation, and system interactions**.

### Characteristics:

* **Scope:** Entire app or feature
* **Dependencies:** Use **real app** or **test doubles for network**
* **Speed:** Slowest
* **Tools:** Espresso, UI Automator, Compose Test Rule

### Example:

```kotlin
@Test
fun testLoginFlow() {
    val scenario = launchActivity<LoginActivity>()
    onView(withId(R.id.username)).perform(typeText("test"))
    onView(withId(R.id.password)).perform(typeText("1234"))
    onView(withId(R.id.loginButton)).perform(click())
    onView(withText("Welcome")).check(matches(isDisplayed()))
}
```

---

## 5️⃣ Focus and Scope Comparison

| Test Type         | Focus                 | Scope                    | Speed  | Tools                                   |
| ----------------- | --------------------- | ------------------------ | ------ | --------------------------------------- |
| Unit Tests        | Isolated logic        | Single class/function    | Fast   | JUnit, Mockito, MockK                   |
| Integration Tests | Component interaction | Multiple classes/modules | Medium | JUnit, Robolectric, MockWebServer       |
| UI / E2E Tests    | Full app/user flows   | Full app or feature      | Slow   | Espresso, UI Automator, ComposeTestRule |

---

## 6️⃣ Key Takeaways

* **Follow the pyramid:** Most tests should be **unit tests**, fewer **integration tests**, and minimal **UI/E2E tests**.
* **Unit tests** are fast and reliable; **UI tests** are slower and more brittle.
* Proper balance improves **test coverage, maintainability, and developer productivity**.
* Android-specific considerations: use **ComposeTestRule** for Compose UI tests and **Robolectric** for JVM-based component integration tests.

---

