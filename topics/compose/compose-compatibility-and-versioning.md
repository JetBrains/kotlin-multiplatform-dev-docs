[//]: # (title: Compatibility and versions)

Compose Multiplatform releases ship separately from Kotlin and Jetpack Compose releases. This page contains information
about Compose Multiplatform releases, the compatibility between different releases, and the release cycles.

## Supported platforms

Compose Multiplatform %composeVersion% supports the following platforms:

* Android
* iOS
* macOS (x86-64, arm64)
* Windows (x86-64)
* Linux (x86-64, arm64)
* Web browsers

> All Compose Multiplatform releases support only 64-bit platforms. 
> 
{type="note"}

## Limitations of Compose Multiplatform for desktop releases

Compose Multiplatform for desktop has the following limitations: 

* Only JDK 11 or later is supported due to the memory management scheme used in [Skia](https://skia.org/) bindings.
* Only JDK 17 or later is supported for packaging native distributions due
  to [`jpackage`](https://docs.oracle.com/en/java/javase/17/docs/specs/man/jpackage.html) limitations.
* There is a known [issue](https://github.com/JetBrains/compose-multiplatform/issues/940) with OpenJDK 11.0.12 when switching keyboard layouts on macOS.
  This issue isn't reproducible in OpenJDK 11.0.15.

## Kotlin compatibility

As long as you are using Compose Multiplatform 1.6.10 or higher and Kotlin 2.0.0 or higher, Compose Multiplatform is 
compatible with Kotlin. There is no need to manually align their versions.
Remember that using an EAP version of either product is still potentially unstable.

Compose Multiplatform requires Compose Compiler Gradle plugin applied with the same version as the Kotlin one.
See [](compose-compiler.md#migrating-a-compose-multiplatform-project) for details.

> It's strongly recommended that you update your Compose app created with Kotlin 2.0.0 to version 2.0.10 or later. The Compose
> compiler 2.0.0 has an issue where it sometimes incorrectly infers the stability of types in multiplatform projects with
> non-JVM targets, which can lead to unnecessary (or even endless) recompositions.
>
> If your app is built with Compose compiler 2.0.10 or later but uses dependencies built with Compose compiler 2.0.0,
> these older dependencies may still cause recomposition issues.
> To prevent this, update your dependencies to versions built with the same Compose compiler as your app.
>
{type="warning"}

### Older versions of Kotlin

To use Compose Multiplatform with a Kotlin version earlier than 2.0.0, you can manually specify the `kotlinCompilerPlugin` version
in your `build.gradle.kts` file.

Find the right version in the following table:

| Kotlin version | Compose Multiplatform compiler version |
|----------------|----------------------------------------|
| 1.9.25         | `1.5.14`                               |
| 1.9.24         | `1.5.14`                               |
| 1.9.23         | `1.5.10.1`                             |
| 1.9.22         | `1.5.8.1`                              |
| 1.9.21         | `1.5.4`                                |
| 1.9.20         | `1.5.3`                                |
| 1.9.10         | `1.5.2`                                |
| 1.9.0          | `1.5.1`                                |
| 1.8.22         | `1.4.8`                                |
| 1.8.21         | `1.4.7`                                |
| 1.8.20         | `1.4.5`                                |
| 1.8.10         | `1.4.2`                                |
| 1.8.0          | `1.4.0`                                |
| 1.7.20         | `1.3.2.2`                              |
| 1.7.10         | `1.3.0`                                |

For example, to use Kotlin 1.9.24, you need Compose Multiplatform compiler version `1.5.14`:

```kotlin
compose {
    kotlinCompilerPlugin.set("1.5.14")
}
```

Early access versions of the Compose Multiplatform compiler plugin, like `1.7.0-alpha03`,
are not available in [Maven Central](https://central.sonatype.com/).
To use early access versions, add this line to your list of repositories:

```kotlin
maven("https://maven.pkg.jetbrains.space/public/p/compose/dev")
```

## Jetpack Compose and Compose Multiplatform release cycles

Compose Multiplatform shares a lot of code with [Jetpack Compose](https://developer.android.com/jetpack/compose) for
Android, a framework developed by Google. We align our Compose Multiplatform release cycle with the release cycle of
Jetpack Compose so that the common code is properly tested and stabilized.

When a new version of Jetpack Compose is released, we:

* Use the release commit as a base for the next [Compose Multiplatform](https://github.com/JetBrains/androidx) version.
* Add support for new platform features.
* Stabilize all platforms.
* Release a new version of Compose Multiplatform.

The gap between a Compose Multiplatform release and a Jetpack Compose release is usually 1–3 months.

When you build your application for Android, the artifacts published by Google are used. For example, if you apply the
Compose Multiplatform 1.5.0 Gradle plugin and add `implementation(compose.material3)` to your `dependencies`, then your
project will use the `androidx.compose.material3:material3:1.1.1` artifact in the Android target (
but `org.jetbrains.compose.material3:material3:1.5.0` in other targets). See the following table to find out exactly
which version of Jetpack Compose artifact is used:

| Compose Multiplatform version                                                     | Jetpack Compose version | Jetpack Compose Material3 version |
|-----------------------------------------------------------------------------------|-------------------------|-----------------------------------|
| [1.6.11](https://github.com/JetBrains/compose-multiplatform/releases/tag/v1.6.11) | 1.6.7                   | 1.2.1                             |
| [1.6.10](https://github.com/JetBrains/compose-multiplatform/releases/tag/v1.6.10) | 1.6.7                   | 1.2.1                             |
| [1.6.2](https://github.com/JetBrains/compose-multiplatform/releases/tag/v1.6.2)   | 1.6.4                   | 1.2.1                             |
| [1.6.1](https://github.com/JetBrains/compose-multiplatform/releases/tag/v1.6.1)   | 1.6.3                   | 1.2.1                             |
| [1.6.0](https://github.com/JetBrains/compose-multiplatform/releases/tag/v1.6.0)   | 1.6.1                   | 1.2.0                             |
| [1.5.12](https://github.com/JetBrains/compose-multiplatform/releases/tag/v1.5.12) | 1.5.4                   | 1.1.2                             |
| [1.5.11](https://github.com/JetBrains/compose-multiplatform/releases/tag/v1.5.11) | 1.5.4                   | 1.1.2                             |
| [1.5.10](https://github.com/JetBrains/compose-multiplatform/releases/tag/v1.5.10) | 1.5.4                   | 1.1.2                             |
| [1.5.1](https://github.com/JetBrains/compose-multiplatform/releases/tag/v1.5.1)   | 1.5.0                   | 1.1.1                             |
| [1.5.0](https://github.com/JetBrains/compose-multiplatform/releases/tag/v1.5.0)   | 1.5.0                   | 1.1.1                             |
| [1.4.3](https://github.com/JetBrains/compose-multiplatform/releases/tag/v1.4.3)   | 1.4.3                   | 1.0.1                             |
| [1.4.1](https://github.com/JetBrains/compose-multiplatform/releases/tag/v1.4.1)   | 1.4.3                   | 1.0.1                             |
| [1.4.0](https://github.com/JetBrains/compose-multiplatform/releases/tag/v1.4.0)   | 1.4.0                   | 1.0.1                             |
| [1.3.1](https://github.com/JetBrains/compose-multiplatform/releases/tag/v1.3.1)   | 1.3.3                   | 1.0.1                             |
| [1.3.0](https://github.com/JetBrains/compose-multiplatform/releases/tag/v1.3.0)   | 1.3.3                   | 1.0.1                             |
| [1.2.1](https://github.com/JetBrains/compose-multiplatform/releases/tag/v1.2.1)   | 1.2.1                   | 1.0.0-alpha14                     |
| [1.2.0](https://github.com/JetBrains/compose-multiplatform/releases/tag/v1.2.0)   | 1.2.1                   | 1.0.0-alpha14                     |
| [1.1.1](https://github.com/JetBrains/compose-multiplatform/releases/tag/v1.1.1)   | 1.1.0                   | 1.0.0-alpha05                     |
| [1.1.0](https://github.com/JetBrains/compose-multiplatform/releases/tag/v1.1.0)   | 1.1.0                   | 1.0.0-alpha05                     |
| [1.0.1](https://github.com/JetBrains/compose-multiplatform/releases/tag/v1.0.1)   | 1.1.0-beta02            | 1.0.0-alpha03                     |
| [1.0.0](https://github.com/JetBrains/compose-multiplatform/releases/tag/v1.0.0)   | 1.1.0-beta02            | 1.0.0-alpha03                     |
