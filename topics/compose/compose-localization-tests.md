# Localization tests
<show-structure depth="2"/>

To test localization, verify that the correct translated strings are displayed for different locales,
and ensure that formatting and layout adapt to the locale’s requirements.

## Testing locales on different platforms

### Android

On Android, you can change the device's system locale via **Settings | System | Languages & input | Languages**.
For automated tests, you can modify the locale directly on the emulator using the `adb` shell:

```shell
adb -e shell
setprop persist.sys.locale [BCP-47 language tag];stop;sleep 5;start
```

This command restarts the emulator, allowing you to relaunch the app with the new locale.

Alternatively, you can configure the locale programmatically before running tests using frameworks like Espresso. 
For example, you can use `LocaleTestRule()` to automate locale switching during tests.

### iOS

On iOS, you can change the device's system language and region via **Settings | General | Language & Region**.
For automated UI tests using the XCUITest framework, use launch arguments to simulate locale changes:

```swift
app.launchArguments = [
    "-AppleLanguages", "(es)",
    "-AppleLocale", "es_ES"
]
```

### Desktop

On desktop, the JVM locale typically defaults to the operating system's locale. 
The settings location varies for different desktop platforms.

You can programmatically set the JVM default locale in your test setup or application entry point before the UI is initialized:

```java
java.util.Locale.setDefault(java.util.Locale("es_ES"))
``` 

### Web

For a quick check, you can change language settings in the browser preferences.
For automated testing, browser automation tools like Selenium or Puppeteer can simulate locale changes. 

Alternatively, you can try bypassing the read-only restriction of the `window.navigator.languages` property to introduce
a custom locale. Learn more in the [](compose-resource-environment.md) tutorial.

## Key testing scenarios

### Custom locale

* Override the locale programmatically.
* Assert that UI elements, formatted strings, and layouts adapt correctly for the selected locale, 
   including handling of right-to-left text if applicable.

### Default resources

Default resources are used when no translation is available for the specified locale. 
The application must correctly fall back to these defaults.

* Configure the locale to an unsupported value using platform-specific methods described above.
* Verify that fallback mechanisms load the default resources correctly and display them properly.

### Locale-specific cases

To avoid common localization issues, consider these locale-specific cases:

* Test [locale-specific formatting](compose-regional-format.md), such as date formatting (`MM/dd/yyyy` vs. `dd/MM/yyyy`) 
  and number formatting.
* Validate [RTL and LTR behavior](compose-rtl.md), ensuring that right-to-left languages like Arabic and Hebrew display strings, 
  layouts, and alignment properly.
