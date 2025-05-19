# Localizing strings

Localization is the process of adapting your app to different languages, regions, and cultural conventions. 
This guide explains how to set up translation directories, work with region-specific formats, 
handle right-to-left (RTL) languages, and test localization across platforms.

To localize strings in Compose Multiplatform, you need to provide translated texts for your application's user interface 
elements in all supported languages. Compose Multiplatform simplifies this process by offering a common resource 
management library and code generation for easy access to translations.

## Set up translation directories

Store all string resources in a dedicated `composeResources` directory within your common source set. 
Place the default texts in the `values` directory, and create corresponding directories for each language.
Use the following structure:

```
commonMain/composeResources/
├── values/
│   └── strings.xml
├── values-es/
│   └── strings.xml
├── values-fr/
│   └── strings.xml
└── ... (other locale directories)
```

Within the `values` directories and its localized variants, define string resources in `strings.xml` files using key-value pairs.
For example, add English texts to `commonMain/composeResources/values/strings.xml`:

```xml
<resources>
    <string name="app_name">My Application</string>
    <string name="greeting">Hello, world!</string>
    <string name="welcome_message">Welcome, %s!</string>
</resources>
```

Then, create corresponding localized files for translations. For example, add Spanish translations 
to `commonMain/composeResources/values-es/strings.xml`:

```xml
<resources>
    <string name="app_name">Mi Aplicación</string>
    <string name="greeting">¡Hola mundo!</string>
    <string name="welcome_message">¡Bienvenido, %s!</string>
</resources>
```

## Generate class for static access

Once you've added all translations, build the project to generate a special class that provides access to resources.
Compose Multiplatform processes the `strings.xml` resource files in `composeResources` and creates static accessor properties 
for each string resource.

The resulting `Res.strings` object allows you to safely access localized strings from your shared code.
To display strings in your app's UI, use the `stringResource()` composable function. 
This function retrieves the correct text based on the user's current locale:

```kotlin
import project.composeapp.generated.resources.Res

@Composable
fun MyApp() {
    Text(stringResource(Res.strings.app_name))
    Text(stringResource(Res.strings.greeting))
    Text(stringResource(Res.strings.welcome_message, "User"))
}
```

In the example above, the `welcome_message` string includes a placeholder (`%s`) to insert a dynamic value. 
Both the generated accessor and the `stringResource()` function support passing such parameters.

## What's next

* [Learn how to manage regional formats](compose-regional-format.md)
* [Read about handling Right-to-left languages](compose-rtl.md)