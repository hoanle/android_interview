
---

# 🧪 Testing Jetpack Compose UIs

## 📌 Overview

Jetpack Compose provides a **testing framework** that integrates with JUnit to test UI behavior:

* Verify composables render correctly.
* Simulate user interactions.
* Assert UI state and side effects.

The key tool is **`ComposeTestRule`**.

---

## ⚙️ `ComposeTestRule`

* A JUnit rule that sets up and tears down the Compose environment for each test.
* Provides APIs to **set content**, **find UI nodes**, **perform actions**, and **make assertions**.

```kotlin
@get:Rule
val composeTestRule = createComposeRule()

@Test
fun myFirstTest() {
    composeTestRule.setContent {
        MyScreen()
    }
}
```

---

## 🔍 Finding Composables

Compose testing uses **semantics** to locate UI elements.

### Common finders:

```kotlin
composeTestRule.onNodeWithText("Login")   // Find by text
composeTestRule.onNodeWithContentDescription("Profile Pic") // Find by contentDescription
composeTestRule.onNodeWithTag("UsernameField") // Find by testTag
composeTestRule.onAllNodesWithText("Item")     // Multiple matches
```

### Adding a Test Tag:

```kotlin
TextField(
    value = username,
    onValueChange = { username = it },
    modifier = Modifier.testTag("UsernameField")
)
```

---

## 🎬 Performing Actions

You can simulate user interactions like clicks, typing, and scrolling.

```kotlin
composeTestRule.onNodeWithText("Login").performClick()

composeTestRule.onNodeWithTag("UsernameField")
    .performTextInput("Alice")

composeTestRule.onNodeWithTag("List")
    .performScrollToNode(hasText("Item 20"))
```

---

## ✅ Making Assertions

Assertions verify that the UI behaves as expected after actions.

```kotlin
composeTestRule.onNodeWithText("Welcome Alice")
    .assertExists()

composeTestRule.onNodeWithTag("UsernameField")
    .assertTextEquals("Alice")

composeTestRule.onNodeWithText("Loading...")
    .assertIsDisplayed()

composeTestRule.onNodeWithText("Error").assertDoesNotExist()
```

---

## 📝 Example Test

```kotlin
@get:Rule
val composeTestRule = createComposeRule()

@Test
fun loginScreen_displaysWelcomeMessage() {
    composeTestRule.setContent {
        LoginScreen()
    }

    // Type username
    composeTestRule.onNodeWithTag("UsernameField")
        .performTextInput("Alice")

    // Click login
    composeTestRule.onNodeWithText("Login").performClick()

    // Assert welcome message
    composeTestRule.onNodeWithText("Welcome Alice")
        .assertIsDisplayed()
}
```

---

## ✅ Key Takeaways

* **`ComposeTestRule`** sets up UI tests.
* Use **finders** (`onNodeWithText`, `onNodeWithTag`) to locate composables.
* Use **actions** (`performClick`, `performTextInput`, `performScrollToNode`) to simulate user behavior.
* Use **assertions** (`assertIsDisplayed`, `assertTextEquals`, `assertExists`) to validate UI state.

---
