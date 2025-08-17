
---

# 🔗 Integration Testing in Android

**Integration tests** focus on testing **how multiple components work together**, rather than testing a single unit in isolation. In Android, this often involves **databases, network services, repositories, and combinations of these layers**.

---

## 1️⃣ What Constitutes an Integration Test

* **Definition:** Tests that verify the **interaction between multiple modules or layers**.
* **Purpose:** Ensure that **components integrate correctly** (e.g., DAO ↔ Room, Repository ↔ API, ViewModel ↔ Repository).
* **Scope:** Larger than unit tests but smaller than full end-to-end tests.
* **Dependencies:** Often use **in-memory databases, test doubles, or local servers**.

---

## 2️⃣ Testing DAOs and Room Database

### Using In-Memory Room Database

* Use an **in-memory database** so that tests do not affect actual app data.
* Execute DAO methods and verify **queries, inserts, updates, deletes**.

### Example:

```kotlin
@RunWith(AndroidJUnit4::class)
class UserDaoTest {

    private lateinit var database: AppDatabase
    private lateinit var userDao: UserDao

    @Before
    fun setup() {
        database = Room.inMemoryDatabaseBuilder(
            ApplicationProvider.getApplicationContext(),
            AppDatabase::class.java
        ).allowMainThreadQueries().build()
        userDao = database.userDao()
    }

    @After
    fun teardown() {
        database.close()
    }

    @Test
    fun insertUser_andGetUserById_returnsInsertedUser() = runBlocking {
        val user = User(id = 1, name = "Alice")
        userDao.insert(user)
        
        val result = userDao.getUserById(1)
        assertEquals(user, result)
    }

    @Test
    fun deleteUser_removesUser() = runBlocking {
        val user = User(id = 2, name = "Bob")
        userDao.insert(user)
        userDao.delete(user)

        val result = userDao.getUserById(2)
        assertNull(result)
    }
}
```

**Key Points:**

* `inMemoryDatabaseBuilder` ensures a **clean database for each test**.
* Test **real SQL queries and relationships** without touching production DB.

---

## 3️⃣ Testing Retrofit Service Integrations

* **Goal:** Verify that **API requests and responses** work as expected.
* **Tool:** `MockWebServer` from OkHttp library.

### Example:

```kotlin
@RunWith(AndroidJUnit4::class)
class ApiServiceTest {

    private lateinit var mockWebServer: MockWebServer
    private lateinit var apiService: ApiService

    @Before
    fun setup() {
        mockWebServer = MockWebServer()
        mockWebServer.start()

        apiService = Retrofit.Builder()
            .baseUrl(mockWebServer.url("/"))
            .addConverterFactory(MoshiConverterFactory.create())
            .build()
            .create(ApiService::class.java)
    }

    @After
    fun teardown() {
        mockWebServer.shutdown()
    }

    @Test
    fun `fetchUsers returns expected data`() = runBlocking {
        val mockResponse = """
            [{"id":1,"name":"Alice"},{"id":2,"name":"Bob"}]
        """.trimIndent()

        mockWebServer.enqueue(MockResponse().setBody(mockResponse).setResponseCode(200))

        val users = apiService.getUsers()
        assertEquals(2, users.size)
        assertEquals("Alice", users[0].name)
    }
}
```

**Key Points:**

* `MockWebServer` allows **simulating HTTP responses** without a real server.
* Can test **error responses, delays, or timeouts** for robust network handling.
* Ensures **Repository → API integration** works before hitting production API.

---

## 4️⃣ Best Practices for Integration Testing

1. **Isolate tests** using in-memory DB and MockWebServer.
2. **Test realistic scenarios** including success and failure cases.
3. **Clean up after tests** to avoid shared state between tests.
4. **Combine layers** for meaningful tests (e.g., Repository + Room + Network).
5. **Keep tests deterministic and repeatable**.

---

## 5️⃣ Summary

| Component              | Example Integration Test                            |
| ---------------------- | --------------------------------------------------- |
| DAO + Room             | Insert, update, delete, query using in-memory DB    |
| Repository + DAO       | Verify Repository correctly fetches and saves data  |
| Repository + API       | Use `MockWebServer` to test API calls and responses |
| ViewModel + Repository | Test state changes after repository operations      |

* **Integration tests** verify **correct interaction between components**.
* They sit **above unit tests** in the testing pyramid and **below full UI/E2E tests**.
* They are **slower than unit tests** but faster than full end-to-end tests.

---

