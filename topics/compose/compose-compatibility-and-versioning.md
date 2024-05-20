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

## Limitations of Compose Multiplatform releases

The following limitations apply to all Compose Multiplatform releases:

* Only 64-bit platforms are supported.
* Only JDK 11 or later is supported due to the memory management scheme used in [Skia](https://skia.org/) bindings.
* Only JDK 17 or later is supported for packaging native distributions due
  to [`jpackage`](https://docs.oracle.com/en/java/javase/17/docs/specs/man/jpackage.html) limitations.

There is a known [issue](https://github.com/JetBrains/compose-multiplatform/issues/940) with OpenJDK 11.0.12 when switching
keyboard layouts on macOS. This issue isn't reproducible in OpenJDK 11.0.15.

## Kotlin compatibility

Compose Multiplatform needs a compatible version of Kotlin to work correctly. Starting from Compose Multiplatform 1.2.0,
Compose Multiplatform supports multiple versions of Kotlin. Earlier versions of Compose Multiplatform can only be used
with one specific version of Kotlin. See the following table for a list of Kotlin releases and the minimum supported
version of Compose Multiplatform:

| Kotlin version | Min. Compose Multiplatform version | Notes                                                    |
|----------------|------------------------------------|----------------------------------------------------------|
| 1.9.23         | 1.6.1                              |
| 1.9.22         | 1.5.12                             |
| 1.9.21         | 1.5.11                             |
| 1.9.20         | 1.5.10                             |
| 1.9.10         | 1.5.1                              |
| 1.9.0          | 1.4.3                              |
| 1.8.22         | 1.4.3                              |
| 1.8.21         | 1.4.3                              |
| 1.8.20         | 1.4.0                              |
| 1.8.10         | 1.3.1                              |
| 1.8.0          | 1.3.0                              | 1.3.0 is not supported by earlier Kotlin/Native versions |
| 1.7.20         | 1.2.1                              |
| 1.7.20         | 1.2.0                              | JavaScript is not supported. (Fixed in release 1.2.1.)   |
| 1.7.10         | 1.2.0                              |
| 1.6.20         | 1.1.1                              |
| 1.5.31         | 1.0.0                              |

As long as there is a compatible version of Kotlin, Compose Multiplatform automatically configures the Compose
Multiplatform compiler plugin. The Compose Multiplatform compiler plugin is responsible for compiling your composable
functions. Each Compose Multiplatform compiler plugin is strictly bound to a single version of Kotlin.

As Compose Multiplatform and Kotlin releases ship separately, there may be times when a stable release of Compose
Multiplatform doesn't yet support the latest Kotlin release. However, you can still try the latest Kotlin release by
manually configuring the compiler plugin for your project. When manually configuring a compiler plugin for your project,
you must check that it is compatible with the Kotlin version that you want to use.

Use one of the following approaches to try the latest Kotlin release by manually configuring a different compiler
plugin:

* [Use a developer version of the Compose Multiplatform compiler](#use-a-developer-version-of-compose-multiplatform-compiler)
* [Use the Jetpack Compose compiler](#use-a-jetpack-compose-compiler)
* [Use a compiler version for a different version of Kotlin](#use-a-compiler-for-a-different-version-of-kotlin)

> With these approaches, stability isn't guaranteed, so use them at your own risk. Even if
> compilation is successful, there can be hidden runtime errors. We don't recommend
> upgrading to the latest version of Kotlin in production until it is officially supported by
> Compose Multiplatform.
>
{type="warning"}

### Use a developer version of Compose Multiplatform compiler

If there isn't a stable Compose Multiplatform version that supports the Kotlin version that you want to use, you
can try a developer (`dev`) version of the Compose Multiplatform compiler.
`-dev` versions of Compose Multiplatform, such as `1.5.0-dev1084`, contain actual version mappings from Kotlin to the
Compose Multiplatform compiler. This includes Beta and RC (Release Candidate) versions of Kotlin.

If you want to test a Beta or RC version of Kotlin that isn't directly supported by a stable release of Compose
Multiplatform, there are two options:

* Use the most recent `-dev` build of Compose Multiplatform. For information on the latest releases available,
  see [Releases](https://github.com/JetBrains/compose-multiplatform/releases).
* Manually specify the `kotlinCompilerPlugin` version in your `build.gradle.kts` file. Find the right version by
  consulting the following table:

| Kotlin version | Compose Multiplatform compiler version |
|----------------|----------------------------------------|
| 2.0.0-RC1      | `1.5.11-kt-2.0.0-RC1`                  |
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

For example, to use Kotlin 1.9.23, you need Compose Multiplatform compiler version `1.5.2.10`:

```kotlin
compose {
    kotlinCompilerPlugin.set("1.5.2.10")
}
```

> Early access versions of the Compose Multiplatform compiler plugin, like `1.5.2.1-rc01`,
> are not available in [Maven Central](https://central.sonatype.com/). To use early access versions,
> add [`maven`](https://maven.pkg.jetbrains.space/public/p/compose/dev) to your list of repositories.
>
{type="note"}

### Use a Jetpack Compose compiler

If there is no suitable developer version of the Compose Multiplatform compiler plugin, you can try using a Jetpack
Compose compiler plugin. Check the [pre-release Kotlin compatibility](https://developer.android.com/jetpack/androidx/releases/compose-kotlin#pre-release_kotlin_compatibility)
table from Android to find a compatible compiler version. Then, in your `build.gradle.kts` file, set it in
the `kotlinCompilerPlugin.set` parameter:

```kotlin
compose {
    kotlinCompilerPlugin.set("androidx.compose.compiler:compiler:1.5.12")
}
```

In this example, Jetpack Compose compiler version 1.5.12 corresponds to Kotlin 1.9.23.

> The Jetpack Compose compiler plugin works for Kotlin/JVM targets, including both desktop and Android platforms.
> However, its reliability may not extend to Kotlin/JS and Kotlin/Native targets. For these scenarios, we recommend
> [using the Compose Multiplatform compiler plugin](#use-a-developer-version-of-compose-multiplatform-compiler) to ensure compatibility.
>
{type="warning"}

### Use a compiler for a different version of Kotlin

If there is no compatible Compose Multiplatform or Jetpack Compose compiler for your Kotlin version (or you
encountered errors), you can try to use a compiler for a different version of Kotlin. If you do this, you **must**
disable the Kotlin version compatibility check. This approach can work if you upgrade to a minor version of Kotlin, but
probably won't work if you upgrade to a major version of Kotlin.

```kotlin
compose {
    kotlinCompilerPlugin.set(dependencies.compiler.forKotlin("1.7.20"))
    kotlinCompilerPluginArgs.add("suppressKotlinVersionCompatibilityCheck=1.7.21")
}
```

This example sets a fixed version for the Compose Multiplatform compiler and configures it by specifying additional
arguments. The argument `suppressKotlinVersionCompatibilityCheck` disables the internal Kotlin compatibility check
inside the compiler. The argument specifies the version of Kotlin that is applied to your project.

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
