
---

# 📤 State Hoisting in Jetpack Compose

## 📌 What is State Hoisting?

**State hoisting** is the practice of **moving state out of a composable** and making it **controlled by a parent composable** instead.

Instead of a composable managing its own state internally, the state is *“hoisted”* up so that:

* The **parent owns the state**.
* The **child is stateless** (receives state as parameters and exposes events as callbacks).

This makes composables more **reusable, testable, and predictable**.

---

## 💡 Why is it Fundamental?

1. **Separation of concerns** → UI components don’t own business logic.
2. **Reusability** → Stateless components can be reused in different contexts.
3. **Single source of truth** → Avoids multiple copies of state drifting out of sync.
4. **Easier testing** → You can test UI independently of state.

---

## 📝 Example: Text Input Field

### ❌ Without State Hoisting (internal state)

```kotlin
@Composable
fun NameInput() {
    var name by remember { mutableStateOf("") }

    TextField(
        value = name,
        onValueChange = { name = it }
    )
}
```

* The state (`name`) is managed **inside** the composable.
* Parent cannot control or observe the state.

---

### ✅ With State Hoisting (external state)

```kotlin
@Composable
fun NameInput(
    name: String,
    onNameChange: (String) -> Unit
) {
    TextField(
        value = name,
        onValueChange = onNameChange
    )
}

@Composable
fun NameScreen() {
    var name by remember { mutableStateOf("") }

    Column {
        NameInput(
            name = name,
            onNameChange = { name = it }
        )
        Text("Hello, $name")
    }
}
```

* `NameInput` is **stateless** → It just renders UI.
* `NameScreen` is **stateful** → Owns and controls the `name`.
* This makes `NameInput` reusable anywhere.

---

## ✅ Key Takeaway

* **State hoisting = Moving state up to the parent**.
* Pattern:

  * Child composable → `value` + `onValueChange` parameters.
  * Parent composable → owns `mutableStateOf`.
* This ensures **unidirectional data flow**, making Compose apps more scalable.

---
