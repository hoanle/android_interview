
---

# 🏛️ Room Database in Android

Room is a **SQLite abstraction library** in Android that provides **compile-time checks, type safety, and a reactive API** for database operations. Its main components include **Entity, DAO, and Database**.

---

## 1️⃣ Main Components

### a. **Entity**

* Represents a **table in the database**.
* Annotated with `@Entity`.
* Each property corresponds to a **column**.

```kotlin
@Entity(tableName = "users")
data class UserEntity(
    @PrimaryKey val id: String,
    val name: String,
    val email: String
)
```

---

### b. **DAO (Data Access Object)**

* Provides **methods to interact with the database**.
* Annotated with `@Dao`.
* Supports `@Insert`, `@Update`, `@Delete`, and `@Query`.

```kotlin
@Dao
interface UserDao {
    @Insert(onConflict = OnConflictStrategy.REPLACE)
    suspend fun insert(user: UserEntity)

    @Query("SELECT * FROM users WHERE id = :userId")
    fun getUserFlow(userId: String): Flow<UserEntity>

    @Query("DELETE FROM users")
    suspend fun clearAll()
}
```

---

### c. **Database**

* Annotated with `@Database`.
* Connects **entities and DAOs**.
* Provides a singleton **RoomDatabase instance**.

```kotlin
@Database(entities = [UserEntity::class], version = 2)
abstract class AppDatabase : RoomDatabase() {
    abstract fun userDao(): UserDao
}
```

---

## 2️⃣ Database Migrations

* When **schema changes**, Room requires **version increment**.
* Use `Migration` objects to handle changes without losing data.

```kotlin
val MIGRATION_1_2 = object : Migration(1, 2) {
    override fun migrate(database: SupportSQLiteDatabase) {
        database.execSQL("ALTER TABLE users ADD COLUMN age INTEGER DEFAULT 0 NOT NULL")
    }
}

val db = Room.databaseBuilder(context, AppDatabase::class.java, "app-db")
    .addMigrations(MIGRATION_1_2)
    .build()
```

---

## 3️⃣ Observing Data Changes with Kotlin Flow

* Room **supports reactive streams** using `Flow`.
* UI automatically updates when data changes.

```kotlin
@Query("SELECT * FROM users")
fun getAllUsersFlow(): Flow<List<UserEntity>>

// Collect in ViewModel
val usersFlow: StateFlow<List<UserEntity>> = userDao.getAllUsersFlow()
    .stateIn(viewModelScope, SharingStarted.Lazily, emptyList())
```

---

## 4️⃣ Type Converters

* Room only supports **primitive types** natively.
* Use `@TypeConverter` to **store custom types** in the database.

```kotlin
class Converters {
    @TypeConverter
    fun fromDate(date: Date): Long = date.time

    @TypeConverter
    fun toDate(time: Long): Date = Date(time)
}

@Database(entities = [UserEntity::class], version = 2)
@TypeConverters(Converters::class)
abstract class AppDatabase : RoomDatabase() { ... }
```

---

## 5️⃣ Complex Queries & Transactions

### a. Complex Queries

* Use **`@Query` with SQL** to join tables, filter, sort, or aggregate.

```kotlin
@Query("""
    SELECT users.*, COUNT(posts.id) as postCount
    FROM users
    LEFT JOIN posts ON users.id = posts.userId
    GROUP BY users.id
    ORDER BY postCount DESC
""")
fun getUsersWithPostCount(): Flow<List<UserWithPostCount>>
```

### b. Transactions

* Ensure **atomic operations** using `@Transaction`.

```kotlin
@Transaction
suspend fun refreshUsers(users: List<UserEntity>) {
    clearAll()
    users.forEach { insert(it) }
}
```

---

## ✅ Key Takeaways

* **Entity** → defines table structure.
* **DAO** → defines database operations.
* **Database** → connects DAOs and entities.
* **Migrations** → safely evolve schema without data loss.
* **Flow** → observe real-time data updates.
* **Type Converters** → store custom types.
* **Transactions** → ensure atomic operations.
* Supports **complex queries, joins, and aggregations** using SQL annotations.

---

