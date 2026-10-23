[//]: # (title: Display PDF files in a Compose Multiplatform app)

To display PDF files in a Compose Multiplatform app, you can use a community library or a platform-native viewer. 
Choose an approach based on how users work with PDFs in your app:

* For document-centric apps, use a document engine. See [Display a PDF with a document engine](#display-a-pdf-with-a-document-engine).
* For PDFs that support another app feature, use a simple PDF viewer. See [Add a PDF viewer](#add-a-pdf-viewer).
* For basic open, print, download, or share actions, use the platform-native viewer.

If you also need to create PDFs, see [Generate and display a PDF](#generate-and-display-a-pdf).

## Display an existing PDF

To display existing PDFs, you can use libraries such as [Nucleus PDF reader](https://nucleusframework.dev/en/docs/pdf/) 
and [KitePDF](https://github.com/yuroyami/KitePDF).

Nucleus PDF Reader provides a ready-made PDF viewer component. 
KitePDF provides a document engine and a Compose viewer, and can create and edit PDF documents.

### Add a PDF viewer

To display a PDF file with [Nucleus PDF reader](https://nucleusframework.dev/en/docs/pdf/), add `PdfReader()` to your app layout:

```kotlin
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.runtime.Composable
import androidx.compose.runtime.LaunchedEffect
import androidx.compose.ui.Modifier
import dev.nucleusframework.pdfium.PdfReader
import dev.nucleusframework.pdfium.rememberPdfReaderState
import nucleuspdfdemo.sharedui.generated.resources.Res

@Composable
fun App() {
    val reader = rememberPdfReaderState()

    LaunchedEffect(Unit) {
        reader.open(Res.readBytes("files/kotlin-reference.pdf"))
    }

    PdfReader(
        state = reader,
        modifier = Modifier.fillMaxSize(),
    )
}
```

![PDF file in a Compose Multiplatform app](compose-pdf.png){width=300 style="block"}

You can also display a PDF from a byte array:

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

For more information, see the [Nucleus PDF reader documentation](https://nucleusframework.dev/en/docs/pdf/).

### Display a PDF with a document engine

[KitePDF](https://github.com/yuroyami/KitePDF) provides a document engine and Compose viewer for reading, creating, and editing PDFs.

Open a PDF with `PdfDocument.open(bytes)`, then pass the document to the viewer:

```kotlin
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.runtime.Composable
import androidx.compose.runtime.remember
import androidx.compose.ui.Modifier
import io.github.yuroyami.kitepdf.compose.KiteDocView
import io.github.yuroyami.kitepdf.compose.rememberKiteDocViewState
import io.github.yuroyami.kitepdf.pdf.PdfDocument

@Composable
fun PdfScreen(bytes: ByteArray) {
    val document = remember(bytes) {
        PdfDocument.open(bytes)
    }

    KiteDocView(
        state = rememberKiteDocViewState(document),
        modifier = Modifier.fillMaxSize(),
    )
}
```

KitePDF can also open documents from file paths, `File` objects, `InputStream` instances, 
Android content `Uri` values, Apple `NSData` or `NSURL` objects, and Base64 text or `data:` URIs.

For more information, see the [KitePDF documentation](https://yuroyami.github.io/KitePDF/).

## Generate and display a PDF

[PdfKmp](https://github.com/ConaMobileDev/PdfKmp) creates PDF documents and provides an optional Compose Multiplatform viewer module.

The following example creates a simple document and displays it:

```kotlin
import androidx.compose.runtime.Composable
import androidx.compose.runtime.remember
import com.conamobile.pdfkmp.pdf
import com.conamobile.pdfkmp.viewer.KmpPdfViewer

@Composable
fun PdfScreen(onBack: () -> Unit) {
    val document = remember {
        pdf {
            page {
                text("Hello, PDF!")
            }
        }
    }

    KmpPdfViewer(
        document = document,
        title = "Greeting",
        fileName = "greeting.pdf",
        onBack = onBack,
    )
}
```

PdfKmp is focused on generating documents such as reports, invoices, forms, and receipts. 
Its viewer can display the generated document, but it is not intended as a general-purpose PDF editor.

For more information, see the [PdfKmp viewer documentation](https://conamobiledev.github.io/PdfKmp/guides/viewer/).

## What's next

* Learn how to [add dependencies on multiplatform libraries](multiplatform-add-dependencies.md).
* Learn how to [work with multiplatform resources](compose-multiplatform-resources.md).