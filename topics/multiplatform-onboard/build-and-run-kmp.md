[//]: # (title: Build and run Kotlin Multiplatform application)

Kotlin Multiplatform uses Gradle as a build system.
The KMP IDE plugin detects targets declared in the build script and provides run configurations for all supported targets.

The [KMP IDE plugin](https://plugins.jetbrains.com/plugin/14936-kotlin-multiplatform)
for IntelliJ IDEs provides further support,
automatically creating tailored run configurations, handling Compose Hot Reload integration, and so on.

## Build and run KMP applications

To build your KMP apps, you only need Gradle and Java.
However, IntelliJ IDEA and Android Studio provide a lot of quality-of-life features for KMP development,
from managing the environment to writing build scripts and multiplatform code.

You can run the application on any supported platform using the same IDE:
* the Android app runs on available Android Virtual Devices,
* the iOS app runs on iOS Simulator (you need a macOS machine to be able to run apps on Apple targets),
* the desktop app runs on the system JVM,
* the web app runs in the default browser.

The run configurations provided by the KMP IDE plugin are more efficient than the general Gradle build task:
They only trigger builds for corresponding targets while the default Gradle build task
builds debug and release versions of all targets.

### Run your application on Android Emulator

The default run configuration automatically suggests the list of available Android virtual devices.
If there are none, or you'd like to simulate a different device, you can configure one using the Android Device Manager
(in IDEA, **View | Tool Window | Device Manager**,
or following the [Android Studio guide](https://developer.android.com/studio/run/managing-avds)).

> The Device Manager in Android Studio usually offers a wider set of available devices,
> but after you create a device, it is available system-wide — including run configurations in IntelliJ IDEA.
>
{style="tip"}

The device will be available to the run configuration as soon as it is created.

1. In the list of run configurations, select **androidApp**.
2. Choose your Android virtual device and then click **Run**: Your IDE starts the selected virtual device if it
   is powered down and runs the app.

![Run the Compose Multiplatform app on Android](compose-run-android.png){width=352}

### Run on a real Android device

To make a hardware Android device available to KMP run configurations,
[configure it and connect it to your machine](https://developer.android.com/studio/run/device).

When it's set up correctly, it will show up in the list of available devices along with the virtual devices.

### Run your application on iOS Simulator

If you haven't launched Xcode as part of the initial setup, do that before running the iOS app.
You need the iOS platform support installed:
In Xcode, check **Xcode | Settings | Components** to make sure at least one iOS simulator is installed.

In your Kotlin Multiplatform IDE, select the iOS entry in the list of run configurations
and select a simulated device from the list next to it,
then click **Run**:

![Run the Compose Multiplatform app on iOS](compose-run-ios.png){width=405}

#### Run on a real iOS device {initial-collapse-state="collapsed" collapsible="true"}

You can run your multiplatform application on a real iOS device. Before you start,
you'll need to set the Team ID associated with your [Apple ID](https://support.apple.com/en-us/HT204316).

##### Set your Team ID

To set new Team ID for your project for the first time, open the project in Xcode
(**File | Open Project in Xcode**):

1. In the Project navigator on the left-hand side, select **iosApp**.
2. Select **iosApp** under **Targets** and switch to the **Signing & Capabilities** tab.
3. In the **Team** list, select your team.

   If you haven't set up your team yet, use the **Add an Account** option in the **Team** list and follow Xcode instructions.

4. Make sure that the Bundle Identifier is unique and a Signing Certificate is successfully assigned.

After you set up a team in Xcode, you can set or change the team in IntelliJ IDEA:

1. Edit the run configuration for **iosApp**:

   ![Edit iOS run configuration](ios-edit-configurations.png){width=450}

2. Switch to the **Options** tab and make the necessary changes in the **Development team** dropdown, then click **OK**.

##### Run the app

Connect your iPhone with a cable. If you already have the device registered in Xcode, IntelliJ IDEA should show it
in the list of run configurations. Run the corresponding `iosApp` configuration.

If you haven't registered your iPhone in Xcode yet, follow [Apple recommendations](https://developer.apple.com/documentation/xcode/running-your-app-in-simulator-or-on-a-device/).
In short, you should:

1. Connect your iPhone with a cable.
2. On your iPhone, enable the developer mode in **Settings** | **Privacy & Security**.
3. In Xcode, go to the top menu and choose **Window** | **Devices and Simulators**.
4. If your iPhone is not shown as connected, click the plus sign at the bottom left and select it.
5. Follow the on-screen instructions to complete the pairing process.

Once you've registered your iPhone in Xcode, it will become available in the list of available devices in IntelliJ IDEA
when you select the **iosApp** run configuration.

### Run your application on desktop

Select **desktopApp [hot] 🔥** in the list of run configurations and click **Run**:

![Run the Compose Multiplatform app on desktop](compose-run-desktop.png){width=350}

By default, the app starts with a [Compose Hot Reload](compose-hot-reload.md) running.
This allows reloading the UI almost instantly as soon as you manually save the file with the changes.

### Run your web application

The default options for web targets are:

* **webApp[js]**: To run your Kotlin/JS application.
* **webApp[wasmJs]**: To run your Kotlin/Wasm application.

The web application opens automatically in your default browser
and is available by default at [http://localhost:8080/](http://localhost:8080/). 

> Port 8080 may be unavailable, in this case the build picks another one.
> You can always find the actual port number in the Gradle build console by searching for the phrase "Project is running at".
>
{style="note"}

![Compose web application](first-compose-project-on-web.png){width=600}

#### Compatibility mode for web targets

You can enable compatibility mode for your web application to ensure it works on all browsers out of the box.
In this mode, modern browsers use the Wasm version, while older ones fall back to the JS version.
This mode is achieved through cross-compilation for both the `js` and `wasmJs` targets.

To enable compatibility mode for your web application:

1. Open the Gradle tool window by selecting **View | Tool Windows | Gradle**.
2. In **ComposeDemo | Tasks | compose**, select and run the **composeCompatibilityBrowserDistribution** task.

   > You need at least Java 11 as your Gradle JVM for the tasks to load successfully, and we recommend at least
   > Java 17 for Compose Multiplatform projects in general.
   >
   {style="note"}

   ![Run compatibility task](web-compatibility-gradle-task.png){width=500}

   Alternatively, you can run the following command in the terminal from the root project directory:

    ```bash
    ./gradlew composeCompatibilityBrowserDistribution
    ```

Once the Gradle task completes, compatible artifacts are generated in the web application module directory, for example
`webApp/build/dist/composeWebCompatibility/productionExecutable`.
You can use these artifacts to [publish your application](https://kotlinlang.org/docs/wasm-get-started.html#publish-the-application) 
for both the `js` and `wasmJs` targets.
