
---

# 🏗️ Clean Architecture Principles in Android

Clean Architecture is a design philosophy that **separates code into layers** to improve maintainability, testability, and scalability. It is framework-agnostic but widely applied in Android development.

---

## 📌 Core Principles

### 1. **Dependency Rule**

* Source code dependencies **can only point inwards**.
* Outer layers (UI, frameworks) can depend on inner layers (use cases, entities), but **inner layers must not depend on outer layers**.
* Inner layers are **pure, framework-independent**, while outer layers handle implementation details.

```
[UI Layer]       --> depends on
[Use Cases Layer] --> depends on
[Entities Layer]  --> no dependency outwards
```

---

### 2. **Entities**

* Represent **core business objects** and rules.
* Should be **independent of frameworks, UI, or databases**.
* Often implemented as **plain Kotlin classes or data classes**.

```kotlin
data class User(val id: String, val name: String)
```

---

### 3. **Use Cases (Interactors)**

* Encapsulate **business logic and operations** on entities.
* **Application-specific rules**, independent of UI or data source.
* Exposed as methods or classes that the UI layer can call.

```kotlin
class GetUserProfileUseCase(private val repository: UserRepository) {
    suspend operator fun invoke(userId: String): User {
        return repository.getUser(userId)
    }
}
```

---

### 4. **Interface Adapters**

* Responsible for **converting data from outer layers into inner layers’ formats** (and vice versa).
* Examples:

  * Mapping network models → domain models
  * Mapping Room entities → domain entities
  * Adapting domain models to UI state

```kotlin
fun UserEntity.toDomain(): User = User(id, name)
```

* Keeps **inner layers clean and framework-independent**.

---

### 5. **Frameworks & Drivers**

* Outermost layer interacting with the real world: UI, databases, network, Android SDK.
* Implements **interface adapters** for communication with inner layers.
* Changes in frameworks should **not affect core business rules**.

---

## ⚙️ Applying Clean Architecture to Android

| Layer                   | Android Components                  | Responsibility                                                                      |
| ----------------------- | ----------------------------------- | ----------------------------------------------------------------------------------- |
| UI Layer                | Composables, Activities, Fragments  | Displays data, collects events, calls use cases                                     |
| Use Case Layer          | Interactors / UseCase classes       | Contains business logic, orchestrates data from repositories                        |
| Entities Layer          | Kotlin data classes / domain models | Core business objects and rules                                                     |
| Interface Adapters      | Mappers, Repository implementations | Converts between data models, handles database/network specifics                    |
| Frameworks / Data Layer | Retrofit, Room, SharedPreferences   | Provides concrete implementations for repositories, communicates with outside world |

### Example Flow

1. **Composable** triggers an event → calls **ViewModel**.
2. **ViewModel** calls a **Use Case**.
3. **Use Case** fetches data from **Repository**.
4. **Repository** interacts with **Room/Retrofit** and maps results to **Domain Models**.
5. **ViewModel** exposes state → **Composable** updates UI.

---

## ✅ Key Benefits

* **Testability**: Inner layers are pure and easy to unit test.
* **Maintainability**: Clear separation reduces coupling.
* **Scalability**: Easy to add new features without impacting existing layers.
* **Framework-agnostic**: Core logic doesn’t depend on Android APIs.

---
