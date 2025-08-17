
---

# 🏷️ Hilt Qualifiers: Distinguishing Multiple Dependencies

In Hilt (and Dagger), **qualifiers** are used to **differentiate between multiple bindings of the same type**. Without a qualifier, Hilt cannot distinguish which dependency to inject when multiple implementations exist.

---

## 1️⃣ Why Qualifiers Are Needed

Consider this scenario:

```kotlin
@Provides
fun provideLocalDatabase(): Database = Room.databaseBuilder(...).build()

@Provides
fun provideRemoteDatabase(): Database = RetrofitDatabase(...)
```

* Both provide a `Database` instance.
* If you try to inject `Database`, Hilt will throw an **ambiguity error**.
* **Solution:** Use **qualifiers** to distinguish between them.

---

## 2️⃣ Built-in Qualifier: @Named

The simplest way to differentiate is using `@Named("name")`.

### Example

```kotlin
@Module
@InstallIn(SingletonComponent::class)
object DatabaseModule {

    @Singleton
    @Provides
    @Named("local")
    fun provideLocalDatabase(): Database = Room.databaseBuilder(...).build()

    @Singleton
    @Provides
    @Named("remote")
    fun provideRemoteDatabase(): Database = RetrofitDatabase(...)
}

// Injecting in ViewModel
@HiltViewModel
class MainViewModel @Inject constructor(
    @Named("local") private val localDb: Database,
    @Named("remote") private val remoteDb: Database
) : ViewModel()
```

* `@Named("local")` and `@Named("remote")` **tell Hilt which instance to inject**.

---

## 3️⃣ Custom Qualifiers

For better type safety and readability, you can create **custom qualifiers** instead of using strings.

### Example

```kotlin
@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class LocalDb

@Qualifier
@Retention(AnnotationRetention.BINARY)
annotation class RemoteDb

@Module
@InstallIn(SingletonComponent::class)
object DatabaseModule {

    @Singleton
    @Provides
    @LocalDb
    fun provideLocalDatabase(): Database = Room.databaseBuilder(...).build()

    @Singleton
    @Provides
    @RemoteDb
    fun provideRemoteDatabase(): Database = RetrofitDatabase(...)
}

// Injecting in ViewModel
@HiltViewModel
class MainViewModel @Inject constructor(
    @LocalDb private val localDb: Database,
    @RemoteDb private val remoteDb: Database
) : ViewModel()
```

**Advantages of Custom Qualifiers:**

* **Compile-time safety** (no risk of typos in `@Named`).
* **Clearer intent** in code.
* Easier to **refactor and maintain** in large projects.

---

## 4️⃣ When to Use Qualifiers

* When you have **multiple bindings of the same type**.
* When **two or more dependencies share the same interface**.
* When injecting **different configurations of the same class** (e.g., local vs remote, debug vs prod).

---

## 5️⃣ Summary

* **Qualifiers** allow Hilt to distinguish between **multiple dependencies of the same type**.
* **Built-in:** `@Named("name")` – quick and simple.
* **Custom:** `@Qualifier` – type-safe and self-documenting.
* **Use cases:** multiple implementations, different configurations, or same type dependencies.

---

