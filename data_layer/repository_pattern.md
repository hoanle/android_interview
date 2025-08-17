
---

# 🗄️ Repository Pattern in Android

The **Repository pattern** is a design pattern used to **abstract data access** and **provide a clean API** for the rest of the app. It acts as a **single source of truth** for data.

---

## 📌 Responsibilities of a Repository

1. **Abstract Data Sources**

   * Hides the details of where data comes from (network, database, cache).
   * Provides a unified API to the rest of the application.

```kotlin
interface UserRepository {
    suspend fun getUser(userId: String): User
}
```

2. **Data Mapping**

   * Converts raw data from data sources into **domain models**.

```kotlin
fun UserEntity.toDomain(): User = User(id, name)
```

3. **Caching and Synchronization**

   * Can manage **local cache** and **remote sync** transparently.
   * Ensures that UI always receives **consistent data**.

4. **Single Source of Truth**

   * All consumers of the repository access data **through a single interface**, reducing inconsistencies.

---

## 📌 Benefits of Using Repository Pattern

| Benefit                    | Description                                                                   |
| -------------------------- | ----------------------------------------------------------------------------- |
| **Separation of Concerns** | UI/ViewModel doesn’t need to know if data comes from network or database.     |
| **Testability**            | Repositories can be **mocked** in tests for predictable behavior.             |
| **Maintainability**        | Data fetching and caching logic is centralized and easier to update.          |
| **Flexibility**            | Easy to swap data sources or add new sources (e.g., remote API, local cache). |
| **Consistency**            | Ensures that all consumers of the data use the same source and mapping logic. |

---

## 📌 Example in Android

```kotlin
class UserRepositoryImpl(
    private val api: UserApi,
    private val dao: UserDao
) : UserRepository {

    override suspend fun getUser(userId: String): User {
        // Try local cache first
        val cached = dao.getUser(userId)
        return if (cached != null) {
            cached.toDomain()
        } else {
            // Fetch from network and save to local db
            val networkUser = api.getUser(userId)
            dao.insert(networkUser.toEntity())
            networkUser.toDomain()
        }
    }
}
```

* UI/ViewModel calls `UserRepository` without knowing whether the data is from **network or database**.

```kotlin
class UserViewModel(private val repository: UserRepository) : ViewModel() {
    val user = MutableStateFlow<User?>(null)

    fun loadUser(userId: String) = viewModelScope.launch {
        user.value = repository.getUser(userId)
    }
}
```

---

## ✅ Key Takeaways

* **Repository pattern abstracts data access** and provides a clean interface.
* Handles **mapping, caching, and data source coordination**.
* Improves **separation of concerns, testability, maintainability, and consistency**.
* Essential in **MVVM/Clean Architecture** Android apps.

---
