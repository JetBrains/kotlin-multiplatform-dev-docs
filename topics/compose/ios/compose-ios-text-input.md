[//]: # (title: iOS-specific text input options)
<primary-label ref="Experimental"/>

<web-summary>
    Learn how to configure iOS-specific text input options in Compose Multiplatform, including keyboard traits,
    native editing behavior, and password saving and autofill.
</web-summary>

Compose Multiplatform supports text input on all platforms through the shared `TextField` and `BasicTextField` APIs.
In shared (`commonMain`) code, use `KeyboardOptions` to configure multiplatform input behavior. In iOS-specific (`iosMain`)
code, add `PlatformImeOptions` to `KeyboardOptions` to configure native iOS traits or override common settings.
If both APIs configure the same trait, the value from `PlatformImeOptions` takes precedence on iOS.

Using `PlatformImeOptions` requires opting in with `@OptIn(ExperimentalComposeUiApi::class)`.

## Configure keyboard and input traits

`PlatformImeOptions` lets you configure native iOS text input traits, including keyboard type, autocapitalization,
autocorrection, and Return key behavior.

For example, the following option keeps the keyboard's Return key disabled until the user enters text:

```kotlin
val text = rememberTextFieldState()

BasicTextField(
    state = text,
    keyboardOptions = KeyboardOptions(
        platformImeOptions = PlatformImeOptions {
            enablesReturnKeyAutomatically(true)
        }
    )
)
```

You can also customize the input interface displayed when a field gains focus:

* `inputView()` replaces the default system keyboard with a custom input view.
* `inputAccessoryView()` attaches a custom accessory view to the system keyboard or a custom input view.

## Use native text input

The default Compose Multiplatform text input implementation provides consistent behavior across platforms.
On iOS, it supports native-like caret interaction: a single tap positions the caret, while a long press and drag moves it.

For editing behavior tailored specifically to iOS, you can opt in to fully native text input handling.
This implementation uses an iOS `UIView` to manage input through the `UITextInput` and `UIKeyInput` protocols.
It supports native caret movement, gestures, selection handling, and system context menus.

To enable native text input, use `usingNativeTextInput` in your iOS source set:

```kotlin
@OptIn(ExperimentalComposeUiApi::class)
@Composable
fun NativeTextInputField() {
    val text = rememberTextFieldState()

    BasicTextField(
        state = text,
        keyboardOptions = KeyboardOptions(
            platformImeOptions = PlatformImeOptions {
                usingNativeTextInput(true)
            }
        )
    )
}
```

Native text input is compatible with the context menu API enabled with the `isNewContextMenuEnabled` flag.

## Save and autofill login credentials

> Due to a known issue, this feature is only partially available in Compose Multiplatform 1.13.0.
> Full support is coming in Compose Multiplatform 1.13.1.
>
{style="note"}

Compose Multiplatform can show native prompts to save login credentials and autofill Compose text fields.
Keep both the username and password fields on screen so iOS can recognize the form as a login form.

<img src="compose-ios-autofill.png" alt="iOS AutoFill menu" width="400" border-effect="line"/>

A simple login form with `TextField()` and `SecureTextField()` can look like this:

```kotlin
val email = rememberTextFieldState()
val password = rememberTextFieldState()

Column {
    TextField(state = email)
    SecureTextField(state = password)
}
```

For `BasicTextField()`, set `keyboardType` to `KeyboardType.Password` or `KeyboardType.NumberPassword`:

```kotlin
val email = rememberTextFieldState()
val password = rememberTextFieldState()

Column {
    BasicTextField(
        state = email,
        keyboardOptions = KeyboardOptions(
            keyboardType = KeyboardType.Email
        )
    )

    BasicTextField(
        state = password,
        outputTransformation = remember {
            OutputTransformation { replace(0, length, "*".repeat(length)) }
        },
        keyboardOptions = KeyboardOptions(
            keyboardType = KeyboardType.Password
        )
    )
}
```
