[//]: # (title: iOS migration guides)

This page walks you through iOS considerations for upgrading the Compose Multiplatform library in your project to a newer version,
starting with 1.7.0.

## Compose Multiplatform 1.6.11 to 1.7.0

### Removed background parameter in UIKitView and UIKitViewController

Deprecated `UIKitView` and `UIKitViewController` APIs have the `background` parameter, while the new ones don't.
The parameter was deemed redundant and removed:

* If you need to set the interop view background for new instances, you can do it using the `factory` parameter.
* If you need the background to be updatable, put the corresponding code in the `update` lambda.

### Touches or gestures may stop working as expected

The new default [touch behavior](compose-ios-touch.md) uses delays to determine if the touch is meant for the interop view
or for the Compose container of that view: the user has to hold still for at least 150 ms before the interop view will receive it.

If you need Compose Multiplatform to process touches like it used to, consider the new experimental `UIKitInteropProperties`
constructor.
It has the `interactionMode` parameter, which you can set to `UIKitInteropInteractionMode.NonCooperative` to make Compose
transfer touches directly to interop views.

The constructor is marked experimental because we ultimately intend to keep the interactability of interop views described
by a single bool flag.
The behavior described explicitly in the `interactionMode` parameter will most likely be derived automatically in the future.

### accessibilityEnabled replaced by isNativeAccessibilityEnabled, and turned off by default

The `accessibilityEnabled` parameter of the old `UIKitView` and `UIKitViewController` constructors was moved and renamed
to be available as the `UIKitInteropProperties.isNativeAccessibilityEnabled` property.
It is also set to `false` by default.

The `isNativeAccessibilityEnabled` property taints the merged Compose subtree with native accessibility resolution.
So it is not recommended to set to true unless you need rich accessibility capabilities of the interop view (such as web views).

For the rationale behind this property and its default value, see the [in-code documentation for the `UIKitInteropProperties` class](https://github.com/JetBrains/compose-multiplatform-core/blob/jb-main/compose/ui/ui/src/uikitMain/kotlin/androidx/compose/ui/viewinterop/UIKitInteropProperties.uikit.kt).

### onResize parameter removed

The `onResize` parameter of the old `UIKitView` and `UIKitViewController` constructors set a custom frame based
on the `rect` argument but didn't affect the Compose layout itself, so it was not intuitive to use.
On top of that, the default implementation of the `onResize` parameter needed to properly set the frame of interop view
and contained some implementation details about properly clipping the view. <!-- TODO: what's wrong with that exactly? -->

How to make do without `onResize`:

* If you need to react to the interop view frame change, you can:
    * override [`layoutSubviews`](https://developer.apple.com/documentation/uikit/uiview/1622482-layoutsubviews)
    of the interop `UIView`,
    * override [`viewDidLayoutSubviews`](https://developer.apple.com/documentation/uikit/uiviewcontroller/1621398-viewdidlayoutsubviews)
      of the interop `UIViewController`,
    * or add `onGloballyPositioned` to the `Modifier` chain.
* If you need to set the frame of your interop view, use the corresponding Compose modifiers: `size`, `fillMaxSize`, and so on.

### Some onReset usage patterns were invalidated

It is incorrect to use a non-null `onReset` lambda along with `remember { UIView() }`.

Consider the following code:

```kotlin
val view = remember { UIView() }

UIKitView(factory = { view }, onReset = { /* ... */ })
```

When a `UIKitView` enters the composition, either `factory` or `onReset` is called, never both.
So, if `onReset` is not null, the remembered `view` can be different from the one that is shown onscreen:
A composable can leave the composition and leave behind an instance of the view
that will be reused after resetting it in `onReset` instead of allocating a new one using `factory`.

To avoid such mistakes, don't specify an `onReset` value in the constructor.
You may need to perform callbacks from within the interop view based on the context in which the function emitting it entered the composition:
In this case, consider storing the callback inside the view using `update` on `onReset`.
