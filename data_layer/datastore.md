
---

# 🗄️ Jetpack DataStore in Android

Jetpack **DataStore** is a modern, asynchronous, and **type-safe data storage solution** for Android, designed to replace **SharedPreferences**. It provides better **thread safety**, **reactive APIs**, and **structured data support**.

---

## 1️⃣ Types of DataStore

### a. **Preferences DataStore**

* Stores **key-value pairs**, similar to SharedPreferences.
* Keys are strongly typed using `preferencesKey`.
* Does **not enforce a schema**—keys and values are independent.
* Ideal for **simple settings and flags**.

**Example:**

```kotlin
val Context.dataStore: DataStore<Preferences> by preferencesDataStore(name = "settings")

suspend fun saveDarkMode(enabled: Boolean) {
    context.dataStore.edit { prefs ->
        prefs[booleanPreferencesKey("dark_mode")] = enabled
    }
}

val darkModeFlow: Flow<Boolean> = context.dataStore.data
    .map { prefs -> prefs[booleanPreferencesKey("dark_mode")] ?: false }
```

---

### b. **Proto DataStore**

* Stores **typed objects** using **Protocol Buffers (protobuf)**.
* Enforces **schema and type safety**, ideal for **structured or complex data**.
* Useful for **user profiles, settings objects, or multi-field configurations**.

**Proto Definition:**

```proto
syntax = "proto3";

message UserPreferences {
    bool dark_mode = 1;
    string username = 2;
}
```

**Kotlin Usage:**

```kotlin
val Context.userPreferencesStore: DataStore<UserPreferences> by dataStore(
    fileName = "user_prefs.pb",
    serializer = UserPreferencesSerializer
)

val userPrefsFlow: Flow<UserPreferences> = context.userPreferencesStore.data
```

---

## 2️⃣ Comparison with SharedPreferences

| Feature          | SharedPreferences   | Preferences DataStore                     | Proto DataStore                                                    |
| ---------------- | ------------------- | ----------------------------------------- | ------------------------------------------------------------------ |
| API Type         | Synchronous         | Asynchronous (Flow + suspend)             | Asynchronous (Flow + suspend)                                      |
| Thread Safety    | Manual              | Built-in thread-safe                      | Built-in thread-safe                                               |
| Data Type        | Primitive key-value | Primitive key-value                       | Typed objects (protobuf)                                           |
| Error Handling   | Minimal             | Exception-safe                            | Exception-safe                                                     |
| Schema           | None                | None                                      | Enforced via proto                                                 |
| Reactive Updates | No                  | Yes, via Flow                             | Yes, via Flow                                                      |
| Migration        | Manual              | Supports migration from SharedPreferences | Supports migration from SharedPreferences or Preferences DataStore |

**Advantages of DataStore over SharedPreferences:**

* **Non-blocking I/O** (suspend functions + Flow)
* **Thread-safe** by design
* **Type-safe** access (especially Proto DataStore)
* **Reactive updates** with Kotlin Flow
* **Better error handling** and corruption recovery

---

## 3️⃣ When to Choose Proto DataStore

| Scenario                                                       | Recommended DataStore |
| -------------------------------------------------------------- | --------------------- |
| Simple key-value preferences (theme, toggle flags)             | Preferences DataStore |
| Structured, multi-field data (user profile, settings object)   | Proto DataStore       |
| Need **type safety** and **compile-time schema checks**        | Proto DataStore       |
| Expect **future schema evolution** with backward compatibility | Proto DataStore       |

**Rule of thumb:**

* Use **Preferences DataStore** for small, simple settings.
* Use **Proto DataStore** when data is **structured, complex, or requires schema enforcement**.

---

## ✅ Summary

* **DataStore** is a modern alternative to SharedPreferences.
* **Preferences DataStore** → simple key-value storage.
* **Proto DataStore** → typed, schema-enforced storage using protobuf.
* **Advantages**: thread safety, reactive updates, non-blocking, type safety, better error handling.
* **Decision**: choose **Proto** for structured data, **Preferences** for simple flags or small settings.

---
