# Manage local resource environment

You may need to manage in-app settings that allow users customize their experience, such as changing the language or theme.
To dynamically update the application's resource environment, you can configure the following resource-related 
settings used by the application:

* [Locale (language and region)](#locale)
* [Theme](#theme)
* [Resolution density](#density)

## Locale

Each platform handles locale settings such as language and region differently. As a temporary workaround, until a common 
public API is implemented ([CMP-4197](https://youtrack.jetbrains.com/issue/CMP-4197)), you need to define 
a common entry point in the shared code. Then, provide the corresponding declarations for each platform 
using the platform-specific API:

* **Android**: `context.resources.configuration.locale`
* **iOS**: `NSLocale.preferredLanguages`
* **desktop**: `Locale.getDefault()`
* **web**: `window.navigator.languages`

1. In the common source set, define the expected `LocalAppLocale` object with the `expect` keyword:

    ```kotlin
    var customAppLocale by mutableStateOf<String?>(null)
    expect object LocalAppLocale {
        val current: String @Composable get
        @Composable infix fun provides(value: String?): ProvidedValue<*>
    }
    
    @Composable
    fun AppEnvironment(content: @Composable () -> Unit) {
        CompositionLocalProvider(
            LocalAppLocale provides customAppLocale,
        ) {
            key(customAppLocale) {
                content()
            }
        }
    }
    ```

2. In the Android source set, add the `actual` implementation that uses `context.resources.configuration.locale`:

    ```kotlin
    actual object LocalAppLocale {
        private var default: Locale? = null
        actual val current: String
            @Composable get() = Locale.getDefault().toString()
    
        @Composable
        actual infix fun provides(value: String?): ProvidedValue<*> {
            val configuration = LocalConfiguration.current
    
            if (default == null) {
                default = Locale.getDefault()
            }
    
            val new = when(value) {
                null -> default!!
                else -> Locale(value)
            }
            Locale.setDefault(new)
            configuration.setLocale(new)
            val resources = LocalContext.current.resources
    
            resources.updateConfiguration(configuration, resources.displayMetrics)
            return LocalConfiguration.provides(configuration)
        }
    }
    ```

3. In the iOS source set, add the `actual` implementation that modifies `NSLocale.preferredLanguages`:
 
    ```kotlin
    @OptIn(InternalComposeUiApi::class)
    actual object LocalAppLocale {
        private const val LANG_KEY = "AppleLanguages"
        private val default = NSLocale.preferredLanguages.first() as String
        private val LocalAppLocale = staticCompositionLocalOf { default }
        actual val current: String
            @Composable get() = LocalAppLocale.current
    
        @Composable
        actual infix fun provides(value: String?): ProvidedValue<*> {
            val new = value ?: default
            if (value == null) {
                NSUserDefaults.standardUserDefaults.removeObjectForKey(LANG_KEY)
            } else {
                NSUserDefaults.standardUserDefaults.setObject(arrayListOf(new), LANG_KEY)
            }
            return LocalAppLocale.provides(new)
        }
    }
    ```

4. In the desktop source set, add the `actual` implementation that uses `Locale.getDefault()` to update the JVM's default locale:

    ```kotlin
    actual object LocalAppLocale {
        private var default: Locale? = null
        private val LocalAppLocale = staticCompositionLocalOf { Locale.getDefault().toString() }
        actual val current: String
            @Composable get() = LocalAppLocale.current
    
        @Composable
        actual infix fun provides(value: String?): ProvidedValue<*> {
            if (default == null) {
                default = Locale.getDefault()
            }
            val new = when(value) {
                null -> default!!
                else -> Locale(value)
            }
            Locale.setDefault(new)
            return LocalAppLocale.provides(new.toString())
        }
    }
    ```

5. For the web platform, bypass the read-only restriction of the `window.navigator.languages` property to introduce 
 a custom locale logic:

    ```kotlin
    external object window {
        var __customLocale: String?
    }
    
    actual object LocalAppLocale {
        private val LocalAppLocale = staticCompositionLocalOf { Locale.current }
        actual val current: String
            @Composable get() = LocalAppLocale.current.toString()
    
        @Composable
        actual infix fun provides(value: String?): ProvidedValue<*> {
            window.__customLocale = value?.replace('_', '-')
            return LocalAppLocale.provides(Locale.current)
        }
    }
    ```

    Then, in your browser's `index.html`, put the following code before loading the application scripts:

    ```html    
    <html lang="en">
        <head>
            <meta charset="UTF-8">
            ...
            <script>
                var currentLanguagesImplementation = Object.getOwnPropertyDescriptor(Navigator.prototype, "languages");
                var newLanguagesImplementation = Object.assign({}, currentLanguagesImplementation, {
                    get: function () {
                        if (window.__customLocale) {
                            return [window.__customLocale];
                        } else {
                            return currentLanguagesImplementation.get.apply(this);
                        }
                    }
                });
        
                Object.defineProperty(Navigator.prototype, "languages", newLanguagesImplementation)
            </script>
            <script src="skiko.js"></script>
            ...
        </head>
        <body></body>
        <script src="composeApp.js"></script>
    </html>
    ```  

## Theme 

Compose Multiplatform defines the current theme via `isSystemInDarkTheme()`. 
Themes are handled differently across platforms:

* Android uses `Resources.getConfiguration().uiMode and Configuration.UI_MODE_NIGHT_MASK`.
* iOS, desktop, and web platforms use `LocalSystemTheme.current`.

As a temporary workaround, while a common public API is yet to be implemented 
([CMP-4197](https://youtrack.jetbrains.com/issue/CMP-4197)), this difference requires an `expect-actual` mechanism to 
handle platform-specific theme customization:

1. In the common code, define the expected `LocalAppTheme` object with the `expect` keyword:
 
    ```kotlin
    var customAppThemeIsDark by mutableStateOf<Boolean?>(null)
    expect object LocalAppTheme {
        val current: Boolean @Composable get
        @Composable infix fun provides(value: Boolean?): ProvidedValue<*>
    }
    
    @Composable
    fun AppEnvironment(content: @Composable () -> Unit) {
        CompositionLocalProvider(
            LocalAppTheme provides customAppThemeIsDark,
        ) {
            key(customAppThemeIsDark) {
                content()
            }
        }
    }
    ```

2. In Android code, add the actual implementation that uses the `LocalConfiguration` API:

    ```kotlin
    actual object LocalAppTheme {
        private var default: Int? = null
        actual val current: Boolean
            @Composable get() = (LocalConfiguration.current.uiMode and UI_MODE_NIGHT_MASK) == UI_MODE_NIGHT_YES
    
        @Composable
        actual infix fun provides(value: Boolean?): ProvidedValue<*> {
            val configuration = LocalConfiguration.current
    
            if (default == null) {
                default = configuration.uiMode
            }
    
            val new = when(value) {
                true -> (configuration.uiMode and UI_MODE_NIGHT_MASK.inv()) or UI_MODE_NIGHT_YES
                false -> (configuration.uiMode and UI_MODE_NIGHT_MASK.inv()) or UI_MODE_NIGHT_NO
                null -> default!!
            }
            configuration.uiMode = new
            return LocalConfiguration.provides(configuration)
        }
    }
    ```

3. On iOS, desktop, and web platforms, you can change `LocalSystemTheme` directly:

    ```kotlin
    @OptIn(InternalComposeUiApi::class)
    actual object LocalAppTheme {
        private var default: SystemTheme? = null
        actual val current: Boolean
            @Composable get() = LocalSystemTheme.current == SystemTheme.Dark
    
        @Composable
        actual infix fun provides(value: Boolean?): ProvidedValue<*> {
            if (default == null) {
                default = LocalSystemTheme.current
            }
            val new = when(value) {
                true -> SystemTheme.Dark
                false -> SystemTheme.Light
                null -> default!!
            }
    
            return LocalSystemTheme.provides(new)
        }
    }
    ```

## Density

To change the application's `Density`, you can use the common `LocalDensity` API, supported on all platforms:

```kotlin
var customAppDensity by mutableStateOf<Density?>(null)
object LocalAppDensity {
    val current: Density
        @Composable get() = LocalDensity.current

    @Composable
    infix fun provides(value: Density?): ProvidedValue<*> {
        val new = value ?: LocalDensity.current
        return LocalDensity.provides(new)
    }
}

@Composable
fun AppEnvironment(content: @Composable () -> Unit) {
    CompositionLocalProvider(
        LocalAppDensity provides customAppDensity,
    ) {
        key(customAppDensity) {
            content()
        }
    }
}
```

## What's next?

* Get more details on resource [qualifiers](compose-multiplatform-resources-setup.md#qualifiers).
* Learn how to [localize resources](compose-rtl.md).

[//]: # (todo: replace localization link)