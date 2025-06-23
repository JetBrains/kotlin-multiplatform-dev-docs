[//]: # (title: Resources overview)

Compose Multiplatform provides a special `compose-multiplatform-resources` library and 
Gradle plugin support for accessing resources in common code across all supported platforms.
Resources are static content, such as images, fonts, and strings, which you can use in your application.

When working with resources in Compose Multiplatform, consider the current conditions:

* Almost all resources are read synchronously in the caller thread. The only exceptions are raw files
  and web resources that are read asynchronously.
* Reading big raw files, like long videos, as a stream is not supported yet.
  Use the [`getUri()`](compose-multiplatform-resources-usage.md#accessing-multiplatform-resources-from-external-libraries) function to pass separate files to a system API,
  for example, the [kotlinx-io](https://github.com/Kotlin/kotlinx-io) library.
* Starting with 1.6.10, you can place resources in any module or source set,
  as long as you are using Kotlin 2.0.0 or newer, and Gradle 7.6 or newer.

To learn how to work with resources in Compose Multiplatform, refer to the following key sections:

* [](compose-multiplatform-resources-setup.md)

  Add the `resources` library dependency and set up all resources that your app should be able to access.

* [](compose-multiplatform-resources-usage.md)

  Learn how to use the automatically generated accessors to access resources directly in your UI code.

* [Local resource environment](compose-resource-environment.md)
  
  Manage your app's resource environment like in-app theme and language.
