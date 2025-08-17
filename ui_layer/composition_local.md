
---

# 🌐 CompositionLocals in Jetpack Compose

## 📌 What are CompositionLocals?

**CompositionLocals** are a way to implicitly pass data down the Compose tree **without having to pass it through every composable parameter**.

* Think of them like **React’s Context** or **dependency injection at the composition level**.
* They provide values to **all descendants** in the composition unless overridden.
* They help avoid **parameter drilling** (passing the same parameter through multiple layers).

---

## ⚙️ How They Work

1. **Define** a `CompositionLocal` using `staticCompositionLocalOf` or `compositionLocalOf`.
2. **Provide** a value with `CompositionLocalProvider`.
3. **Consume** it with `LocalXXX.current`.

---

## 📝 Example: Passing a Theme Color

### Define

```kotlin
val LocalAccentColor = staticCompositionLocalOf { Color.Blue }
```

### Provide

```kotlin
@Composable
fun App() {
    CompositionLocalProvider(LocalAccentColor provides Color.Red) {
        HomeScreen()
    }
}
```

### Consume

```kotlin
@Composable
fun HomeScreen() {
    val accent = LocalAccentColor.current
    Text("Welcome!", color = accent)
}
```

👉 Here, `HomeScreen` (and its children) automatically receive `Color.Red` without needing `accentColor` passed as a parameter.

---

## 🎯 Practical Use Cases

1. **Theming**

   * Colors, typography, shapes → avoid passing theme objects everywhere.
   * Jetpack Compose’s `MaterialTheme` itself is built on CompositionLocals.

2. **Ambient context (dependencies)**

   * Example: current `Context` (`LocalContext`), `LifecycleOwner`, or `NavController`.

```kotlin
val navController = LocalNavController.current
```

3. **Configuration & settings**

   * Language/locale, feature flags, user preferences → easily accessed deep in UI hierarchy.

4. **Dependency injection at UI level**

   * Provide a `Repository`, `ViewModel`, or mock object in previews/tests without rewriting every parameter.

---

## ✅ Key Takeaways

* **CompositionLocals = implicit data providers** for the Compose tree.
* Prevent **boilerplate parameter passing** through deep UI hierarchies.
* Useful for **theming, context, navigation, configuration, or lightweight DI**.
* Should be used **sparingly** → overusing them can hurt code readability and make dependencies implicit.

---

