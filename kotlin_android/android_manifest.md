
---

# 📄 AndroidManifest.xml

The **`AndroidManifest.xml`** file is a **central configuration file** in every Android app. It **declares essential app information** that the Android system must know before running the app.

---

## 1️⃣ Purpose of AndroidManifest.xml

* Declares **app components** (Activities, Services, Broadcast Receivers, Content Providers).
* Specifies **permissions** the app requires.
* Defines **app metadata** like theme, icons, version, and minimum SDK.
* Configures **intent filters** to handle actions like deep links.
* Declares **hardware or software features** the app uses (camera, sensors, GPS).

---

## 2️⃣ Key Components

### 2.1 `<manifest>` Tag

The root tag of the file; includes the **package name** and optional **permissions**.

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.myapp">

    <!-- Permissions, application, and components go here -->

</manifest>
```

---

### 2.2 `<application>` Tag

Contains the **app-wide settings and components**.

**Attributes & Purpose:**

* `android:icon` → App icon
* `android:label` → App name
* `android:theme` → App-wide theme
* Declares **Activities, Services, Receivers, Providers**

```xml
<application
    android:icon="@mipmap/ic_launcher"
    android:label="@string/app_name"
    android:theme="@style/Theme.MyApp">
    
    <!-- Components go here -->

</application>
```

---

### 2.3 `<activity>` Tag

Defines a **screen** in the app.

* Must declare each Activity in the manifest.
* Can specify **launch mode, intent filters, theme**.

```xml
<activity android:name=".MainActivity">
    <intent-filter>
        <action android:name="android.intent.action.MAIN" />
        <category android:name="android.intent.category.LAUNCHER" />
    </intent-filter>
</activity>
```

* **Intent Filter:** Determines how the activity can be launched or respond to system intents.

---

### 2.4 `<service>` Tag

Declares a **background service**.

* Runs without UI to perform **long-running operations**.
* Can be **started** or **bound**.

```xml
<service android:name=".MyService" />
```

---

### 2.5 `<receiver>` Tag

Defines a **BroadcastReceiver** to listen to system or app events.

```xml
<receiver android:name=".MyReceiver">
    <intent-filter>
        <action android:name="android.net.conn.CONNECTIVITY_CHANGE"/>
    </intent-filter>
</receiver>
```

* Common use cases: network changes, alarms, SMS reception.

---

### 2.6 `<provider>` Tag

Declares a **ContentProvider** to share **data between apps**.

```xml
<provider
    android:name=".MyContentProvider"
    android:authorities="com.example.myapp.provider"
    android:exported="false"/>
```

---

### 2.7 `<uses-permission>` Tag

Specifies **permissions required by the app**.

```xml
<uses-permission android:name="android.permission.INTERNET"/>
<uses-permission android:name="android.permission.CAMERA"/>
```

* Modern apps also request **dangerous permissions at runtime**.

---

## 3️⃣ Other Useful Tags

| Tag               | Purpose                                                                 |
| ----------------- | ----------------------------------------------------------------------- |
| `<uses-sdk>`      | Specifies **minimum, target, and maximum SDK versions**.                |
| `<meta-data>`     | Provides **additional information** to the system or libraries.         |
| `<uses-feature>`  | Declares required **hardware/software features** (camera, GPS).         |
| `<intent-filter>` | Declares **how components can be invoked** by the system or other apps. |

---

## 4️⃣ Best Practices

* Keep **manifest minimal**; only declare required permissions.
* Use **exported=false** for components not meant to be accessed externally.
* Leverage **meta-data** for library configurations.
* Always declare **activities and services** even if referenced in code.
* Use **intent filters** wisely to prevent security issues.

---

## 5️⃣ Summary

* The **AndroidManifest.xml** is the **central configuration file** for an Android app.
* Key components include:

  * `<application>` → app-wide settings
  * `<activity>` → screens
  * `<service>` → background services
  * `<receiver>` → broadcast listeners
  * `<provider>` → data sharing
  * `<uses-permission>` → required permissions
* Proper configuration ensures the app **runs correctly, securely, and efficiently**.

---
