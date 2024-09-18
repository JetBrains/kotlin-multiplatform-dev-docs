[//]: # (title: Resources overview)

Compose Multiplatform provides a special library and Gradle plugin support for accessing resources in common code
across all supported platforms.
Resources are static content, such as images, fonts, and strings, which you can use in your application.

When working with resources in Compose Multiplatform, consider the current conditions:

* Almost all resources are read synchronously in the caller thread. The only exceptions are raw files
  and all of the resources on the JS platform that are read asynchronously.
* Reading big raw files, like long videos, as a stream is not supported yet.
  Use the [`getUri()`](compose-multiplatform-resources-usage#accessing-resources-from-external-libraries) function to pass separate files to a system API,
  for example, the [kotlinx-io](https://github.com/Kotlin/kotlinx-io) library.
* Starting with 1.6.10, you can place resources in any module or source set,
  as long as you are using Kotlin 2.0.0 or newer, and Gradle 7.6 or newer.

### Set up a project with multiplatform resources

To set up and configure all resources that your app should be able to access, see [](compose-multiplatform-resources-setup).

### Access the available resources in your code

To access resources in your UI, use the automatically generated accessors.
For the reference and various use cases, see [](compose-multiplatform-resources-usage).
