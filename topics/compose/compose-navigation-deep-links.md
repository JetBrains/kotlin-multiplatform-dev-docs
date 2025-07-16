[//]: # (title: Deep links)

Deep linking is a navigation mechanism that allows the operating system to handle custom links
by taking the user to a specific destination in the corresponding app.

Deep links are a more general case of app links (as they are called on Android) or universal links (the iOS term):
these are verified connections of the app with a specific web address.
To learn about them specifically, see documentation on [Android App Links](https://developer.android.com/training/app-links)
and [iOS universal links](https://developer.apple.com/documentation/xcode/allowing-apps-and-websites-to-link-to-your-content/).

Deep links can also be useful for getting outside input into the app,
for example, in the case of OAuth authorization:
you can parse the deep link and get the OAuth token without necessarily visually navigating the users.

> Since outside input can be malicious, be sure to follow the [security guidelines](https://developer.android.com/privacy-and-security/risks/unsafe-use-of-deeplinks)
> to properly mitigate risks associated with processing raw deep link URIs.
> 
{style="warning"}

To implement a deep link in Compose Multiplatform:

1. [Register your deep link schema in the app configuration](#register-deep-links-schemas-in-the-operating-system)
2. [Assign specific deep links to destinations in the navigation graph](#assign-deep-links-to-destinations)
3. [Handle deep links received by the app](#handle-received-deep-links)

## Setup

To use deep links with Compose Multiplatform, set up the dependencies as follows.

List these versions, libraries, and plugins in your Gradle catalog:

```ini
[versions]
compose-multiplatform = "%org.jetbrains.compose%"
agp = "8.9.0"

# The multiplatform Navigation library version with deep link support 
androidx-navigation = "%org.jetbrains.androidx.navigation%"

# Minimum Kotlin version to use with Compose Multiplatform 1.8.0
kotlin = "2.1.0"

# Serialization library necessary to implement type-safe routes
kotlinx-serialization = "1.7.3"

[libraries]
navigation-compose = { module = "org.jetbrains.androidx.navigation:navigation-compose", version.ref = "androidx-navigation" }
kotlinx-serialization-json = { module = "org.jetbrains.kotlinx:kotlinx-serialization-json", version.ref = "kotlinx-serialization" }

[plugins]
multiplatform = { id = "org.jetbrains.kotlin.multiplatform", version.ref = "kotlin" }
compose-compiler = { id = "org.jetbrains.kotlin.plugin.compose", version.ref = "kotlin" }
compose = { id = "org.jetbrains.compose", version.ref = "compose-multiplatform" }
kotlinx-serialization = { id = "org.jetbrains.kotlin.plugin.serialization", version.ref = "kotlin" }
android-application = { id = "com.android.application", version.ref = "agp" }
```

Add the additional dependencies to the shared module's `build.gradle.kts`:

```kotlin
plugins {
    // ...
    alias(libs.plugins.kotlinx.serialization)
}

// ...

kotlin {
    // ...
    sourceSets {
        commonMain.dependencies {
            // ...
            implementation(libs.androidx.navigation.compose)
            implementation(libs.kotlinx.serialization.json)
        }
    }
}
```

## Register deep links schemas in the operating system

Each operating system has its own way of handling deep links.
It's more reliable to refer to the documentation for your particular targets:

* For Android apps, deep link schemes are declared as intent filters in the `AndroidManifest.xml` file.
  See [Android documentation](https://developer.android.com/training/app-links/deep-linking?hl=en#adding-filters)
  to learn how to properly set up intent filters.
* For iOS and macOS apps, deep link schemes are declared in `Info.plist` files,
    in the [CFBundleURLTypes](https://developer.apple.com/documentation/bundleresources/information-property-list/cfbundleurltypes)
    key.

    > Compose Multiplatform [provides a Gradle DSL](compose-native-distribution.md#information-property-list-on-macos)
    > to add values to a macOS app's `Info.plist`.
    > For iOS, you can edit the file directly in your KMP project or [register schemes using the Xcode GUI](https://developer.apple.com/documentation/xcode/defining-a-custom-url-scheme-for-your-app#Register-your-URL-scheme).
    >
    {style="note"}
* For Windows apps, deep link schemes can be declared by adding [keys with necessary information to the Windows registry](https://learn.microsoft.com/en-us/previous-versions/windows/internet-explorer/ie-developer/platform-apis/aa767914(v=vs.85))
    (for Windows 8 and earlier)
    or by specifying the [extension in the package manifest](https://learn.microsoft.com/en-us/windows/apps/develop/launch/handle-uri-activation) (for Windows 10 and 11).
    This can be done with an installation script or a third-party distribution package generator like [Hydraulic Conveyor](https://conveyor.hydraulic.dev/).
    Compose Multiplatform does not support configuring this within the project itself.
    
    > Make sure you're not using one of the [schemes reserved by Windows](https://learn.microsoft.com/en-us/windows/apps/develop/launch/reserved-uri-scheme-names#reserved-uri-scheme-names).
    >
    {style="tip"}
* For Linux, deep link schemes can be registered in a `.desktop` file included in the distribution.

## Assign deep links to destinations

A destination declared as a part of a navigation graph has an optional `deepLinks` parameter
which can hold the list of corresponding `NavDeepLink` objects.
Each `NavDeeplink` describes a URI pattern that should match a destination – you can define multiple URI patterns
that should lead to the same screen.

There is no limit to the number of deep links you can define for a route.

### General URI patterns for deep links

A general URI pattern should match the entire URI.
You can use placeholders for parameters to extract them from the received URI within the destination.

Rules for general URI patterns:

* URIs without a scheme are assumed to start with `http://` or `https://`.
  So `uriPattern = "example.com"` matches `http://example.com` and `https://example.com`.
* `{placeholder}` matches one or more characters (`example.com/name={name}` matches `https://example.com/name=Bob`).
  To match zero or more characters, use the `.*` wildcard (`example.com/name={.*}` matches `https://example.com/name=`
  as well as any value of `name`).
* Parameters for path placeholders are required while matching query placeholders is optional.
  For example, the pattern `example.com/users/{id}?arg1={arg1}&arg2={arg2}`:
    * Doesn't match `http://www.example.com/users?arg1=one&arg2=two` because the required part of the path (`id`) is missing.
    * Matches both `http://www.example.com/users/4?arg2=two` and `http://www.example.com/users/4?arg1=one`.
    * Also matches `http://www.example.com/users/4?other=random` because extraneous query parameters don't affect matching.
* If several composables have a `navDeepLink` that matches the received URI, behavior is indeterminate.
  Make sure that your deep link patterns don't intersect.
  If you need multiple composables to handle the same deep link pattern, consider adding path or query parameters,
  or use an intermediate destination to route the user predictably.

### Generated URI pattern for a route type

You can avoid writing out the URI pattern fully:
the Navigation library can automatically generate a URI pattern based on the parameters of a route.

To use this approach, define a deep link like this:

```kotlin
composable<PlantDetail>(
    deepLinks = listOf(
        navDeepLink<PlantDetail>(basePath = "demo://example.com/plant")
    )
) { ... }
```

Here `PlantDetail` is the route type you're using for the destination, and the "plant" in `basePath`
is the serial name of the `PlantDetail` data class.

The rest of the URI pattern will be generated as follows:

* Required parameters are appended as path parameters (example: `/{id}`)
* Parameters with a default value (optional parameters) are appended as query parameters (example: `?name={name}`)
* Collections are appended as query parameters (example: `?items={value1}&items={value2}`)
* The order of parameters matches the order of the fields in the route definition.

So, for example, this route type:

```kotlin
@Serializable data class PlantDetail(
  val id: String,
  val name: String,
  val colors: List<String>,
  val latinName: String? = null,
)
```

has the following generated URI pattern generated by the library:

```none
<basePath>/{id}/{name}/?colors={color1}&colors={color2}&latinName={latinName}
```

### Example of adding deep links to a destination

In this example we assign several deep links to a destination and then
extract parameter values from the received URIs:

```kotlin
@Serializable @SerialName("dlscreen") data class DeepLinkScreen(val name: String)

// ...

val firstBasePath = "demo://example1.org"

NavHost(
    navController = navController,
    startDestination = FirstScreen
) {
    // ...
    
    composable<DeepLinkScreen>(
        deepLinks = listOf(
            // This composable should handle links both for demo://example1.org and demo://example2.org
            navDeepLink { uriPattern = "$firstBasePath?name={name}" },
            navDeepLink { uriPattern = "demo://example2.org/name={name}" },
            // The generated pattern only handles the parameters,
            // so we add the serial name for the route type
            navDeepLink<Screen3>(basePath = "$firstBasePath/dlscreen"),
        )
    ) {
        // If the app receives the URI `demo://example1.org/dlscreen/Jane/`,
        // it matches the generated URI pattern (name is a required parameter and is given in the path),
        // and you can map it to the route type automatically
        val deeplink: DeepLinkScreen = backStackEntry.toRoute()
        val nameGenerated = deeplink.name
        
        // If the app receives a URI matching only a general pattern,
        // like `demo://example1.com/?name=Jane`
        // you need to parse the URI directly
        val nameGeneral = backStackEntry.arguments?.read { getStringOrNull("name") }
        
        // Composable content
    }
}
```

For web, deep links work a bit differently: since Compose Multiplatform for Web makes single-page apps,
you need to put all parameters of the deep link URI pattern in a URL fragment (after the `#` character),
and make sure that all parameters are URL-encoded.

You can still use the `backStackEntry.toRoute()` method to parse the parameters if the URL fragment conforms
to the URI pattern rules.
For details on accessing and parsing a URL in a web app, as well as particulars on navigation in the browser,
see [](compose-navigation-routing.md#support-for-browser-navigation-in-web-apps).

```kotlin
composable<DeepLinkScreen>(
        deepLinks = listOf(
            // For the default Compose Multiplatform setup, localhost:8080
            // is the local dev endpoint that runs with the wasmJsBrowserDevelopmentRun Gradle task
            navDeepLink { uriPattern = "localhost:8080/#dlscreen%2F{name}" },
        )
    ) { ... }
```

> Like with any other single-page web app, you can avoid using URL fragments on the web.
> To do that, you have to configure your web server to redirect appropriate requests to the app
> and rewrite the [default mapping of navigation route to the browser address](compose-navigation-routing.md#full-url-customization).
>
{style="tip"}

## Handle received deep links

On Android, the deep link URIs sent to the app are available as a part of the `Intent` that triggered the deep link.
A cross-platform implementation needs a universal way to listen for deep links.

Let's create a bare-bones implementation:

1. Declare a singleton in common code for storing and caching the URIs with a listener for external URIs.
2. Where necessary, implement platform-specific calls sending URIs received from the operating system.
3. Set up the listener for new deep links in the main composable.

### Declare a singleton with a URI listener

In `commonMain`, declare the singleton object at the top level:

```kotlin
object ExternalUriHandler {
    // Storage for when a URI arrives before the listener is set up
    private var cached: String? = null
    
    var listener: ((uri: String) -> Unit)? = null
        set(value) {
            field = value
            if (value != null) {
                // When a listener is set and `cached` is not empty,
                // immediately invoke the listener with the cached URI
                cached?.let { value.invoke(it) }
                cached = null
            }
        }

    // When a new URI arrives, cache it.
    // If the listener is already set, invoke it and clear the cache immediately.
    fun onNewUri(uri: String) {
        cached = uri
        listener?.let {
            it.invoke(uri)
            cached = null
        }
    }
}
```

### Implement platform-specific calls to the singleton

Both for desktop JVM and for iOS you need to explicitly pass the URI received from the system.

In `jvmMain/.../main.kt`, parse the command-line arguments for every necessary operating system
and pass the received URI on to the singleton:

```kotlin
// Import the singleton
import org.company.app.ExternalUriHandler

fun main() {
    if(System.getProperty("os.name").indexOf("Mac") > -1) {
        Desktop.getDesktop().setOpenURIHandler { uri ->
            ExternalUriHandler.onNewUri(uri.uri.toString())
        }
    }
    else {
        ExternalUriHandler.onNewUri(args.getOrNull(0).toString())
    }

    application {
         // ...
    }
}
```

For iOS, in Swift code add an `application()` variant that handles incoming URIs:

```swift
// Imports the KMP module to access the singleton
import ComposeApp

func application(
    _ application: UIApplication,
    open uri: URL,
    options: [UIApplication.OpenURLOptionsKey: Any] = [:]
) -> Bool {
    // Sends the full URI on to the singleton
    ExternalUriHandler.shared.onNewUri(uri: uri.absoluteString)    
        return true
    }
```

> For naming conventions for accessing singletons from Swift, see [Kotlin/Native documentation](https://kotlinlang.org/docs/native-objc-interop.html#kotlin-singletons).
> 
{style="tip"}

### Set up the listener

You can use a `DisposableEffect(Unit)` to set up the listener and clean it up after the composable is no longer active.
For example:

```kotlin
internal fun App(navController: NavHostController = rememberNavController()) = AppTheme {

    // The effect is produced only once, as `Unit` never changes
    DisposableEffect(Unit) {
        // Sets up the listener to call `NavController.navigate()`
        // for the composable that has a matching `navDeepLink` listed
        ExternalUriHandler.listener = { uri ->
            navController.navigate(NavUri(uri))
        }
        // Removes the listener when the composable is no longer active
        onDispose {
            ExternalUriHandler.listener = null
        }
    }

    // Reusing the example from earlier in this article
    NavHost(
        navController = navController,
        startDestination = FirstScreen
    ) {
        // ...

        composable<DeepLinkScreen>(
            deepLinks = listOf(
                navDeepLink { uriPattern = "$firstBasePath?name={name}" },
                navDeepLink { uriPattern = "demo://example2.com/name={name}" },
            )
        ) {
            // Composable content
        }
    }
}
```

## Result

Now you can see the full workflow:
when the user opens a `demo://` URI, the operating system matches it with the registered scheme.
Then:
  * If the app handling the deep link is closed, the singleton receives the URI and caches it.
    When the main composable function starts,
    it calls the singleton and navigates to the deep link matching the cached URI.
  * If the app handling the deep link is open, the listener is already set up, so when the singleton receives the URI
    the app immediately navigates to it.
