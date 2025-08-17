
---

# 📄 Implementing Pagination in Android with Paging 3

Loading **large datasets** efficiently is crucial for performance and user experience. **Paging 3** (Jetpack) provides a **robust, reactive, and lifecycle-aware solution** for paginated data loading.

---

## 1️⃣ Why Use Paging?

* Avoids **loading all data at once** → reduces memory usage.
* Supports **incremental loading** as the user scrolls.
* Handles **network + database integration** seamlessly.
* Works with both **RecyclerView (Adapter)** and **Jetpack Compose**.

---

## 2️⃣ Key Components of Paging 3

### a. **PagingSource**

* Defines **how to load data from a single source** (network or database).
* Extends `PagingSource<Key, Value>`.
* Implements `load()` method to **fetch pages** based on a key (offset, page number, or token).

**Example (Network PagingSource):**

```kotlin
class UserPagingSource(
    private val api: UserApi
) : PagingSource<Int, User>() {

    override suspend fun load(params: LoadParams<Int>): LoadResult<Int, User> {
        val page = params.key ?: 1
        return try {
            val response = api.getUsers(page, params.loadSize)
            LoadResult.Page(
                data = response.users,
                prevKey = if (page == 1) null else page - 1,
                nextKey = if (response.users.isEmpty()) null else page + 1
            )
        } catch (e: Exception) {
            LoadResult.Error(e)
        }
    }
}
```

---

### b. **RemoteMediator**

* Integrates **network + database** for **offline caching**.
* Helps **merge remote and local data**.
* Used with **PagingSource from Room** to **sync data automatically**.

**Example:**

```kotlin
@OptIn(ExperimentalPagingApi::class)
class UserRemoteMediator(
    private val api: UserApi,
    private val dao: UserDao
) : RemoteMediator<Int, UserEntity>() {

    override suspend fun load(
        loadType: LoadType,
        state: PagingState<Int, UserEntity>
    ): MediatorResult {
        try {
            val page = when (loadType) {
                LoadType.REFRESH -> 1
                LoadType.PREPEND -> return MediatorResult.Success(endOfPaginationReached = true)
                LoadType.APPEND -> state.pages.lastOrNull()?.nextKey ?: 1
            }

            val response = api.getUsers(page, state.config.pageSize)
            dao.insertUsers(response.users.map { it.toEntity() })
            return MediatorResult.Success(endOfPaginationReached = response.users.isEmpty())
        } catch (e: Exception) {
            return MediatorResult.Error(e)
        }
    }
}
```

---

### c. **Pager**

* Coordinates **PagingSource or RemoteMediator** with **PagingData**.
* Configures **page size, placeholders, prefetch distance**, etc.

**Example:**

```kotlin
val pager = Pager(
    config = PagingConfig(pageSize = 20, enablePlaceholders = false),
    remoteMediator = UserRemoteMediator(api, dao),
    pagingSourceFactory = { dao.getAllUsersPagingSource() }
).flow
```

* Returns a `Flow<PagingData<T>>` that **emits pages as data loads**.

---

### d. **PagingDataAdapter / Compose Paging**

* **RecyclerView Adapter** for PagingData (`PagingDataAdapter`).
* **Jetpack Compose** uses `collectAsLazyPagingItems()` for LazyColumn.

**Example (Compose):**

```kotlin
@Composable
fun UserList(userPager: Flow<PagingData<User>>) {
    val lazyPagingItems = userPager.collectAsLazyPagingItems()

    LazyColumn {
        items(lazyPagingItems) { user ->
            user?.let {
                Text(text = user.name)
            }
        }

        lazyPagingItems.apply {
            when {
                loadState.refresh is LoadState.Loading -> item { CircularProgressIndicator() }
                loadState.append is LoadState.Loading -> item { CircularProgressIndicator() }
                loadState.refresh is LoadState.Error -> item { Text("Error") }
            }
        }
    }
}
```

---

## 3️⃣ Benefits of Paging 3

* **Memory Efficient** → only loads visible items + prefetch buffer.
* **Reactive & Coroutine Friendly** → integrates with Kotlin Flow.
* **Supports Offline Cache** → via RemoteMediator + Room.
* **Error Handling & Retry** → built-in `LoadState` management.
* **Composable Ready** → works seamlessly with LazyColumn/LazyRow.

---

## 4️⃣ Architecture Flow

```text
       +-------------+
       |  UI Layer   |
       | (LazyColumn)|
       +------+------+
              |
              v
       +-------------+
       | PagingData  |
       | Flow Stream |
       +------+------+
              |
     ---------------------
     |                   |
PagingSource        RemoteMediator
(Network / DB)     (Sync Network + DB)
     |                   |
     ---------------------
              |
              v
         Local DB / Network
```

* UI observes **PagingData Flow**.
* **PagingSource** fetches pages.
* **RemoteMediator** ensures **offline caching + sync**.
* **Pager** orchestrates the flow.

---

## ✅ Summary

* **PagingSource** → defines how to load pages from a single source.
* **RemoteMediator** → merges remote and local data, handles caching.
* **Pager** → orchestrates PagingData emission.
* **PagingDataAdapter / Compose Paging** → displays paginated data in the UI.
* **Paging 3** provides **efficient, reactive, and offline-ready pagination**.

---

