# How multiplatform Jetpack libraries are packaged

Compose Multiplatform brings the full power of Jetpack Compose and its related AndroidX libraries
to other platforms besides Android.
As shown on the [Android Developers website](https://developer.android.com/kotlin/multiplatform),
many Jetpack libraries (like `androidx.annotation`) are published by the Android team fully multiplatform
and can be used in KMP projects as is.
Others, such as, Compose itself, Navigation, Lifecycle, and ViewModel, need additional support to work in common code.

The Compose Multiplatform team at JetBrains produces artifacts for such libraries for platforms besides Android,
and then publishes them all together with original Android artifacts under a single group ID.
This way, when you add such a multiplatform dependency to your common source set,
the Android distribution of your app uses the Android artifact while distributions for other targets use artifacts
built for other platforms.

Here's an outline of the process:

![](androidx-cmp-artifacts.svg)

For example, "Navigation artifacts for iOS" refers to the collection of the following multiplatform artifacts:
* `org.jetbrains.androidx.navigation.navigation-compose-uikitarm64`
* `org.jetbrains.androidx.navigation.navigation-compose-uikitsimarm64`
* `org.jetbrains.androidx.navigation.navigation-common-iosx64`
* `org.jetbrains.androidx.navigation.navigation-runtime-iossimulatorarm64`
* And so on.

All of these artifacts, together with the artifacts for other platforms and a reference to the original Android library (`androidx.navigation.navigation-compose`),
are published as a group. They are accessible through the unifying 
`org.jetbrains.androidx.navigation.navigation-compose` dependency.
The Compose Multiplatform Gradle plugin handles the mapping of platform-specific artifacts to distributions.

With this approach, the Android app produced by a Kotlin Multiplatform (KMP) project with that dependency uses the original Android Navigation library. The iOS app, on the other hand, uses the corresponding iOS library built by JetBrains.

## Compose packages available for multiplatform projects

Among the base Compose libraries, the fundamental `androidx.compose.runtime` is fully multiplatform.
  ([Previously used](whats-new-compose-190.md#multiplatform-targets-in-androidx-compose-runtime-runtime)
  `org.jetbrains.compose.runtime` artifact now serves as an alias.)
In addition, Compose Multiplatform implements:
   * Multiplatform versions of `androidx.compose.ui` and `androidx.compose.foundation`,
     which are available in Compose Multiplatform projects as `org.jetbrains.compose.ui` and `org.jetbrains.compose.foundation`.
   * Multiplatform versions of `androidx.compose.material3` and `androidx.compose.material`, which are similarly
     packaged (`org.jetbrains.compose.material3` and `org.jetbrains.compose.material`).
     Unlike the others, the Material 3 library is not coupled with Compose Multiplatform versions.
     So instead of the `material3` alias, you can provide a direct dependency. For example, you might use an EAP version.
   * Material 3 adaptive library as standalone artifacts (`org.jetbrains.compose.material3.adaptive:adaptive*`)

## Additional multiplatform libraries

Some functionality required for building Compose apps is outside the scope of AndroidX,
so JetBrains implements it as multiplatform libraries bundled with Compose Multiplatform, such as:

* Compose Multiplatform Gradle plugin, which:
    * Provides a Gradle DSL for configuring Compose Multiplatform projects.
    * Helps create distribution packages for desktop and web targets.
    * Supports the multiplatform resource library in making resources available correctly for each target.
* `org.jetbrains.compose.components.resources`, which provides  support for [cross-platform resources](compose-multiplatform-resources.md).
* `org.jetbrains.compose.components.uiToolingPreview`, which supports Compose UI previews for common code in IntelliJ IDEA and Android Studio.
* `org.jetbrains.compose.components.animatedimage`, which supports displaying animated images.
* `org.jetbrains.compose.components.splitpane`, which implements an analog of Swing's [JSplitPane](https://docs.oracle.com/javase/8/docs/api/javax/swing/JSplitPane.html).


