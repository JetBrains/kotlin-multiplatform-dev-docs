[//]: # (title: Deep links)

Deep linking is a navigation mechanism that allows the operating system to handle custom links
by taking the user to a specific destination in the corresponding app.

To implement a deep link in Compose Multiplatform:

1. Register your deep link schema in the app configuration.
2. Assign specific deep links to composables in the navigation graph.
3. Handle deep links received by the app by converting them into navigation commands.

## Registering deep links schemas for operating systems

Each operating system has its own way of handling deep links.
It's more reliable to refer to the documentation for your particular targets:

* Deep links for Android apps are declared as intent filters in the `AndroidManifest.xml` file.
  See [Android documentation](https://developer.android.com/training/app-links/deep-linking?hl=en#adding-filters)
  to learn how to properly set up intent filters.
* Deep links for iOS and macOS apps are declared in `Info.plist` files,
    in the [CFBundleURLTypes](https://developer.apple.com/documentation/bundleresources/information-property-list/cfbundleurltypes)
    key.

    > Compose Multiplatform [provides a Gradle DSL](compose-native-distribution.md#information-property-list-on-macos)
    > to add values to a macOS app's `Info.plist`.
    > For iOS, you can edit the file directly in your KMP project or [register schemes using the Xcode GUI](https://developer.apple.com/documentation/xcode/defining-a-custom-url-scheme-for-your-app#Register-your-URL-scheme).
    >
    {style="note"}
* Deep links for Windows apps can be declared by adding a key to the Windows registry. (TODO link to the correct one or mention if there is no link)
    This can be done with an installation script.
    Compose Multiplatform does not support configuring this within the project itself.

## Assign deep links to composables

A composable that is declared as a part of a navigation graph has an optional `deepLinks` parameter
which holds the list of `NavDeepLink` objects.
Each `NavDeeplink` describes a URI pattern that should match the composable – you can define multiple URI patterns
to lead to the same screen.

For example:

```kotlin
// ...

val firstUrl = "demo://example1.com"

NavHost(
    navController = navController,
    startDestination = FirstScreen
) {
    // ...
    
    composable<DeepLinkScreen>(
        deepLinks = listOf(
            // This composable should handle links both for demo://example1.com and demo://example2.com
            navDeepLink { uriPattern = "$firstUrl?name={name}" },
            navDeepLink { uriPattern = "demo://example2.com/name={name}" },
        )
    ) {
        // Composable content
    }
}
```

Rules for writing URI patterns:

* URIs without a scheme are assumed to be `http://` or `https://`.
    So `uriPattern = "example.com"` matches both `http://example.com` and `https://example.com`.
* `{placeholder}` matches one or more characters (`example.com/name={name}` matches `https://example.com/name=Bob`).
    To match zero or more characters, use the `.*` wildcard (`example.com/name={.*}`)
* Parameters for path placeholders are required while query placeholders are optional.
  For example, the pattern `example.com/users/{id}?arg1={arg1}&arg2={arg2}`
    * Matches both `http://www.example.com/users/4?arg2=two` and `http://www.example.com/users/4?arg1=one`.
    * Doesn't match `http://www.example.com/users?arg1=one&arg2=two` because the required part of the path is missing.
    * Also matches `http://www.example.com/users/4?other=random` as extraneous query parameters don't affect matching.
* If several have a `navDeepLink` that matches the received URI, behavior is indeterminate.
    Make sure that your deep link patterns don't intersect.

## Handle received deep links

On Android, the received deep link URIs are available as a part of the `Intent` that triggered the deep link.
A cross-platform implementation needs a way to catch and store deep links.

Let's create a bare-bones implementation:

1. Declare a singleton in common code for storing and caching the URIs with a listener for external URLs.
2. Where necessary, implement platform-specific calls sending URIs received from the operating system.
3. Set up the listener for new deep links in the main composable.

### Declare a singleton with a URI listener

In `commonMain`, declare the singleton object at the top level:

```kotlin
object ExternalUrlHandler {
    // Storage for when a URI arrives before the listener is set up.
    private var cached: String? = null
    
    var listener: ((url: String) -> Unit)? = null
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
    fun onNewUrl(url: String) {
        cached = url
        listener?.let {
            it.invoke(url)
            cached = null
        }
    }
}
```

### Implement platform-specific calls to the singleton

Both for desktop JVM and for iOS you need to explicitly pass the URI received from the system.

For desktop, in `jvmMain/.../main.kt` add a call to the `main()` function:

```kotlin
// Import the singleton
import org.company.app.ExternalUrlHandler

fun main() {
    Desktop.getDesktop().setOpenURIHandler { url ->
        // Call the `onNewUrl` function with the URI from the `OpenURIEvent` object.
        ExternalUrlHandler.onNewUrl(url.uri.toString())
    }

    application {
         // ...
    }
}
```

For iOS, in Swift code add an `application()` variant that handles incoming URLs:

```swift
// Imports the KMP module to access the singleton
import ComposeApp

func application(
    _ application: UIApplication,
    open url: URL,
    options: [UIApplication.OpenURLOptionsKey: Any] = [:]
) -> Bool {
    // Sends the full URL on to the singleton.
    ExternalUrlHandler.shared.onNewUrl(url: url.absoluteString)    
        return true
    }
```

> For conventions on accessing singletons from Swift, see [Kotlin/Native documentation](https://kotlinlang.org/docs/native-objc-interop.html#kotlin-singletons).
> 
{style="tip"}

### Set up the listener

You can use a `DisposableEffect(Unit)` that sets the listener and cleans it up after the composable is no longer active.
For example:

```kotlin
internal fun App(navController: NavHostController = rememberNavController()) = AppTheme {

    // The effect is produced only once, as `Unit` never changes.
    DisposableEffect(Unit) {
        // Sets up the listener to call `NavController.navigate()`
        // for the composable that has a matching `navDeepLink` listed.
        ExternalUrlHandler.listener = { url ->
            navController.navigate(parseStringAsNavUri(url))
        }
        // Removes the listener when the composable is no longer active.
        onDispose {
            ExternalUrlHandler.listener = null
        }
    }

    // Reusing the example from earlier in this article.
    NavHost(
        navController = navController,
        startDestination = FirstScreen
    ) {
        // ...

        composable<DeepLinkScreen>(
            deepLinks = listOf(
                navDeepLink { uriPattern = "$firstUrl?name={name}" },
                navDeepLink { uriPattern = "demo://example2.com/name={name}" },
            )
        ) {
            // Composable content
        }
    }
}
```

### Full cycle

Now you can see the full workflow:
when the user opens the `demo://example2.com/name=Alice` URL, the operating system matches it with the registered scheme. 
  * If the app handling the deep link is closed, the singleton receives the URL and caches it.
    When the main composable function starts,
    it calls the singleton and navigates to the deep link matching the cached URI.
  * If the app handling the deep link is open, the listener is already set up, so when the singleton receives the URL
    the app immediately navigates to it.