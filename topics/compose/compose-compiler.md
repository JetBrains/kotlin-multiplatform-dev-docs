[//]: # (title: Updating Compose compiler)

The Compose compiler is supplemented by a Gradle plugin, which simplifies setup and offers
easier access to compiler options.
When applied with the Android Gradle plugin (AGP), this Compose compiler plugin will override the coordinates
of the Compose compiler supplied automatically by AGP.

The Compose compiler has been merged into the Kotlin repository since Kotlin 2.0.0.
This helps smooth the migration of your projects to Kotlin 2.0.0 and later, as the Compose compiler ships
simultaneously with Kotlin and will always be compatible with Kotlin of the same version.

> It is strongly recommended that you update your Compose Multiplatform app created with Kotlin 2.0.0 to version 2.0.10 or later. The Compose
> compiler 2.0.0 has an issue where it sometimes incorrectly infers the stability of types in multiplatform projects with
> non-JVM targets, which can lead to unnecessary (or even endless) recompositions.
>
> If your app is built with Compose compiler 2.0.10 or later but uses dependencies built with Compose compiler 2.0.0,
> these older dependencies may still cause recomposition issues.
> To prevent this, update your dependencies to versions built with the same Compose compiler as your app.
>
{style="warning"}

To use the new Compose compiler plugin in your project, apply it for each module that uses Compose.
Read on for details on how to [migrate a Compose Multiplatform project](#migrating-a-compose-multiplatform-project). For a Jetpack Compose project,
refer to the [migration guide](https://kotlinlang.org/docs/compose-compiler-migration-guide.html#migrating-a-jetpack-compose-project).

## Migrating a Compose Multiplatform project

Starting with Compose Multiplatform 1.6.10, you should apply the `org.jetbrains.kotlin.plugin.compose` Gradle plugin to each
module that uses the `org.jetbrains.compose` plugin:

1. Add the Compose compiler Gradle plugin to the [Gradle version catalog](https://docs.gradle.org/current/userguide/platforms.html#sub:conventional-dependencies-toml):

    ```
    [versions]
    # ...
    kotlin = "%kotlinVersion%"
    compose-plugin = "%org.jetbrains.compose%"
 
    [plugins]
    # ...
    jetbrainsCompose = { id = "org.jetbrains.compose", version.ref = "compose-plugin" }
    kotlinMultiplatform = { id = "org.jetbrains.kotlin.multiplatform", version.ref = "kotlin" }
    compose-compiler = { id = "org.jetbrains.kotlin.plugin.compose", version.ref = "kotlin" }
    ```

2. Add the Gradle plugin to the root `build.gradle.kts` file:

    ```kotlin
    plugins {
     // ...
     alias(libs.plugins.jetbrainsCompose) apply false
     alias(libs.plugins.compose.compiler) apply false
    }
    ```

3. Apply the plugin to every module that uses Compose Multiplatform:

    ```kotlin
    plugins { 
        // ...
        alias(libs.plugins.jetbrainsCompose)
        alias(libs.plugins.compose.compiler)
    }
    ```

4. If you are using compiler options for the Jetpack Compose compiler, set them in the `composeCompiler {}` block.
   See [Compose compiler options DSL](https://kotlinlang.org/docs/compose-compiler-options.html) for reference.

#### Possible issue: "Missing resource with path"

When switching from Kotlin 1.9.0 to 2.0.0, or from 2.0.0 to 1.9.0, you may encounter the following error:

```
org.jetbrains.compose.resources.MissingResourceException: Missing resource with path: ...
```

To resolve this, delete all the `build` directories: at the root of your project and in each of the modules.

## What's next

* See [Google's announcement](https://android-developers.googleblog.com/2024/04/jetpack-compose-compiler-moving-to-kotlin-repository.html) about the Compose compiler moving to the Kotlin repository.
* See [Compose compiler options DSL](https://kotlinlang.org/docs/compose-compiler-options.html) for reference.
* To migrate a Jetpack Compose app, check out [Compose compiler documentation](https://kotlinlang.org/docs/compose-compiler-migration-guide.html).
