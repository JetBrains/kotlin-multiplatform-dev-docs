[//]: # (title: Compose Multiplatform and Jetpack Compose)

Compose Multiplatform is a cross-platform UI toolkit developed by JetBrains.
It extends Google's [Jetpack Compose](https://developer.android.com/jetpack/compose) toolkit for Android by providing support for additional platforms.

Compose Multiplatform makes Compose APIs available for use from [common Kotlin code](https://kotlinlang.org/docs/multiplatform-discover-project.html#common-code),
allowing you to write shared Compose UI code that can run on Android, Desktop, iOS, and Web.

|                  | **Compose Multiplatform**  | **Jetpack Compose** |
|------------------|----------------------------|---------------------|
| **Platforms**    | Android, iOS, Desktop, Web | Android             |
| **Supported by** | JetBrains                  | Google              |

As Compose Multiplatform is based on Jetpack Compose, using these frameworks is a very similar experience. 
Both are powered by the Compose compiler and runtime. They use the same core concepts and the same APIs for building UI,
including `@Composable` functions, state handling APIs like `remember`, UI components such as `Row` and `Column`,
modifiers, animation APIs, and so on.
This means that you can reuse existing knowledge of Jetpack Compose in Compose Multiplatform, 
and you can learn from almost any Jetpack Compose material, including [Google's official documentation](https://developer.android.com/jetpack/compose/documentation).

Naturally, Compose Multiplatform has platform-specific features and considerations:
The [Android-only components page](compose-android-only-components.md) lists APIs that are closely tied to the 
Android platform and are therefore not available from common Compose Multiplatform code.
Some platform-specific APIs, such as window handling APIs for Desktop or the UIKit compatibility APIs for iOS,
are only available on their respective platforms.

Here’s an overview of the availability of popular components and APIs:

|                                                                                                                     | **Compose Multiplatform**                                                                                 | **Jetpack Compose**                                                                                    |
|---------------------------------------------------------------------------------------------------------------------|-----------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------|
| [Compose Animation](https://developer.android.com/jetpack/androidx/releases/compose-animation)                      | Yes                                                                                                       | Yes                                                                                                    |
| [Compose Compiler](https://developer.android.com/jetpack/androidx/releases/compose-compiler)                        | Yes                                                                                                       | Yes                                                                                                    |
| [Compose Foundation](https://developer.android.com/jetpack/androidx/releases/compose-foundation)                    | Yes                                                                                                       | Yes                                                                                                    |
| [Compose Material](https://developer.android.com/jetpack/androidx/releases/compose-material)                        | Yes                                                                                                       | Yes                                                                                                    |
| [Compose Material 3](https://developer.android.com/jetpack/androidx/releases/compose-material30)                    | Yes <sup>1</sup>                                                                                          | Yes                                                                                                    |
| [Compose Runtime](https://developer.android.com/jetpack/androidx/releases/compose-runtime)                          | Yes <sup>2</sup>                                                                                          | Yes                                                                                                    |
| [Compose UI](https://developer.android.com/jetpack/androidx/releases/compose-ui)                                    | Yes                                                                                                       | Yes                                                                                                    |
| [Jetpack Lifecycle](https://developer.android.com/jetpack/androidx/releases/lifecycle)                              | [Yes](compose-lifecycle.md)                                                                               | Yes                                                                                                    |
| [Jetpack ViewModel](https://developer.android.com/topic/libraries/architecture/viewmodel)                           | [Yes](compose-viewmodel.md)                                                                               | Yes                                                                                                    |
| [Jetpack Navigation Compose](https://developer.android.com/jetpack/androidx/releases/navigation)                    | [Yes](compose-navigation-routing.md)                                                                      | Yes                                                                                                    |
| Resources                                                                                                           | [Compose Multiplatform resources library](compose-multiplatform-resources.md) using the `Res` class       | [Android resource system](https://developer.android.com/jetpack/compose/resources) using the `R` class |
| [Maps Compose](https://developers.google.com/maps/documentation/android-sdk/maps-compose)                           | No                                                                                                        | Yes                                                                                                    |
| [Third-party libraries](#libraries-for-compose-multiplatform) for UI components, navigation, architecture, and more | [Compose Multiplatform libraries](https://github.com/terrakok/kmp-awesome?tab=readme-ov-file#-compose-ui) | Jetpack Compose and Compose Multiplatform libraries                                                    |

<sup>1</sup> except `material3-adaptive` and `material3-window-size-class` (a third-party library available)  
<sup>2</sup> except `androidx.compose.runtime.rxjava2` and `androidx.compose.runtime.rxjava3`

## Technical details

Compose Multiplatform builds on code and releases published by Google.
While Google's focus is Jetpack Compose for Android,
there is close collaboration between Google and JetBrains to enable Compose Multiplatform.

Jetpack includes first-party libraries such as Foundation and Material, 
which Google publishes for Android.
To make the APIs provided by these libraries available from common code, 
JetBrains maintains multiplatform versions of these libraries, which are published for targets other than Android.

> Learn more about the release cycles on the 
> [Compatibility and versions page](compose-compatibility-and-versioning.md#jetpack-compose-and-compose-multiplatform-release-cycles).
> 
{style="tip"}

When you build your Compose Multiplatform application for Android, you use the Jetpack Compose artifacts published by Google.
For example, if you add `compose.material3` to your dependencies, your project will use `androidx.compose.material3:material3` 
in the Android target, and `org.jetbrains.compose.material3:material3` in other targets. 
This is done automatically, based on Gradle Module Metadata in the multiplatform artifacts.

## Libraries for Compose Multiplatform

By using Compose Multiplatform, you can publish your libraries that use Compose APIs as [Kotlin Multiplatform libraries](https://kotlinlang.org/docs/multiplatform-publish-lib.html). 
This makes them available for use from common Kotlin code, targeting multiple platforms.

So if you're building a new library with Compose, consider taking advantage of that and building it as a multiplatform library using Compose Multiplatform.
If you've already built a Jetpack Compose library for Android, consider making that library multiplatform. 
There are already [many Compose Multiplatform libraries](https://github.com/terrakok/kmp-awesome#-compose-ui) available in the ecosystem.

When a library is published with Compose Multiplatform, apps that only use Jetpack Compose are still able to consume it seamlessly;
they simply use the Android artifacts of the library.

## What's next

Read more about Compose Multiplatform implementation for the following components:
  * [](compose-lifecycle.md)
  * [Resources](compose-multiplatform-resources.md)
  * [](compose-viewmodel.md)
  * [](compose-navigation-routing.md)