[//]: # (title: Resources overview)
<web-summary>Learn how to work with resources in Compose Multiplatform.</web-summary>

Compose Multiplatform provides a special resources library (`org.jetbrains.compose.components:components-resources`) and 
Gradle plugin support for accessing resources in common code across all supported platforms.
Resources are static content, such as images, fonts, and strings, which you can use in your application.

When working with resources in Compose Multiplatform, consider the current conditions:

* Almost all resources are read synchronously in the caller thread. The only exceptions are raw files
  and [web resources](compose-web-resources.md) that are read asynchronously.
* A missing resource causes an exception on all platforms, with the following exceptions:
  * On web, composables such as `painterResource()` and `stringResource()` shows an empty placeholder if a resource fails to load.
  * On iOS and web, [`Res.getUri()`](compose-multiplatform-resources-usage.md#accessing-multiplatform-resources-from-external-libraries)
    doesn't check whether the resource exists.
* Reading big raw files, like long videos, as a stream is not supported yet.
  Use the [`getUri()`](compose-multiplatform-resources-usage.md#accessing-multiplatform-resources-from-external-libraries)
  function to pass separate files to a system API, for example, the [kotlinx-io](https://github.com/Kotlin/kotlinx-io) library.
* Starting with 1.6.10, you can place resources in any module or source set,
  as long as you are using Kotlin 2.0.0 or newer, and Gradle 7.6 or newer.

To learn how to work with resources in Compose Multiplatform, refer to the following key sections:

* [](compose-multiplatform-resources-setup.md)

  Add the `resources` library dependency and set up all resources that your app should be able to access.

* [](compose-multiplatform-resources-usage.md)

  Learn how to use the automatically generated accessors to access resources directly in your UI code.

* [Local resource environment](compose-resource-environment.md)
  
  Manage your app's resource environment like in-app theme and language.
