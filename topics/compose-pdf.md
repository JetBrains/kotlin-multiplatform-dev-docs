[//]: # (title: Display PDF files in a Compose Multiplatform app)

You can use third-party libraries to display, edit, or generate PDFs in a Compose Multiplatform app.
Platform support and version requirements may change over time, so check each library’s documentation for 
supported platforms and required Kotlin and Compose Multiplatform versions.

Choose an approach based on what users need to do with PDFs in your app:

* To display existing PDFs, use a [PDF viewer](#add-a-pdf-viewer).
* To create and edit PDFs, use a [document engine](#display-a-pdf-with-a-document-engine).
* To generate PDFs from app data, use a [PDF generator](#generate-and-display-a-pdf).

For PDFs that users can open outside your Compose UI, [use a platform viewer](#use-a-platform-viewer).

## Display an existing PDF

To display an existing PDF, use a ready-made viewer or a document engine with a viewer component.

### Add a PDF viewer

[Nucleus PDF reader](https://nucleusframework.dev/en/docs/pdf/) uses PDFium on Android, iOS, desktop,
and web. It integrates through native binaries, or through a WebAssembly module running in a Web Worker on web.

To display a PDF that’s already loaded into memory, pass its bytes to the reader state and add `PdfReader()` to your layout:

```kotlin
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.runtime.Composable
import androidx.compose.runtime.LaunchedEffect
import androidx.compose.ui.Modifier
import dev.nucleusframework.pdfium.PdfReader
import dev.nucleusframework.pdfium.rememberPdfReaderState

@Composable
fun PdfScreen(bytes: ByteArray) {
    val reader = rememberPdfReaderState()

    LaunchedEffect(bytes) {
        reader.open(bytes)
    }

    PdfReader(
        state = reader,
        modifier = Modifier.fillMaxSize(),
    )
}
```

![PDF file in a Compose Multiplatform app](compose-pdf.png){width=300 style="block"}

For more information, see the [Nucleus PDF reader documentation](https://nucleusframework.dev/en/docs/pdf/).

### Display a PDF with a document engine

[KitePDF](https://github.com/yuroyami/KitePDF) provides a document engine and Compose viewer for reading, creating, and editing PDFs.
It supports Android, iOS, web, and desktop targets.

Open a PDF with `PdfDocument.open(bytes)`, then pass the document to the viewer:

```kotlin
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.runtime.Composable
import androidx.compose.runtime.remember
import androidx.compose.ui.Modifier
import io.github.yuroyami.kitepdf.PdfDocument
import io.github.yuroyami.kitepdf.compose.PdfView
import io.github.yuroyami.kitepdf.compose.rememberPdfViewState

@Composable
fun PdfScreen(bytes: ByteArray) {
    val document = remember(bytes) {
        PdfDocument.open(bytes)
    }

    PdfView(
        state = rememberPdfViewState(document),
        modifier = Modifier.fillMaxSize(),
    )
}
```

Available document inputs depend on the library version and target platform.
For more information, see the [KitePDF documentation](https://yuroyami.github.io/KitePDF/).

## Generate and display a PDF

[PdfKmp](https://github.com/ConaMobileDev/PdfKmp) can create PDFs from app data, such as recipes, invoices, or receipts.
It supports Android, iOS, desktop, and web via Kotlin/Wasm.

The optional `pdfkmp-viewer` module embeds a Compose viewer on Android, iOS, and desktop. 
This viewer module is currently not available on web. For web apps, see the browser [viewing and saving](https://conamobiledev.github.io/PdfKmp/guides/web/#viewing-saving) guidance.

The following example generates a PDF from app data and displays it with the Compose viewer:

```kotlin
import androidx.compose.runtime.Composable
import androidx.compose.runtime.remember
import com.conamobile.pdfkmp.pdf
import com.conamobile.pdfkmp.viewer.KmpPdfViewer

@Composable
fun RecipePdfScreen(
    recipeTitle: String,
    ingredients: String,
    instructions: String,
    onBack: () -> Unit,
) {
    val document = remember(recipeTitle, ingredients, instructions) {
        pdf {
            page {
                text(recipeTitle)
                text(ingredients)
                text(instructions)
            }
        }
    }

    KmpPdfViewer(
        document = document,
        title = recipeTitle,
        fileName = "recipe.pdf",
        onBack = onBack,
    )
}
```

For more information, see the [PdfKmp viewer documentation](https://conamobiledev.github.io/PdfKmp/guides/viewer/).

## Use a platform viewer

For PDFs that don’t require an embedded Compose viewer, use platform APIs to present, open, or share the document.

For example, a desktop app can open a saved PDF with `java.awt.Desktop.open()`.
On mobile, use platform APIs to present or share the document, such as PDFKit on iOS or a file-sharing intent on Android.
A web app can offer the PDF for the browser to open or download. 

For platform-specific examples, see [PdfKmp's save and share guide](https://conamobiledev.github.io/PdfKmp/getting-started/#save-share-per-platform).
These examples hand a document to platform APIs and do not require PdfKmp's Compose viewer.

## What's next

* Learn how to [add dependencies on multiplatform libraries](multiplatform-add-dependencies.md).
* Learn how to [work with multiplatform resources](compose-multiplatform-resources.md).