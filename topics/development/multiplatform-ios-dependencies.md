[//]: # (title: Adding iOS dependencies)

Apple SDK dependencies (such as Foundation or Core Bluetooth) are available as a set of prebuilt libraries in Kotlin
Multiplatform projects. They do not require any additional configuration.

You can also reuse other libraries and frameworks from the iOS ecosystem in your iOS source sets. Kotlin supports
interoperability with Objective-C dependencies and Swift dependencies if their APIs are exported to Objective-C with
the `@objc` attribute. Pure Swift dependencies are not yet supported.

To handle iOS dependencies in Kotlin Multiplatform projects, you can manage them with the [cinterop tool](#with-cinterop)
or use the [CocoaPods dependency manager](#with-cocoapods) (pure Swift pods are not supported).

### With cinterop

You can use the cinterop tool to create Kotlin bindings for Objective-C or Swift
declarations. This will allow you to call them from the Kotlin code.

The steps differ a bit for [libraries](#add-a-library) and [frameworks](#add-a-framework),
but the general workflow looks like this:

1. Download your dependency.
2. Build it to get its binaries.
3. Create a special `.def` [definition file](https://kotlinlang.org/docs/native-definition-file.html) that describes this dependency to cinterop.
4. Adjust your build script to generate bindings during the build.

#### Add a library

1. Download the library source code and place it somewhere where you can reference it from your project.
2. Build a library (library authors usually provide a guide on how to do this) and get a path to the binaries.
3. In your project, create a `.def` file, for example `DateTools.def`.
4. Add the first string to this file: `language = Objective-C`. If you want to use a pure C dependency, omit the language
   property.
5. Provide values for two mandatory properties:

    * `headers` describes which headers will be processed by cinterop.
    * `package` sets the name of the package these declarations should be put into.

   For example:

    ```none
    headers = DateTools.h
    package = DateTools
    ```

6. Add information about interoperability with this library to the build script:

    * Pass the path to the `.def` file. This path can be omitted if your `.def` file has the same name as cinterop and
      is placed in the `src/nativeInterop/cinterop/` directory.
    * Tell cinterop where to look for header files using the `includeDirs` option.
    * Configure linking to library binaries.

    <tabs group="build-script">
    <tab title="Kotlin" group-key="kotlin">

    ```kotlin
    kotlin {
        iosArm64() {
            compilations.getByName("main") {
                val DateTools by cinterops.creating {
                    // Path to the .def file
                    definitionFile.set(project.file("src/nativeInterop/cinterop/DateTools.def"))

                    // Directories for header search (an analogue of the -I<path> compiler option)
                    includeDirs("include/this/directory", "path/to/another/directory")
                }
                val anotherInterop by cinterops.creating { /* ... */ }
            }

            binaries.all {
                // Linker options required to link to the library.
                linkerOpts("-L/path/to/library/binaries", "-lbinaryname")
            }
        }
    }
    ```

    </tab>
    <tab title="Groovy" group-key="groovy">

    ```groovy
    kotlin {
        iosArm64 {
            compilations.main {
                cinterops {
                    DateTools {
                        // Path to the .def file
                        definitionFile = project.file("src/nativeInterop/cinterop/DateTools.def")

                        // Directories for header search (an analogue of the -I<path> compiler option)
                        includeDirs("include/this/directory", "path/to/another/directory")
                    }
                    anotherInterop { /* ... */ }
                }
            }

            binaries.all {
                // Linker options required to link to the library.
                linkerOpts "-L/path/to/library/binaries", "-lbinaryname"
            }
        }
    }
    ```

    </tab>
    </tabs>

7. Build the project.

Now you can use this dependency in your Kotlin code. To do that, import the package you've set up in the `package`
property in the `.def` file. For the example above, this will be:

```kotlin
import DateTools.*
```

> See the sample project that [uses the cinterop tool and the libcurl library](https://github.com/Kotlin/kotlin-hands-on-intro-kotlin-native).
>
{style="tip"}

#### Add a framework

1. Download the framework source code and place it somewhere that you can reference it from your project.
2. Build the framework (framework authors usually provide a guide on how to do this) and get a path to the binaries.
3. In your project, create a `.def` file, for example `MyFramework.def`.
4. Add the first string to this file: `language = Objective-C`. If you want to use a pure C dependency, omit the
   language property.
5. Provide values for these two mandatory properties:

    * `modules` – the name of the framework that should be processed by the cinterop.
    * `package` – the name of the package these declarations should be put into.

    For example:
    
    ```none
    modules = MyFramework
    package = MyFramework
    ```

6. Add information about interoperability with the framework to the build script:

    * Pass the path to the .def file. This path can be omitted if your `.def` file has the same name as the cinterop and
      is placed in the `src/nativeInterop/cinterop/` directory.
    * Pass the framework name to the compiler and linker using the `-framework` option. Pass the path to the framework
      sources and binaries to the compiler and linker using the `-F` option.

    <tabs group="build-script">
    <tab title="Kotlin" group-key="kotlin">

    ```kotlin
    kotlin {
        iosArm64() {
            compilations.getByName("main") {
                val DateTools by cinterops.creating {
                    // Path to the .def file
                    definitionFile.set(project.file("src/nativeInterop/cinterop/DateTools.def"))

                    compilerOpts("-framework", "MyFramework", "-F/path/to/framework/")
                }
                val anotherInterop by cinterops.creating { /* ... */ }
            }

            binaries.all {
                // Tell the linker where the framework is located.
                linkerOpts("-framework", "MyFramework", "-F/path/to/framework/")
            }
       }
    }
    ```

    </tab>
    <tab title="Groovy" group-key="groovy">

    ```groovy
    kotlin {
        iosArm64 {
            compilations.main {
                cinterops {
                    DateTools {
                        // Path to the .def file
                        definitionFile = project.file("src/nativeInterop/cinterop/MyFramework.def")

                        compilerOpts("-framework", "MyFramework", "-F/path/to/framework/")
                    }
                    anotherInterop { /* ... */ }
                }
            }

            binaries.all {
                // Tell the linker where the framework is located.
                linkerOpts("-framework", "MyFramework", "-F/path/to/framework/")
            }
        }
    }
    ```

    </tab>
    </tabs>

7. Build the project.

Now you can use this dependency in your Kotlin code. To do this, import the package you've set up in the package
property in the `.def` file. For the example above, this will be:

```kotlin
import MyFramework.*
```

Learn more about [Swift/Objective-C interop](https://kotlinlang.org/docs/native-objc-interop.html) and
[configuring cinterop from Gradle](multiplatform-dsl-reference.md#cinterops).

### With CocoaPods

1. Perform [initial CocoaPods integration setup](multiplatform-cocoapods-overview.md#set-up-an-environment-to-work-with-cocoapods).
2. Add a dependency on a Pod library from the CocoaPods repository that you want to use by including the `pod()`
   function call in `build.gradle(.kts)` of your project.

    <tabs group="build-script">
    <tab title="Kotlin" group-key="kotlin">

    ```kotlin
    kotlin {
        cocoapods {
            version = "2.0"
            //..
            pod("SDWebImage") {
                version = "5.20.0"
            }
        }
    }
    ```

    </tab>
    <tab title="Groovy" group-key="groovy">

    ```groovy
    kotlin {
        cocoapods {
            version = '2.0'
            //..
            pod('SDWebImage') {
                version = '5.20.0'
            }
        }
    }
    ```

    </tab>
    </tabs>

   You can add the following dependencies on a Pod library:

   * [From the CocoaPods repository](multiplatform-cocoapods-libraries.md#from-the-cocoapods-repository)
   * [On a locally stored library](multiplatform-cocoapods-libraries.md#on-a-locally-stored-library)
   * [From a custom Git repository](multiplatform-cocoapods-libraries.md#from-a-custom-git-repository)
   * [From a custom Podspec repository](multiplatform-cocoapods-libraries.md#from-a-custom-podspec-repository)
   * [With custom cinterop options](multiplatform-cocoapods-libraries.md#with-custom-cinterop-options)

3. Run **Build** | **Reload All Gradle Projects** in IntelliJ IDEA (or **File** | **Sync Project with Gradle Files** in Android Studio)
   to re-import the project.

To use the dependency in your Kotlin code, import the package `cocoapods.<library-name>`. For the example above, it's:

```kotlin
import cocoapods.SDWebImage.*
```

> * See the sample project with [different Pod dependencies set up in a Kotlin project](https://github.com/Kotlin/kmp-with-cocoapods-multitarget-xcode-sample).
> * Check out the sample project where [an Xcode project with several targets depends on a Kotlin library](https://github.com/Kotlin/kmp-with-cocoapods-multitarget-xcode-sample).
> 
{style="tip"}

## What's next?

Check out other resources on adding dependencies in multiplatform projects and learn more about:

* [Connecting platform libraries](https://kotlinlang.org/docs/native-platform-libs.html)
* [Adding dependencies on multiplatform libraries or other multiplatform projects](multiplatform-add-dependencies.md)
* [Adding Android dependencies](multiplatform-android-dependencies.md)