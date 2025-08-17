
---

# ⚙️ Gradle in Android Projects

**Gradle** is the **build system used in Android** to compile, package, and manage dependencies for your app. It provides a **flexible and declarative way** to configure builds, manage versions, and automate tasks.

---

## 1️⃣ Role of Gradle

* **Build Automation:** Compiles code, processes resources, runs tests, and generates APKs/AABs.
* **Dependency Management:** Downloads libraries, plugins, and ensures correct versions.
* **Multi-Variant Support:** Handles different **build types** (debug/release) and **product flavors** (free/paid).
* **Task Management:** Allows defining custom tasks like code generation or file processing.
* **Integration:** Works with Android Studio to provide instant builds and sync.

---

## 2️⃣ build.gradle / build.gradle.kts Files

Android projects usually have **two levels** of Gradle files:

### 2.1 Project-Level `build.gradle` (or `build.gradle.kts`)

* Located in the **root folder**.
* Defines **global configuration** and **dependencies for all modules**.
* Example:

```kotlin
// build.gradle.kts (Project-level)
buildscript {
    repositories {
        google()
        mavenCentral()
    }
    dependencies {
        classpath("com.android.tools.build:gradle:8.2.0")
        classpath("org.jetbrains.kotlin:kotlin-gradle-plugin:1.9.10")
    }
}

allprojects {
    repositories {
        google()
        mavenCentral()
    }
}
```

### 2.2 Module-Level `build.gradle` (or `build.gradle.kts`)

* Located in **app/** or other module folders.
* Configures **Android-specific settings**, **build variants**, and **dependencies**.
* Example:

```kotlin
plugins {
    id("com.android.application")
    kotlin("android")
}

android {
    compileSdk = 34

    defaultConfig {
        applicationId = "com.example.myapp"
        minSdk = 24
        targetSdk = 34
        versionCode = 1
        versionName = "1.0"
    }

    buildTypes {
        getByName("debug") {
            isMinifyEnabled = false
        }
        getByName("release") {
            isMinifyEnabled = true
            proguardFiles(
                getDefaultProguardFile("proguard-android-optimize.txt"),
                "proguard-rules.pro"
            )
        }
    }

    flavorDimensions += "version"
    productFlavors {
        create("free") {
            dimension = "version"
        }
        create("paid") {
            dimension = "version"
        }
    }
}

dependencies {
    implementation("androidx.core:core-ktx:1.12.0")
    api("com.squareup.retrofit2:retrofit:2.9.0")
    kapt("com.google.dagger:hilt-compiler:2.47")
}
```

---

## 3️⃣ Build Types

**Build types** define how your app is **compiled, packaged, and optimized**.

| Build Type  | Description                                        | Use Case                |
| ----------- | -------------------------------------------------- | ----------------------- |
| **debug**   | Non-obfuscated, includes debug info and logging    | Development and testing |
| **release** | Minified, optimized, may include ProGuard/R8 rules | Production release      |

* You can also **add custom build types** if needed.

---

## 4️⃣ Product Flavors

**Product flavors** allow creating **different versions of your app** from the same codebase.

* Example: Free vs Paid, Staging vs Production.
* Each flavor can have **different resources, dependencies, and configs**.

```kotlin
productFlavors {
    create("free") {
        applicationIdSuffix = ".free"
        versionNameSuffix = "-free"
    }
    create("paid") {
        applicationIdSuffix = ".paid"
        versionNameSuffix = "-paid"
    }
}
```

* **Combine flavors and build types** → multi-variant builds (e.g., `freeDebug`, `paidRelease`).

---

## 5️⃣ Managing Dependencies

Gradle provides **scopes** for dependencies:

| Scope                         | Description                                                                                                  |
| ----------------------------- | ------------------------------------------------------------------------------------------------------------ |
| **implementation**            | Dependency is internal; not exposed to other modules                                                         |
| **api**                       | Dependency is exposed to modules depending on this module                                                    |
| **kapt/ksp**                  | Annotation processors for Kotlin (`kapt` for Java annotation processors, `ksp` for Kotlin Symbol Processing) |
| **testImplementation**        | Dependency only for local unit tests                                                                         |
| **androidTestImplementation** | Dependency only for instrumented tests                                                                       |

### Example

```kotlin
dependencies {
    implementation("androidx.lifecycle:lifecycle-viewmodel-ktx:2.6.2")
    api("com.squareup.okhttp3:okhttp:4.11.0")
    kapt("com.google.dagger:hilt-compiler:2.47")
}
```

---

## 6️⃣ Version Catalogs

* Introduced in **Gradle 7+**, allows **centralized dependency management**.
* Define all versions and dependencies in `gradle/libs.versions.toml`:

```toml
[versions]
kotlin = "1.9.10"
coroutines = "1.7.3"

[libraries]
kotlin-stdlib = { module = "org.jetbrains.kotlin:kotlin-stdlib", version.ref = "kotlin" }
coroutines-core = { module = "org.jetbrains.kotlinx:kotlinx-coroutines-core", version.ref = "coroutines" }
```

* Use in `build.gradle.kts`:

```kotlin
dependencies {
    implementation(libs.kotlin.stdlib)
    implementation(libs.coroutines.core)
}
```

**Benefits:**

* Avoid version conflicts.
* Centralize upgrades.
* Simplify multi-module projects.

---

## 7️⃣ Summary

* **Gradle** manages **building, testing, packaging, and dependencies** in Android.
* **Project-level build.gradle:** Global config, plugins, and repositories.
* **Module-level build.gradle:** Android config, build types, flavors, and dependencies.
* **Build types:** Control debug/release configurations.
* **Product flavors:** Support multiple versions of the app.
* **Dependencies:** Scoped using `implementation`, `api`, `kapt/ksp`.
* **Version catalogs:** Centralized dependency management for easier maintenance.

---
