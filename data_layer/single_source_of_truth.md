
---

# 🏛️ Single Source of Truth (SSOT) in Android

The **Single Source of Truth** (SSOT) is a design principle in which **one authoritative source holds the definitive state of data** in an application. All other layers and components consume data from this source to ensure consistency.

---

## 📌 Concept

* Ensures that **all consumers of data observe the same state**.
* Reduces **data inconsistency and duplication** across UI, cache, and network.
* Often implemented using **a combination of local database and network sources**.

---

## 📌 Benefits

| Benefit                  | Description                                                  |
| ------------------------ | ------------------------------------------------------------ |
| **Consistency**          | UI always displays the latest, authoritative data.           |
| **Simplified Data Flow** | Reduces multiple conflicting copies of data.                 |
| **Offline Support**      | Local database acts as the SSOT when network is unavailable. |
| **Testability**          | Makes it easier to test data flows and caching logic.        |

---

## 📌 Implementing SSOT in Android

### 1. Use Local Database as the Primary Source

* **Room Database** is commonly used as the SSOT.
* The network **updates the database**, not the UI directly.

```kotlin
@Dao
interface UserDao {
    @Query("SELECT * FROM users WHERE id = :userId")
    fun getUserFlow(userId: String): Flow<UserEntity>

    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insert(user: UserEntity)
}
```

* The **UI observes the database** using `Flow` or `LiveData`:

```kotlin
class UserRepository(
    private val api: UserApi,
    private val dao: UserDao
) {
    fun getUser(userId: String): Flow<User> =
        dao.getUserFlow(userId).map { it.toDomain() }

    suspend fun refreshUser(userId: String) {
        val networkUser = api.getUser(userId)
        dao.insert(networkUser.toEntity())
    }
}
```

---

### 2. Unidirectional Data Flow

1. **UI** observes **Room database** (SSOT).
2. **Repository** fetches from network when needed.
3. **Repository** updates the database.
4. **Room triggers UI updates automatically**.

```text
[Network] → [Repository] → [Database (SSOT)] → [UI]
```

* Even when offline, **UI reads from database**, ensuring **consistent experience**.

---

### 3. Optional Caching & Conflict Handling

* Use **timestamp or versioning** to resolve conflicts between network and local data.
* Ensure **atomic updates** to maintain data integrity.

---

### 4. Example with Jetpack Compose + Flow

```kotlin
@Composable
fun UserScreen(userId: String, viewModel: UserViewModel = viewModel()) {
    val user by viewModel.userFlow.collectAsState(initial = null)

    if (user == null) {
        CircularProgressIndicator()
    } else {
        Text("Hello, ${user.name}")
    }
}

class UserViewModel(private val repository: UserRepository) : ViewModel() {
    val userFlow = repository.getUser("123").stateIn(
        viewModelScope,
        SharingStarted.Lazily,
        null
    )

    init {
        viewModelScope.launch {
            repository.refreshUser("123")
        }
    }
}
```

* UI observes **database Flow** as the **SSOT**.
* Network updates database asynchronously.

---

## ✅ Key Takeaways

* **Single Source of Truth** centralizes data in **one authoritative source**.
* **Room Database** often serves as SSOT for Android apps.
* **Repositories** coordinate network + database updates.
* Benefits: **consistency, offline support, testability, and simplified data flow**.

---

