[//]: # (title: Sharing content in Compose Multiplatform)

In Compose Multiplatform apps, you can share text, URLs, files, and other content with the system or other apps. 

Sharing capabilities differ across platforms:

* Android and iOS provide native system share dialogs.
* Desktop applications generally do not have a universal system share dialog. Common alternatives include using the clipboard, opening files or URLs with the default application, or calling platform-specific APIs.
* Web applications can use the [Web Share API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Share_API), depending on browser support.

To share content in a Compose Multiplatform app, use [community libraries](#community-libraries) or call [platform-specific APIs](#platform-specific-sharing-apis) directly.
To open a link in the default application on any platform, use the [common `UriHandler` API](#related-features).

## Community libraries

Community libraries wrap native sharing APIs and provide a common API that you can call from shared code.

[KMP Sharing](https://klibs.io/project/software-mansion/kmp-sharing) provides a common API for the system share dialog on Android and iOS.
It supports sharing text or a URL, optionally with file URIs:

```kotlin
val share = rememberShare()
val sharingOptions = SharingOptions(androidDialogTitle = "Share with")

Button(onClick = { share("https://kotlinlang.org", sharingOptions) }) {
    Text("Share link")
}

Button(onClick = { share("Kotlin programming language", sharingOptions) }) {
    Text("Share text")
}
```

[FileKit](https://klibs.io/project/vinceglb/FileKit) provides cross-platform file picking, saving, and file operations with native dialogs.
You can use these libraries together. For example, pick files with FileKit and share them with KMP Sharing:

```kotlin
val share = rememberShare()
val sharingOptions = SharingOptions(androidDialogTitle = "Share with")

val photosPicker = rememberFilePickerLauncher(
    type = FileKitType.Image,
    mode = FileKitMode.Multiple(),
    onError = {},
    onResult = { photos ->
        photos?.takeIf { it.isNotEmpty() }?.let { selectedPhotos ->
            share(selectedPhotos.map(PlatformFile::sharingUri), sharingOptions)
        }
    },
)

Button(onClick = photosPicker::launch) {
    Text("Pick and share photos")
}
```

KMP Sharing expects file values as URIs. If you get a local path from FileKit, convert the `PlatformFile` path to a URI before sharing it:

```kotlin
private fun PlatformFile.sharingUri(): String = path.let { filePath ->
    when {
        filePath.startsWith("content://") || filePath.startsWith("file://") -> filePath
        else -> "file://$filePath"
    }
}
```

Browse other library options on [klibs.io](https://klibs.io/).
For use cases not covered by the libraries, you can implement sharing with
[platform-specific APIs](#platform-specific-sharing-apis).

## Platform-specific sharing APIs

You can implement sharing directly with the native APIs on each platform,
for example, to control content types, share options, or the sharing flow.

### Android

Use an `ACTION_SEND` intent to share text, a URL, or a file.
For text or URLs, pass the content with `Intent.EXTRA_TEXT`:

```kotlin
val intent = Intent(Intent.ACTION_SEND).apply {
    type = "text/plain"
    putExtra(Intent.EXTRA_TEXT, "https://kotlinlang.org")
}

context.startActivity(Intent.createChooser(intent, "Share with"))
```

To share a file, use a `content://` URI and grant temporary read permission:

```kotlin
val intent = Intent(Intent.ACTION_SEND).apply {
    type = "image/jpeg"
    putExtra(Intent.EXTRA_STREAM, contentUri)
    addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION)
}

context.startActivity(Intent.createChooser(intent, "Share with"))
```

For app-private files, expose the file through `FileProvider`, `MediaStore`, or another content provider.
For multiple files, use `ACTION_SEND_MULTIPLE` with `Intent.EXTRA_STREAM` and pass an `ArrayList<Uri>`:

```kotlin
val intent = Intent(Intent.ACTION_SEND_MULTIPLE).apply {
    type = "image/jpeg"
    putParcelableArrayListExtra(Intent.EXTRA_STREAM, ArrayList(contentUris))
    addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION)
}

context.startActivity(Intent.createChooser(intent, "Share with"))
```

Learn more in the Android Developers documentation: [Sharing data between apps](https://developer.android.com/develop/ui/compose/sharing).

### iOS

For UIKit-based UIs, present a `UIActivityViewController`:

```swift
let items: [Any] = [URL(string: "https://kotlinlang.org")!]

let controller = UIActivityViewController(
    activityItems: items,
    applicationActivities: nil
)

viewController.present(controller, animated: true)
```

In SwiftUI, you can use `ShareLink` for values supported by the `Transferable` protocol. 
For custom or large files, implement a `Transferable` representation or use `UIActivityViewController`
with a platform bridge.

Learn more in the Apple Developer documentation: [Collaborating and sharing copies of your data](https://developer.apple.com/documentation/uikit/collaborating-and-sharing-copies-of-your-data).

### Desktop

Desktop applications do not have a universal share dialog. Common approaches include:

* copying content to the clipboard,
* opening a file with the default application,
* using platform-specific APIs, such as native macOS sharing APIs.

For example, you can open a file with its default application using `java.awt.Desktop` when it is supported:

```kotlin
val file = java.io.File("/path/to/document.pdf")

if (java.awt.Desktop.isDesktopSupported()) {
    val desktop = java.awt.Desktop.getDesktop()
    if (desktop.isSupported(java.awt.Desktop.Action.OPEN)) {
        desktop.open(file)
    }
}
```

### Web

Web applications can use the Web Share API via JavaScript interop when supported by the browser and operating system. 
It usually requires HTTPS and a user gesture, such as a button click.

Check for support before calling the API and provide a fallback, for example, copying the content to the clipboard:

```javascript
if (navigator.share) {
    await navigator.share({
        title: "Kotlin",
        text: "Kotlin programming language",
        url: "https://kotlinlang.org"
    })
} else {
    await navigator.clipboard.writeText("https://kotlinlang.org")
}
```

For files, check support with `navigator.canShare()`:

```javascript
if (navigator.canShare && navigator.canShare({ files })) {
    await navigator.share({
        files,
        title: "Photos"
    })
}
```

Support for file sharing depends on the browser, platform, file types, and file sizes.
If files cannot be shared, common fallbacks include offering the file as a download
or copying a link to the content instead.

## Related features

* **Opening links and deep links.** Compose Multiplatform provides `UriHandler` as a common API for asking the platform
  to open a URI: `https://` links, custom application schemes, `mailto:`, `geo:`, and others.
  Get the handler with `LocalUriHandler.current` and call `openUri()` from an event handler:

  ```kotlin
  val uriHandler = LocalUriHandler.current

  Button(onClick = { uriHandler.openUri("mailto:user@example.com?subject=Hello") }) {
      Text("Send email")
  }
  ```
* **Drag and drop.** Compose Multiplatform provides a common API for sharing content by dragging it to a destination. See [Drag and drop](https://kotlinlang.org/docs/multiplatform/compose-drag-drop.html).
* **Clipboard.** A common Compose Multiplatform clipboard API is coming soon. Track progress in [CMP-7624](https://youtrack.jetbrains.com/issue/CMP-7624).