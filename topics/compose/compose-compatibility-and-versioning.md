[//]: # (title: Compatibility and versions)

Compose Multiplatform releases ship separately from Kotlin and Jetpack Compose releases. This page contains information
about Compose Multiplatform releases, Compose release cycles, and component compatibility. 

## Supported platforms

Compose Multiplatform %composeVersion% supports the following platforms:

| Platform | Minimum version                                                                                        |
|----------|--------------------------------------------------------------------------------------------------------|
| Android  | Android 5.0 (API level 21)                                                                             |
| iOS      | iOS 13                                                                                                 |
| macOS    | macOS 12 x64, macOS 13 arm64                                                                           |
| Windows  | Windows 10 (x86-64, arm64)                                                                             |
| Linux    | Ubuntu 20.04 (x86-64, arm64)                                                                           |
| Web      | Browsers with [WasmGC support](https://kotlinlang.org/docs/wasm-troubleshooting.html#browser-versions) |

[//]: # (https://youtrack.jetbrains.com/issue/CMP-7539)

> All Compose Multiplatform releases support only 64-bit platforms. 
> 
{style="note"}

## Kotlin compatibility

The latest Compose Multiplatform is always compatible with the latest version of Kotlin.
There is no need to manually align their versions.
Remember that using an EAP version of either product is still potentially unstable.

Compose Multiplatform requires the Compose Compiler Gradle plugin applied with the same version
as the Kotlin Multiplatform plugin.
See [](compose-compiler.md#migrating-a-compose-multiplatform-project) for details.

> Starting with Compose Multiplatform 1.8.0, the UI framework fully transitioned to the K2 compiler.
> So, to use the latest Compose Multiplatform release, you should:
> * use at least Kotlin 2.1.0 for your projects,
> * depend on libraries based on Compose Multiplatform only if they are compiled against at least Kotlin 2.1.0.
> 
> As a workaround for backward compatibility problems until all your dependencies are updated,
> you may turn off Gradle cache by adding `kotlin.native.cacheKind=none` to your `gradle.properties` file.
> This will increase compilation time.
>
{style="warning"}

## Limitations of Compose Multiplatform for desktop releases

Compose Multiplatform for desktop has the following limitations:

* Only JDK 11 or later is supported due to the memory management scheme used in [Skia](https://skia.org/) bindings.
* Only JDK 17 or later is supported for packaging native distributions due
  to [`jpackage`](https://docs.oracle.com/en/java/javase/17/docs/specs/man/jpackage.html) limitations.
* There is a known [issue](https://github.com/JetBrains/compose-multiplatform/issues/940) with OpenJDK 11.0.12 when switching keyboard layouts on macOS.
  This issue isn't reproducible in OpenJDK 11.0.15.

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

### Development versions of Compose Multiplatform

Development versions of the Compose Multiplatform compiler plugin (for example, `1.8.2+dev2544`) are built without a set schedule,
to test updates between formal releases.

These builds are not available in [Maven Central](https://central.sonatype.com/).
To access them, add this line to your list of repositories:

```kotlin
maven("https://maven.pkg.jetbrains.space/public/p/compose/dev")
```

### Jetpack Compose artifacts used

When you build your application for Android, Compose Multiplatform uses artifacts published by Google.
For example, if you apply the Compose Multiplatform 1.5.0 Gradle plugin and add `implementation(compose.material3)` to your `dependencies`, then your
project will use the `androidx.compose.material3:material3:1.1.1` artifact in the Android target (but `org.jetbrains.compose.material3:material3:1.5.0` in other targets).

The following table lists Jetpack Compose artifact versions used by each version of Compose Multiplatform:

| Compose Multiplatform version                                                     | Jetpack Compose version | Jetpack Compose Material3 version |
|-----------------------------------------------------------------------------------|-------------------------|-----------------------------------|
| [1.8.2](https://github.com/JetBrains/compose-multiplatform/releases/tag/v1.8.2)   | 1.8.2                   | 1.3.2                             |
| [1.7.3](https://github.com/JetBrains/compose-multiplatform/releases/tag/v1.7.3)   | 1.7.6                   | 1.3.1                             |
| [1.7.1](https://github.com/JetBrains/compose-multiplatform/releases/tag/v1.7.1)   | 1.7.5                   | 1.3.1                             |
| [1.7.0](https://github.com/JetBrains/compose-multiplatform/releases/tag/v1.7.0)   | 1.7.1                   | 1.3.0                             |
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
