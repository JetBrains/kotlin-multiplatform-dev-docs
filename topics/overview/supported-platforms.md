[//]: # (title: Stability of supported platforms)

Kotlin Multiplatform allows you to create applications for various platforms and share code across them, allowing you to
reach users on their favorite devices. Different platforms may have varying levels of stability based on their support
from the core Kotlin Multiplatform technology for code-sharing and the Compose Multiplatform UI framework.

This page contains information to help you identify which platforms align with your project needs, based on their
stability level.

## Core Kotlin Multiplatform technology stability levels

Here's a quick guide to platform stability levels for the Core Kotlin Multiplatform technology and their meaning:

**Experimental** means "for trial use only":

* We are just trying out an idea and want some users to play with it and give feedback. If it doesn't work out, we may
  drop it any minute.

**Best effort** means "it is safe to use in most scenarios":

* There might be unexpected breaking changes.
* You may experience issues during migration.

**Stable** means "use it even in most conservative scenarios":

* It's done. We will be evolving it according to our strict [backward compatibility rules](https://kotlinfoundation.org/language-committee-guidelines/).

### Current platform stability levels for Core Kotlin Multiplatform technology

| Platform                 | Stability level |
|--------------------------|-----------------|
| Android                  | Stable          |
| iOS                      | Stable          |
| Desktop (JVM)            | Stable          |
| Server-side (JVM)        | Stable          |
| Web based on Kotlin/Wasm | Experimental    |
| Web based on Kotlin/JS   | Stable          |
| watchOS                  | Best effort     |
| tvOS                     | Best effort     |

Kotlin Multiplatform supports more native platforms than are listed here. To understand the level of support for each of
them, see [Kotlin/Native target support](https://kotlinlang.org/docs/native-target-support.html).

For more information on the stability levels of Kotlin components like Kotlin Multiplatform,
see [Stability levels of Kotlin components](https://kotlinlang.org/docs/components-stability.html#current-stability-of-kotlin-components).

## Compose Multiplatform UI framework stability levels

Here's a quick guide to platform stability levels for the Compose Multiplatform UI framework and their meaning:

**Experimental** means "it's under development":

* Some features might not be available yet and those features that are present might have performance issues or bugs.
* There might be changes in the future, and breaking changes can occur often.

**Alpha** means "use at your own risk, expect migration issues":

* We have decided to productize the platform support but it hasn't reached its final shape yet.

**Beta** means "you can use it, we'll do our best to minimize migration issues for you":

* It's almost done, user feedback is especially important now.
* Still, it's not 100% finished, so changes are possible (including ones based on your own feedback).
  We collectively refer to **Experimental**, **Alpha** and **Beta** as **pre-stable** levels.

**Stable** means "use it even in most conservative scenarios":

* The framework provides a comprehensive API surface that allows you to write beautiful, production-ready applications,
  without encountering performance or other issues in the framework itself.
* API-breaking changes can only be made 2 versions after an official deprecation announcement.

### Current platform stability levels for Compose Multiplatform UI framework

| Platform                 | Stability level |
|--------------------------|-----------------|
| Android                  | Stable          |
| iOS                      | Alpha           |
| Desktop (JVM)            | Stable          |
| Web based on Kotlin/Wasm | Experimental    |

<!-- Learn more about which IDE we recommend for your code-sharing scenario across different combinations of platforms. -->
