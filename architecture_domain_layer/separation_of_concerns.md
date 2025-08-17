
---

# 🎯 Ensuring Separation of Concerns in Android Apps

**Separation of Concerns (SoC)** is a design principle where **different responsibilities are divided across distinct components**, improving maintainability, testability, and readability.

---

## 1️⃣ Layered Architecture

### Approach

* Split the app into **UI layer**, **Domain layer**, and **Data layer**.
* Each layer has a **clear responsibility** and communicates only through **defined interfaces**.

### Example

```kotlin
// Domain Layer (Use Case)
class GetUserProfileUseCase(private val repository: UserRepository) {
    suspend operator fun invoke(userId: String): User = repository.getUser(userId)
}

// Data Layer (Repository)
class UserRepositoryImpl(private val api: UserApi) : UserRepository {
    override suspend fun getUser(userId: String) = api.getUser(userId)
}

// UI Layer (ViewModel)
class UserViewModel(private val getUserProfile: GetUserProfileUseCase) : ViewModel() {
    private val _uiState = MutableStateFlow<User?>(null)
    val uiState: StateFlow<User?> = _uiState

    fun loadUser(userId: String) = viewModelScope.launch {
        _uiState.value = getUserProfile(userId)
    }
}
```

✅ Each layer focuses on **its own responsibility**.

---

## 2️⃣ ViewModel for UI State

* Keep **UI state and event handling** in the ViewModel.
* Avoid **business logic in Composables** or Activities/Fragments.

```kotlin
@Composable
fun UserScreen(viewModel: UserViewModel = viewModel()) {
    val user by viewModel.uiState.collectAsState()
    Text(user?.name ?: "Loading...")
}
```

* Composables remain **stateless**, focused only on rendering.

---

## 3️⃣ Use Cases / Interactors for Business Logic

* Encapsulate **core operations** in **use cases** rather than spreading logic across ViewModel or UI.

```kotlin
class UpdateUserProfileUseCase(private val repository: UserRepository) {
    suspend operator fun invoke(user: User) = repository.updateUser(user)
}
```

* Helps **unit testing** without Android dependencies.

---

## 4️⃣ Repositories and Data Sources

* Abstract data fetching and storage from the UI/domain layers.
* Can combine multiple sources (network, database, cache) transparently.

```kotlin
interface UserRepository {
    suspend fun getUser(userId: String): User
}
```

* UI and domain layers **don’t care where the data comes from**.

---

## 5️⃣ Modifiers and Composables in UI

* UI-specific logic like layout, gestures, and theming should remain **in Composables or modifiers**.
* Avoid **embedding network or database calls** in Composables.

---

## 6️⃣ Dependency Injection

* Use **Hilt or Dagger** to inject dependencies instead of creating them in UI components.
* Promotes **loose coupling** between layers.

```kotlin
@HiltViewModel
class UserViewModel @Inject constructor(
    private val getUserProfile: GetUserProfileUseCase
) : ViewModel()
```

---

## ✅ Key Practices

1. **One responsibility per class/layer**.
2. **UI observes state, doesn’t compute it**.
3. **Domain layer encapsulates business rules**.
4. **Data layer abstracts network/db details**.
5. **Use dependency injection** to reduce tight coupling.
6. **Composable functions remain stateless and declarative**.

---

