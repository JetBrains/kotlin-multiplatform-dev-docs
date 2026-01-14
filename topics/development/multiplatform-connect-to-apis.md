[//]: # (title: Use platform-specific APIs)

In this article, you'll learn how to use platform-specific APIs when developing multiplatform applications and libraries.

<video src="https://www.youtube.com/v/bSNumV04y_w" title="Using Platform-Specific APIs in KMP Apps"/>

## Kotlin multiplatform libraries

Before writing code that uses a platform-specific API, check whether you can use a multiplatform library instead.
This type of library provides a common Kotlin API that has a different implementation for different platforms.

There are already many libraries available that you can use to implement networking, logging, and analytics, as well as access device
functionality and more. For more information, see [this curated list](https://github.com/terrakok/kmm-awesome).

## Expected and actual functions and properties

Kotlin provides a language mechanism to access platform-specific APIs while developing common logic:
[expected and actual declarations](multiplatform-expect-actual.md).

With this mechanism, the common source set of a multiplatform module defines an expected declaration, and every platform
source set must provide the actual declaration that corresponds to the expected declaration. The compiler ensures that
every declaration marked with the `expect` keyword in the common source set has the corresponding declarations marked
with the `actual` keyword in all targeted platform source sets.

This works for most Kotlin declarations, such as functions, classes, interfaces, enumerations, properties, and
annotations. This section focuses on using expected and actual functions and properties.

![Using expected and actual functions and properties](expect-functions-properties.svg){width=700}

In this example, you'll define an expected `platform()` function in the common source set and provide the actual
implementations in the platform source sets. While generating the code for a specific platform, the Kotlin compiler merges
the expected and actual declarations. It generates one `platform()` function with its actual implementation. The expected and
actual declarations should be defined in the same package and merged into _one declaration_ in the resulting
platform code. Any invocation of the expected `platform()` function in the generated platform code will call the correct
actual implementation.

### Example: generate a UUID

Let's assume that you are developing iOS and Android applications using Kotlin Multiplatform and you want to generate a
universally unique identifier (UUID).

To do so, declare the expected function `randomUUID()` with the `expect` keyword in the common source set of
your Kotlin Multiplatform module. Do **not** include any implementation code.

```kotlin
// In the common source set:
expect fun randomUUID(): String
```

In each platform-specific source set (iOS and Android), provide the actual implementation for the `randomUUID()`
function expected in the common module. Use the `actual` keyword to mark these actual implementations.

![Generating UUID with expected and actual declarations](expect-generate-uuid.svg){width=700}

The following snippets show the implementations for Android and iOS. Platform-specific code uses the `actual` keyword
and the same name for the function:

```kotlin
// In the android source set:
import java.util.*

actual fun randomUUID() = UUID.randomUUID().toString()
```

```kotlin
// In the iOS source set:
import platform.Foundation.NSUUID

actual fun randomUUID(): String = NSUUID().UUIDString()
```

The Android implementation uses the APIs available on Android, while the iOS implementation uses the APIs available on iOS.
You can access iOS APIs from Kotlin/Native code.

While producing the resulting platform code for Android, the Kotlin compiler automatically merges the expected and actual
declarations and generates a single `randomUUID()` function with its actual Android-specific implementation. The same
process is repeated for iOS.

For simplicity, this and the following examples use the simplified source set names "common", "ios", and "android".
Typically, this implies `commonMain`, `iosMain`, and `androidMain`, and similar logic can be defined in the test source
sets `commonTest`, `iosTest`, and `androidTest`.

Similar to expected and actual functions, expected and actual properties allow you to use different values on
different platforms. Expected and actual functions and properties are most useful for simple cases.

## Interfaces in common code

If the platform-specific logic is too big and complex, you can simplify your code by defining an interface to represent
it in the common code and then providing different implementations in the platform source sets.

![Using interfaces](expect-interfaces.svg){width=700}

The implementations in the platform source sets use their corresponding dependencies:

```kotlin
// In the commonMain source set:
interface Platform {
    val name: String
}
```

```kotlin
// In the androidMain source set:
import android.os.Build

class AndroidPlatform : Platform {
    override val name: String = "Android ${Build.VERSION.SDK_INT}"
}
```

```kotlin
// In the iosMain source set:
import platform.UIKit.UIDevice

class IOSPlatform : Platform {
    override val name: String = UIDevice.currentDevice.systemName() + " " + UIDevice.currentDevice.systemVersion
}
```

To inject the appropriate platform implementations when you need a common interface, you can choose one of the following options, each of which is explained in more detail below:

* [Use expected and actual functions](#expected-and-actual-functions)
* [Provide implementations through different entry points](#different-entry-points)
* [Use a dependency injection framework](#dependency-injection-framework)

### Expected and actual functions

Define an expected function that returns a value of this interface, and then define actual functions that return its subclasses:

```kotlin
// In the commonMain source set:
interface Platform

expect fun platform(): Platform
```

```kotlin
// In the androidMain source set:
class AndroidPlatform : Platform

actual fun platform() = AndroidPlatform()
```

```kotlin
// In the iosMain source set:
class IOSPlatform : Platform

actual fun platform() = IOSPlatform()
```

When you call the `platform()` function in the common code, it can work with an object of the `Platform` type.
When you run this common code on Android, the `platform()` call returns an instance of the `AndroidPlatform` class.
When you run it on iOS, `platform()` returns an instance of the `IOSPlatform` class.

### Different entry points

If you control the entry points, you can construct implementations of each platform artifact without using
expected and actual declarations. To do so, define the platform implementations in the shared Kotlin Multiplatform module,
but instantiate them in the platform modules:

```kotlin
// Shared Kotlin Multiplatform module
// In the commonMain source set:
interface Platform

fun application(p: Platform) {
    // application logic
}
```

```kotlin
// In the androidMain source set:
class AndroidPlatform : Platform
```

```kotlin
// In the iosMain source set:
class IOSPlatform : Platform
```

```kotlin
// In the androidApp platform module:
import android.app.Application
import mysharedpackage.*

class MyApp : Application() {
    override fun onCreate() {
        super.onCreate()
        application(AndroidPlatform())
    }
}
```

```Swift
// In the iosApp platform module (in Swift):
import shared

@main
struct iOSApp : App {
    init() {
        application(IOSPlatform())
    }
}
```

On Android, you should create an instance of `AndroidPlatform` and pass it to the `application()` function, while on iOS, you
should similarly create and pass an instance of `IOSPlatform`. These entry points don't need to be the entry points of your
applications, but this is where you can call the specific functionality of your shared module.

Providing the right implementations with expected and actual functions or directly through entry points works well for
simple scenarios. However, if you use a dependency injection framework in your project,
we recommend using it in simple cases to ensure consistency.

### Dependency injection framework

A modern application typically uses a dependency injection (DI) framework to create a loosely coupled architecture. The
DI framework allows injecting dependencies into components based on the current environment.

Any DI framework that supports Kotlin Multiplatform can help you inject different dependencies for different platforms.

For example, [Koin](https://insert-koin.io/) is a dependency injection framework that supports Kotlin Multiplatform:

```kotlin
// In the common source set:
import org.koin.dsl.module

interface Platform

expect val platformModule: Module
```

```kotlin
// In the androidMain source set:
class AndroidPlatform : Platform

actual val platformModule: Module = module {
    single<Platform> {
        AndroidPlatform()
    }
}
```

```kotlin
// In the iosMain source set:
class IOSPlatform : Platform

actual val platformModule = module {
    single<Platform> { IOSPlatform() }
}
```

Here, Koin DSLs create modules that define components for injection. You declare a module in common code with
the `expect` keyword and then provide a platform-specific implementation for each platform using the `actual` keyword.
The framework takes care of selecting the correct implementation at runtime.

When you use a DI framework, you inject all of the dependencies through this framework. The same logic applies to handling
platform dependencies. We recommend continuing to use DI if you already have it in your project, rather than using the expected
and actual functions manually. This way, you can avoid mixing two different ways of injecting dependencies.

You also don't have to always implement the common interface in Kotlin. You can do it in another language, like
Swift, in a different _platform module_. If you choose this approach, you should then provide the implementation from the iOS platform module using the DI
framework:

![Using dependency injection framework](expect-di-framework.svg){width=700}

This approach only works if you put the implementations in the platform modules. It isn't very scalable, as your Kotlin
Multiplatform module can't be self-sufficient, and you'll need to implement the common interface in a different module.

<!-- If you're interested in having this functionality expanded to a shared module, please vote for this issue in Youtrack and describe your use case. -->

## What's next?

For more examples and information on the expect/actual mechanism, see [Expected and actual declarations](multiplatform-expect-actual.md).