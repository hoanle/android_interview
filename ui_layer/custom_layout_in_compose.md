
---

# 🛠️ Creating Custom Layouts in Jetpack Compose

In Jetpack Compose, you can create **custom layouts** when the standard `Row`, `Column`, or `Box` do not meet your needs. Two primary APIs are used: **`Layout`** and **`SubcomposeLayout`**.

---

## 1️⃣ `Layout` Composable

### Overview

* The `Layout` composable allows you to define **custom measurement and placement logic** for its children.
* You provide a **measure block** that calculates size and positions children.

### Syntax

```kotlin
@Composable
fun CustomRow(
    modifier: Modifier = Modifier,
    content: @Composable () -> Unit
) {
    Layout(
        content = content,
        modifier = modifier
    ) { measurables, constraints ->
        // Measure children
        val placeables = measurables.map { it.measure(constraints) }

        // Determine layout size
        val width = placeables.sumOf { it.width }
        val height = placeables.maxOfOrNull { it.height } ?: constraints.minHeight

        // Set layout size and place children
        layout(width, height) {
            var xPosition = 0
            placeables.forEach { placeable ->
                placeable.placeRelative(x = xPosition, y = 0)
                xPosition += placeable.width
            }
        }
    }
}
```

### ✅ Key Points

* `measurables`: Represents the children to measure.
* `constraints`: The constraints from parent (min/max width/height).
* `layout(width, height) { ... }`: Defines the size of the layout and **child placement**.

---

## 2️⃣ `SubcomposeLayout`

### Overview

* Allows **delayed measurement** of children.
* Useful when the size of some children **depends on other children**.
* Common use cases: adaptive layouts, headers affecting content, or measuring children before deciding layout.

### Example

```kotlin
@Composable
fun TwoPassLayout(
    modifier: Modifier = Modifier,
    content: @Composable () -> Unit
) {
    SubcomposeLayout(modifier = modifier) { constraints ->
        // Step 1: subcompose and measure header
        val headerPlaceables = subcompose("header") { Header() }
            .map { it.measure(constraints) }

        val headerHeight = headerPlaceables.maxOf { it.height }

        // Step 2: subcompose and measure content based on header height
        val contentPlaceables = subcompose("content") { Content() }
            .map { it.measure(constraints.copy(minHeight = 0)) }

        val totalHeight = headerHeight + (contentPlaceables.maxOfOrNull { it.height } ?: 0)

        layout(constraints.maxWidth, totalHeight) {
            var yPosition = 0
            headerPlaceables.forEach { it.placeRelative(0, yPosition) }
            yPosition += headerHeight
            contentPlaceables.forEach { it.placeRelative(0, yPosition) }
        }
    }
}
```

### ✅ Key Points

* `subcompose(key) { ... }`: Compose children on-demand.
* Useful when **child sizes depend on each other**.
* Enables **two-pass layout** measurement.

---

## 💡 When to Use Which

| API                | When to Use                                                                                               |
| ------------------ | --------------------------------------------------------------------------------------------------------- |
| `Layout`           | Simple custom layouts where children are independent.                                                     |
| `SubcomposeLayout` | Complex layouts where child measurement depends on other children (e.g., adaptive headers, lazy layouts). |

---

## ✅ Summary

* **Custom Layouts** let you define **measurement and placement** logic.
* `Layout` is for most **straightforward custom arrangements**.
* `SubcomposeLayout` is for **dependent or two-pass measurement scenarios**.
* Both integrate with Compose’s **declarative and reactive system**.

---
