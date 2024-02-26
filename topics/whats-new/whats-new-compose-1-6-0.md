[//]: # (title: Compose Multiplatform 1.6.0)

The Compose Multiplatform 1.6.0 release is out. Here are the highlights:

* [New and improved Resources API](#improved-resources-api-all-platforms)
* [Basic support for iOS accessibility features](#ios-accessibility-support)
* [UI testing API](#ui-testing-api-experimental-all-platforms)
* [Popup, Dialog, and DropdownMenu components are now placed in separate platform views](#separate-platform-views-for-popups-dialogs-and-dropdowns-ios-desktop).
* Jetpack Compose 1.6.1 and Material 3 1.2.0 were merged into Compose Multiplatform. Here is a [summary of changes](#changes-from-jetpack-compose-and-material-3-all-platforms)
  with comments on what this means for Compose Multiplatform consumers.
* [Compose Multiplatform for Web (Wasm) artifacts are available with the stable version of the framework](#compose-multiplatform-for-web-wasm-artifacts-are-available-with-the-stable-version-of-the-framework).

## Dependencies

This version of Compose Multiplatform is based on the following Jetpack Compose libraries:

* [Compiler 1.5.8](https://developer.android.com/jetpack/androidx/releases/compose-compiler#1.5.8)
* [Runtime 1.6.1](https://developer.android.com/jetpack/androidx/releases/compose-runtime#1.6.1)
* [UI 1.6.1](https://developer.android.com/jetpack/androidx/releases/compose-ui#1.6.1)
* [Foundation 1.6.1](https://developer.android.com/jetpack/androidx/releases/compose-foundation#1.6.1)
* [Material 1.6.1](https://developer.android.com/jetpack/androidx/releases/compose-material#1.6.1)
* [Material3 1.2.0](https://developer.android.com/jetpack/androidx/releases/compose-material3#1.2.0)

## Breaking changes

### Padding for text with lineHeight set is trimmed by default

With the added support for [LineHeightStyle.Trim](https://developer.android.com/reference/kotlin/androidx/compose/ui/text/style/LineHeightStyle.Trim)
Compose Multiplatform aligns with Android in the way text padding is trimmed.
See [the pull request](https://github.com/JetBrains/compose-multiplatform-core/pull/897) for details.



TODO mention the includeFontPadding

### Using fontSize in MaterialTheme now needs lineHeight as well

> This affects only `material` components. `material3` already had this restriction.
>
{type="note"}

If you set the `fontSize` attribute for a `Text` component in `MaterialTheme` but don't include `lineHeight`, actual line
height will not be modified to match the font. Now, you have to explicitly specify the `lineHeight` attribute every time you
set the corresponding `fontSize`.

Jetpack Compose now [recommends](https://issuetracker.google.com/issues/321872412) not setting the font size directly:

>To support non-standard text sizes, we encourage users to follow the Material design system and use a different [type scale](https://m2.material.io/design/typography/the-type-system.html#type-scale)
> rather than changing the font size directly. Alternatively, users can overwrite the line height like so:
> `style = LocalTextStyle.current.copy(lineHeight = TextUnit.Unspecified)`, or create a custom `Typography` entirely.

### New approach to resource organization

If you have been using the Resources API in preview versions of Compose Multiplatform 1.6.0, familiarize yourself with
[the documentation for the current version](compose-images-resources.md): 1.6.0-beta01 changed the way resource files
should be stored in the project folders to be available to the project code.

## Across platforms

### Improved Resources API (all platforms)

The new experimental API adds support for strings and fonts, and allows you to more comfortably share and access resources
in common Kotlin:

* Resources can be organized according to specific settings or constraints they are designed for, supporting:
  * locales,
  * image resolutions,
  * dark and light themes.
* Compose Multiplatform tooling now generates a `Res` object for each project to provide straightforward resource access.

For a closer look at resource qualifiers, as well as a more in-depth overview of the new resources API,
see [the documentation page](compose-images-resources.md).

### UI testing API (experimental, all platforms)

The experimental API for UI testing with Compose Multiplatform allows you to write common tests that
validate the behavior of your application's UI across platforms supported by the framework. The API uses the same
finders, assertions, actions, and matchers as Jetpack Compose on Android.

> JUnit-based tests are supported only for Compose Multiplatform for Desktop.
>
{type="note"}

See the setup instructions and test examples in [the API documentation](https://www.jetbrains.com/help/kotlin-multiplatform-dev/compose-test.html).

### Changes from Jetpack Compose and Material 3 (all platforms)

#### Jetpack Compose 1.6.1

Merging the latest release of Jetpack Compose positively affects performance on all platforms. For details,
see [the announcement on the Android Developers Blog](https://android-developers.googleblog.com/2024/01/whats-new-in-jetpack-compose-january-24-release.html).

Other notable features from this release:
* The change to default font padding came into effect only for Android targets. However, make sure to take into account
  a [side effect](#using-fontsize-in-materialtheme-now-needs-lineheight-as-well) of this change.
* Mouse selection was already supported in Compose Multiplatform for other targets. With 1.6.0, this includes Android, too.

Jetpack Compose features that are not ported to Compose Multiplatform yet:
* [BasicTextField2](https://github.com/JetBrains/compose-multiplatform/issues/4218)
* [Support for nonlinear font scaling](https://github.com/JetBrains/compose-multiplatform/issues/4305)
* [MultiParagraph.fillBoundingBoxes](https://github.com/JetBrains/compose-multiplatform/issues/4236)
* [Multiplatform Drag and drop](https://github.com/JetBrains/compose-multiplatform/issues/4235). Works only for Android
  right now. For Desktop, you can use an existing API, `Modifier.onExternalDrag`.

The JetBrains team is working on adopting these features in upcoming versions of Compose Multiplatform.

#### Compose Material 3 1.2.0

Release highlights:
* New experimental component `Segmented Button`, with single and multiple selection.
* Expanded color set with more surface options to make it easier to emphasize information in your UI.
  * Implementation note: the `ColorScheme` object is now immutable. If your code currently modifies the colors in `ColorScheme` directly,
    you will need to make use of the [copy](https://developer.android.com/reference/kotlin/androidx/compose/material3/ColorScheme#copy(androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color,androidx.compose.ui.graphics.Color)) method now to change colors.
  * Instead of a single surface value, there are now several options for surface color and surface container, for more
    flexible color management.

For more details on changes in Material 3, see [the release post on the Material Design Blog](https://material.io/blog/material-3-compose-1-2).

### Separate platform views for popups, dialogs, and dropdowns (iOS, Desktop)

Sometimes it’s important that popup elements (for example, tooltips and dropdown menus) are not limited by initial
Compose canvas or the app window. This becomes especially relevant if the Compose view is not occupying the full screen,
but needs to spawn an alert dialog. In 1.6.0 there is a way to reliably achieve that.

Note that popups and dialogs are still unable to draw anything out of their own bounds (for example, a shadow of the topmost container).

#### iOS (stable)

On iOS, the feature is active by default.
To switch back to the old behavior, set the `platformLayers` parameter to `false`:

```kotlin
ComposeUIViewController(
  configure = {
    platformLayers = false
  }
) {
  // your Compose code
}
```

#### Desktop (experimental)

To use the feature with Compose Multiplatform for Desktop, set the `compose.layers.type`
system property. Supported values:
* `WINDOW`, for creating `Popup` and `Dialog` components as separate undecorated windows.
* `COMPONENT`, for creating `Popup` or `Dialog` as a separate Swing component in the same window. It works only with offscreen
  rendering: `compose.swing.render.on.graphics` set to `true`, see the [Enhanced Swing interop](https://blog.jetbrains.com/kotlin/2023/08/compose-multiplatform-1-5-0-release/#enhanced-swing-interop)
  section of the 1.5.0 Compose Multiplatform release notes. Note that offscreen rendering only works for `ComposePanel`,
  not for full window Compose applications.

Example (TODO check and screenshot):

```kotlin
@OptIn(ExperimentalComposeUiApi::class)
fun main() = SwingUtilities.invokeLater {
    System.setProperty("compose.swing.render.on.graphics", "true")
    System.setProperty("compose.layers.type", "COMPONENT")

    val window = JFrame()
    window.defaultCloseOperation = WindowConstants.EXIT_ON_CLOSE

    val contentPane = JLayeredPane()
    contentPane.layout = null

    val composePanel = ComposePanel()
    composePanel.setBounds(200, 200, 200, 200)
    composePanel.setContent {
        ComposeContent()
    }
    composePanel.windowContainer = contentPane  // Use the full window for dialogs
    contentPane.add(composePanel)
    
    window.contentPane.add(contentPane)
    window.setSize(800, 600)
    window.isVisible = true
}

@Composable
fun ComposeContent() {
    Box(Modifier.fillMaxSize().background(Color.Green)) {
        Dialog(onDismissRequest = {}) {
            Box(Modifier.size(100.dp).background(Color.Yellow))
        }
    }
}

```

### Support for text decoration line styles (iOS, Desktop, Web)

Compose Multiplatform now allows setting underline styles for text using the `PlatformTextStyle` class.

> The class is not available in the common source set and needs to be called in platform-specific code.
>
{type="warning"}

Example of setting a dotted underline style:

```kotlin
Text(
    "Hello, Compose",
    style = TextStyle(
        textDecoration = TextDecoration.Underline,
        platformStyle = PlatformTextStyle(
            textDecorationLineStyle = TextDecorationLineStyle.Dotted
        )
    )
)
```

You can use solid, double-width solid, dotted, dashed, and wavy line style. All available options can be found in
[the source code](https://github.com/JetBrains/compose-multiplatform-core/blob/jb-main/compose/ui/ui-text/src/skikoMain/kotlin/androidx/compose/ui/text/TextDecorationLineStyle.kt#L21).

### Add a way to use fonts installed on the system (Desktop, Web, iOS)

> The feature will be supported on iOS in the next release. There are no plans for Android support.
>
{type="note"}

You can now access fonts installed on the system from your Compose Multiplatform app: use the `SystemFont` class to load
fonts with appropriate font styles and font weights:

```kotlin
import androidx.compose.ui.text.platform.SystemFont

FontFamily(SystemFont("Menlo", weight = 700))
FontFamily(SystemFont("Times New Roman", FontWeight.Bold))
FontFamily(SystemFont("Webdings"))
```

On desktop, you can use the `FontFamily` function to load all possible font styles by specifying a font family name only
(see [the code sample](https://github.com/JetBrains/compose-multiplatform-core/blob/jb-main/compose/desktop/desktop/samples/src/jvmMain/kotlin/androidx/compose/desktop/examples/fonts/Fonts.jvm.kt)
for an extensive example):

```kotlin
FontFamily("Menlo")
```

## iOS

### iOS accessibility support

Compose Multiplatform for iOS now allows people with disabilities to interact with Compose UI with the same level of comfort
as with native UIs:

* Screen readers and VoiceOver can access the content of the Compose UI.
* Compose UI supports the same gestures as the native UIs for navigation and interaction.

This also means that you can make the Compose semantic data available to Accessibility Services and the XCTest framework.

For details on implementation and customization API, see Support for iOS accessibility features (TODO link).

### Changing opacity for Compose view

The `ComposeUIViewController` class has one more configuration option now, to change the opacity of the view's background.
Now, instead of the default white color, you can set the background to be transparent.

> Please note that transparent background negatively affects performance as it leads to an additional blending step.
>
{type="note"}

Sample code:
```kotlin
val viewController = ComposeUIViewController(
    // Makes the Compose scene background transparent
    configure = { opaque = false },
    content = ::SomeComposeNode
)
```

TODO screenshots with opaque true and false

### Allow selecting Text in SelectionContainer by double and triple tap

Previously, Compose Multiplatform for iOS allowed users to use multiple tap for selecting text only in text input fields.
Now, double and triple tap gestures also work for selecting text displayed in `Text` components inside a `SelectionContainer`.

### Introduce @Composable fun UIKitViewController

Some native APIs that are not implemented as `UIView`, for example, `UITabBarController`, or `UINavigationController` could not
be embedded into a Compose Multiplatform UI using [the existing interop mechanism](compose-ios-ui-integration.md#use-uikit-inside-compose-multiplatform).

Now, CMP implements the `UIKitViewController` function that allows you to embed native view controllers.

TODO: update the linked article

### Native-like caret behavior by long/single taps in text fields

Compose Multiplatform for iOS is now closer to the native behavior of a caret in a text field:
* The position of the caret after a single tap in a text field is determined with more precision.
* Long tap and drag in a text field leads to moving the cursor, not entering selection mode like on Android.

## Web

### Compose Multiplatform for Web (Wasm) artifacts are available with the stable version of the framework

Stable version of Compose Multiplatform for Web supports Kotlin/Wasm targets now. After you switch to 1.6.0, you don't
have to specify a particular `dev-wasm` version of the `compose-ui` library in your list of dependencies.

> To build Compose Multiplatform with a Wasm target, you need to have a Kotlin version no older than 1.9.22.
>
{type="warning"}

## Known issues and solutions

### Missing dependencies

There are a couple of libraries that can be missing with a default project configuration:

* `org.jetbrains.compose.annotation-internal:annotation` or `org.jetbrains.compose.collection-internal:collection`

  These may be missing if a library depends on Compose Multiplatform 1.6.0-beta02, which isn't binary compatible with 1.6.0.
  To figure out which library that is, run this command (replace `shared` with the name of your main module):

  ```shell
  ./gradlew shared:dependencies
  ```

  Downgrade the library to a version depending on Compose Multiplatform 1.5.12, or ask the library author to upgrade it
  to Compose Multiplatform 1.6.0.

* `androidx.annotation:annotation:...` or `androidx.collection:collection:...`

  Compose Multiplatform 1.6.0 depends on [collection](https://developer.android.com/jetpack/androidx/releases/collection)
  and [annotation](https://developer.android.com/jetpack/androidx/releases/annotation) libraries that are available only
  in the Google Maven repository.

  To make this repository available to your project, add the following line to the `build.gradle.kts` file of the module:

  ```kotlin
  repositories {
      //...
      google()
  }
  ```
