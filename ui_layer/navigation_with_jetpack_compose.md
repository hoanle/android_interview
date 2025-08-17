
---

# 🧭 Navigation in Jetpack Compose

## 📌 Core Components

### **`NavController`**

* Central API for navigation in Compose.
* Manages the **back stack** of destinations.
* Exposes methods like `navigate()`, `popBackStack()`, and back stack entry state.

```kotlin
val navController = rememberNavController()
```

---

### **`NavHost`**

* Defines the **navigation graph** inside Compose.
* Maps routes (`String`) to composable destinations.

```kotlin
NavHost(navController, startDestination = "home") {
    composable("home") { HomeScreen(navController) }
    composable("profile") { ProfileScreen(navController) }
}
```

---

## 📦 Passing Arguments

### **Primitive arguments**

```kotlin
composable("profile/{userId}") { backStackEntry ->
    val userId = backStackEntry.arguments?.getString("userId")
    ProfileScreen(userId)
}
navController.navigate("profile/123")
```

---

### **Complex data**

* Use `Parcelable` / `Serializable` with `Bundle`.
* Or store data in **`SavedStateHandle`** (preferred).

```kotlin
// Sender
navController.currentBackStackEntry?.savedStateHandle?.set("user", user)

// Receiver
val user = navController.previousBackStackEntry
    ?.savedStateHandle?.get<User>("user")
```

✅ This avoids putting large/complex data in route strings.

---

## 🌲 Nested Navigation Graphs

* Breaks navigation into **modular graphs** (auth, main app, settings).
* Helps in large apps with multiple flows.

```kotlin
NavHost(navController, startDestination = "auth") {
    navigation(startDestination = "login", route = "auth") {
        composable("login") { LoginScreen(navController) }
        composable("signup") { SignupScreen(navController) }
    }
    navigation(startDestination = "home", route = "main") {
        composable("home") { HomeScreen(navController) }
        composable("profile") { ProfileScreen(navController) }
    }
}
```

---

## 🛡️ Type-Safe Navigation Strategies

String routes can be error-prone → define **sealed classes or enums** for routes.

```kotlin
sealed class Screen(val route: String) {
    object Home : Screen("home")
    object Profile : Screen("profile/{userId}") {
        fun createRoute(userId: String) = "profile/$userId"
    }
}

// Usage
navController.navigate(Screen.Profile.createRoute("123"))
```

✅ Avoids typos and makes navigation **type-safe**.

---

## 🔗 Deep Linking

* Compose Navigation supports deep links to open specific screens.

```kotlin
composable(
    "profile/{userId}",
    deepLinks = listOf(navDeepLink { uriPattern = "app://profile/{userId}" })
) { entry ->
    val userId = entry.arguments?.getString("userId")
    ProfileScreen(userId)
}
```

Opening `app://profile/123` from a browser or intent will navigate directly.

---

## 🎭 Custom Transitions

* Use **`Accompanist Navigation-Animation`** or `AnimatedNavHost`.

```kotlin
AnimatedNavHost(navController, startDestination = "home") {
    composable("home",
        enterTransition = { fadeIn() },
        exitTransition = { fadeOut() }
    ) {
        HomeScreen(navController)
    }
}
```

✅ Enables smooth animations between screens.

---

## 🧪 Testing Navigation

* Use **`createComposeRule`** and inject `NavController`.
* Verify navigation with `assertThat(navController.currentBackStackEntry?.destination?.route)`.

```kotlin
@get:Rule
val composeTestRule = createComposeRule()

@Test
fun testNavigationToProfile() {
    composeTestRule.setContent {
        val navController = TestNavHostController(LocalContext.current)
        NavHost(navController, startDestination = "home") {
            composable("home") {
                Button(onClick = { navController.navigate("profile/123") }) {
                    Text("Go")
                }
            }
            composable("profile/{userId}") { /* ... */ }
        }
    }

    composeTestRule.onNodeWithText("Go").performClick()
    assertThat(navController.currentBackStackEntry?.destination?.route)
        .isEqualTo("profile/{userId}")
}
```

---

## 🧩 Managing Complex Navigation State

### 1. **Conditional navigation**

* Decide route based on state (e.g., auth vs main flow).

```kotlin
if (isLoggedIn) {
    navController.navigate("main") {
        popUpTo("auth") { inclusive = true }
    }
} else {
    navController.navigate("auth")
}
```

---

### 2. **Backstack manipulation**

* Control navigation stack for scenarios like logout or replacing screens.

```kotlin
navController.navigate("login") {
    popUpTo("home") { inclusive = true } // Clear stack
}
```

---

## ✅ Key Takeaways

* **NavController** manages back stack.
* **NavHost** defines navigation graph.
* Use **SavedStateHandle** for complex data.
* Break into **nested graphs** for modularity.
* Adopt **sealed classes** for type-safe navigation.
* Add **deep links** for external entry points.
* Use **Accompanist animations** for transitions.
* Write **navigation tests** with `TestNavHostController`.
* Manage **conditional flows** and **backstack rules** for real-world apps.

---
