
---
# Using Retrofit to Interact with a REST API

Retrofit is a type-safe HTTP client for Android and Java developed by Square. It simplifies API integration by mapping REST endpoints to Java/Kotlin interfaces and converting JSON or XML responses into Kotlin/Java objects.

---

## 1. Setting Up Retrofit

1. **Add Dependencies** (Gradle example):

```gradle
implementation 'com.squareup.retrofit2:retrofit:2.9.0'
implementation 'com.squareup.retrofit2:converter-gson:2.9.0' // JSON converter
implementation 'com.squareup.okhttp3:logging-interceptor:4.9.0' // Optional for logging
````

2. **Define API Interface**

```kotlin
interface ApiService {
    @GET("users/{id}")
    suspend fun getUser(@Path("id") userId: String): Response<User>

    @POST("users")
    suspend fun createUser(@Body user: User): Response<User>
}
```

3. **Create Retrofit Instance**

```kotlin
val retrofit = Retrofit.Builder()
    .baseUrl("https://api.example.com/")
    .addConverterFactory(GsonConverterFactory.create()) // Converts JSON to objects
    .build()

val apiService = retrofit.create(ApiService::class.java)
```

4. **Make API Calls**

```kotlin
try {
    val response = apiService.getUser("123")
    if (response.isSuccessful) {
        val user = response.body()
        // Use the user object
    } else {
        // Handle HTTP errors (like 404 or 500)
        val errorBody = response.errorBody()?.string()
    }
} catch (e: IOException) {
    // Handle network errors (timeout, no connection)
}
```

---

## 2. Handling API Errors

Retrofit separates **HTTP errors** from **network errors**:

* **HTTP Errors**: Occur when the server responds with a non-2xx status code.

  ```kotlin
  if (!response.isSuccessful) {
      // HTTP error code: response.code()
      // Error response body: response.errorBody()
  }
  ```

* **Network Errors**: IOException occurs when there is no internet or timeout.

  ```kotlin
  try {
      val response = apiService.getUser("123")
  } catch (e: IOException) {
      // Handle network issues
  }
  ```

* **Different Response Types**:
  Retrofit allows returning:

  * `Response<T>` → gives both body and HTTP info.
  * `T` → throws an exception for non-2xx responses.
  * `Call<T>` → can execute synchronously (`execute()`) or asynchronously (`enqueue()`).

---

## 3. Retrofit Interceptors

**Interceptors** are components that can monitor, rewrite, and retry HTTP requests and responses. They are part of OkHttp (Retrofit uses OkHttp internally).

* **Use cases:**

  * **Logging**: Print request and response for debugging.
  * **Adding headers**: e.g., authentication token.
  * **Retry logic**: Automatically retry failed requests.

```kotlin
val logging = HttpLoggingInterceptor().apply {
    level = HttpLoggingInterceptor.Level.BODY
}

val client = OkHttpClient.Builder()
    .addInterceptor(logging)
    .addInterceptor { chain ->
        val request = chain.request().newBuilder()
            .addHeader("Authorization", "Bearer TOKEN")
            .build()
        chain.proceed(request)
    }
    .build()

val retrofit = Retrofit.Builder()
    .baseUrl("https://api.example.com/")
    .client(client)
    .addConverterFactory(GsonConverterFactory.create())
    .build()
```

---

## 4. Retrofit ConverterFactory

**ConverterFactory** is responsible for serializing request bodies and deserializing response bodies.

* **Use cases:**

  * `GsonConverterFactory` → Convert JSON responses to Kotlin/Java objects.
  * `MoshiConverterFactory` → Use Moshi for JSON parsing.
  * `ScalarsConverterFactory` → For plain text responses.
  * `ProtoConverterFactory` → For Protocol Buffers.

Example with Gson:

```kotlin
val retrofit = Retrofit.Builder()
    .baseUrl("https://api.example.com/")
    .addConverterFactory(GsonConverterFactory.create())
    .build()
```

---

## Summary

| Concept                    | Purpose / Use Case                                                      |
| -------------------------- | ----------------------------------------------------------------------- |
| **Retrofit API Interface** | Maps REST endpoints to Kotlin/Java methods                              |
| **Response Handling**      | `isSuccessful` checks HTTP status; `IOException` catches network errors |
| **Interceptor**            | Modify requests/responses, add headers, logging, retry logic            |
| **ConverterFactory**       | Converts JSON/XML/plain text to Kotlin/Java objects                     |

By combining Retrofit with Interceptors and ConverterFactories, you get a **flexible, type-safe, and maintainable way to consume REST APIs** in Android or Java applications.

---
