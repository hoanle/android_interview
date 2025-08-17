
---

# 📐 Adaptive Layouts in Jetpack Compose

## 📌 What is an Adaptive Layout?

An **adaptive layout** is a UI design approach where the interface **dynamically adjusts** to different:

* Screen sizes (phones, tablets, ChromeOS)
* Orientations (portrait, landscape)
* Form factors (foldables, multi-window modes)

### Why It’s Crucial

* **Modern Android devices** come in a wide variety of screen sizes and densities.
* Adaptive layouts **improve usability, accessibility, and user experience**.
* Ensures your app is **future-proof** for emerging devices like foldables and large-screen tablets.

---

## ⚙️ Strategies and APIs for Adaptive Layouts

### 1. **WindowSizeClass**

* Jetpack Compose provides **`WindowSizeClass`** to categorize screen width/height:

  * **Compact** → small screens (phones)
  * **Medium** → medium screens (large phones, small tablets)
  * **Expanded** → large screens (tablets, desktop, foldables)

#### Example:

```kotlin
@Composable
fun AdaptiveScreen() {
    val windowSizeClass = calculateWindowSizeClass()

    when (windowSizeClass.widthSizeClass) {
        WindowWidthSizeClass.Compact -> CompactLayout()
        WindowWidthSizeClass.Medium -> MediumLayout()
        WindowWidthSizeClass.Expanded -> ExpandedLayout()
    }
}
```

* Makes **layout decisions predictable and declarative**.
* Supports **responsive navigation patterns** (bottom nav for compact, rail or sidebar for expanded).

---

### 2. **BoxWithConstraints**

* Provides **maxWidth, maxHeight, minWidth, minHeight** at runtime.
* Useful for **component-level adaptivity**.

```kotlin
@Composable
fun ResponsiveCard() {
    BoxWithConstraints {
        if (maxWidth < 600.dp) {
            Column { /* stacked layout */ }
        } else {
            Row { /* horizontal layout */ }
        }
    }
}
```

* Enables **fine-grained, conditional layout adjustments** based on available space.

---

### 3. **Custom Layout Composables**

* Use `Layout` or `SubcomposeLayout` for **dynamic, content-driven adjustments**.
* Example: adjusting item positions, wrapping content, or resizing children based on constraints.

---

### 4. **ConstraintLayout**

* Provides **flexible, constraint-based UI design**.
* Allows defining **relationships between elements** that adjust naturally to different screen sizes.

```kotlin
ConstraintLayout(modifier = Modifier.fillMaxSize()) {
    val (title, image) = createRefs()
    Text("Title", Modifier.constrainAs(title) { top.linkTo(parent.top) })
    Image(..., Modifier.constrainAs(image) { top.linkTo(title.bottom) })
}
```

---

## 💡 Recommended Practices for Adaptive Layouts

1. **Canonical Layouts**

   * Define base layouts for **small, medium, and large screens**.
   * Reuse components where possible.

2. **Dynamic Content**

   * Adjust **content density** and **visibility** based on screen size.
   * Example: show extra info in expanded mode, hide in compact mode.

3. **Responsive Navigation**

   * Bottom navigation for **Compact**
   * Navigation rail or sidebar for **Expanded**

4. **Flexible Spacing & Typography**

   * Use **`dp` and `sp` scaling**, **padding adjustments**, and **weight-based sizing** to improve responsiveness.

---

## 🧪 Testing Adaptive Layouts

* **Preview in Compose** with different devices/orientations:

```kotlin
@Preview(device = Devices.PIXEL_4)
@Preview(device = Devices.TABLET)
@Composable
fun AdaptiveScreenPreview() {
    AdaptiveScreen()
}
```

* **Emulators & physical devices**:

  * Rotate screen
  * Test foldables / multi-window modes
  * Resize ChromeOS windows

* **Automated tests**:

  * Use `ComposeTestRule` with different constraints
  * Validate layouts for visibility, alignment, and content wrapping

---

## ✅ Key Takeaways

* Adaptive layouts = **dynamic UI for various screens and form factors**.
* **WindowSizeClass** helps make **high-level layout decisions**.
* **BoxWithConstraints & custom layouts** enable **component-level adaptivity**.
* Follow best practices: canonical layouts, dynamic content, responsive navigation.
* Test across devices, orientations, and configurations to ensure **consistent experience**.

---

