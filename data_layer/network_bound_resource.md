
---

# 🌐 NetworkBoundResource Pattern in Android

The **NetworkBoundResource (NBR) pattern** is a common architecture pattern used to **coordinate between local and remote data sources**, providing a **single source of truth (SSOT)** while supporting offline-first behavior.

It is widely used in **MVVM / Clean Architecture apps** with Room + Retrofit + Kotlin Flow or LiveData.

---

## 1️⃣ Core Idea

* **Local database** is the SSOT.
* **Network fetch** happens only when necessary (cache miss, data stale, or refresh).
* UI observes the **local database**, while the repository manages **syncing with the network**.
* **Emits states**: Loading, Success, Error.

```text
      +----------------+
      |      UI        |
      +-------+--------+
              |
        Observe Data
              v
      +----------------+
      |   Repository   |
      | NetworkBoundRes|
      +-------+--------+
              |
     ----------------------
     |                    |
 Local DB               Network API
```

---

## 2️⃣ Responsibilities of NetworkBoundResource

1. **Load from DB**

   * Initially, return **cached data** from local database.

2. **Decide whether to fetch from Network**

   * Example: data is stale or user triggers refresh.

3. **Fetch from Network**

   * Make API call via Retrofit (or any network client).

4. **Save Network Response to DB**

   * Transform and insert/update database, so UI sees updated data automatically.

5. **Emit Resource / UI State**

   * Typical states: `Loading`, `Success`, `Error`.

---

## 3️⃣ Typical Implementation (Kotlin + Flow)

```kotlin
sealed class Resource<T> {
    data class Success<T>(val data: T): Resource<T>()
    data class Loading<T>(val data: T? = null): Resource<T>()
    data class Error<T>(val message: String, val data: T? = null): Resource<T>()
}

fun <T, A> networkBoundResource(
    query: () -> Flow<T>,
    fetch: suspend () -> A,
    saveFetchResult: suspend (A) -> Unit,
    shouldFetch: (T) -> Boolean = { true }
): Flow<Resource<T>> = flow {
    emit(Resource.Loading())
    val data = query().first() // get current cached data
    if (shouldFetch(data)) {
        try {
            val response = fetch()
            saveFetchResult(response)
            emitAll(query().map { Resource.Success(it) })
        } catch (e: Exception) {
            emitAll(query().map { Resource.Error(e.message ?: "Unknown Error", it) })
        }
    } else {
        emitAll(query().map { Resource.Success(it) })
    }
}
```

**Usage in Repository:**

```kotlin
class UserRepository(
    private val api: UserApi,
    private val dao: UserDao
) {
    fun getUser(userId: String): Flow<Resource<User>> =
        networkBoundResource(
            query = { dao.getUserFlow(userId) },
            fetch = { api.getUser(userId) },
            saveFetchResult = { dao.insert(it.toEntity()) },
            shouldFetch = { it == null } // fetch if local data is null
        )
}
```

---

## 4️⃣ Advantages

* **Single Source of Truth** → UI always observes local database.
* **Offline-first** → works seamlessly when network is unavailable.
* **Automatic caching** → network updates automatically stored in DB.
* **Reactive UI** → updates emitted via Flow or LiveData.
* **Simplifies state handling** → encapsulates loading, success, error states.

---

## 5️⃣ Considerations

* **Stale Data Strategy** → define when to fetch network (time-based, manual refresh).
* **Error Handling** → can show cached data along with error messages.
* **Mapping & Transformation** → network DTOs often need to be mapped to local entities.
* **Pagination** → integrate with Paging 3 when datasets are large.

---

## ✅ Key Takeaways

* **NetworkBoundResource** is a **repository-level pattern** for **syncing local + remote data**.
* **Local DB is the source of truth**, network updates DB.
* Provides **consistent, reactive, offline-capable UI**.
* Encapsulates **loading, success, and error states**, reducing boilerplate.

---
