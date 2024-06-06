[//]: # (title: Compose compiler)

<!-- this tip should be moved lower and possibly reworded after the 2.0.0 release -->
> The Compose compiler has been merged into the Kotlin repository since Kotlin 2.0.0-RC2.
> This helps smooth the migration of your projects to Kotlin 2.0, as the Compose compiler will ship simultaneously
> with Kotlin from now on and will always be compatible with Kotlin of the same version.
>
{type="tip"}

The Compose compiler is supplemented by a Gradle plugin, which simplifies setup and offers
easier access to compiler options.
When applied with the Android Gradle plugin (AGP), this Compose compiler plugin will override the coordinates
of the Compose compiler supplied automatically by AGP.

To use it in your project, apply the plugin for each module that uses Compose. See the migration instructions below:

* [Migrating a Compose Multiplatform project](#migrating-a-compose-multiplatform-project)
* [Migrating a Jetpack Compose project](#migrating-a-jetpack-compose-project)
* [Compose Compiler options DSL](#compose-compiler-options-dsl)

## Migrating a Compose Multiplatform project

Starting with 1.6.10, you should apply the `org.jetbrains.kotlin.plugin.compose` Gradle plugin to each
module that uses the `org.jetbrains.compose` plugin:

1. Add the Compose compiler Gradle plugin to the [Gradle version catalog](https://docs.gradle.org/current/userguide/platforms.html#sub:conventional-dependencies-toml):

    ```toml
    [versions]
    # ...
    kotlin = "2.0.0-RC2"
    
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
   See [Compose Compiler options DSL](#compose-compiler-options-dsl) for reference.

   > The Gradle plugin provides defaults for several compiler options that were only specified manually before.
   > If you have any of them set up with `freeCompilerArgs`, for example, Gradle will report a duplicate options error.
   >
   {type="warning"}

#### Possible issue: "Missing resource with path"

When switching from Kotlin 1.9.0 to 2.0.0, or from 2.0.0 to 1.9.0, you may encounter the following error:

```
org.jetbrains.compose.resources.MissingResourceException: Missing resource with path: ...
```

To resolve this, delete all of the `build` directories: at the root of your project and in each of the modules. 

### Test with the stable Compose Multiplatform 

To try the Compose compiler 2.0.0 with the stable version of Compose Multiplatform (1.6.2 or older), use this configuration:

```kotlin
compose {
    kotlinCompilerPlugin = "org.jetbrains.kotlin:kotlin-compose-compiler-plugin-embeddable:2.0.0-RC2"
}
```

> This configuration will be deprecated with the stable 1.6.10 release:
> you will have to apply the `org.jetbrains.kotlin.plugin.compose` plugin instead.
> 
{type="note"}

## Migrating a Jetpack Compose project

For Android modules that do not rely on Compose Multiplatform: 

1. Add the Compose compiler Gradle plugin to the [Gradle version catalog](https://docs.gradle.org/current/userguide/platforms.html#sub:conventional-dependencies-toml):

    ```toml
    [versions]
    # ...
    kotlin = "2.0.0-RC2"
    
    [plugins]
    # ...
    org-jetbrains-kotlin-android = { id = "org.jetbrains.kotlin.android", version.ref = "kotlin" }
    compose-compiler = { id = "org.jetbrains.kotlin.plugin.compose", version.ref = "kotlin" }
    ```

2. Add the Gradle plugin to the root `build.gradle.kts` file:

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

4. If you reference the old Maven artifacts directly, you'll need to update these references:

   * Change `androidx.compose.compiler:compiler` to `org.jetbrains.kotlin:kotlin-compose-compiler-plugin-embeddable`
   * Change `androidx.compose.compiler:compiler-hosted` to `org.jetbrains.kotlin:kotlin-compose-compiler-plugin`

5. If you are using compiler options for the Jetpack Compose compiler, set them in the `composeCompiler {}` block.
   See [Compiler options](#compose-compiler-options-dsl) for reference.

   > The Gradle plugin provides defaults for several compiler options that were only specified manually before.
   > If you have any of them set up with `freeCompilerArgs`, for example, Gradle will report a duplicate options error.
   >
   {type="warning"}

## Compose compiler options DSL

The Compose compiler Gradle plugin offers a DSL for various compiler options.
You can use it to configure the compiler in the `composeCompiler {}` block of the `build.gradle.kts` file for the module
you're applying the plugin to, for example:

```kotlin
composeCompiler {
   enableStrongSkippingMode = true
}
```

### generateFunctionKeyMetaClasses

**Type**: `Property<Boolean>`

**Default**: `false`

If `true`, generate function key meta classes with annotations indicating the functions and their group keys.

### includeSourceInformation

**Type**: `Property<Boolean>`

**Default**: `false`

If `true`, include source information in generated code.

Records source information that can be used for tooling to determine the source location of the corresponding composable function.
This option does not affect the presence of symbols or line information normally added by the Kotlin compiler;
it only controls source information added by the Compose compiler.

### metricsDestination

**Type**: `DirectoryProperty`

When a directory is specified, the Compose compiler will use the directory to dump [compiler metrics](https://github.com/androidx/androidx/blob/androidx-main/compose/compiler/design/compiler-metrics.md#reports-breakdown).
They can be useful for debugging and optimizing your application's runtime performance:
the metrics show which composable functions are skippable, restartable, read-only, and so on.

The [reportsDestination](#reportsdestination) option allows dumping descriptive reports as well.

For a deep dive into the compiler metrics, see this [Composable metrics blog post](https://chrisbanes.me/posts/composable-metrics/).

### reportsDestination

**Type**: `DirectoryProperty`

When a directory is specified, the Compose compiler will use the directory to dump [compiler metrics reports](https://github.com/androidx/androidx/blob/androidx-main/compose/compiler/design/compiler-metrics.md#reports-breakdown).
They can be useful for optimizing your application's runtime performance:
the reports show which composable functions are skippable, restartable, read-only, and so on.

The [metricsDestination](#metricsdestination) option allows dumping raw metrics.

For a deep dive into the compiler metrics, see this [Composable metrics blog post](https://chrisbanes.me/posts/composable-metrics/).

### enableIntrinsicRemember

**Type**: `Property<Boolean>`

**Default**: `true`

If `true`, enable intrinsic remember performance optimization.

Intrinsic remember is an optimization mode which inlines `remember` invocations and replaces `.equals` comparisons for keys
with comparisons of the `$changed` meta parameter when possible.
This results in fewer slots being used and fewer comparisons being made at runtime.

### enableNonSkippingGroupOptimization

**Type**: `Property<Boolean>`

**Default**: `false`

If `true`, remove groups around non-skipping composable functions.

This optimization improves the runtime performance of your application by skipping
unnecessary groups around composables which do not skip (and thus do not require a group).
This optimization will remove the groups, for example, around functions explicitly marked as `@NonSkippableComposable`
and functions that are implicitly not skippable (inline functions and functions that return a non-`Unit` value such as `remember`).

> This feature is considered [Experimental](supported-platforms.md#core-kotlin-multiplatform-technology-stability-levels) and is thus disabled by default.
> 
{type="warning"}

### enableStrongSkippingMode

**Type**: `Property<Boolean>`

**Default**: `false`

If `true`, enable Strong Skipping mode.

Strong Skipping is an [Experimental](supported-platforms.md#core-kotlin-multiplatform-technology-stability-levels) mode that improves the runtime performance of your application by skipping unnecessary
invocations of composable functions whose parameters have not changed.
For example, composables with unstable parameters become skippable, and lambdas with unstable captures are memoized.

For details, see the [description of Strong Skipping mode](https://github.com/androidx/androidx/blob/androidx-main/compose/compiler/design/strong-skipping.md)
in Androidx GitHub repo.

### stabilityConfigurationFile

**Type**: `RegularFileProperty`

Path to the stability configuration file.
For details, see [Stability configuration file](https://developer.android.com/develop/ui/compose/performance/stability/fix#configuration-file)
in the Jetpack Compose documentation.

### includeTraceMarkers

**Type**: `Property<Boolean>`

**Default**: `false`

If `true`, include composition trace markers in the generated code.

The Compose compiler can inject additional tracing information into the bytecode, which allows it to show composable functions
in the Android Studio system trace profiler.

For details, see this [Android Developers blog post](https://medium.com/androiddevelopers/jetpack-compose-composition-tracing-9ec2b3aea535).

### targetKotlinPlatforms

**Type**: `SetProperty<KotlinPlatformType>`

Indicates Kotlin platforms to which the Compose compiler Gradle plugin should be applied.
By default, the plugin is applied to all Kotlin platforms.

To enable only one specific Kotlin platform, for example, Kotlin/JVM:

```kotlin
composeCompiler {
   targetKotlinPlatforms.set(setOf(KotlinPlatformType.jvm))
}
```

To disable the Gradle plugin for one or more Kotlin platforms, for example, Kotlin/Native and Kotlin/JS:

```kotlin
composeCompiler {
    targetKotlinPlatforms.set(
       KotlinPlatformType.values()
           .filterNot { it == KotlinPlatformType.native || it == KotlinPlatformType.js }
           .asIterable()
    )
}
```

## What's next

* See the [Google's announcement](https://android-developers.googleblog.com/2024/04/jetpack-compose-compiler-moving-to-kotlin-repository.html) about Compose compiler moving to the Kotlin repository.
* If you are using Jetpack Compose to build an Android app, check out [our guide on how to make it multiplatform](multiplatform-integrate-in-existing-app.md).
