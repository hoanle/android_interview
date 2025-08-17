
---

# 🧩 Test Doubles in Android

**Test doubles** are objects that **stand in for real dependencies** during testing. They help **isolate the unit under test** and control its interactions with external components.

There are several types of test doubles, each with a specific purpose.

---

## 1️⃣ Mocks

### Definition

* Objects that **record interactions** and allow verification of method calls.
* Often created with **Mockito, MockK, or other mocking frameworks**.

### Use Case

* Verify that a **dependency is called correctly**.
* Test **behavior or interactions** rather than the result.

### Example (MockK)

```kotlin
val repository: UserRepository = mockk()
val viewModel = UserViewModel(repository)

every { repository.getUsers() } returns listOf(User("Alice"))

viewModel.loadUsers()

verify { repository.getUsers() }  // Ensure getUsers() was called
```

---

## 2️⃣ Stubs

### Definition

* Objects that **provide pre-defined responses** to method calls.
* Do not track interactions.

### Use Case

* Provide **controlled data** for the unit under test.
* Focus on **state/output testing**, not behavior verification.

### Example

```kotlin
class UserRepositoryStub : UserRepository {
    override fun getUsers() = listOf(User("Alice"), User("Bob"))
}

val stubRepo = UserRepositoryStub()
val viewModel = UserViewModel(stubRepo)
viewModel.loadUsers()
assertEquals(2, viewModel.users.value?.size)
```

---

## 3️⃣ Fakes

### Definition

* Objects with a **working implementation**, but simpler or in-memory.
* Can be **used for real operations without external dependencies**.

### Use Case

* Replace **real components like databases, network services, or caches** during tests.
* Useful for **integration-style unit tests**.

### Example: In-memory Room database

```kotlin
val db = Room.inMemoryDatabaseBuilder(context, AppDatabase::class.java).build()
val userDao = db.userDao()
userDao.insert(User(1, "Alice"))
val users = userDao.getAllUsers()
assertEquals(1, users.size)
```

---

## 4️⃣ Spies

### Definition

* Objects that **wrap real instances** and allow **partial mocking**.
* Can **record interactions** and optionally override some behavior.

### Use Case

* Verify interactions on real objects while keeping some real behavior.
* Useful when testing **side effects or method calls** on actual implementations.

### Example (MockK)

```kotlin
val realRepo = UserRepositoryImpl()
val spyRepo = spyk(realRepo)

spyRepo.getUsers() // Calls the real method
verify { spyRepo.getUsers() } // Verify it was called
```

---

## 5️⃣ Comparison Table

| Type     | Purpose                              | Tracks Interactions? | Implementation         | Use Case                                        |
| -------- | ------------------------------------ | -------------------- | ---------------------- | ----------------------------------------------- |
| **Mock** | Behavior verification                | ✅                    | Fake object            | Verify method calls                             |
| **Stub** | Provide canned responses             | ❌                    | Minimal implementation | Control returned data                           |
| **Fake** | Lightweight working implementation   | ❌                    | Simplified real logic  | Replace real component (DB, API)                |
| **Spy**  | Partial real object with observation | ✅/Partial            | Wraps real object      | Verify interactions while keeping real behavior |

---

## 6️⃣ Summary

* **Mocks**: Verify interactions.
* **Stubs**: Provide canned responses for controlled testing.
* **Fakes**: Simplified working implementation, often in-memory.
* **Spies**: Wrap real objects to observe interactions while keeping behavior.

**Best Practice:**

* Use **stubs/fakes** for unit testing logic and **integration testing**.
* Use **mocks/spies** for **behavior verification**.
* Choose based on whether you care about **output/state** or **interaction/behavior**.

---

