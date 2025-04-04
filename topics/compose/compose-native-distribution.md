[//]: # (title: Native distributions)

Here, you'll learn about native distributions: how to create installers and packages for all of the supported systems, and
how to run an application locally with the same settings as for distributions.

Read on for the details about the following topics:

* [What is Compose Multiplatform Gradle plugin](#gradle-plugin)?
* [Details on basic tasks](#basic-tasks) such as running an application locally, and [advanced tasks](#minification-and-obfuscation) like minification and obfuscation.
* [How to include JDK modules](#including-jdk-modules) and deal with `ClassNotFoundException`.
* [How to specify distribution properties](#specifying-distribution-properties): package version, JDK version, output directory, launcher properties, and metadata.
* [How to manage resources](#managing-resources) using resources library, JVM resource loading, or adding files to packaged applications.
* [How to customize source sets](#custom-source-sets) using Gradle source set, Kotlin JVM target, or manually.
* [How to specify application icon](#application-icon) for each OS.
* [Platform-specific options](#platform-specific-options), such as email of the package maintainer on Linux and the app category for the Apple App Store on macOS.
* [macOS-specific configuration](#macos-specific-configuration): signing, notarization, and `Info.plist`.

## Gradle plugin

This guide is primarily focused on packaging Compose applications using the Compose Multiplatform Gradle plugin. The `org.jetbrains.compose` plugin 
provides tasks for basic packaging, obfuscation, and macOS code signing.

The plugin simplifies the process of packaging applications into native distributions using `jpackage` and running an application locally.
Distributable applications are self-contained, installable binaries that include all the necessary Java runtime components,
without requiring a JDK to be installed on the target system.

To minimize package size, the Gradle plugin uses the [jlink](https://openjdk.org/jeps/282) tool that ensures bundling only the necessary Java modules in the distributable package. 
However, you still must configure the Gradle plugin to specify which modules you need.
For more information, see the `Configuring included JDK modules` section.

As an alternative, you can use [Conveyor](https://www.hydraulic.software), an external tool not developed by JetBrains.
Conveyor supports online updates, cross-building, and various other features but requires a [license](https://hydraulic.software/pricing.html) for non-open source projects.
For more information, refer to the [Conveyor documentation](https://conveyor.hydraulic.dev/latest/tutorial/hare/jvm).

## Basic tasks

The basic configurable unit in the plugin is an `application`. The `application` DSL method defines a shared configuration for a set of final binaries, which means
it allows you to pack a collection of files, together with a JDK distribution, into a set of compressed binary installers 
in various formats.

The following formats are available for the supported operating systems:

* **macOS**: `.dmg` (`TargetFormat.Dmg`), `.pkg` (`TargetFormat.Pkg`)
* **Windows**: `.exe` (`TargetFormat.Exe`), `.msi` (`TargetFormat.Msi`)
* **Linux**: `.deb` (`TargetFormat.Deb`), `.rpm` (`TargetFormat.Rpm`)

Here is an example of a `build.gradle.kts` file with a basic desktop configuration:

```kotlin
import org.jetbrains.compose.desktop.application.dsl.TargetFormat

plugins {
    kotlin("jvm")
    id("org.jetbrains.compose")
}

dependencies {
    implementation(compose.desktop.currentOs)
}

compose.desktop {
    application {
        mainClass = "example.MainKt"

        nativeDistributions {
            targetFormats(TargetFormat.Dmg, TargetFormat.Msi, TargetFormat.Exe)
        }
    }
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="compose.desktop { application { mainClass = "}

When you build a project, the plugin creates the following tasks:

<table>
    <tr>
        <td>Gradle task</td>
        <td>Description</td>
    </tr>
    <tr>
        <td><code>package&lt;FormatName&gt;</code></td> 
        <td>Packages the application into the corresponding <code>FormatName</code> binary. Cross-compilation is currently not supported, 
            meaning you can build the specific format using the corresponding compatible OS only.
            For example, to build a <code>.dmg</code> binary, you have to run the <code>packageDmg</code> task on macOS. 
            If any tasks are incompatible with the current OS, they are skipped by default.</td>
    </tr>
    <tr>
        <td><code>packageDistributionForCurrentOS</code></td>
        <td>Aggregates all package tasks for an application. It is a <a href="https://docs.gradle.org/current/userguide/more_about_tasks.html#sec:task_categories">lifecycle task</a>.</td>
    </tr>
    <tr>
        <td><code>packageUberJarForCurrentOS</code></td>
        <td>Creates a single jar file containing all dependencies for the current operating system. 
        The task expects <code>compose.desktop.currentOS</code> to be used as a <code>compile</code>, <code>implementation</code>, or <code>runtime</code> dependency.</td>
    </tr>
    <tr>
        <td><code>run</code></td>
        <td>Runs an application locally from the entry point specified in <code>mainClass</code>. The <code>run</code> task starts a non-packaged JVM application with the full runtime. 
        This approach is faster and easier to debug compared to creating a compact binary image with a minified runtime. 
        To run a final binary image, use the <code>runDistributable</code> task instead.</td>
    </tr>
    <tr>
        <td><code>createDistributable</code></td>
        <td>Creates a final application image without creating an installer.</td>
    </tr>
    <tr>
        <td><code>runDistributable</code></td>
        <td>Runs a prepackaged application image.</td>
    </tr>
</table>

All available tasks are listed in the Gradle tool window. After you execute a task, Gradle generates output binaries in the `${project.buildDir}/compose/binaries` directory.

## Including JDK modules

To reduce the distributable size, the Gradle plugin uses [jlink](https://openjdk.org/jeps/282) that helps bundle only the necessary JDK modules.

For now, the Gradle plugin does not automatically determine necessary JDK Modules. While this will not cause compilation issues, 
failure to provide the necessary modules can lead to `ClassNotFoundException` at runtime.

If you encounter a `ClassNotFoundException` when running a packaged application or the `runDistributable` task, 
you can include additional JDK modules using the `modules` DSL method:

```kotlin
compose.desktop {
    application {
        nativeDistributions {
            modules("java.sql")
            // Alternatively: includeAllModules = true
        }
    }
}
```

You can specify the required modules manually, or run `suggestModules`. The `suggestModules` task uses the 
[jdeps](https://docs.oracle.com/javase/9/tools/jdeps.htm) static analysis tool to determine possible missing modules.
Note that the tool's output might be incomplete or list unnecessary modules.

If the size of the distributable is not a crucial factor and can be ignored, you may opt to include all runtime modules by using the `includeAllModules` DSL property.


## Specifying distribution properties

### Package version

Native distribution packages must have specific package versions.
To specify the package version, you can use the following DSL properties, listed from the highest priority level to the lowest:

* `nativeDistributions.<os>.<packageFormat>PackageVersion` specifies a version for a single package format.
* `nativeDistributions.<os>.packageVersion` specifies a version for a single target OS.
* `nativeDistributions.packageVersion` specifies a version for all packages.

On macOS, you can also specify the build version using the following DSL properties, listed, again, from the highest priority level to the lowest:

* `nativeDistributions.macOS.<packageFormat>PackageBuildVersion` specifies a build version for a single package format.
* `nativeDistributions.macOS.packageBuildVersion` specifies a build version for all macOS packages.

If you don't specify a build version, Gradle uses the package version instead. For more information about versioning on macOS,
see the [`CFBundleShortVersionString`](https://developer.apple.com/documentation/bundleresources/information_property_list/cfbundleshortversionstring) 
and [`CFBundleVersion`](https://developer.apple.com/documentation/bundleresources/information_property_list/cfbundleversion) documentation.

Here is a template for specifying package versions in order of priority:

```kotlin
compose.desktop {
    application {
        nativeDistributions {
            // Version for all packages
            packageVersion = "..." 
          
            macOS {
              // Version for all macOS packages
              packageVersion = "..."
              // Version for the dmg package only
              dmgPackageVersion = "..." 
              // Version for the pkg package only
              pkgPackageVersion = "..." 
              
              // Build version for all macOS packages
              packageBuildVersion = "..."
              // Build version for the dmg package only
              dmgPackageBuildVersion = "..." 
              // Build version for the pkg package only
              pkgPackageBuildVersion = "..." 
            }
            windows {
              // Version for all Windows packages
              packageVersion = "..."  
              // Version for the msi package only
              msiPackageVersion = "..."
              // Version for the exe package only
              exePackageVersion = "..." 
            }
            linux {
              // Version for all Linux packages
              packageVersion = "..."
              // Version for the deb package only
              debPackageVersion = "..."
              // Version for the rpm package only
              rpmPackageVersion = "..."
            }
        }
    }
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="compose.desktop { application { nativeDistributions { packageVersion ="}

To define a package version, follow these rules:

<table>
    <tr>
        <td>File Type</td>
        <td>Version format</td>
        <td>Details</td>
    </tr>
    <tr>
        <td><code>dmg</code>, <code>pkg</code></td>
        <td><code>MAJOR[.MINOR][.PATCH]</code></td>
        <td>
            <ul>
                <li><code>MAJOR</code> is an integer > 0</li>
                <li><code>MINOR</code> is an optional non-negative integer</li>
                <li><code>PATCH</code> is an optional non-negative integer</li>
            </ul>
        </td>
    </tr>
    <tr>
        <td><code>msi</code>, <code>exe</code></td>
        <td><code>MAJOR.MINOR.BUILD</code></td>
        <td>
            <ul>
                <li><code>MAJOR</code> is a non-negative integer with a maximum value of 255</li>
                <li><code>MINOR</code> is a non-negative integer with a maximum value of 255</li>
                <li><code>BUILD</code> is a non-negative integer with a maximum value of 65535</li>
            </ul>
        </td>
    </tr>
    <tr>
        <td><code>deb</code></td>
        <td><code>[EPOCH:]UPSTREAM_VERSION[-DEBIAN_REVISION]</code></td>
        <td>
            <ul>
                <li><code>EPOCH</code> is an optional non-negative integer</li>
                <li><code>UPSTREAM_VERSION</code>:
                    <ul>
                        <li>May contain only alphanumerics and the <code>.</code>, <code>+</code>, <code>-</code>, <code>~</code> characters</li>
                        <li>Must start with a digit</li>
                    </ul>
                </li>
                <li><code>DEBIAN_REVISION</code>:
                    <ul>
                        <li>Optional</li>
                        <li>May contain only alphanumerics and the <code>.</code>, <code>+</code>, <code>~</code> characters</li>
                    </ul>
                </li>
            </ul>
            For more details, see <a href="https://www.debian.org/doc/debian-policy/ch-controlfields.html#version">Debian documentation</a>.
        </td>
    </tr>
    <tr>
        <td><code>rpm</code></td>
        <td>Any format</td>
        <td>Version must not contain the <code>-</code> (dash) character.</td>
    </tr>
</table>

### JDK version

The plugin uses `jpackage`, which requires a JDK version not lower than [JDK 17](https://openjdk.java.net/projects/jdk/17/). 
When specifying a JDK version, ensure you meet at least one of the following requirements:

* The `JAVA_HOME` environment variable points to the compatible JDK version.
* The `javaHome` property is set via the DSL:

  ```kotlin
  compose.desktop {
      application {
          javaHome = System.getenv("JDK_17")
      }
  }
  ```

### Output directory

To use custom output directory for native distributions, configure the `outputBaseDir` property as shown below:

```kotlin
compose.desktop {
    application {
        nativeDistributions {
            outputBaseDir.set(project.layout.buildDirectory.dir("customOutputDir"))
        }
    }
}
```

### Launcher properties

To tailor the application startup process, you can customize the following properties:

<table>
  <tr>
    <td>Property</td>
    <td>Description</td>
  </tr>
  <tr>
    <td><code>mainClass</code></td>
    <td>The fully-qualified name of the class containing the <code>main</code> method.</td>
  </tr>
  <tr>
    <td><code>args</code></td>
    <td>Arguments for the application's <code>main</code> method.</td>
  </tr>
  <tr>
    <td><code>jvmArgs</code></td>
    <td>Arguments for the application's JVM.</td>
  </tr>
</table>

Here's an example configuration:

```kotlin
compose.desktop {
    application {
        mainClass = "MainKt"
        args += listOf("-customArgument")
        jvmArgs += listOf("-Xmx2G")
    }
}
```

### Metadata

Within the `nativeDistributions` DSL block, you can configure the following properties:

<table>
  <tr>
    <td>Property</td>
    <td>Description</td>
    <td>Default value</td>
  </tr>
  <tr>
    <td><code>packageName</code></td>
    <td>The application's name.</td>
    <td>The Gradle project's <a href="https://docs.gradle.org/current/javadoc/org/gradle/api/Project.html#getName--">name</a></td>
  </tr>
  <tr>
    <td><code>packageVersion</code></td>
    <td>The application's version.</td>
    <td>The Gradle project's <a href="https://docs.gradle.org/current/javadoc/org/gradle/api/Project.html#getVersion--">version</a></td>
  </tr>
  <tr>
    <td><code>description</code></td>
    <td>The application's description.</td>
    <td>None</td>
  </tr>
  <tr>
    <td><code>copyright</code></td>
    <td>The application's copyright information.</td>
    <td>None</td>
  </tr>
  <tr>
    <td><code>vendor</code></td>
    <td>The application's vendor.</td>
    <td>None</td>
  </tr>
  <tr>
    <td><code>licenseFile</code></td>
    <td>The application's license file.</td>
    <td>None</td>
  </tr>
</table> 

Here's an example configuration:

```kotlin
compose.desktop {
    application {
        nativeDistributions {
            packageName = "ExampleApp"
            packageVersion = "0.1-SNAPSHOT"
            description = "Compose Multiplatform App"
            copyright = "© 2024 My Name. All rights reserved."
            vendor = "Example vendor"
            licenseFile.set(project.file("LICENSE.txt"))
        }
    }
}
```

## Managing resources

To package and load resources, you can use the Compose Multiplatform resources library, the JVM resource loading, or add files to packaged applications.

### Resources library

The most straightforward way to set up the resources for your project is to use the resources library.
With the resources library, you can access resources in common code across all supported platforms.
See [Multiplatform resources](compose-multiplatform-resources.md) for details.

### JVM resource loading

Compose Multiplatform for desktop operates on the JVM platform, meaning you can load resources from
a `.jar` file using the `java.lang.Class` API. You can access a file in the `src/main/resources` directory via
[`Class::getResource`](https://docs.oracle.com/en/java/javase/15/docs/api/java.base/java/lang/Class.html#getResource(java.lang.String))
or [`Class::getResourceAsStream`](https://docs.oracle.com/en/java/javase/15/docs/api/java.base/java/lang/Class.html#getResourceAsStream(java.lang.String)).

### Adding files to packaged application

There are scenarios where loading resources from `.jar` files may be less practical, for example, when you have target-specific assets 
and need to include files only in a macOS package but not in a Windows one.

In these cases, you can configure the Gradle plugin to include additional resource files in the installation directory.
Specify a root resource directory using the DSL as follows:

```kotlin
compose.desktop {
    application {
        mainClass = "MainKt"
        nativeDistributions {
            targetFormats(TargetFormat.Dmg, TargetFormat.Msi, TargetFormat.Deb)
            packageVersion = "1.0.0"

            appResourcesRootDir.set(project.layout.projectDirectory.dir("resources"))
        }
    }
}
```

In the example above, the root resource directory is defined as `<PROJECT_DIR>/resources`.

The Gradle plugin will include files from the resources subdirectories as follows:

1. **Common resources:**
Files located in `<RESOURCES_ROOT_DIR>/common` will be included in all packages regardless of the target OS or architecture.

2. **OS-specific resources:**
Files located in `<RESOURCES_ROOT_DIR>/<OS_NAME>` will be included only in packages built for a specific operating system. 
Valid values for `<OS_NAME>` are: `windows`, `macos`, and `linux`.

3. **OS and architecture-specific resources:**
Files located in `<RESOURCES_ROOT_DIR>/<OS_NAME>-<ARCH_NAME>` will be included only in packages built for a specific combination of operating system and CPU architecture. 
Valid values for `<ARCH_NAME>` are: `x64` and `arm64`.
For example, files in `<RESOURCES_ROOT_DIR>/macos-arm64` will be included in packages intended for Apple Silicon Macs only.

You can access included resources using the `compose.application.resources.dir` system property:

```kotlin
import java.io.File

val resourcesDir = File(System.getProperty("compose.application.resources.dir"))

fun main() {
    println(resourcesDir.resolve("resource.txt").readText())
}
```

## Custom source sets

You can rely on default configuration, if you use `org.jetbrains.kotlin.jvm` or 
`org.jetbrains.kotlin.multiplatform` plugins:

* Configuration with `org.jetbrains.kotlin.jvm` includes content from the `main` [source set](https://docs.gradle.org/current/userguide/java_plugin.html#source_sets).
* Configuration with `org.jetbrains.kotlin.multiplatform` includes content from a single [JVM target](multiplatform-dsl-reference.md#targets).
  If you define multiple JVM targets, the default configuration is disabled. In this case, you need to configure the plugin manually, 
  or specify a single target (see below).

If the default configuration is ambiguous or insufficient, you can customize it in several ways:

Using a Gradle [source set](https://docs.gradle.org/current/userguide/java_plugin.html#source_sets):

``` kotlin
plugins {
    kotlin("jvm")
    id("org.jetbrains.compose")
}
val customSourceSet = sourceSets.create("customSourceSet")
compose.desktop {
    application {
        from(customSourceSet)
    }
}
``` 

Using a Kotlin [JVM target](multiplatform-dsl-reference.md#targets):

``` kotlin
plugins {
    kotlin("multiplatform")
    id("org.jetbrains.compose")
} 
kotlin {
    jvm("customJvmTarget") {}
}
compose.desktop {
    application {
        from(kotlin.targets["customJvmTarget"])
    }
}
```

Manually:

* Use `disableDefaultConfiguration` to disable the default settings.
* Use `fromFiles` to specify files to include.
* Specify the `mainJar` file property to point to a `.jar` file containing the main class.
* Use `dependsOn` to add task dependencies to all plugin tasks.
``` kotlin
compose.desktop {
    application {
        disableDefaultConfiguration()
        fromFiles(project.fileTree("libs/") { include("**/*.jar") })
        mainJar.set(project.file("main.jar"))
        dependsOn("mainJarTask")
    }
}
```

## Application icon

Make sure your app icon is available in the following OS-specific formats:

* `.icns` for macOS
* `.ico` for Windows
* `.png` for Linux

```kotlin
compose.desktop {
    application {
        nativeDistributions {
            macOS {
                iconFile.set(project.file("icon.icns"))
            }
            windows {
                iconFile.set(project.file("icon.ico"))
            }
            linux {
                iconFile.set(project.file("icon.png"))
            }
        }
    }
}
```

## Platform-specific options

Platform-specific settings can be configured using the corresponding DSL blocks:

``` kotlin
compose.desktop {
    application {
        nativeDistributions {
            macOS {
                // Options for macOS
            }
            windows {
                // Options for Windows
            }
            linux {
                // Options for Linux
            }
        }
    }
}
```

The following table describes all supported platform-specific options. It is **not recommended** to use undocumented properties.

<table>
    <tr>
        <td>Platform</td>
        <td>Option</td>
        <td width="500">Description</td>
    </tr>
    <tr>
        <td rowspan="3">All platforms</td>
        <td><code>iconFile.set(File("PATH_TO_ICON"))</code></td>
        <td>Specifies the path to a platform-specific icon for the application. For details, see the <a anchor="application-icon">Application icon</a> section.</td>
    </tr>
    <tr>
        <td><code>packageVersion = "1.0.0"</code></td>
        <td>Sets a platform-specific package version. For details, see the <a anchor="package-version">Package version</a> section.</td>
    </tr>
    <tr>
        <td><code>installationPath = "PATH_TO_INST_DIR"</code></td>
        <td>Specifies the absolute or relative path to the default installation directory.
            On Windows, you can also use <code>dirChooser = true</code> to enable customizing the path during installation.</td>
    </tr>
    <tr>
        <td rowspan="8">Linux</td>
        <td><code>packageName = "custom-package-name"</code></td>
        <td>Overrides the default application name.</td>
    </tr>
    <tr>
        <td><code>debMaintainer = "maintainer@example.com"</code></td>
        <td>Specifies the email of the package maintainer.</td>
    </tr>
    <tr>
        <td><code>menuGroup = "my-example-menu-group"</code></td>
        <td>Defines a menu group for the application.</td>
    </tr>
    <tr>
        <td><code>appRelease = "1"</code></td>
        <td>Sets a release value for the rpm package or a revision value for the deb package.</td>
    </tr>
    <tr>
        <td><code>appCategory = "CATEGORY"</code></td>
        <td>Assigns a group value for the rpm package or a section value for the deb package.</td>
    </tr>
    <tr>
        <td><code>rpmLicenseType = "TYPE_OF_LICENSE"</code></td>
        <td>Indicates the type of license for the rpm package.</td>
    </tr>
    <tr>
        <td><code>debPackageVersion = "DEB_VERSION"</code></td>
        <td>Sets a deb-specific package version. For details, see the <a anchor="package-version">Package version</a> section.</td>
    </tr>
    <tr>
        <td><code>rpmPackageVersion = "RPM_VERSION"</code></td>
        <td>Sets an rpm-specific package version. For details, see the <a anchor="package-version">Package version</a> section.</td>
    </tr>
    <tr>
        <td rowspan="15">macOS</td>
        <td><code>bundleID</code></td>
        <td>
            Specifies a unique application identifier, which can contain only alphanumeric characters 
            (<code>A-Z</code>, <code>a-z</code>, <code>0-9</code>), hyphen (<code>-</code>), and 
            period (<code>.</code>). It is recommended to use reverse DNS notation (<code>com.mycompany.myapp</code>).
        </td>
    </tr>
    <tr>
        <td><code>packageName</code></td>
        <td>The name of the application.</td>
    </tr>
    <tr>
        <td><code>dockName</code></td>
        <td>
            The name of the application as displayed in the menu bar, the "About &lt;App&gt;" menu item, 
            and in the dock. Default value is <code>packageName</code>.
        </td>
    </tr>
    <tr>
        <td><code>minimumSystemVersion</code></td>
        <td>
            The minimum macOS version required to run the application. For details, see
            <a href="https://developer.apple.com/documentation/bundleresources/information_property_list/lsminimumsystemversion">
                <code>LSMinimumSystemVersion</code></a>.
        </td>
    </tr>
    <tr>
        <td><code>signing</code>, <code>notarization</code>, <code>provisioningProfile</code>, <code>runtimeProvisioningProfile</code></td>
        <td>
            See the
            <a href="https://github.com/JetBrains/compose-multiplatform/tree/master/tutorials/Signing_and_notarization_on_macOS">
               Signing and notarizing distributions for macOS</a> tutorial.
        </td>
    </tr>
    <tr>
        <td><code>appStore = true</code></td>
        <td>Specifies whether to build and sign the app for the Apple App Store. Requires at least JDK 17.</td>
    </tr>
    <tr>
        <td><code>appCategory</code></td>
        <td>
            The category of the app for the Apple App Store. When building for the App Store, default value is 
            <code>public.app-category.utilities</code>, otherwise it is <code>Unknown</code>.
            See 
            <a href="https://developer.apple.com/documentation/bundleresources/information_property_list/lsapplicationcategorytype">
                <code>LSApplicationCategoryType</code>
            </a> for a list of valid categories.
        </td>
    </tr>
    <tr>
        <td><code>entitlementsFile.set(File("PATH_ENT"))</code></td>
        <td>
            Specifies the path to the file containing entitlements used when signing. When you provide a custom file, 
            make sure to add the entitlements required for Java applications. See 
            <a href="https://github.com/openjdk/jdk/blob/master/src/jdk.jpackage/macosx/classes/jdk/jpackage/internal/resources/sandbox.plist">
                sandbox.plist</a> for the default file used when building for the App Store. Note that this default file may differ
            depending on your JDK version. If no file is specified, the plugin will use the default entitlements provided by 
            <code>jpackage</code>. For details, see the
            <a href="https://github.com/JetBrains/compose-multiplatform/tree/master/tutorials/Signing_and_notarization_on_macOS">
               Signing and notarizing distributions for macOS</a> tutorial.
        </td>
    </tr>
    <tr>
        <td><code>runtimeEntitlementsFile.set(File("PATH_R_ENT"))</code></td>
        <td>
            Specifies the path to the file containing entitlements used when signing the JVM runtime. When you provide a custom file, 
            make sure to add the entitlements required for Java applications. See 
            <a href="https://github.com/openjdk/jdk/blob/master/src/jdk.jpackage/macosx/classes/jdk/jpackage/internal/resources/sandbox.plist">
                sandbox.plist</a> for the default file used when building for the App Store. Note that this default file may differ
            depending on your JDK version. If no file is specified, the plugin will use the default entitlements provided by 
            <code>jpackage</code>. For details, see the
            <a href="https://github.com/JetBrains/compose-multiplatform/tree/master/tutorials/Signing_and_notarization_on_macOS">
               Signing and notarizing distributions for macOS</a> tutorial.
        </td>
    </tr>
    <tr>
        <td><code>dmgPackageVersion = "DMG_VERSION"</code></td>
        <td>
            Sets the DMG-specific package version. For details, see the <a anchor="package-version">Package version</a> section.
        </td>
    </tr>
    <tr>
        <td><code>pkgPackageVersion = "PKG_VERSION"</code></td>
        <td>
            Sets the PKG-specific package version. For details, see the <a anchor="package-version">Package version</a> section.
        </td>
    </tr>
    <tr>
        <td><code>packageBuildVersion = "DMG_VERSION"</code></td>
        <td>
            Sets the package build version. For details, see the <a anchor="package-version">Package version</a> section.
        </td>
    </tr>
    <tr>
        <td><code>dmgPackageBuildVersion = "DMG_VERSION"</code></td>
        <td>
            Sets the DMG-specific package build version. For details, see the <a anchor="package-version">Package version</a> section.
        </td>
    </tr>
    <tr>
        <td><code>pkgPackageBuildVersion = "PKG_VERSION"</code></td>
        <td>
            Sets the PKG-specific package build version. For details, see the <a anchor="package-version">Package version</a> section.
        </td>
    </tr>
    <tr>
        <td><code>infoPlist</code></td>
        <td>See the <a anchor="information-property-list-on-macos"><code>Info.plist</code> on macOS</a> section.</td>
    </tr>
        <tr>
            <td rowspan="7">Windows</td>
            <td><code>console = true</code></td>
            <td>Adds a console launcher for the application.</td>
        </tr>
        <tr>
            <td><code>dirChooser = true</code></td>
            <td>Enables customizing the installation path during installation.</td>
        </tr>
        <tr>
            <td><code>perUserInstall = true</code></td>
            <td>Enables installing the application on a per-user basis.</td>
        </tr>
        <tr>
            <td><code>menuGroup = "start-menu-group"</code></td>
            <td>Adds the application to the specified Start menu group.</td>
        </tr>
        <tr>
            <td><code>upgradeUuid = "UUID"</code></td>
            <td>Specifies a unique ID which enables users to update the application via the installer, 
            when there is a version newer than an installed version. The value must remain constant for a single application. 
            For details, see <a href="https://wixtoolset.org/documentation/manual/v3/howtos/general/generate_guids.html">How To: Generate a GUID</a>.</td>
        </tr>
        <tr>
            <td><code>msiPackageVersion = "MSI_VERSION"</code></td>
            <td>Sets the MSI-specific package version. For details, see the <a anchor="package-version">Package version</a> section.</td>
        </tr>
        <tr>
            <td><code>exePackageVersion = "EXE_VERSION"</code></td>
            <td>Sets the EXE-specific package version. For details, see the <a anchor="package-version">Package version</a> section.</td>
        </tr>
</table>

## macOS-specific configuration

### Signing and notarization on macOS

Modern macOS versions do not permit users to execute unsigned applications downloaded from the internet. If you attempt to run such an application, you'll encounter
the following error: "YourApp is damaged and can't be open. You should eject the disk image".

To learn how to sign and notarize your application, see our [tutorial](https://github.com/JetBrains/compose-multiplatform/blob/master/tutorials/Signing_and_notarization_on_macOS/README.md).

### Information property list on macOS

While the DSL supports essential platform-specific customizations, there can still be cases beyond the provided capabilities. 
If you need to specify `Info.plist` values that are not represented in the DSL, 
you can include a snippet of raw XML as a workaround. This XML will be appended to the application's `Info.plist`.

#### Example: Deep linking

1. Define a custom URL scheme in the `build.gradle.kts` file:

  ``` kotlin
  compose.desktop {
      application {
          mainClass = "MainKt"
          nativeDistributions {
              targetFormats(TargetFormat.Dmg)
              packageName = "Deep Linking Example App"
              macOS {
                  bundleID = "org.jetbrains.compose.examples.deeplinking"
                  infoPlist {
                      extraKeysRawXml = macExtraPlistKeys
                  }
              }
          }
      }
  }
  
  val macExtraPlistKeys: String
      get() = """
        <key>CFBundleURLTypes</key>
        <array>
          <dict>
            <key>CFBundleURLName</key>
            <string>Example deep link</string>
            <key>CFBundleURLSchemes</key>
            <array>
              <string>compose</string>
            </array>
          </dict>
        </array>
      """
  ```
  {initial-collapse-state="collapsed" collapsible="true" collapsed-title="infoPlist { extraKeysRawXml = macExtraPlistKeys"}

2. Use the `java.awt.Desktop` class to set up a URI handler in the `src/main/main.kt` file:

  ``` kotlin 
  import androidx.compose.material.MaterialTheme
  import androidx.compose.material.Text
  import androidx.compose.runtime.getValue
  import androidx.compose.runtime.mutableStateOf
  import androidx.compose.runtime.setValue
  import androidx.compose.ui.window.singleWindowApplication
  import java.awt.Desktop
  
  fun main() {
      var text by mutableStateOf("Hello, World!")
  
      try {
          Desktop.getDesktop().setOpenURIHandler { event ->
              text = "Open URI: " + event.uri
          }
      } catch (e: UnsupportedOperationException) {
          println("setOpenURIHandler is unsupported")
      }
  
      singleWindowApplication {
          MaterialTheme {
              Text(text)
          }
      }
  }
  ```
  {initial-collapse-state="collapsed" collapsible="true" collapsed-title="Desktop.getDesktop().setOpenURIHandler { event ->"}

3. Execute the `runDistributable` task: `./gradlew runDistributable`.

As a result, links like `compose://foo/bar` can now be redirected from a browser to your application.

## Minification and obfuscation

The Compose Multiplatform Gradle plugin includes built-in support for [ProGuard](https://www.guardsquare.com/proguard). 
ProGuard is an [open-source tool](https://github.com/Guardsquare/proguard) for code minification and obfuscation.

For each *default* (without ProGuard) packaging task, the Gradle plugin provides a *release* task (with ProGuard):

<table>
  <tr>
    <td width="400">Gradle task</td>
    <td>Description</td>
  </tr>
  <tr>
    <td>
        <p>Default: <code>createDistributable</code></p>
        <p>Release: <code>createReleaseDistributable</code></p>
    </td>
    <td>Creates an application image with bundled JDK and resources.</td>
  </tr>
  <tr>
    <td>
        <p>Default: <code>runDistributable</code></p>
        <p>Release: <code>runReleaseDistributable</code></p>
    </td>
    <td>Runs an application image with bundled JDK and resources.</td>
  </tr>
  <tr>
    <td>
        <p>Default: <code>run</code></p>
        <p>Release: <code>runRelease</code></p>
    </td>
    <td>Runs a non-packaged application <code>.jar</code> using the Gradle JDK.</td>
  </tr>
  <tr>
    <td>
        <p>Default: <code>package&lt;FORMAT_NAME&gt;</code></p>
        <p>Release: <code>packageRelease&lt;FORMAT_NAME&gt;</code></p>
    </td>
    <td>Packages an application image into a <code>&lt;FORMAT_NAME&gt;</code> file.</td>
  </tr>
  <tr>
    <td>
        <p>Default: <code>packageDistributionForCurrentOS</code></p>
        <p>Release: <code>packageReleaseDistributionForCurrentOS</code></p>
    </td>
    <td>Packages an application image into a format compatible with the current OS.</td>
  </tr>
  <tr>
    <td>
        <p>Default: <code>packageUberJarForCurrentOS</code></p>
        <p>Release: <code>packageReleaseUberJarForCurrentOS</code></p>
    </td>
    <td>Packages an application image into an uber (fat) `.jar`.</td>
  </tr>
  <tr>
    <td>
        <p>Default: <code>notarize&lt;FORMAT_NAME&gt;</code></p>
        <p>Release: <code>notarizeRelease&lt;FORMAT_NAME&gt;</code></p>
    </td>
    <td>Uploads a <code>&lt;FORMAT_NAME&gt;</code> application image for notarization (macOS only).</td>
  </tr>
  <tr>
    <td>
        <p>Default: <code>checkNotarizationStatus</code></p>
        <p>Release: <code>checkReleaseNotarizationStatus</code></p>
    </td>
    <td>Checks if notarization succeeded (macOS only).</td>
  </tr>
</table>

The default configuration enables some pre-defined ProGuard rules:

* The application image is minified, meaning unused classes are removed.
* `compose.desktop.application.mainClass` is used as the entry point.
* Several `keep` rules are included to ensure the Compose runtime remains functional.

In most cases, you don't need any additional configurations to get a minified application. 
However, ProGuard might not track certain usages in bytecode, for example, when a class is used via reflection. 
If you encounter issues that occur only after ProGuard processing, you may need to add custom rules.

To specify a custom configuration file, use the DSL as follows:

```kotlin
compose.desktop {
    application {
        buildTypes.release.proguard {
            configurationFiles.from(project.file("compose-desktop.pro"))
        }
    }
}
```

For more information about ProGuard rules and configuration options, refer to the Guardsquare [manual](https://www.guardsquare.com/manual/configuration/usage).

Obfuscation is disabled by default. To enable it, set the following property via the Gradle DSL:

```kotlin
compose.desktop {
    application {
        buildTypes.release.proguard {
            obfuscate.set(true)
        }
    }
}
```

ProGuard's optimizations are enabled by default. To disable them, set the following property via the Gradle DSL:

```kotlin
compose.desktop {
    application {
        buildTypes.release.proguard {
            optimize.set(false)
        }
    }
}
```

Producing an uber JAR is disabled by default, and ProGuard produces a corresponding `.jar` file for every input `.jar`. To enable it, set the following property via the Gradle DSL:

```kotlin
compose.desktop {
    application {
        buildTypes.release.proguard {
            joinOutputJars.set(true)
        }
    }
}
```

## What's next?

Explore the tutorials about [desktop components](https://github.com/JetBrains/compose-multiplatform/tree/master/tutorials#desktop).