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

## Migrating a Compose Multiplatform project

Compose Multiplatform is going to support the Kotlin Compose compiler in the 1.6.10 release. Until then, no changes
to your projects are necessary as Compose Multiplatform manages the compiler on its own by default.

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

## Compiler options

The Kotlin Compose compiler offers an options DSL. You can use it to configure the compiler in the `composeCompiler{}`
block of a `build.gradle.kts` file, for example:

```kotlin
composeCompiler {
   enableStrongSkippingMode = true
}
```

### generateFunctionKeyMetaClasses

Type: `Property<Boolean>`

If `true`, generate function key meta classes with annotations indicating the functions and their group keys.

### includeSourceInformation

Type: `Property<Boolean>`

If `true`, include source information in generated code.

Records source information that can be used for tooling to determine the source location of the corresponding composable function.
This option does not affect the presence of symbols or line information normally added by the Kotlin compiler;
it only controls source information added by the Kotlin Compose compiler.

### metricsDestination

Type: `DirectoryProperty`

When a directory is specified, the Compose compiler will use the directory to dump [compiler metrics](https://github.com/androidx/androidx/blob/androidx-main/compose/compiler/design/compiler-metrics.md#reports-breakdown).
They can be useful for optimizing your application's runtime performance:
the metrics show which of your composable functions are skippable, which are restartable, which are readonly, etc.

For a deep dive into the compiler metrics, see this [Composable metrics blog post](https://chrisbanes.me/posts/composable-metrics/).

### reportsDestination

Type: `DirectoryProperty`

When a directory is specified, the Compose compiler will use the directory to write an overall compiler metrics report.
Generally used in conjunction with the [metricsDestination](#metricsdestination) option.

### enableIntrinsicRemember

Type: `Property<Boolean>`

If `true`, enable intrinsic remember performance optimization.

Intrinsic remember is an optimization mode which inlines `remember` invocations and replaces `.equals` comparison for keys
with comparisons of the `$changed` meta parameter when possible.
This results in fewer slots being used and fewer comparisons being done at runtime.

### enableNonSkippingGroupOptimization

Type: `Property<Boolean>`

If `true`, remove groups around non-skipping composable functions.

This is an experimental optimization that improves the runtime performance of your application by skipping
unnecessary groups around composables which do not skip (and thus do not require a group).
This optimization will remove the groups, for example, around functions explicitly marked as `@NonSkippableComposable`
and functions that are implicitly not skippable (inline functions and functions that return a non-`Unit` value such as `remember`).

This feature is considered experimental and is thus disabled by default.

### enableStrongSkippingMode

Type: `Property<Boolean>`

If `true`, enable strong skipping mode.

Strong Skipping is an experimental mode that improves the runtime performance of your application by skipping unnecessary
invocations of composable functions whose parameters have not changed.
For example, composables with unstable parameters become skippable and lambdas with unstable captures will be memoized.

For details, see the [description for Strong Skipping Mode](https://github.com/androidx/androidx/blob/androidx-main/compose/compiler/design/strong-skipping.md)
in Androidx GitHub repo.

### stabilityConfigurationFile

Type: `RegularFileProperty`

Path to the stability configuration file.
For details, see [Stability configuration file](https://developer.android.com/develop/ui/compose/performance/stability/fix#configuration-file)
in Jetpack Compose documentation.

### includeTraceMarkers

Type: `Property<Boolean>`

If `true`, include composition trace markers in the generated code.

Compose compiler can inject additional tracing information into the bytecode, which allows showing composable functions
in the Android Studio system trace profiler.

For details, see this [Android Developers blog post](https://medium.com/androiddevelopers/jetpack-compose-composition-tracing-9ec2b3aea535).

### targetKotlinPlatforms

Type: `SetProperty<KotlinPlatformType>`

Indicates Kotlin platforms to which the Compose compiler plugin should be applied.
By default, the plugin is applied to all Kotlin platforms.

To enable only one specific Kotlin platform (for example, Kotlin/JVM):

```kotlin
composeCompiler {
   targetKotlinPlatforms.set(setOf(KotlinPlatformType.jvm))
}
```

To disable the plugin for one or more Kotlin platforms (for example, Kotlin/Native and Kotlin/JS):

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