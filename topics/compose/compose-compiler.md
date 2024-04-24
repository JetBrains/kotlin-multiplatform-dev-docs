[//]: # (title: Compose compiler)

<!-- this tip should be moved lower and possibly reworded after the 2.0.0 release -->
> The Jetpack Compose compiler [is merged into the Kotlin repository]() (TODO link to Google) as of Kotlin version 2.0.0-RC2.
> This helps with smoother releases of the Compose compiler, as it will ship simultaneously with Kotlin and
> will always be compatible with Kotlin of the same version.
>
{type="tip"}

To use the Kotlin Compose compiler in your project, add it to the `plugins{}` block in your `build.gradle.kts` file:

```kotlin
plugins {
    //...
    id("org.jetbrains.kotlin.plugin.compose")
}
```

When applied together with the Android Gradle plugin, the Kotlin Compose plugin will override the coordinates of the
Compose compiler supplied automatically by AGP.

## Compiler options

The Kotlin Compose compiler offers an options DSL. You can use it to configure the compiler in the `composeCompiler{}`
block of a `build.gradle.kts` file, for example:

```kotlin
composeCompiler {
    enableStrongSkippingMode = true
}
```

Available compiler options:

TODO reference of the options here

## Migrating a Compose Multiplatform project

Compose Multiplatform is going to support the Kotlin Compose compiler in the 1.6.10 release. Until then, no changes
to your projects are necessary as Compose Multiplatform manages a compiler on its own by default.

## Migrating a Jetpack Compose project

For Android modules that do not rely on Compose Multiplatform: 

1. Add the plugin to the Gradle version catalog:

    ```Ini
    [versions]
    # ...
    kotlin = "2.0.0"
    
    [plugins]
    # ...
    org-jetbrains-kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
    compose-compiler = { id = "org.jetbrains.kotlin.plugin.compose", version.ref = "kotlin" }
    ```

2. Add the plugin to the root `build.gradle.kts` file:

    ```kotlin
    plugins {
        // ...
        alias(libs.plugins.compose.compiler) apply false
    }
    ```

3. Apply the plugin to every module that uses Jetpack Compose:

    ```kotlin
    plugins {
        // ...
        alias(libs.plugins.compose.compiler)
    }
    ```

4. If you use the Android build feature in your Gradle configuration (`android.buildFeatures.compose = true`),
   or directly reference the old Maven artifacts, you'll need to update these references:

   * Change `androidx.compose.compiler:compiler` to `org.jetbrains.kotlin:kotlin-compose-compiler-plugin-embeddable`
   * Change `androidx.compose.compiler:compiler-hosted` to `org.jetbrains.kotlin:kotlin-compose-compiler-plugin`

5. If you are using compiler options for the Jetpack Compose compiler, they can be set in the `composeCompiler{}` block.
   See [Compiler options](#compiler-options) for reference.

## What's next