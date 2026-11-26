[//]: # (title: Setup and configuration for multiplatform resources)
<web-summary>Learn how to set up your Compose Multiplatform project to use resources.</web-summary>

<show-structure depth="3"/>

To properly configure the project to use multiplatform resources:

1. Add the library dependency.
2. Create the necessary directories for each kind of resource.
3. Create additional directories for qualified resources (for example, different images for the dark UI theme
    or localized strings).

## Build script and directory setup

To access resources in your multiplatform projects, add the library dependency and organize files within your project directory:

1. In the `build.gradle.kts` file of the module where you plan to place the resources (for example, `shared`),
   add a dependency to the `commonMain` source set:

   ```kotlin
   kotlin {
       //...
       sourceSets {
           commonMain.dependencies {
               implementation("org.jetbrains.compose.components:components-resources:%org.jetbrains.compose%")
           }
       }
   }
   ```

2. Create a new directory `composeResources` in the source set directory you want to add the resources to
   (`commonMain` in this example):

   ![Compose resources project structure](compose-resources-structure.png){width=250}

3. Organize the `composeResources` directory structure according to these rules:

   * Images should be in the `drawable` directory. Compose Multiplatform supports rasterized images (JPEG, PNG, BMP, and WebP),
     vector Android XML images (without references to Android resources), and SVG images on all platforms except Android.
     
     Put SVG images into the `drawable` directory of a source set that is not compiled for Android. If an SVG image is placed in 
     `commonMain`, the project builds, but the app fails at runtime on Android.
   * Fonts should be in the `font` directory.
   * Strings should be in the `values` directory.
   * Other files should be in the `files` directory, with any folder hierarchy you may find appropriate.

### Custom resource directories

In the `compose.resources {}` block of the `build.gradle.kts` file, you can specify custom resource directories for each 
source set. A custom directory replaces the default `composeResources` directory of its source set. Resources from other 
source sets, such as `commonMain`, are still merged as usual.

Custom directories should follow the same file structure as the default `composeResources`: with a `drawable` subdirectory
for images, a `font` subdirectory for fonts, and so on.

To specify a custom resource directory, point to a specific project folder. The path should be relative to the module 
directory, not the root project:

```kotlin
compose.resources {
    customDirectory(
        sourceSetName = "jvmMain",
        directoryProvider = provider { layout.projectDirectory.dir("desktopResources") }
    )
}
```

You can also set up a folder populated by a Gradle task, for example, with downloaded files.
The task runs automatically before the resources of the source set are processed:

```kotlin
abstract class DownloadRemoteFiles : DefaultTask() {

    @get:OutputDirectory
    abstract val outputDir: DirectoryProperty

    @TaskAction
    fun run() { /* your code for downloading files into outputDir */ }
}

compose.resources {
    customDirectory(
        sourceSetName = "iosMain",
        directoryProvider = tasks.register<DownloadRemoteFiles>("downloadedRemoteFiles") {
            outputDir.set(layout.buildDirectory.dir("downloadedRemoteFiles"))
        }.flatMap { it.outputDir }
    )
}
```
{initial-collapse-state="collapsed" collapsible="true"  collapsed-title="directoryProvider = tasks.register<DownloadRemoteFiles>"}

Learn more about customizing access to resources in [Access and usage](compose-multiplatform-resources-usage.md#customizing-accessor-class-generation).

### Resources in the Android library target

To include multiplatform resources in the Android library target, enable Android resources in the `android {}` block
of the `kotlin {}` block:

```kotlin
kotlin {
    android {
        androidResources.enable = true
    }
}
```

The `Res` class and resource accessors are generated regardless of this setting, and the project compiles without it.
However, if the option is disabled, resources are not packaged into the Android artifact, 
and the app throws a `MissingResourceException` at runtime when you access a resource.

## Custom web resource paths

You can set a custom path to your web resources using the `configureWebResources()` function.
This is a runtime Kotlin function from the `org.jetbrains.compose.resources` package.
Call it in the `main()` function of your web application before the first composition.

```kotlin
// Maps resources to an application-specific path
configureWebResources {
    resourcePathMapping { path -> "/myApp/resources/$path" }
}

// Maps resources to an external CDN
configureWebResources {
    resourcePathMapping { path -> "https://mycdn.com/myApp/res/$path" }
}
```

The `resourcePathMapping` lambda receives the full internal path of a resource, 
for example, `composeResources/<package of the Res class>/files/myDir/someFile.bin`,
and returns the path or URL to load the resource from.

The browser loads the resource from the returned path in the same way as it follows a link in HTML:

* A path starting with `/` is resolved from the domain root.
* An absolute URL (starting with `http://` or `https://`) is used as is. Use it for resources hosted on an external domain or CDN.
* Any other value (for example, `./my-resources/`) is resolved relative to the directory of the current page.

Be default, the mapping is `"./$path"`: resources are loaded from the `composeResources` directory next to the page.
This matches the layout of the web distribution built by Gradle.

> `configureWebResources()` applies mapping to the resources of all modules, including libraries, and each call 
> replaces the previous mapping. 
> To return to the default behavior, set it explicitly: `configureWebResources { resourcePathMapping { path -> "./$path" } }`.
> 
{style="note"}

## Qualifiers

Sometimes, the same resource should be presented in different ways depending on the environment, such as locale,
screen density, or interface theme. For example, you might need to localize texts for different languages or adjust
images for the dark theme. For that, the library provides special qualifiers.

> Learn how to handle resource-related settings in the [Manage local resource environment](compose-resource-environment.md) tutorial.
>
{style="note"}

All resource types, except for raw files in the `files` directory, support qualifiers. Add qualifiers to directory
names using a hyphen:

![Qualifiers in multiplatform resources](compose-resources-qualifiers.png){width=250}

The library supports (in the order of priority) the following qualifiers: [language](#language-and-regional-qualifiers)
(including region and script), [theme](#theme-qualifier), and [density](#density-qualifier).

* Different types of qualifiers can be applied together. For example, `drawable-en-rUS-mdpi-dark` is an image for the
  English language in the United States region, suitable for 160 DPI screens in the dark theme.
* If no resource matches the requested language or theme, the default resource (without a qualifier) is used instead.
  Density qualifiers are resolved differently, see [Density qualifier](#density-qualifier).
  For languages with multiple scripts, see [Fallback for languages with multiple scripts](#fallback-for-languages-with-multiple-scripts).
* Qualifiers are checked in the order of priority, and the search stops as soon as only one resource is left.
  If only one resource matches the requested language, it is used even if its theme or density qualifier doesn't match.
  For example, if a project contains only `drawable` and `drawable-en-rUS-mdpi-dark` directories, a device with
  the US English locale and the light theme gets the image from `drawable-en-rUS-mdpi-dark`.
  To avoid this, provide a variant for each theme, for example, `drawable-en-rUS-mdpi-light` next to `drawable-en-rUS-mdpi-dark`.

### Language and regional qualifiers

You can combine language and region qualifiers:

* The language is defined by a two-letter (ISO 639-1)
    or a three-letter (ISO 639-2) [language code](https://www.loc.gov/standards/iso639-2/php/code_list.php).
* You can add a two-letter [ISO 3166-1-alpha-2](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2)
    regional code to your language code.
    The regional code must have a lowercase `r` prefix, for example: `drawable-es-rMX`.

The language and regional codes are case-sensitive.

> The qualifier must match the language code that the platform reports, and platforms usually report two-letter codes.
> Two-letter and three-letter codes for the same language are not related: for example, a device set to Mexican Spanish
> reports `es-MX`, so resources in `drawable-spa-rMX` are never selected on that device.
> 
{style="note"}

To specify a script, or any other BCP 47 subtag, use the `b+` prefix and replace all subsequent subtag hyphens (`-`) with a `+` sign,
for example: `values-b+zh+Hans` and `values-b+zh+Hant`.

Use the same syntax for three-digit numeric region codes, for example, `values-b+es+419` for Latin American Spanish.
The `r` prefix form, such as `values-es-r419`, is not resolved correctly.

Learn more about working with region-specific formats in [Localization](compose-regional-format.md).

#### Fallback for languages with multiple scripts

The platform often reports a locale without a script: for example, `zh` or `zh-CN` instead of `zh-Hans` or `zh-Hant`.
This happens on Android, desktop, and web. Browsers, in particular, usually report Chinese locales as `zh-CN` or `zh-TW`.

A directory with a script qualifier, such as `values-b+zh+Hans`, matches a script-less request for the same language.
The result depends on how many script-specific directories the language has:

* If there is only one, for example, `values-b+sr+Latn`, it is used for script-less requests, such as `sr`.
* If there are several, for example, `values-b+zh+Hans` and `values-b+zh+Hant`, all of them match `zh`, and the library can't choose one.
  The default resource (without a qualifier) is not used in this case. Instead, the app throws an `IllegalStateException`
  with the `Resource with ID='...' has more than one file` message.

To avoid the exception, add a script-less directory for every language that has several script-specific directories.
Script-less requests, such as `zh` or `zh-CN`, then use this directory, and requests with a script, such as `zh-Hans`,
use the script-specific directories:

```
commonMain/composeResources/
├── values/                 # Default resources
├── values-zh/              # Resources for `zh` without a script
├── values-b+zh+Hans/
└── values-b+zh+Hant/
```

The same applies to all languages that have more than one script, such as Serbian
(`sr-Cyrl`, `sr-Latn`) or Uzbek (`uz-Cyrl`, `uz-Latn`).

### Theme qualifier

You can add `light` or `dark` qualifiers. Compose Multiplatform then selects the necessary resource depending on the
current system theme.

### Density qualifier

You can use the following density qualifiers:

* `ldpi` – 120 DPI, 0.75x density
* `mdpi` – 160 DPI, 1x density
* `hdpi` – 240 DPI, 1.5x density
* `xhdpi` – 320 DPI, 2x density
* `xxhdpi` – 480 DPI, 3x density
* `xxxhdpi` – 640 DPI, 4x density

The resource is selected depending on the screen density defined in the system.
If there is no resource for the exact density, the library looks for the closest higher density first,
then for the closest lower density, and only then uses the default resource (without a qualifier).
For example, on a 1.5x screen with only `drawable`, `drawable-mdpi`, and `drawable-xxhdpi` directories,
the image from `drawable-xxhdpi` is used.

## Publication

Starting with Compose Multiplatform 1.6.10, all necessary resources are included in the publication
maven artifacts.

To enable this functionality, your project needs to use Kotlin 2.0.0 or newer and Gradle 7.6 or newer.

## What's next?

* Learn how to access the resources you set up and how to customize the accessors generated by default on the
    [](compose-multiplatform-resources-usage.md) page.
* Check out the official [demo project](https://github.com/JetBrains/compose-multiplatform/tree/master/components/resources/demo)
    that shows how resources can be handled in a Compose Multiplatform project targeting iOS, Android, and desktop.
