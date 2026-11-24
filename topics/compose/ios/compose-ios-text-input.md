[//]: # (title: iOS-specific text input options)
<primary-label ref="Experimental"/>

<web-summary>
    Learn how to configure iOS-specific text input options in Compose Multiplatform, including keyboard traits,
    native editing behavior, and password saving and autofill.
</web-summary>

Compose Multiplatform supports text input on all platforms through the shared `TextField` and `BasicTextField` APIs.
This page describes additional options available specifically for iOS. In your iOS source set, use `PlatformImeOptions`
alongside the common `KeyboardOptions` to configure the system keyboard and editing behavior.

Using `PlatformImeOptions` requires opting in with `@OptIn(ExperimentalComposeUiApi::class)`.

## Configure keyboard and input traits

`PlatformImeOptions` provides options for the keyboard type and appearance, return key behavior, autocapitalization,
autocorrection, and other native text input traits. Values specified in `PlatformImeOptions` override the corresponding
settings from the enclosing `ImeOptions`.

The following text field displays a keyboard layout optimized for entering email addresses:

```kotlin
val email = rememberTextFieldState()

BasicTextField(
    state = email,
    keyboardOptions = KeyboardOptions(
        platformImeOptions = PlatformImeOptions {
            keyboardType(UIKeyboardTypeEmailAddress)
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

## Save and autofill passwords

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
