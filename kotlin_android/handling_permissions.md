
---

# 🔐 Runtime Permissions in Android

Since **Android 6.0 (Marshmallow, API 23)**, **dangerous permissions** must be requested at runtime instead of only being declared in the `AndroidManifest.xml`. This allows users to have **more control over sensitive data and features**.

---

## 1️⃣ Permission Types

| Type                      | Examples                               | Notes                                                  |
| ------------------------- | -------------------------------------- | ------------------------------------------------------ |
| **Normal Permissions**    | INTERNET, ACCESS\_WIFI\_STATE          | Granted automatically at install time.                 |
| **Dangerous Permissions** | CAMERA, READ\_CONTACTS, LOCATION       | Must be requested at runtime.                          |
| **Special Permissions**   | SYSTEM\_ALERT\_WINDOW, WRITE\_SETTINGS | Requires explicit settings screen or special handling. |

---

## 2️⃣ Handling Runtime Permissions

### Step 1: Declare Permission in Manifest

```xml
<uses-permission android:name="android.permission.CAMERA"/>
```

### Step 2: Check if Permission is Granted

```kotlin
if (ContextCompat.checkSelfPermission(this, Manifest.permission.CAMERA)
    == PackageManager.PERMISSION_GRANTED) {
    // Permission already granted
} else {
    // Request permission
}
```

### Step 3: Request Permission

```kotlin
ActivityCompat.requestPermissions(
    this,
    arrayOf(Manifest.permission.CAMERA),
    REQUEST_CAMERA_CODE
)
```

### Step 4: Handle Permission Result

```kotlin
override fun onRequestPermissionsResult(
    requestCode: Int,
    permissions: Array<out String>,
    grantResults: IntArray
) {
    super.onRequestPermissionsResult(requestCode, permissions, grantResults)
    if (requestCode == REQUEST_CAMERA_CODE) {
        if (grantResults.isNotEmpty() && grantResults[0] == PackageManager.PERMISSION_GRANTED) {
            // Permission granted, proceed with camera
        } else {
            // Permission denied, show rationale or disable feature
        }
    }
}
```

---

## 3️⃣ Showing Permission Rationale

* Users may **deny permission** or select “Don’t ask again”.
* Use `shouldShowRequestPermissionRationale()` to **explain why the permission is needed**.

```kotlin
if (ActivityCompat.shouldShowRequestPermissionRationale(this, Manifest.permission.CAMERA)) {
    // Show dialog explaining why camera is needed
}
```

---

## 4️⃣ Using Activity Result API (Modern Approach)

AndroidX provides **`ActivityResultContracts.RequestPermission`** for a cleaner API:

```kotlin
private val requestPermissionLauncher =
    registerForActivityResult(ActivityResultContracts.RequestPermission()) { isGranted ->
        if (isGranted) {
            // Permission granted
        } else {
            // Permission denied
        }
    }

// Request permission
requestPermissionLauncher.launch(Manifest.permission.CAMERA)
```

**Advantages:**

* No need to override `onRequestPermissionsResult`.
* Lifecycle-aware and safer for **Fragments & Compose**.

---

## 5️⃣ Permissions in Jetpack Compose

* Use `Accompanist Permissions` library for **Compose-friendly permission handling**:

```kotlin
val cameraPermissionState = rememberPermissionState(Manifest.permission.CAMERA)

Button(onClick = { cameraPermissionState.launchPermissionRequest() }) {
    Text("Request Camera Permission")
}
```

---

## 6️⃣ Best Practices

1. **Only request necessary permissions** to increase trust.
2. **Explain rationale** before requesting dangerous permissions.
3. **Handle denial gracefully**; provide alternative UX or disable feature.
4. **Use modern Activity Result API** or Compose-friendly libraries for cleaner code.
5. **Check permissions every time** before performing sensitive operations.

---

## 7️⃣ Summary

* Runtime permissions provide **user control over sensitive features**.
* Dangerous permissions must be **checked and requested at runtime**.
* Modern APIs (Activity Result API, Accompanist) simplify lifecycle handling.
* Always **handle denial and show rationale** to maintain a good user experience.

---

