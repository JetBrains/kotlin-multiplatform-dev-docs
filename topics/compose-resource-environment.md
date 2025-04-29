# Manage local resource environment

Learn how to manage the application's resource environment to match dynamically updated resource-related settings.
For example, you may need to handle changes to the in-app language or theme.

The `ResourceEnvironment` class allows you to configure locale (language and region), theme, and resolution density 
used by the application:

```kotlin
class ResourceEnvironment(
    val language: LanguageQualifier,
    val region: RegionQualifier,
    val theme: ThemeQualifier,
    val density: DensityQualifier
)
```

## Configure locale

In Compose Multiplatform, the resource library uses `Locale.current` to define and manage the current locale settings, 
such as language and region. However, platforms apply locale settings differently, and common API is not implemented yet.

Each platform has its own API for updating the global locale settings:

* **Android**: `context.resources.configuration.locale`
* **iOS**: `NSLocale.preferredLanguages`
* **desktop**: `Locale.getDefault()`
* **web**: `window.navigator.languages`

For multiplatform locale configuration, define a common interface in the shared code.
Then, implement the interface for each platform using the corresponding platform-specific API.

1. In the common source set, define the interface with the `expect` keyword:

```kotlin
var customAppLocaleIso by mutableStateOf<String?>(null)
expect object LocalAppLocaleIso {
    val current: String @Composable get
    @Composable infix fun provides(value: String?): ProvidedValue<*>
}

@Composable
fun AppEnvironment(content: @Composable () -> Unit) {
    CompositionLocalProvider(
        LocalAppLocaleIso provides customAppLocaleIso,
    ) {
        key(customAppLocaleIso) {
            content()
        }
    }
}
```

2. In the Android source set, add the `actual` implementation that uses `context.resources.configuration.locale`:

```kotlin
actual object LocalAppLocaleIso {
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
actual object LocalAppLocaleIso {
    private const val LANG_KEY = "AppleLanguages"
    private val default = NSLocale.preferredLanguages.first() as String
    private val LocalAppLocaleIso = staticCompositionLocalOf { default }
    actual val current: String
        @Composable get() = LocalAppLocaleIso.current

    @Composable
    actual infix fun provides(value: String?): ProvidedValue<*> {
        val new = value ?: default
        if (value == null) {
            NSUserDefaults.standardUserDefaults.removeObjectForKey(LANG_KEY)
        } else {
            NSUserDefaults.standardUserDefaults.setObject(arrayListOf(new), LANG_KEY)
        }
        return LocalAppLocaleIso.provides(new)
    }
}
```

4. In the desktop source set, add the `actual` implementation that uses `Locale.getDefault()` to update the JVM's default locale:

```kotlin
actual object LocalAppLocaleIso {
    private var default: Locale? = null
    private val LocalAppLocaleIso = staticCompositionLocalOf { Locale.getDefault().toString() }
    actual val current: String
        @Composable get() = LocalAppLocaleIso.current

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
        return LocalAppLocaleIso.provides(new.toString())
    }
}
```

5. For the web platform, workaround the `window.navigator.languages` property to introduce a custom locale logic:

```kotlin
external object window {
    var __hackedLocale: String?
}

actual object LocalAppLocaleIso {
    private val LocalAppLocaleIso = staticCompositionLocalOf { Locale.current }
    actual val current: String
        @Composable get() = LocalAppLocaleIso.current.toString()

    @Composable
    actual infix fun provides(value: String?): ProvidedValue<*> {
        window.__hackedLocale = value?.replace('_', '-')
        return LocalAppLocaleIso.provides(Locale.current)
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
                    if (window.__hackedLocale) {
                        return [window.__hackedLocale];
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

## Configure theme 

Compose Multiplatform defines the current theme via `isSystemInDarkTheme()`. 
Themes are handled differently across platforms:

* Android uses `Resources.getConfiguration().uiMode and Configuration.UI_MODE_NIGHT_MASK`.
* iOS, desktop, and web platforms use `LocalSystemTheme.current`.

The difference requires an `expect-actual` mechanism to handle platform-specific theme customization:

1. In the common code, define the `LocalAppTheme` interface with the `expect` keyword:

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

## Configure density

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

Learn how to [localize resources](compose-rtl.md).

[//]: # (todo: replace localization link)