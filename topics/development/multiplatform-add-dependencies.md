[//]: # (title: Adding dependencies on multiplatform libraries)

Every program requires a set of libraries to operate successfully.
A Kotlin Multiplatform project can depend on cross-platform libraries that support multiple target platforms,
platform-specific libraries, and other multiplatform projects.

This page describes management of multiplatform dependencies.
Some platform specifics are covered in [](multiplatform-android-dependencies.md) and [](multiplatform-ios-dependencies.md). 

To add a dependency on a multiplatform library, update the `build.gradle(.kts)` file that configures the shared code module.
Set a dependency of the required [type](https://kotlinlang.org/docs/gradle-configure-project.html#dependency-types)
(for example, `implementation`) in the [`dependencies {}`](multiplatform-dsl-reference.md#dependencies)
block:

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

## Dependency on a core Kotlin library

### Standard library

Each source set automatically depends on the standard library (`stdlib`).
The version of the standard library is the same as the version of the `kotlin-multiplatform` Gradle plugin.

For platform-specific source sets, the corresponding platform-specific variant of the library is used, while a common
standard library is added to the rest. The Kotlin Gradle plugin will select the appropriate JVM standard library
depending on the `compilerOptions.jvmTarget` [compiler option](https://kotlinlang.org/docs/gradle-compiler-options.html) of your Gradle build script.

Learn how to [change the default behavior](https://kotlinlang.org/docs/gradle-configure-project.html#dependency-on-the-standard-library).

### Test libraries

For multiplatform tests, the [`kotlin.test`](https://kotlinlang.org/api/latest/kotlin.test/) API is available.
As it is a multiplatform library,
you can add test dependencies to all source sets by specifying a single dependency in `commonTest`:

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

### kotlinx libraries

kotlinx libraries are multiplatform libraries maintained by the core Kotlin team at JetBrains
([kotlinx.serialization](https://github.com/kotlin/kotlinx.serialization)
and [kotlinx.coroutines](https://github.com/Kotlin/kotlinx.coroutines)
are primary examples).

Just like with [any multiplatform library](#dependency-on-kotlin-multiplatform-libraries),
to add a dependency, refer to a library artifact in the corresponding source set.

## Dependency on Kotlin Multiplatform libraries

You can add dependencies on libraries that have adopted Kotlin Multiplatform technology, such
as [SQLDelight](https://github.com/cashapp/sqldelight). The authors of these libraries usually provide guides for adding
their dependencies to your project.

> Look for Kotlin Multiplatform libraries on the [JetBrains' search platform](https://klibs.io/).
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

> You can also configure a common library in a top-level `dependencies {}` block. See [Configure dependencies at the top level](multiplatform-dsl-reference.md#configure-dependencies-at-the-top-level).
> 
{style="tip"}

### Library used in specific source sets

If you want to use a multiplatform library just for specific source sets, you can add it exclusively to them.
The library declarations will then only be available in those source sets.

> Use a common library name in such cases, not a platform-specific one.
> Like with SQLDelight in the example below, use `native-driver`, not `native-driver-iosx64`.
> Find the exact name in the library's documentation.
>
{style="note"}

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

## Dependency on another multiplatform project

You can connect one multiplatform project to another as a dependency. To do this, add a project-type dependency to the
source set that needs it.
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

## What's next?

Check out other resources on adding dependencies in multiplatform projects and learn more about:

* [Adding Android dependencies](multiplatform-android-dependencies.md)
* [Adding iOS dependencies](multiplatform-ios-dependencies.md)
