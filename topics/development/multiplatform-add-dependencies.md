[//]: # (title: Adding dependencies on multiplatform libraries)

Every program requires a set of libraries to operate successfully.
A Kotlin Multiplatform project can depend on cross-platform libraries that support multiple target platforms,
platform-specific libraries, and other multiplatform projects.

If you have experience developing Android apps, adding a multiplatform dependency is similar to adding a
Gradle dependency to a regular Android project.
The main difference is that you need to add the dependency to a specific source set rather than to the module as a whole.

This page describes the general management of dependencies in a multiplatform project.
Some platform specifics are covered in [](multiplatform-android-dependencies.md) and [](multiplatform-ios-dependencies.md).

## Dependency types

There are two types of dependencies that you can use in Kotlin Multiplatform projects:

* _Multiplatform dependencies_. These are multiplatform libraries that support multiple targets and can be used in the
  common source set.

  Many modern Android libraries already have multiplatform support, like [Koin](https://insert-koin.io/),
  [Coil](https://coil-kt.github.io/coil/), and [SQLDelight](https://sqldelight.github.io/sqldelight/latest/).
  
  Find more multiplatform libraries on [klibs.io](https://klibs.io/),
  a catalog of published Kotlin Multiplatform libraries.

* _Native dependencies_. These are platform-specific libraries from corresponding ecosystems.
  In native projects, you typically manage these libraries through platform-specific tools
  such as Gradle for Android and Swift Package Manager for iOS.

  When you work with a multiplatform project module, typically, you still need native dependencies to use platform APIs
  such as secure storage, system calls, and so on.
  In the build script, you specify native dependencies in the configuration of native source sets, for example, `androidMain` and `iosMain`.

For both types of dependencies, you can use local and external repositories.

## Local multiplatform dependencies

### Dependency on a local multiplatform library

To add a dependency on a multiplatform library, update the `build.gradle(.kts)` file of the dependent shared code module.
Set a dependency of the required [type](https://kotlinlang.org/docs/gradle-configure-project.html#dependency-types)
(for example, `implementation`) in the [`dependencies {}`](multiplatform-dsl-reference.md#dependencies)
block of the `commonMain` source set configuration:

<tabs group="build-script">
<tab title="Kotlin" group-key="kotlin">

```kotlin
kotlin {
    //...
    sourceSets {
        // Makes my-library classes available in all source sets
        commonMain.dependencies {
            implementation("com.example:my-library:1.0")
        }
    }
}
```

</tab>
<tab title="Groovy" group-key="groovy">

```groovy
kotlin {
    //...
    sourceSets {
        // Makes my-library classes available in all source sets
        commonMain {
            dependencies {
                implementation 'com.example:my-library:1.0'
            }
        }
    }
}
```

</tab>
</tabs>

### Dependency on another multiplatform project

One multiplatform project can depend on another.
To set this up, add a project-type dependency to the source set that needs it.
If you want to use a dependency in all source sets, add it to the common one.
In this case, other source sets will get their versions automatically.

<tabs group="build-script">
<tab title="Kotlin" group-key="kotlin">

```kotlin
kotlin {
    //...
    sourceSets {
        commonMain.dependencies {
            implementation(project(":some-other-multiplatform-module"))
        }
        androidMain.dependencies {
            // Platform-specific declarations of :some-other-multiplatform-module
            // will be resolved automatically
        }
    }
}
```

</tab>
<tab title="Groovy" group-key="groovy">

```groovy
kotlin {
    //...
    sourceSets {
        commonMain {
            dependencies {
                implementation project(':some-other-multiplatform-module')
            }
        }
        androidMain {
            dependencies {
                // Platform-specific declarations of :some-other-multiplatform-module
                // will be resolved automatically
            }
        }
    }
}
```

</tab>
</tabs>

## Dependencies on core Kotlin libraries

### Standard library

Each source set in a Kotlin Multiplatform project automatically depends on the Kotlin standard library (`kotlin-stdlib`).
The version of the standard library is the same as the version of the applied [Kotlin Multiplatform Gradle plugin](https://kotlinlang.org/docs/multiplatform/multiplatform-dsl-reference.html#id-and-version).

For platform-specific source sets, Gradle automatically uses the corresponding platform-specific variant of the library,
while the common standard library is added to the rest.
For JVM targets, the Kotlin Gradle plugin selects the appropriate JVM standard library
depending on the `compilerOptions.jvmTarget` [compiler option](https://kotlinlang.org/docs/gradle-compiler-options.html) of your Gradle build script.

Learn how to [change the default `kotlin-stdlib` dependency resolution](https://kotlinlang.org/docs/gradle-configure-project.html#dependency-on-the-standard-library).

### Testing libraries

For multiplatform tests, the [`kotlin.test`](https://kotlinlang.org/api/latest/kotlin.test/) API is available.
As it is a multiplatform library,
you can add test dependencies to all source sets by specifying a single dependency for the `commonTest` source set:

<tabs group="build-script">
<tab title="Kotlin" group-key="kotlin">

```kotlin
kotlin {
    //...
    sourceSets {
        // Makes kotlin.test classes available in all test source sets
        commonTest.dependencies {
            implementation(kotlin("test")) 
        }
    }
}
```

</tab>
<tab title="Groovy" group-key="groovy">

```groovy
kotlin {
    //...
    sourceSets {
        // Makes kotlin.test classes available in all test source sets
        commonTest {
            dependencies {
                implementation kotlin("test")
            }
        }
    }
}
```

</tab>
</tabs>

### `kotlinx` libraries

kotlinx libraries are multiplatform libraries maintained by the core Kotlin team at JetBrains
(primary examples are [kotlinx.serialization](https://github.com/kotlin/kotlinx.serialization)
and [kotlinx.coroutines](https://github.com/Kotlin/kotlinx.coroutines)).

Just like with any other multiplatform library,
to add a dependency, refer to a library artifact in the corresponding source set.

> `kotlinx` libraries sometimes require a more involved setup, for example, for web targets.
> Refer to the library's documentation for comprehensive instructions.
{style="note"}

## Dependencies on Kotlin Multiplatform libraries

You can add dependencies on libraries that have adopted Kotlin Multiplatform,
such as [SQLDelight](https://github.com/cashapp/sqldelight).
The authors of such libraries usually provide guides for adding their dependencies to your project.

> Explore the available Kotlin Multiplatform libraries on [klibs.io](https://klibs.io/).
>
{style="tip"}

### Library shared for all source sets

If you want to have access to the library from all source sets,
or to write shared code using it, add it only for the common source set.
The Kotlin Multiplatform Gradle plugin automatically resolves the corresponding platform-specific artifacts
for other declared source sets.

> Common source set cannot depend on platform-specific artifacts:
> common code needs to compile for every declared target.
>
{style="warning"}

<tabs group="build-script">
<tab title="Kotlin" group-key="kotlin">

```kotlin
kotlin {
    //...
    sourceSets {
        commonMain.dependencies {
            implementation("io.ktor:ktor-client-core:%ktorVersion%")
        }
        androidMain.dependencies {
            // Dependency to a platform part of ktor-client
            // will be resolved automatically at build time
        }
    }
}
```

</tab>
<tab title="Groovy" group-key="groovy">

```groovy
kotlin {
    //...
    sourceSets {
        commonMain {
            dependencies {
                implementation 'io.ktor:ktor-client-core:%ktorVersion%'
            }
        }
        androidMain {
            dependencies {
                // Dependency to a platform part of ktor-client
                // will be resolved automatically at build time
            }
        }
    }
}
```

</tab>
</tabs>

> You can also configure a common library in a top-level `dependencies {}` block.
> See [Configure dependencies at the top level](multiplatform-dsl-reference.md#configure-dependencies-at-the-top-level).
> 
{style="tip"}

### Libraries to be used in specific source sets

If you want to use a multiplatform library just for specific source sets, you can add it exclusively to them.
The library declarations will then only be available in those source sets.

Use a common library name in such cases, not a platform-specific one:
the Kotlin Multiplatform Gradle plugin resolves such references automatically.
For example, the example below uses `native-driver`, not `native-driver-iosx64` for platform-specific SQLDelight
(find the exact name in the library's documentation):

<tabs group="build-script">
<tab title="Kotlin" group-key="kotlin">

```kotlin
kotlin {
    //...
    sourceSets {
        commonMain.dependencies {
            // kotlinx.coroutines will be available in all source sets
            implementation("org.jetbrains.kotlinx:kotlinx-coroutines-core:%coroutinesVersion%")
        }
        androidMain.dependencies {

        }
        iosMain.dependencies {
            // SQLDelight will be available only in the iOS source set, but not in Android or common
            implementation("com.squareup.sqldelight:native-driver:%sqlDelightVersion%")
        }
        wasmJsMain.dependencies {
            
        }
    }
}
```

</tab>
<tab title="Groovy" group-key="groovy">

```groovy
kotlin {
    //...
    sourceSets {
        commonMain {
            dependencies {
                // kotlinx.coroutines will be available in all source sets
                implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-core:%coroutinesVersion%'
            }
        }
        androidMain {
            dependencies {}
        }
        iosMain {
            dependencies {
                // SQLDelight will be available only in the iOS source set, but not in Android or common
                implementation 'com.squareup.sqldelight:native-driver:%sqlDelightVersion%'
            }
        }
        wasmJsMain {
            dependencies {}
        }
    }
}
```

</tab>
</tabs>

## What's next?

Check out other resources on adding dependencies in multiplatform projects and learn more about:

* [Adding Android dependencies](multiplatform-android-dependencies.md)
* [Adding iOS dependencies](multiplatform-ios-dependencies.md)
* Check out the examples of [how to use Android and iOS libraries](multiplatform-samples.md) in sample projects.

## Get help

* **Kotlin Slack**. Get an [invite](https://surveys.jetbrains.com/s3/kotlin-slack-sign-up) and join the [#multiplatform](https://kotlinlang.slack.com/archives/C3PQML5NU) channel.
* **Kotlin issue tracker**. [Report a new issue](https://youtrack.jetbrains.com/newIssue?project=KT).