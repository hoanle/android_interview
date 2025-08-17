
---

# 🧠 ViewModel Injection with Hilt

Hilt provides **built-in support for injecting ViewModels**, simplifying dependency injection in the **MVVM architecture** of Android. Traditionally, creating ViewModels with dependencies required **ViewModelFactory boilerplate**, but Hilt eliminates this need.

---

## 1️⃣ @HiltViewModel Annotation

* `@HiltViewModel` is used to **annotate a ViewModel class** so that Hilt can **provide its dependencies automatically**.
* The ViewModel must have an **@Inject constructor** to receive dependencies.

### Example:

```kotlin
@HiltViewModel
class MainViewModel @Inject constructor(
    private val repository: UserRepository
) : ViewModel() {

    fun fetchUsers() = repository.getUsers()
}
```

* `UserRepository` is automatically injected by Hilt.
* No need to manually create a `ViewModelFactory`.

---

## 2️⃣ Injecting ViewModels in Activities and Fragments

### a. In Activity

```kotlin
@AndroidEntryPoint
class MainActivity : AppCompatActivity() {

    private val viewModel: MainViewModel by viewModels() // Hilt-aware

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        viewModel.fetchUsers()
    }
}
```

### b. In Fragment

```kotlin
@AndroidEntryPoint
class MainFragment : Fragment() {

    private val viewModel: MainViewModel by viewModels() // Hilt-aware

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        viewModel.fetchUsers()
    }
}
```

* Hilt automatically **creates and provides** the ViewModel with its dependencies.
* Works with **Activity-retained scopes**, so ViewModel survives configuration changes.

---

## 3️⃣ Benefits of Hilt ViewModel Injection

| Benefit                    | Description                                                                                |
| -------------------------- | ------------------------------------------------------------------------------------------ |
| **No Factory Boilerplate** | Eliminates the need to manually implement `ViewModelProvider.Factory`.                     |
| **Lifecycle Awareness**    | ViewModels are tied to **Activity/Fragment lifecycles** automatically.                     |
| **Scoped Dependencies**    | Injected dependencies respect Hilt scopes (e.g., `@Singleton`, `@ActivityRetainedScoped`). |
| **Testable**               | Dependencies can be easily **mocked in unit tests**.                                       |
| **Simplified Code**        | Cleaner, more maintainable code with less boilerplate.                                     |

---

## 4️⃣ Scoping in ViewModels

* **@ActivityRetainedScoped** dependencies survive configuration changes and are shared across fragments in the same activity.
* **@ViewModelScoped** dependencies exist only during the ViewModel lifecycle.
* Hilt **automatically resolves scopes** when injecting into a ViewModel annotated with `@HiltViewModel`.

---

## 5️⃣ Summary

* `@HiltViewModel` + `@Inject constructor` allows **automatic dependency injection** into ViewModels.
* Works seamlessly in **Activities and Fragments** with `by viewModels()`.
* Benefits include **eliminating boilerplate, lifecycle management, scoped dependencies, and improved testability**.

---

