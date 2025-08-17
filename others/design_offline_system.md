
---

# 🌐 Designing a Multi-API Feature with Offline Support in Android

When designing a feature that fetches data from multiple API endpoints, combines it, and supports offline mode, the architecture should emphasize **clean separation of concerns, resilience, and offline-first behavior**.

---

## 1️⃣ Architectural Overview

We can follow a **Clean + MVVM architecture**:

```
UI Layer (Composable / Fragment / Activity)
       │
       ▼
ViewModel (exposes UI state via StateFlow/LiveData)
       │
       ▼
Domain Layer (UseCases / Interactors)
       │
       ▼
Repository Layer (or NetworkBoundResource)
       │
       ▼
Data Sources:
   - Remote DataSource (Retrofit API services)
   - Local DataSource (Room database / DataStore cache)
```

---

## 2️⃣ Data Flow

1. **UI triggers** data fetch (e.g., ViewModel on screen load).
2. **Repository coordinates** multiple API calls and merges the data.
3. **Merged result** is stored in **local cache (Room / DataStore)**.
4. **ViewModel exposes** state to UI (loading, success, error) using `StateFlow` or `LiveData`.
5. **Offline support:** UI observes local cache and displays data if network is unavailable.

---

## 3️⃣ Combining Multiple API Endpoints

* Use **Kotlin Coroutines + Flow** for asynchronous and concurrent operations.
* `async/await` or `zip`/`combine` operators help **merge results from multiple endpoints**.

```kotlin
suspend fun fetchCombinedData(): CombinedData {
    coroutineScope {
        val api1Deferred = async { apiService.getData1() }
        val api2Deferred = async { apiService.getData2() }

        val data1 = api1Deferred.await()
        val data2 = api2Deferred.await()

        return mergeData(data1, data2)
    }
}

fun mergeData(data1: List<Item1>, data2: List<Item2>): CombinedData {
    // Combine data logically
}
```

* **Flow operators**:

```kotlin
val combinedFlow = combine(api1Flow, api2Flow) { data1, data2 ->
    mergeData(data1, data2)
}
```

---

## 4️⃣ Offline-First Strategy

* Implement a **NetworkBoundResource**-like pattern:

```kotlin
fun getCombinedData(): Flow<Resource<CombinedData>> = flow {
    emit(Resource.Loading())

    // Step 1: Load cached data
    val cached = localDataSource.getCombinedData()
    if (cached.isNotEmpty()) emit(Resource.Success(cached))

    try {
        // Step 2: Fetch from remote APIs concurrently
        val remoteData = repository.fetchCombinedData()
        // Step 3: Update local cache
        localDataSource.saveCombinedData(remoteData)
        emit(Resource.Success(remoteData))
    } catch (e: Exception) {
        emit(Resource.Error("Failed to fetch data"))
    }
}
```

**Advantages:**

* **UI shows cached data immediately**.
* Network errors handled gracefully.
* Supports **offline mode** seamlessly.

---

## 5️⃣ Handling Caching and Synchronization

* **Room Database** for structured data.
* Use **`@Transaction`** for atomic writes.
* **Data freshness strategy**:

  * Timestamp-based cache invalidation.
  * Background refresh with **WorkManager** or **Pull-to-Refresh**.

---

## 6️⃣ ViewModel & UI State

* Use **`StateFlow`** or **`LiveData`** for exposing states:

```kotlin
class CombinedViewModel(private val repository: Repository) : ViewModel() {
    val combinedData: StateFlow<Resource<CombinedData>> =
        repository.getCombinedData()
            .stateIn(viewModelScope, SharingStarted.Lazily, Resource.Loading())
}
```

* **UI Layer (Compose example)**:

```kotlin
val state by viewModel.combinedData.collectAsState()

when(state) {
    is Resource.Loading -> CircularProgressIndicator()
    is Resource.Success -> DataList((state as Resource.Success).data)
    is Resource.Error -> ErrorMessage((state as Resource.Error).message)
}
```

---

## 7️⃣ Error Handling and Retries

* Use **`retry` operator** for transient network errors.
* Show **offline indicators** in UI when data is from cache.
* Provide **manual refresh** option for users.

---

## 8️⃣ Testing

* **Unit test repositories**: mock multiple APIs and local DB.
* **Integration test**: verify proper combination and caching.
* **UI test**: simulate offline and online scenarios.

---

## ✅ Summary

* **Concurrent API fetching:** Use coroutines/Flow (`async`, `combine`).
* **Offline-first:** Cache combined results in Room.
* **UI state management:** Use `StateFlow` / `LiveData`.
* **Error handling:** Emit cached data on network failure.
* **Architecture:** MVVM + Clean Architecture + Repository pattern ensures modularity, testability, and maintainability.

---
