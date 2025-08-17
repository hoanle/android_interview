
---

# 🧪 Unit Testing in Android

Unit testing focuses on testing **individual components in isolation**. In Android, typical unit test targets include **ViewModels, UseCases, and Repositories**. Dependencies are usually **mocked or faked** to ensure tests are deterministic and fast.

---

## 1️⃣ Unit Testing a ViewModel

### Key Idea

* Test the **logic of the ViewModel** without relying on the actual repository, network, or database.
* Mock or fake **dependencies** like UseCases, Repositories, or other services.

### Example: ViewModel

```kotlin
@HiltViewModel
class UserViewModel @Inject constructor(
    private val getUsersUseCase: GetUsersUseCase
) : ViewModel() {
    private val _users = MutableLiveData<List<User>>()
    val users: LiveData<List<User>> = _users

    fun loadUsers() {
        _users.value = getUsersUseCase.execute()
    }
}
```

### Unit Test

```kotlin
class UserViewModelTest {

    private lateinit var viewModel: UserViewModel
    private val getUsersUseCase: GetUsersUseCase = mockk() // Using MockK

    @Before
    fun setup() {
        viewModel = UserViewModel(getUsersUseCase)
    }

    @Test
    fun `loadUsers updates LiveData`() {
        val fakeUsers = listOf(User("Alice"), User("Bob"))
        every { getUsersUseCase.execute() } returns fakeUsers

        viewModel.loadUsers()

        assertEquals(fakeUsers, viewModel.users.getOrAwaitValue())
    }
}
```

**Key Points:**

* Dependencies (`GetUsersUseCase`) are mocked.
* Test **ViewModel behavior**, not actual repository/network.
* Can test **LiveData** or **StateFlow** with helper extensions like `getOrAwaitValue` or `Turbine`.

---

## 2️⃣ Unit Testing a UseCase

### Key Idea

* Test the **business logic** of the UseCase in isolation.
* Dependencies like repositories can be **mocked**.

### Example: UseCase

```kotlin
class GetUsersUseCase(private val repository: UserRepository) {
    fun execute(): List<User> = repository.getUsers()
}
```

### Unit Test

```kotlin
class GetUsersUseCaseTest {

    private val repository: UserRepository = mockk()
    private lateinit var useCase: GetUsersUseCase

    @Before
    fun setup() {
        useCase = GetUsersUseCase(repository)
    }

    @Test
    fun `execute returns list of users`() {
        val fakeUsers = listOf(User("Alice"))
        every { repository.getUsers() } returns fakeUsers

        val result = useCase.execute()

        assertEquals(fakeUsers, result)
    }
}
```

---

## 3️⃣ Unit Testing a Repository

### Key Idea

* Test repositories in isolation with **in-memory or fake data sources**.
* Use **Room in-memory database**, **Retrofit MockWebServer**, or **mocked network layers**.

### Example: Repository Test with Fake DataSource

```kotlin
class UserRepositoryTest {

    private val localDataSource = FakeLocalDataSource()
    private val remoteDataSource = FakeRemoteDataSource()
    private lateinit var repository: UserRepository

    @Before
    fun setup() {
        repository = UserRepository(localDataSource, remoteDataSource)
    }

    @Test
    fun `getUsers returns combined users from local and remote`() {
        val result = repository.getUsers()
        assertTrue(result.containsAll(localDataSource.users))
        assertTrue(result.containsAll(remoteDataSource.users))
    }
}
```

---

## 4️⃣ Common Libraries for Unit Testing

| Library                        | Purpose                                                                |
| ------------------------------ | ---------------------------------------------------------------------- |
| **JUnit**                      | Core testing framework for defining and running tests.                 |
| **Mockito / MockK**            | Mocking dependencies for isolation. MockK is Kotlin-friendly.          |
| **Turbine**                    | Testing **Flows** in Kotlin Coroutines.                                |
| **Truth / Hamcrest / AssertJ** | Assertion libraries for readable test assertions.                      |
| **Robolectric**                | JVM-based Android component testing (optional for View/Context tests). |

---

## 5️⃣ Best Practices

1. **Isolate the component**: Mock all dependencies.
2. **Test one behavior at a time**: Each test should focus on a single method or scenario.
3. **Prefer in-memory or fake data sources** for repositories.
4. **Test both success and failure cases**.
5. **Use Turbine for Flow testing** to collect emissions and assert values.
6. **Keep tests fast** to run them frequently during development.

---

## 6️⃣ Summary

* **ViewModel Tests**: Focus on UI state and event logic; mock UseCases or Repositories.
* **UseCase Tests**: Focus on business logic; mock repositories.
* **Repository Tests**: Focus on data handling; use fakes or in-memory databases.
* **Libraries**: JUnit, MockK/Mockito, Turbine, and assertion libraries.
* Unit tests form the **foundation of the Android testing pyramid**, ensuring reliability and fast feedback.

---

