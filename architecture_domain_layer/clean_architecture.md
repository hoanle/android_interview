
---

# Clean Architecture in Android

Clean Architecture, introduced by Robert C. Martin (Uncle Bob), is a design philosophy for creating modular, testable, and maintainable software systems. In Android, it helps structure apps to improve scalability, testability, and adaptability to changing requirements by emphasizing separation of concerns and independence from frameworks, UI, and external systems.

## Core Principles

1. **Separation of Concerns**: Divide the app into distinct layers, each with specific responsibilities.
2. **Dependency Rule**: Dependencies flow inward—outer layers depend on inner layers, but inner layers are independent.
3. **Independence from Frameworks**: Business logic should not rely on Android-specific APIs or libraries.
4. **Testability**: Business logic can be tested without UI, database, or external services.
5. **Abstraction**: Use interfaces to decouple layers and define interactions.

## Layers of Clean Architecture

Clean Architecture organizes code into three main concentric layers:

### 1. Presentation Layer (Outer Layer)
- **Purpose**: Manages UI components (Activities, Fragments, ViewModels, Jetpack Compose) and user interactions.
- **Dependencies**: Depends on the Domain layer to fetch or send data.
- **Example**: A ViewModel retrieving data from a Use Case to update the UI.

### 2. Domain Layer (Middle Layer)
- **Purpose**: Contains the core business logic and rules, independent of Android frameworks.
- **Components**:
  - **Entities**: Plain data models representing business objects (e.g., `User`).
  - **Use Cases**: Encapsulate specific business operations (e.g., `GetUserProfileUseCase`).
- **Example**: A Use Case defining how to fetch a user’s profile data.

### 3. Data Layer (Inner Layer)
- **Purpose**: Handles data sources (APIs, databases, local storage).
- **Components**:
  - **Repositories**: Bridge between Domain layer and data sources, implementing Domain interfaces.
  - **Data Models**: API or database-specific models with mappers to convert to Domain entities.
- **Example**: A Repository fetching data from a Room database or Retrofit API.

## Dependency Flow
- **Presentation → Domain**: Presentation layer uses Domain Use Cases.
- **Domain ← Data**: Data layer implements Domain interfaces (e.g., Repository interfaces).
- The Domain layer remains independent of outer layers.

## Implementation in Android

### Project Structure
```plaintext
app/
├── presentation/
│   ├── ui/              # Activities, Fragments, Composables
│   ├── viewmodel/       # ViewModels
│   └── mapper/          # Maps Domain models to UI models
├── domain/
│   ├── model/           # Entities (business objects)
│   ├── usecase/         # Use Cases (business logic)
│   └── repository/      # Repository interfaces
└── data/
    ├── repository/      # Repository implementations
    ├── local/          # Local data sources (Room, SharedPreferences)
    ├── remote/         # Remote data sources (Retrofit, API services)
    └── mapper/         # Maps data models to Domain entities
```

### Example Workflow
**Feature**: Display a user’s profile.
1. **Presentation**: `ProfileActivity` or `ProfileScreen` (Compose) uses `ProfileViewModel` to fetch data.
2. **Domain**: `ProfileViewModel` calls `GetUserProfileUseCase`, which defines logic to retrieve a `User` entity.
3. **Data**: `GetUserProfileUseCase` uses a `UserRepository` interface. `UserRepositoryImpl` fetches data from `UserApi` (remote) or `UserDao` (local) and maps it to a `User` entity.
4. **Flow**:
   - User clicks “Load Profile” in the UI.
   - `ProfileViewModel` invokes `GetUserProfileUseCase`.
   - `GetUserProfileUseCase` calls `UserRepository`.
   - `UserRepositoryImpl` fetches data, maps it to a `User` entity, and returns it.
   - `ProfileViewModel` updates the UI with the data.

### Example Code
```kotlin
// Domain Layer: Entity
data class User(val id: String, val name: String)

// Domain Layer: Repository Interface
interface UserRepository {
    suspend fun getUserProfile(userId: String): User
}

// Domain Layer: Use Case
class GetUserProfileUseCase(private val userRepository: UserRepository) {
    suspend operator fun invoke(userId: String): User {
        return userRepository.getUserProfile(userId)
    }
}

// Data Layer: Repository Implementation
class UserRepositoryImpl(
    private val userApi: UserApi,
    private val userDao: UserDao
) : UserRepository {
    override suspend fun getUserProfile(userId: String): User {
        // Example: Try local, then remote
        val localUser = userDao.getUser(userId)
        return if (localUser != null) {
            localUser.toDomainUser()
        } else {
            val remoteUser = userApi.getUserProfile(userId)
            userDao.saveUser(remoteUser)
            remoteUser.toDomainUser()
        }
    }
}

// Presentation Layer: ViewModel
class ProfileViewModel(
    private val getUserProfileUseCase: GetUserProfileUseCase
) : ViewModel() {
    private val _userState = MutableStateFlow<User?>(null)
    val userState: StateFlow<User?> = _userState

    fun loadUserProfile(userId: String) {
        viewModelScope.launch {
            val user = getUserProfileUseCase(userId)
            _userState.value = user
        }
    }
}
```

### Dependency Injection
- Use Hilt or Koin to provide dependencies (e.g., `UserRepository` to `GetUserProfileUseCase`).
- Ensures loose coupling and easier testing.

## Benefits in Android
- **Testability**: Domain logic can be unit-tested without Android dependencies.
- **Maintainability**: Changes in one layer (e.g., swapping Retrofit) don’t affect others.
- **Scalability**: Modular structure simplifies adding new features.
- **Flexibility**: UI or data sources can change without impacting business logic.

## Challenges
- **Boilerplate Code**: Requires more setup (interfaces, mappers).
- **Learning Curve**: Complex for developers new to the pattern.
- **Overhead for Small Apps**: May be overkill for simple apps.

## When to Use
- Ideal for medium to large apps with complex logic or frequent changes.
- Less suitable for small apps or prototypes.

## Practical Tips
- Use Kotlin coroutines or Flow for asynchronous operations.
- Leverage Jetpack libraries (ViewModel, Room) in Presentation and Data layers.
- Keep Domain layer pure (no Android dependencies).
- Use mappers to convert between Domain entities and Data/UI models.
- Regularly refactor to prevent layer leakage.

## Resources
- Uncle Bob’s [Clean Architecture](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
- Android Jetpack documentation
- GitHub projects (search “Android Clean Architecture”)

---
