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

Compose Multiplatform requires Compose Compiler Gradle plugin applied with the same version as the Kotlin one.
See [](compose-compiler.md#migrating-a-compose-multiplatform-project) for details.

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
