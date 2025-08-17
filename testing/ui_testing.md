
---

# 📱 UI Testing in Android

UI tests verify that the **user interface behaves correctly** and responds to user interactions as expected. Android provides **Jetpack Compose testing** for Compose apps and **Espresso** for XML-based UI.

---

## 1️⃣ Jetpack Compose UI Testing

### Core Component: `ComposeTestRule`

* `ComposeTestRule` is the **entry point for testing Compose UIs**.
* It allows **finding, interacting, and asserting** on Composables.

### Example Setup

```kotlin
@get:Rule
val composeTestRule = createComposeRule()
```

### Finding Composables

* Use **semantics** to locate elements:

```kotlin
composeTestRule.onNodeWithText("Login").performClick()
composeTestRule.onNodeWithContentDescription("Profile Image").assertExists()
```

### Performing Actions

* Click, type, scroll, or set text:

```kotlin
composeTestRule.onNodeWithTag("usernameField")
    .performTextInput("testuser")
composeTestRule.onNodeWithTag("loginButton")
    .performClick()
```

### Making Assertions

* Check **visibility, text, or state**:

```kotlin
composeTestRule.onNodeWithText("Welcome")
    .assertIsDisplayed()
composeTestRule.onNodeWithTag("usernameField")
    .assertTextEquals("testuser")
```

### Handling State and Lifecycle

* Use `setContent` to provide the Composable under test:

```kotlin
composeTestRule.setContent {
    MyAppTheme {
        LoginScreen(viewModel = loginViewModel)
    }
}
```

---

## 2️⃣ UI Testing with XML Views (Espresso)

* **Espresso** is the standard UI testing framework for **View-based layouts**.

### Finding Views

```kotlin
onView(withId(R.id.loginButton)).perform(click())
onView(withText("Welcome")).check(matches(isDisplayed()))
```

### Performing Actions

* Type text, scroll, swipe, or click:

```kotlin
onView(withId(R.id.usernameField))
    .perform(typeText("testuser"), closeSoftKeyboard())
onView(withId(R.id.loginButton)).perform(click())
```

### Assertions

```kotlin
onView(withText("Login successful"))
    .check(matches(isDisplayed()))
```

---

## 3️⃣ Handling Asynchronous Operations

UI tests must wait for **background operations** (network, coroutines, animations).

### Jetpack Compose

* Compose automatically waits for **recomposition and idle state**, so most async tasks are safe.
* For testing **Flows or coroutines**, ensure your state updates are completed before asserting:

```kotlin
composeTestRule.waitForIdle()
```

### Espresso

* Use **Idling Resources** to synchronize async tasks:

```kotlin
val idlingResource = CountingIdlingResource("Network")
IdlingRegistry.getInstance().register(idlingResource)

// Increment before async call, decrement after completion
```

* Espresso waits until all registered **IdlingResources** are idle before performing actions/assertions.

---

## 4️⃣ Best Practices

1. **Use semantic tags** (`Modifier.testTag`) for Compose testing.
2. **Keep tests deterministic**: avoid network dependency by using fakes or MockWebServer.
3. **Test one feature at a time** to make debugging easier.
4. **Handle async operations** with ComposeTestRule's `waitForIdle` or Espresso IdlingResources.
5. **Run UI tests on different device configurations** to ensure adaptive layouts work.

---

## 5️⃣ Summary

| Framework                 | UI Type         | Key Points                                                                          |
| ------------------------- | --------------- | ----------------------------------------------------------------------------------- |
| **ComposeTestRule**       | Jetpack Compose | Find composables by text, tag, or contentDescription; perform actions; assert state |
| **Espresso**              | XML Views       | Find views by ID or text; perform actions; assert visibility/text                   |
| **Asynchronous Handling** | Both            | Compose: `waitForIdle`; Espresso: IdlingResources                                   |

* Compose tests are **tightly integrated with the UI state system**, while Espresso relies on **synchronization mechanisms** for async operations.
* Writing reliable UI tests ensures **correct behavior across user interactions and async state changes**.

---

