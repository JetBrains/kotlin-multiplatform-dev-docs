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

* Deep links for Android apps are declared in the `AndroidManifest.xml` file.
* Deep links for iOS and macOS apps are declared in `Info.plist` files,
    in the [CFBundleURLTypes](https://developer.apple.com/documentation/bundleresources/information-property-list/cfbundleurltypes)
    key.

    > Compose Multiplatform [provides a Gradle DSL](compose-native-distribution.md#information-property-list-on-macos)
    > to add values to a macOS app's `Info.plist`.
    > For iOS, you can edit the file directly or [register schemes using the Xcode GUI](https://developer.apple.com/documentation/xcode/defining-a-custom-url-scheme-for-your-app#Register-your-URL-scheme).
    >
    {style="note"}
* Deep links for Windows apps