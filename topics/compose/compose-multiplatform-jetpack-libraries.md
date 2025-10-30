# How multiplatform Jetpack libraries are packaged

Compose Multiplatform brings the full power of Jetpack Compose and its related AndroidX libraries
to other platforms besides Android.
As shown on the [Android Developers website](https://developer.android.com/kotlin/multiplatform),
many Jetpack libraries (like `androidx.annotation`) are published by the Android team fully multiplatform
and can be used in KMP projects as is.
Others, namely, Compose itself, Navigation, Lifecycle, and ViewModel — need additional support to work in common code.

The Compose Multiplatform team at JetBrains produces artifacts for such libraries for platforms besides Android,
and then publishes them all together with original Android artifacts under a single group ID.
So when you add this single multiplatform dependency to your common source set,
the Android distribution of your app uses the Android artifact while distributions for other targets use the additional platform-specific artifacts.

Here's the outline of the process:

![](org.jetbrains.androidx-diagram.png)

So, for example, "Navigation artifacts for iOS" refers to the collection of the following multiplatform artifacts:
* `org.jetbrains.androidx.navigation.navigation-compose-uikitarm64`
* `org.jetbrains.androidx.navigation.navigation-compose-uikitsimarm64`
* `org.jetbrains.androidx.navigation.navigation-common-iosx64`
* `org.jetbrains.androidx.navigation.navigation-runtime-iossimulatorarm64`
* and so on.

All of them, together with the artifacts for other platforms and a reference to the original Android library (`androidx.navigation.navigation-compose`),
are published as a group and accessible through the unifying 
`org.jetbrains.androidx.navigation.navigation-compose` dependency.
Compose Multiplatform Gradle plugin handles the mapping of platform-specific artifacts to distributions.

This way, the Android app produced by a KMP project with that dependency uses the original Android Navigation library,
while the iOS app uses the corresponding iOS library built by JetBrains.

## Compose packages available for multiplatform projects

Among the base Compose libraries:

* The fundamental `androidx.compose.runtime` is fully multiplatform by itself.
  ([Previously used](whats-new-compose-190.md#multiplatform-targets-in-androidx-compose-runtime-runtime)
  `org.jetbrains.compose.runtime` artifact now serves as an alias.)
* Compose Multiplatform implements:
   * Multiplatform versions of `androidx.compose.ui` and `androidx.compose.foundation`
     which are available in Compose Multiplatform projects using the `compose.ui` and `compose.foundation` aliases.
   * Multiplatform versions of `androidx.compose.material3` and `androidx.compose.material`.
     These are also available through aliases (`compose.material3` and `compose.material`), but, unlike the others,
     the Material 3 library is not coupled with Compose Multiplatform versions.
     So, instead of the `material3` alias you can provide a direct dependency, for example on an EAP version.
   * Material 3 adaptive library as standalone artifacts (`org.jetbrains.compose.material3.adaptive:adaptive*`)

## Additional multiplatform libraries

Some functionality required for building Compose apps is out of scope of AndroidX,
so JetBrains implements it as multiplatform libraries bundled with Compose Multiplatform:

* Compose Multiplatform Gradle plugin, which:
    * provides a Gradle DSL for configuring Compose Multiplatform projects,
    * helps create distribution packages for desktop and web targets,
    * supports the multiplatform resource library in making resources available correctly for each target.
* `org.jetbrains.compose.components.resources`, which provides [cross-platform resources](compose-multiplatform-resources.md) support.
* `org.jetbrains.compose.components.uiToolingPreview`, which supports Compose UI previews in IntelliJ IDEA and Android Studio.
* `org.jetbrains.compose.components.animatedimage`, which supports displaying animated images.
* `org.jetbrains.compose.components.splitpane` TODO what's that? Is there anything else?


