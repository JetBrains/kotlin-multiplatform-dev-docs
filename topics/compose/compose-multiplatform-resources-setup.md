[//]: # (title: Setup and configuration for multiplatform resources)

<toc-settings depth="3"/>

To properly configure the project to use multiplatform resources:

1. Add the library dependency.
2. Create the necessary directories for each kind of resource.
3. Create additional directories for qualified resources (for example, different images for the dark UI theme
    or localized strings).

## Dependency and directory setup

To access resources in your multiplatform projects, add the library dependency and organize files within your project directory:

1. In the `build.gradle.kts` file in the `composeApp` directory, add a dependency to the `commonMain` source set:

   ```kotlin
   kotlin {
       sourceSets {
           commonMain.dependencies {
               implementation(compose.components.resources)
           }
       }
   }
   ```

2. Create a new directory `composeResources` in the source set directory you want to add the resources to
   (`commonMain` in this example):

   ![Compose resources project structure](compose-resources-structure.png){width=250}

3. Organize the `composeResources` directory structure according to these rules:

   * Images should be in the `drawable` directory.
   * Fonts should be in the `font` directory.
   * Strings should be in the `values` directory.
   * Other files should be in the `files` directory, with any folder hierarchy you may find appropriate.

### Custom resource directories {label="EAP"}

In the `compose.resources {}` block of the `build.gradle.kts` file, you can specify custom resource directories for each source set.
Each of these custom directories should also contain files in the same way as the default `composeResources`: with a `drawable` subdirectory
for images, a `font` subdirectory for fonts, and so on.

A simple example is to point to a specific folder:

```kotlin
compose.resources {
    customDirectory(
        sourceSetName = "desktopMain",
        directoryProvider = provider { layout.projectDirectory.dir("desktopResources") }
    )
}
```

You can also set up a folder populated by a Gradle task, for example, with downloaded files:

```kotlin
abstract class DownloadRemoteFiles : DefaultTask() {

    @get:OutputDirectory
    val outputDir = layout.buildDirectory.dir("downloadedRemoteFiles")

    @TaskAction
    fun run() { /* your code for downloading files */ }
}

compose.resources {
    customDirectory(
        sourceSetName = "iosMain",
        directoryProvider = tasks.register<DownloadRemoteFiles>("downloadedRemoteFiles").map { it.outputDir.get() }
    )
}
```
{initial-collapse-state="collapsed"  collapsed-title="directoryProvider = tasks.register<DownloadRemoteFiles>"}

## Qualifiers

Sometimes, the same resource should be presented in different ways depending on the environment, such as locale,
screen density, or interface theme. For example, you might need to localize texts for different languages or adjust
images for the dark theme. For that, the library provides special qualifiers.

All resource types, except for raw files in the `files` directory, support qualifiers. Add qualifiers to directory
names using a hyphen:

![Qualifiers in multiplatform resources](compose-resources-qualifiers.png){width=250}

The library supports (in the order of priority) the following qualifiers: [language](#language-and-regional-qualifiers),
[theme](#theme-qualifier), and [density](#density-qualifier).

* Different types of qualifiers can be applied together. For example, "drawable-en-rUS-mdpi-dark" is an image for the
  English language in the United States region, suitable for 160 DPI screens in the dark theme.
* If a resource with the requested qualifier is not accessible, the default resource (without a qualifier) is used instead.

### Language and regional qualifiers

You can combine language and region qualifiers:
* The language is defined by a two-letter (ISO 639-1)
    or a three-letter (ISO 639-2) [language code](https://www.loc.gov/standards/iso639-2/php/code_list.php).
* You can add a two-letter [ISO 3166-1-alpha-2](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2)
    regional code to your language code.
    The regional code must have a lowercase `r` prefix, for example: `drawable-spa-rMX`

The language and regional codes are case-sensitive.

### Theme qualifier

You can add "light" or "dark" qualifiers. Compose Multiplatform then selects the necessary resource depending on the
current system theme.

### Density qualifier

You can use the following density qualifiers:

* "ldpi" – 120 DPI, 0.75x density
* "mdpi" – 160 DPI, 1x density
* "hdpi" – 240 DPI, 1.5x density
* "xhdpi" – 320 DPI, 2x density
* "xxhdpi" – 480 DPI, 3x density
* "xxxhdpi" – 640dpi, 4x density

The resource is selected depending on the screen density defined in the system.

## Publication

Starting with Compose Multiplatform 1.6.10, all necessary resources are included in the publication
maven artifacts.

To enable this functionality, your project needs to use Kotlin 2.0.0 or newer and Gradle 7.6 or newer.

## What's next?

* Learn how to access the resources you set up and how to customize the accessors generated by default on the
    [](compose-multiplatform-resources-usage.md) page.
* Check out the official [demo project](https://github.com/JetBrains/compose-multiplatform/tree/master/components/resources/demo)
    that shows how resources can be handled in a Compose Multiplatform project targeting iOS, Android, and desktop.
