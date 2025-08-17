
---

# 🎨 Modeling UI States and Events in Modern Android Apps

In modern Android apps, especially using **Jetpack Compose** and **MVVM/MVI architectures**, managing UI states and events clearly is crucial for predictable and maintainable UI behavior.

---

## 1️⃣ UI State

### Concept

* **UI state** represents the **current state of the screen**.
* Common states: `Loading`, `Success`, `Error`, `Empty`.
* Should be **immutable** and reflect the **entire UI at a moment in time**.

### Recommended Approach

* Use **sealed classes** or **enums** to model UI states.

#### Sealed Class Example

```kotlin
sealed class UserUiState {
    object Loading : UserUiState()
    data class Success(val user: User) : UserUiState()
    data class Error(val message: String) : UserUiState()
}
```

* ViewModel exposes state via **StateFlow / LiveData**:

```kotlin
class UserViewModel(private val getUserProfile: GetUserProfileUseCase) : ViewModel() {
    private val _uiState = MutableStateFlow<UserUiState>(UserUiState.Loading)
    val uiState: StateFlow<UserUiState> = _uiState

    fun loadUser(userId: String) = viewModelScope.launch {
        _uiState.value = UserUiState.Loading
        try {
            val user = getUserProfile(userId)
            _uiState.value = UserUiState.Success(user)
        } catch (e: Exception) {
            _uiState.value = UserUiState.Error(e.message ?: "Unknown error")
        }
    }
}
```

* **Composable observes the state**:

```kotlin
@Composable
fun UserScreen(viewModel: UserViewModel = viewModel()) {
    val uiState by viewModel.uiState.collectAsState()

    when (uiState) {
        is UserUiState.Loading -> CircularProgressIndicator()
        is UserUiState.Success -> Text("Hello ${(uiState as UserUiState.Success).user.name}")
        is UserUiState.Error -> Text("Error: ${(uiState as UserUiState.Error).message}")
    }
}
```

✅ **Benefits**:

* **Type-safe**: impossible to represent an invalid state.
* **Exhaustive `when` checks** in Kotlin.
* **Immutable** and predictable.

---

## 2️⃣ UI Events

### Concept

* **Events** represent **one-time actions**, like:

  * Navigation
  * Showing a Toast / Snackbar
  * Dialog triggers

### Recommended Approach

* Use **SharedFlow** (or Channel) in ViewModel for events.
* Model events with **sealed classes** for type safety.

#### Example

```kotlin
sealed class UserUiEvent {
    data class ShowToast(val message: String) : UserUiEvent()
    object NavigateToDetail : UserUiEvent()
}

class UserViewModel : ViewModel() {
    private val _uiEvent = MutableSharedFlow<UserUiEvent>()
    val uiEvent: SharedFlow<UserUiEvent> = _uiEvent

    fun onUserClicked() {
        viewModelScope.launch {
            _uiEvent.emit(UserUiEvent.NavigateToDetail)
        }
    }
}
```

* UI observes events:

```kotlin
LaunchedEffect(Unit) {
    viewModel.uiEvent.collect { event ->
        when (event) {
            is UserUiEvent.ShowToast -> Toast.makeText(context, event.message, Toast.LENGTH_SHORT).show()
            UserUiEvent.NavigateToDetail -> navigateToDetailScreen()
        }
    }
}
```

✅ **Benefits**:

* Distinguishes **transient events** from **persistent state**.
* Prevents **re-triggering events on recomposition**.

---

## 3️⃣ Summary of Recommendations

| Aspect    | Modeling                             | Notes                                                                      |
| --------- | ------------------------------------ | -------------------------------------------------------------------------- |
| UI State  | Sealed Class / Enum                  | Immutable, represents full screen state, observed via StateFlow / LiveData |
| UI Events | Sealed Class + SharedFlow / Channel  | One-time actions, unidirectional flow, safe from recomposition issues      |
| Benefits  | Type-safe, predictable, easy to test | Exhaustive checks, clear separation of state vs events                     |

---
