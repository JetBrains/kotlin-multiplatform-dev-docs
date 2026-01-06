[//]: # (title: Direct integration)

<tldr>
   This is a local integration method. It can work for you if:<br/>

   * You've already set up a Kotlin Multiplatform project targeting iOS on your local machine.
   * Your Kotlin Multiplatform project does not have CocoaPods dependencies.<br/>

   [Choose the integration method that suits you best](multiplatform-ios-integration-overview.md)
</tldr>

If you want to develop your Kotlin Multiplatform project and an iOS project simultaneously by sharing code between them,
you can set up direct integration using a special script.

This script automates the process of connecting the Kotlin framework to iOS projects in Xcode:

![Direct integration diagram](direct-integration-scheme.svg){width=700}

The script uses the `embedAndSignAppleFrameworkForXcode` Gradle task designed specifically for the Xcode environment.
During the setup, you add it to the run script phase of the iOS app build. After that, the Kotlin artifact
is built and included in the derived data before running the iOS app build.

In general, the script:

* Copies the compiled Kotlin framework into the correct directory within the iOS project structure.
* Handles the code signing process of the embedded framework.
* Ensures that code changes in the Kotlin framework are reflected in the iOS app in Xcode.

## How to set up

If you're currently using the CocoaPods plugin to connect your Kotlin framework, migrate first.
If your project doesn't have CocoaPods dependencies, [skip this step](#connect-the-framework-to-your-project).

### Migrate from the CocoaPods plugin

To migrate from the CocoaPods plugin:

1. In Xcode, clean build directories using **Product** | **Clean Build Folder** or with the
   <shortcut>Cmd + Shift + K</shortcut> shortcut.
2. In the directory with the Podfile, run the following command:

    ```none
   pod deintegrate
   ```

3. Remove the `cocoapods {}` block from your `build.gradle(.kts)` files.
4. Delete the `.podspec` file and the Podfile.

### Connect the framework to your project

To connect the Kotlin framework generated from the multiplatform project to your Xcode project:

1. The `embedAndSignAppleFrameworkForXcode` task only registers if the `binaries.framework` configuration option is
   declared. In your Kotlin Multiplatform project, check the iOS target declaration in the `build.gradle.kts` file.
2. In Xcode, open the iOS project settings by double-clicking the project name.
3. In the **Targets** section on the left, select your target, then navigate to the **Build Phases** tab.
4. Click **+** and select **New Run Script Phase**.

   ![Add run script phase](xcode-run-script-phase-1.png){width=700}

5. Adjust the following script and paste it in the script text field of the new phase:

   ```bash
   if [ "YES" = "$OVERRIDE_KOTLIN_BUILD_IDE_SUPPORTED" ]; then
       echo "Skipping Gradle build task invocation due to OVERRIDE_KOTLIN_BUILD_IDE_SUPPORTED environment variable set to \"YES\""
       exit 0
   fi
   cd "<Path to the root of the multiplatform project>"
   ./gradlew :<Shared module name>:embedAndSignAppleFrameworkForXcode
   ```

   * In the `cd` command, specify the path to the root of your Kotlin Multiplatform project, for example, `$SRCROOT/..`.
   * In the `./gradlew` command, specify the name of the shared module, for example, `:shared` or `:composeApp`.
   
   When you start an iOS run configuration, IntelliJ IDEA and Android Studio build the Kotlin framework dependency
     before starting the Xcode build, and set the `OVERRIDE_KOTLIN_BUILD_IDE_SUPPORTED` environment variable to "YES".
     The provided shell script checks this variable and prevents the Kotlin framework from being built a second time from Xcode.
     
   > When you launch an iOS run configuration for a project that does not support this,
     the IDE suggests a fix to set up the build guard.
   >
   {style="note"}

6. Disable **Based on dependency analysis** option.

   ![Add the script](xcode-run-script-phase-2.png){width=700}

   This ensures that Xcode runs the script during every build and doesn't warn about missing output dependencies every time.

7. Move the **Run Script** phase higher, placing it before the **Compile Sources** phase.

   ![Drag the Run Script phase](xcode-run-script-phase-3.png){width=700}

8. On the **Build Settings** tab, disable the **User Script Sandboxing** option under **Build Options**:

   ![User Script Sandboxing](disable-sandboxing-in-xcode-project-settings.png){width=700}

   > This may require restarting your Gradle daemon if you built the iOS project without disabling sandboxing first.
   > Stop the Gradle daemon process that might have been sandboxed:
   > ```shell
   > ./gradlew --stop
   > ```
   >
   > {style="tip"}

9. Build the project in Xcode. If everything is set up correctly, the project will successfully build.

> If you have a custom build configuration different from the default `Debug` or `Release`, on the **Build Settings**
> tab, add the `KOTLIN_FRAMEWORK_BUILD_TYPE` setting under **User-Defined** and set it to `Debug` or `Release`.
>
{style="note"}

## What's next?

You can also take advantage of local integration when working with the Swift package manager. [Learn how to add a
dependency on a Kotlin framework in a local package](multiplatform-spm-local-integration.md).