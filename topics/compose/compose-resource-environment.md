[//]: # (title: Manage local resource environment)
<web-summary>Learn how to change the locale, theme, and screen density used for resources at runtime in a 
Compose Multiplatform app.</web-summary>

<show-structure depth="2"/>

You may need to manage in-app settings that allow users to customize their experience, such as changing the language or theme.
To dynamically update the application's resource environment, you can configure the following resource-related 
settings used by the application:

* [Locale (language and region)](#locale)
* [Theme](#theme)
* [Resolution density](#density)

Each setting in the examples below is controlled by a variable, such as `customAppLocale`, and applied by an `AppEnvironment()` composable.
If you need several settings, [combine them in a single `AppEnvironment()` composable](#combine-the-settings).

## Locale

Each platform handles locale settings such as language and region differently. As a temporary workaround, until a common 
public API is implemented, you need to define a common entry point in the shared code. Then, 
provide the corresponding declarations for each platform using the platform-specific API:

* **Android**: [`context.resources.configuration.locale`](https://developer.android.com/reference/android/content/res/Configuration#setLocale(java.util.Locale))
* **iOS**: [`NSLocale.preferredLanguages`](https://developer.apple.com/documentation/foundation/nslocale/preferredlanguages)
* **desktop**: [`Locale.getDefault()`](https://docs.oracle.com/en/java/javase/21/docs/api/java.base/java/util/Locale.html#getDefault())
* **web**: [`window.navigator.languages`](https://developer.mozilla.org/en-US/docs/Web/API/Navigator/languages)

1. In the common source set, define the expected `LocalAppLocale` object with the `expect` keyword.
   Locales are specified as BCP 47 language tags, such as `es`, `es-ES`, or `zh-Hans`. 
   Set `customAppLocale` to `null` to use the system locale:

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

   The platform implementations of `provides()` change the platform locale settings, which Compose can't observe.
   The `key(customAppLocale)` call recreates the content when the locale changes, so the resources are loaded again
   with the new locale.

   > Expected and actual classes are in Beta, so the compiler reports a warning when using them.
   > To suppress it, add the `-Xexpect-actual-classes` compiler option, as described in 
   > [Expected and actual classes](multiplatform-expect-actual.md#expected-and-actual-classes).
   >
   {style="note"}

2. In the Android source set, add the `actual` implementation that uses `context.resources.configuration.locale`:

    ```kotlin
    import androidx.compose.runtime.Composable
    import androidx.compose.runtime.ProvidedValue
    import androidx.compose.ui.platform.LocalConfiguration
    import androidx.compose.ui.platform.LocalContext
    import java.util.Locale

    actual object LocalAppLocale {
        private var default: Locale? = null
        actual val current: String
            @Composable get() = LocalConfiguration.current.locales[0].toLanguageTag()
    
        @Composable
        actual infix fun provides(value: String?): ProvidedValue<*> {
            val configuration = LocalConfiguration.current
    
            if (default == null) {
                default = configuration.locales[0]
            }
    
            val new = when(value) {
                null -> default!!
                else -> Locale.forLanguageTag(value.replace('_', '-'))
            }
            Locale.setDefault(new)
            configuration.setLocale(new)
            val resources = LocalContext.current.resources
    
            resources.updateConfiguration(configuration, resources.displayMetrics)
            return LocalConfiguration.provides(configuration)
        }
    }
    ```
   
   > `Resources.updateConfiguration()` is deprecated, so this code produces a deprecation warning.
   > Its replacement, `Context.createConfigurationContext()`, returns a new `Context` instead of changing the current one.
   > If you provide the new context through `LocalContext`, resources switch the language, but `LocalConfiguration` 
   > keeps the system locale. As a result, `LocalAppLocale.current` returns the system locale instead of the selected one.
   > For production apps, consider the Android [per-app language preferences](https://developer.android.com/guide/topics/resources/app-languages) API.
   >
   {style="note"}

3. In the iOS source set, add the `actual` implementation that modifies `NSLocale.preferredLanguages`:
 
    ```kotlin
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
        private val LocalAppLocale = staticCompositionLocalOf { Locale.getDefault().toLanguageTag() }
        actual val current: String
            @Composable get() = LocalAppLocale.current
    
        @Composable
        actual infix fun provides(value: String?): ProvidedValue<*> {
            if (default == null) {
                default = Locale.getDefault()
            }
            val new = when(value) {
                null -> default!!
                else -> Locale.forLanguageTag(value.replace('_', '-'))
            }
            Locale.setDefault(new)
            return LocalAppLocale.provides(new.toLanguageTag())
        }
    }
    ```

5. For the web platform, bypass the read-only restriction of the `window.navigator.languages` property to introduce 
   a custom locale logic. Add the following implementation to the `webMain`/`wasmJsMain` source set:

    ```kotlin
    actual object LocalAppLocale {
        private val LocalAppLocale = staticCompositionLocalOf { Locale.current }
        actual val current: String
            @Composable get() = LocalAppLocale.current.toString()
    
        @Composable
        actual infix fun provides(value: String?): ProvidedValue<*> {
            updateCustomLocale(value?.replace('_', '-'))
            return LocalAppLocale.provides(Locale.current)
        }
    }
    
    @OptIn(ExperimentalWasmJsInterop::class)
    private fun updateCustomLocale(value: String?) {
        js(
            """
            if (window.__customLocale !== value) {
                window.__customLocale = value;
                window.dispatchEvent(new Event("languagechange"));
            }
            """
        )
    }
    ```

   Compose caches the browser locale and reads it again only when the `languagechange` event is dispatched.
   Without the event, the custom locale is ignored.

   > Starting with Compose Multiplatform 1.13.0, setting `window.__customLocale` without dispatching the `languagechange` event
   > no longer changes the locale. If you used an earlier version of this workaround, update it as shown above.
   >
   {style="warning"}

   In your browser's `index.html`, put the following code before loading the application scripts, so that the 
   application reads the overridden `navigator.languages` property:

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
            ...
        </head>
        <body>
            <script src="webApp.js"></script>
        </body>
    </html>
    ```

To change the locale, assign a language tag to the `customAppLocale` variable, for example, from a settings screen:

```kotlin
Button(onClick = { customAppLocale = "de" }) {
    Text("Deutsch")
}
```

## Theme 

Compose Multiplatform defines the current theme via `isSystemInDarkTheme()`. 
Themes are handled differently across platforms:

* Android defines the theme via the following bitwise operation: 
  `Resources.getConfiguration().uiMode and Configuration.UI_MODE_NIGHT_MASK`.
* iOS, desktop, and web platforms don't provide a public API for overriding the theme. 
  The workaround below relies on the internal `LocalSystemTheme` property.

As a temporary workaround, until a common public API is implemented, 
you can address this difference using the `expect`/`actual` mechanism to manage platform-specific theme customization:

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
    import android.content.res.Configuration
    import android.content.res.Configuration.UI_MODE_NIGHT_MASK
    import android.content.res.Configuration.UI_MODE_NIGHT_NO
    import android.content.res.Configuration.UI_MODE_NIGHT_YES
    import androidx.compose.runtime.Composable
    import androidx.compose.runtime.ProvidedValue
    import androidx.compose.ui.platform.LocalConfiguration

    actual object LocalAppTheme {
        actual val current: Boolean
            @Composable get() = (LocalConfiguration.current.uiMode and UI_MODE_NIGHT_MASK) == UI_MODE_NIGHT_YES
    
        @Composable
        actual infix fun provides(value: Boolean?): ProvidedValue<*> {
            val new = if (value == null) {
                LocalConfiguration.current
            } else {
                Configuration(LocalConfiguration.current).apply {
                    uiMode = when (value) {
                        true -> (uiMode and UI_MODE_NIGHT_MASK.inv()) or UI_MODE_NIGHT_YES
                        false -> (uiMode and UI_MODE_NIGHT_MASK.inv()) or UI_MODE_NIGHT_NO
                    }
                }
            }
            return LocalConfiguration.provides(new)
        }
    }
    ```

3. On iOS, desktop, and web platforms, you can change `LocalSystemTheme` directly:

    ```kotlin
    import androidx.compose.ui.InternalComposeUiApi
    import androidx.compose.ui.LocalSystemTheme
    import androidx.compose.ui.SystemTheme

    @OptIn(InternalComposeUiApi::class)
    actual object LocalAppTheme {
        actual val current: Boolean
            @Composable get() = LocalSystemTheme.current == SystemTheme.Dark
    
        @Composable
        actual infix fun provides(value: Boolean?): ProvidedValue<*> {
            val new = when(value) {
                true -> SystemTheme.Dark
                false -> SystemTheme.Light
                null -> LocalSystemTheme.current
            }
    
            return LocalSystemTheme.provides(new)
        }
    }
    ```

## Density

To change the application's resolution `Density`, you can use the common `LocalDensity` API, supported on all platforms:

```kotlin
var customAppDensity by mutableStateOf<Density?>(null)

@Composable
fun AppEnvironment(content: @Composable () -> Unit) {
    CompositionLocalProvider(
        LocalDensity provides (customAppDensity ?: LocalDensity.current),
    ) {
        content()
    }
}
```

When `customAppDensity` is `null`, `AppEnvironment()` passes on the `LocalDensity` value that it receives.
This is the system density, unless you override `LocalDensity` higher in the composition.

Compose observes changes of `LocalDensity`, so unlike locale and theme, the content doesn't need to be wrapped in `key()`.

## Combine the settings

Each of the sections above defines its own `AppEnvironment()` composable. If you need several settings in one project,
don't copy all of these definitions, as they conflict with each other.
Instead, define a single `AppEnvironment()` composable that provides all the values:

```kotlin
@Composable
fun AppEnvironment(content: @Composable () -> Unit) {
    CompositionLocalProvider(
        LocalAppLocale provides customAppLocale,
        LocalAppTheme provides customAppThemeIsDark,
        LocalDensity provides (customAppDensity ?: LocalDensity.current),
    ) {
        key(customAppLocale, customAppThemeIsDark) {
            content()
        }
    }
}
```

Wrap the content of your app in `AppEnvironment()` to apply the settings:

```kotlin
@Composable
fun App() {
    AppEnvironment {
        // Your app's content
    }
}
```

## What's next?

* Get more details on [resource qualifiers](compose-multiplatform-resources-setup.md#qualifiers).
* Learn how to [localize resources](compose-localize-strings.md).
