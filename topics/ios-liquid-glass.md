[//]: # (title: Liquid Glass in a Compose Multiplatform app)
<show-structure depth="1"/>

[Liquid Glass](https://developer.apple.com/documentation/TechnologyOverviews/liquid-glass) is Apple's visual design 
system introduced in iOS 26, bringing glass-like translucency and fluidity to UI elements.
To adopt it in a Compose Multiplatform app, you need a native SwiftUI shell, because 
Liquid Glass effects are rendered by the system through native `TabView`, `NavigationStack`, and toolbar APIs.

This tutorial walks you through **migrating an iOS app from fully Compose-driven navigation to native SwiftUI navigation** with
iOS 26 Liquid Glass styling, while keeping Compose in charge of rendering each screen's content.
The system applies Liquid Glass effects automatically when the app uses native `TabView` and `NavigationStack` views,
so you don't need to write any Liquid Glass-specific code.

We'll use the official [KotlinConf app](https://apps.apple.com/us/app/kotlinconf/id1299196584) as our example.

> You'll need Xcode 26 or later with the iOS 26 SDK.
>
{style="note"}

* [`main` branch](https://github.com/JetBrains/kotlinconf-app/tree/main) — starting state, with a custom theme fully implemented in Compose.
* [`lg-nav` branch](https://github.com/JetBrains/kotlinconf-app/tree/lg-nav) — final state, with the Liquid Glass design.

![Shared UI](ios-kotlinconf-no-liquid-glass.png){ width="250" style="inline"}
![Native iOS UI with Liquid Glass](ios-kotlinconf-liquid-glass.png){ width="250" style="inline"}

Clone the repo and check out either branch to follow along, or compare them side by side: 
[`main...lg-nav`](https://github.com/JetBrains/kotlinconf-app/compare/main...lg-nav). 

For simplicity, we'll migrate a two-tab version of the app (**Schedule** and **Info**), 
but the same pattern scales to any number of tabs.

## Migration plan

In a Compose Multiplatform setup with fully shared UI code, 
a single `ComposeUIViewController` is responsible for the entire iOS UI:
tabs, navigation stack, back gestures, and screen content. 
Compose Multiplatform's navigation transitions on iOS are designed to feel native,
but some platform-level features, such as iOS 26's Liquid Glass tab bar styling, 
are only available through native iOS components.

The solution is to hand navigation over to SwiftUI, letting the system render the tab bar and navigation stack natively 
while Compose continues to render each screen's content.

**Before:**

```
ContentView
  └── ComposeView (Compose Multiplatform)
```

**After:**

```
ContentView
  └── TabView  (Liquid Glass, iOS 26)
        ├── Tab: Schedule
        │     └── NavigationStack
        │           ├── NativeNavComposeView  ← Compose tab root
        │           └── DetailComposeView     ← Compose detail screen, one per destination
        └── Tab: Info
              └── NavigationStack
                    ├── NativeNavComposeView  ← Compose tab root
                    └── DetailComposeView     ← Compose detail screen, one per destination
```

Here's how navigation flows in the new setup:

* SwiftUI creates a `TabView` containing a `NavigationStack` for each tab.
* Compose still renders each screen's content but no longer manages the back stack.
* When the user triggers navigation from a Compose screen (for example, taps a list row), the event is forwarded to Swift via `onNavigate`.
* The Swift coordinator pushes the route into its `NavigationStack`, which creates a new `UIViewController` hosting a single Compose screen.

The migration touches both the shared Compose Multiplatform code and the native iOS code.
In the shared Kotlin code:

* [Add title metadata to routes](#add-title-metadata-to-routes) so SwiftUI can render navigation bar titles and back stack entries without calling back into Kotlin.
* [Add navigation callbacks to the iOS entry point](#add-navigation-callbacks-to-the-ios-entry-point) so the iOS layer can control which tab is active and respond to navigation events.
* [Intercept navigation at the Compose level](#intercept-navigation-at-the-compose-level) so detail routes are forwarded to Swift instead of being handled by Compose.
  This tutorial shows the Navigation 3 implementation — adapt this step if you use a different navigation library.
* [Build a standalone screen renderer for iOS](#build-a-standalone-screen-renderer-for-ios) so SwiftUI can render any detail route on its own outside the full `App()`.
* [Hide Compose's built-in navigation UI](#hide-compose-s-built-in-navigation-ui) when SwiftUI is in charge so users don't see duplicate title bars and back buttons.
* [Expose new iOS entry points](#expose-new-ios-entry-points) for creating the root view controller and individual screen view controllers.

In the native iOS code (Swift):

* [Build the SwiftUI navigation layer](#build-the-swiftui-navigation-layer) with native `TabView` and `NavigationStack` views, and the bridges that embed Compose screens.

## Add title metadata to routes

On iOS, each destination has a title that appears in the navigation bar, 
as well as in the back stack revealed by long-pressing the back button.
We'll store titles directly on the route objects, so each route is self-describing 
and Swift can read the title without round-trips to Kotlin.

1. In the `navigation/Routes.kt` file, add `title` and `subtitle` properties to `AppRoute`:

    ```kotlin
    @Serializable
    sealed interface AppRoute {
        val title: String? get() = null
        val subtitle: String? get() = null
    }
    ```

2. Override `title` (and `subtitle` where useful) on routes that appear as detail screens. 
   For routes that already carry data, add it as an optional parameter:

    ```kotlin
    @Serializable
    data class SessionScreen(
        val sessionId: SessionId,
        override val title: String? = null,
    ) : AppRoute
    ```

3. Routes that were `data object` also need a title, but a `data object` can't carry per-instance title state.
   Convert them to `data class`:

    <compare type="top-bottom">
        <code-block lang="kotlin">
            data object SettingsScreen : AppRoute
        </code-block>
        <code-block lang="kotlin">
            data class SettingsScreen(override val title: String = "") : AppRoute
        </code-block>
    </compare>

   For the full set of updated route definitions, see [`Routes.kt`](https://github.com/JetBrains/kotlinconf-app/blob/3982334f1c3712fb959f0d20b563d6c8b81e9bbd/app/shared/src/commonMain/kotlin/org/jetbrains/kotlinconf/navigation/Routes.kt).

4. Pass the localized title at the call site in the `NavHost.kt` file.
   Since `stringResource` is a `@Composable` function, resolve it inside the entry scope and capture it in the click callback,
   not inside the callback itself:

    ```kotlin
    entry<InfoScreen> {
        val settingsTitle = stringResource(Res.string.settings_title)
        InfoScreen(
            onSettings = { navigator.add(SettingsScreen(settingsTitle)) },
            // ...
        )
    }
    ```

## Add navigation callbacks to the iOS entry point

`App()` is the Kotlin entry point that iOS calls into. To let Swift drive navigation, 
it needs a way to do three things:

* Choose the starting tab when the app launches via a new `topLevelRoute` parameter.
* React to navigation pushes from Compose (for example, when a list item is tapped) via an `onNavigate` callback.
* React to tab switches initiated from Compose via an `onActivate` callback.

The new callbacks are optional and default to `null`, so Android, desktop, and web targets are unaffected.

In the `App.kt` file, update the signature of `App()` accordingly:

```kotlin
@Composable
fun App(
    appGraph: AppGraph,
    topLevelRoute: TopLevelRoute,
    onThemeChange: ((isDarkTheme: Boolean) -> Unit)? = null,
    onNavigate: ((AppRoute) -> Unit)? = null,
    onActivate: ((TopLevelRoute) -> Unit)? = null,
) {
    // ...
    val startRoute: AppRoute = remember {
        if (isOnboardingComplete) topLevelRoute else StartPrivacyNoticeScreen
    }
    NavHost(startRoute, isDarkTheme, onThemeChange, onNavigate, onActivate)
}
```

For the full implementation, see [`App.kt`](https://github.com/JetBrains/kotlinconf-app/blob/3982334f1c3712fb959f0d20b563d6c8b81e9bbd/app/shared/src/commonMain/kotlin/org/jetbrains/kotlinconf/App.kt).

## Intercept navigation at the Compose level

Now that `App()` exposes navigation callbacks, `NavHost` needs to use them. 
Whenever a detail route appears on Compose's back stack, hand it off to Swift and immediately remove 
it from Compose. This way, Compose renders detail screens only when invoked from Swift.

Two flows need to be set up:

* Detail pushes → Swift. Whenever a non-root route lands on the back stack, 
  forward it through `onNavigate` and remove it from Compose's back stack so SwiftUI's `NavigationStack` becomes the single source of truth.
* Tab switches → Swift. When the top-level route changes from inside Compose, notify Swift via `onActivate` so the SwiftUI `TabView` selection stays in sync.

This step is specific to the Navigation 3 library. 
The same interception pattern applies to any Compose navigation library, 
but the exact API (back stack access, current destination observation) will differ.

In `navigation/NavHost.kt`, add the new parameters and the two interception effects
to the `NavHost()` function:

```kotlin
import androidx.compose.runtime.snapshotFlow

@Composable
internal fun NavHost(
    startRoute: AppRoute,
    isDarkTheme: Boolean,
    onThemeChange: ((Boolean) -> Unit)?,
    onNavigate: ((AppRoute) -> Unit)? = null,
    onActivate: ((TopLevelRoute) -> Unit)? = null,
) {
    // Forwards detail routes to Swift and removes them from Compose's stack
    if (onNavigate != null) {
        LaunchedEffect(navState) {
            snapshotFlow { navState.currentBackstack.toList() }.collect { backstack ->
                val detailRoutes = backstack.drop(1)
                if (detailRoutes.isNotEmpty()) {
                    detailRoutes.forEach { onNavigate(it) }
                    navState.currentBackstack.removeRange(1, navState.currentBackstack.size)
                }
            }
        }
    }
    // Notifies Swift when the user switches tabs from within Compose
    if (onActivate != null) {
        LaunchedEffect(navState) {
            snapshotFlow { navState.topLevelRoute }.collect { route ->
                if (route != null) onActivate(route)
            }
        }
    }
    // ...
}
```

For the full file, see [`NavHost.kt`](https://github.com/JetBrains/kotlinconf-app/blob/3982334f1c3712fb959f0d20b563d6c8b81e9bbd/app/shared/src/commonMain/kotlin/org/jetbrains/kotlinconf/navigation/NavHost.kt).

## Build a standalone screen renderer for iOS

When SwiftUI owns the `NavigationStack`, Compose only needs to render the content of each screen.
`NavHost` is built for managing a back stack, transitions, and lifecycle, 
so we need a simpler entry point for rendering a single route.

### Add a flat screen renderer

`ScreenContent` is that simpler entry point: a flat `when` expression that maps a single detail route to its composable,
with no navigation state of its own. Tab roots are still handled by the full `App()` / `NavHost`.
SwiftUI creates a separate view controller for each destination, each hosting a single `ScreenContent` call.

Add the following to `navigation/NavHost.kt`:

```kotlin
@Composable
fun ScreenContent(
    route: AppRoute,
    onNavigate: (AppRoute) -> Unit,
    onBack: () -> Unit,
    onSet: (AppRoute) -> Unit,
    onActivate: (TopLevelRoute) -> Unit,
) {
    val uriHandler = LocalUriHandler.current
    when (route) {
        is SessionScreen -> SessionScreen(
            sessionId = route.sessionId,
            onBack = onBack,
            onSpeaker = { speakerId -> onNavigate(SpeakerDetailScreen(speakerId)) },
            // ...
        )
        is SpeakerDetailScreen -> SpeakerDetailScreen(
            speakerId = route.speakerId,
            onBack = onBack,
            onSession = { sessionId -> onNavigate(SessionScreen(sessionId)) },
        )
        is SettingsScreen -> SettingsScreen(onBack = onBack)
        is AboutAppScreen -> AboutAppScreen(
            onBack = onBack,
            onLicenses = { onNavigate(LicensesScreen) },
            // ...
        )
        // All other detail routes
        else -> {}
    }
}
```

Titles don't appear in this function: they were attached to the route objects back in the
[Add title metadata to routes](#add-title-metadata-to-routes) step,
so the Swift side can read them directly from each route when configuring its navigation bar.

### Signal to Compose that SwiftUI owns navigation

`ScreenContent` runs in a context where SwiftUI renders the navigation bar and back button. 
Compose screens that draw their own title bars or back buttons must skip them.

To avoid duplication inside the composition tree, use a `CompositionLocal`
that each screen can read without depending on iOS-specific code.

Declare `LocalUseNativeNavigation` as a `CompositionLocal` in the `NavHost.kt` file, 
before the `NavHost()` function:

```kotlin
val LocalUseNativeNavigation = staticCompositionLocalOf { false }
```

### Wrap the renderer for iOS

`ScreenContent` renders a route, but it needs a wrapper that sets the same theme, dependency injection,
and app-wide `CompositionLocal` values that `App()` usually sets up.

Add the `SingleScreenApp` wrapper. It mirrors the setup from `App()` and additionally sets 
`LocalUseNativeNavigation` to `true`, so each screen automatically hides its Compose-rendered title bar 
and back button.

In the `iosMain` source set, create the `SingleScreenApp.kt` file: 

```kotlin
@Composable
internal fun SingleScreenApp(
    appGraph: AppGraph,
    route: AppRoute,
    onNavigate: (AppRoute) -> Unit,
    onGoBack: () -> Unit,
    onSet: (AppRoute) -> Unit,
    onActivate: (TopLevelRoute) -> Unit,
) {
    // Sets theme and flags
    CompositionLocalProvider(
        LocalUseNativeNavigation provides true,
        LocalFlags provides flags,
        LocalAppGraph provides appGraph,
        // Other providers
    ) {
        KotlinConfTheme(colors = colors) {
            Box(Modifier.fillMaxSize().background(KotlinConfTheme.colors.mainBackground)) {
                ScreenContent(route, onNavigate, onGoBack, onSet, onActivate)
            }
        }
    }
}
```

### Apply the flag to tab roots

Tab roots still go through the regular `NavHost`, so they also need to honor the `LocalUseNativeNavigation` value.
Provide it based on whether native navigation callbacks are active.
When they are active, render the navigation content directly and skip `NavScaffold` (the Compose bottom bar):

```kotlin
val useNativeNavigation = onNavigate != null

CompositionLocalProvider(LocalUseNativeNavigation provides useNativeNavigation) {
    Box(
        // ...
    ) {
        val content = @Composable {
            NavDisplay(
                entries = navState.toDecoratedEntries(entryProvider),
                onBack = navigator::goBack,
            )
        }
        if (useNativeNavigation) {
            content()
        } else {
            NavScaffold(
                navState = navState,
                navigator = navigator,
                showGoldenKodee = showGoldenKodee,
                content = content,
            )
        }
    }
}
```

For full implementations, see [`NavHost.kt`](https://github.com/JetBrains/kotlinconf-app/blob/3982334f1c3712fb959f0d20b563d6c8b81e9bbd/app/shared/src/commonMain/kotlin/org/jetbrains/kotlinconf/navigation/NavHost.kt)
and [`SingleScreenApp.kt`](https://github.com/JetBrains/kotlinconf-app/blob/3982334f1c3712fb959f0d20b563d6c8b81e9bbd/app/shared/src/iosMain/kotlin/org/jetbrains/kotlinconf/SingleScreenApp.kt).

## Hide Compose's built-in navigation UI

With `LocalUseNativeNavigation` set wherever SwiftUI renders the navigation UI, 
individual screens now need to read it and hide their own title bars and back buttons. 
Otherwise, the user would see two title bars stacked on top of each other and two competing back buttons.

In `BaseScreens.kt`, update the `ScreenWithTitle()` function to read `LocalUseNativeNavigation` 
and skip the title bar and its divider when it is `true`:

```kotlin
val useNativeNavigation = LocalUseNativeNavigation.current

if (!useNativeNavigation) {
    MainHeaderTitleBar(...)
    HorizontalDivider(...)
}
```

Apply the same pattern to any other screens that draw their own back buttons or headers.

For the full implementation, see [`BaseScreens.kt`](https://github.com/JetBrains/kotlinconf-app/blob/3982334f1c3712fb959f0d20b563d6c8b81e9bbd/app/shared/src/commonMain/kotlin/org/jetbrains/kotlinconf/BaseScreens.kt).

## Expose new iOS entry points

To build the new navigation structure from SwiftUI, expose three Kotlin entry points:
two overloads of `MainViewController` and one `ScreenViewController`.
In `iosMain/main.ios.kt`, add the three functions:

* `MainViewController` without callbacks, used as a pre-iOS 26 fallback. Liquid Glass APIs require iOS 26, so SwiftUI should fall back
  to the original full-Compose setup on older versions. Without this overload, the `#available` branch in Swift won't compile.

    ```kotlin
    // Pre-iOS 26 fallback: full Compose navigation, no native callbacks
    @Suppress("unused")
    fun MainViewController(topLevelRoute: TopLevelRoute): UIViewController = ComposeUIViewController(
        configure = { onFocusBehavior = OnFocusBehavior.DoNothing },
    ) {
        App(appGraph, topLevelRoute)
    }
    ```

* `MainViewController` with callbacks, called by SwiftUI for each tab root. 
  Compose runs the full `App()` and `NavHost`, but navigation events are forwarded to SwiftUI instead of being handled internally.
  The signature includes `onGoBack` and `onSet` for API symmetry with `ScreenViewController`, although they aren't used in this overload.

    ```kotlin
    // Tab root: Compose runs NavHost but hands navigation events to SwiftUI
    @Suppress("unused")
    fun MainViewController(
        topLevelRoute: TopLevelRoute,
        onNavigate: (AppRoute) -> Unit,
        onGoBack: () -> Unit,
        onSet: (AppRoute) -> Unit,
        onActivate: (TopLevelRoute) -> Unit,
    ): UIViewController = ComposeUIViewController(
        configure = { onFocusBehavior = OnFocusBehavior.DoNothing }
    ) {
        App(appGraph, topLevelRoute, onNavigate = onNavigate, onActivate = onActivate)
    }
    ```
  
* `ScreenViewController`, called by SwiftUI for each detail screen. Renders a single route via `SingleScreenApp`, 
  which sets `LocalUseNativeNavigation` to `true` so Compose's built-in title bars and back buttons are hidden.

    ```kotlin
    // Detail screen: renders a single screen with LocalUseNativeNavigation = true
    @Suppress("unused")
    fun ScreenViewController(
        route: AppRoute,
        onNavigate: (AppRoute) -> Unit,
        onGoBack: () -> Unit,
        onSet: (AppRoute) -> Unit,
        onActivate: (TopLevelRoute) -> Unit,
    ): UIViewController = ComposeUIViewController(
        configure = { onFocusBehavior = OnFocusBehavior.DoNothing }
    ) {
        SingleScreenApp(appGraph, route, onNavigate, onGoBack, onSet, onActivate)
    }
    ```

For the full implementation, see [`main.ios.kt`](https://github.com/JetBrains/kotlinconf-app/blob/3982334f1c3712fb959f0d20b563d6c8b81e9bbd/app/shared/src/iosMain/kotlin/org/jetbrains/kotlinconf/main.ios.kt).

## Build the SwiftUI navigation layer

This is the iOS side of the migration. All the Kotlin changes from the previous steps prepare the app for what happens here: 
a SwiftUI `TabView` with per-tab `NavigationStack`s that host Compose views as their destinations.
To build that, complete the following:

1. [Make Kotlin routes usable in `NavigationStack`](#make-kotlin-routes-usable-in-navigationstack)
2. [Track tab and navigation state](#track-tab-and-navigation-state)
3. [Embed Compose screens as SwiftUI views](#embed-compose-screens-as-swiftui-views)
4. [Set up navigation within each tab](#set-up-navigation-within-each-tab)
5. [Build the tab bar](#build-the-tab-bar)
6. [Fall back on older iOS versions](#fall-back-on-older-ios-versions)

Note that none of the code in this section applies Liquid Glass effects directly. 
iOS 26 renders Liquid Glass automatically for native `TabView` and `NavigationStack` views,
so using them is enough to enable it.

### Make Kotlin routes usable in `NavigationStack`

`NavigationStack` requires its path elements to be `Hashable` and `Identifiable`.
To satisfy this for a Kotlin sealed interface, wrap `AppRoute` in a Swift `struct`.
Add the following to the `ContentView.swift` file:

```swift
@available(iOS 26.0, *)
struct RouteWrapper: Hashable, Identifiable {
    let id = UUID()
    let route: AppRoute

    static func ==(lhs: RouteWrapper, rhs: RouteWrapper) -> Bool {
        lhs.id == rhs.id
    }

    func hash(into hasher: inout Hasher) {
        hasher.combine(id)
    }
}
```

Pushing the same route twice must create two distinct stack entries, 
matching the expected navigation behavior. 
To achieve this, identity is based on a UUID rather than the route's value.

### Track tab and navigation state

Each tab has its own navigation stack, and the app tracks which tab is currently selected. 
Add two `@Observable` classes to handle this:

```swift
@available(iOS 26.0, *)
@Observable
class TabNavigationCoordinator {
    var path: [RouteWrapper] = []

    func push(_ route: AppRoute) {
        path.append(RouteWrapper(route: route))
    }

    func pop() {
        if !path.isEmpty {
            path.removeLast()
        }
    }

    func popToRoot() {
        path.removeAll()
    }
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="@Observable class TabNavigationCoordinator { "}

```swift
@available(iOS 26.0, *)
@Observable
class AppNavigationCoordinator {
    enum AppTab {
        case schedule, info
    }

    var selectedTab: AppTab = .schedule
    let scheduleCoordinator = TabNavigationCoordinator()
    let infoCoordinator = TabNavigationCoordinator()

    func activateTab(for route: TopLevelRoute) {
        if route is ScheduleScreen {
            selectedTab = .schedule
        } else if route is InfoScreen {
            selectedTab = .info
        }
    }
}
```
{initial-collapse-state="collapsed" collapsible="true" collapsed-title="@Observable class AppNavigationCoordinator { "}

`AppNavigationCoordinator` is simplified for the two-tab version used in this tutorial. 
See [`ContentView.swift`](https://github.com/JetBrains/kotlinconf-app/blob/b451d80301c50097d4cf5050d865829b49d07c8e/app/iosApp/iosApp/ContentView.swift) for the full version.

### Embed Compose screens as SwiftUI views

Two `UIViewControllerRepresentable` types connect the Kotlin entry points from
the [Expose new iOS entry points](#expose-new-ios-entry-points) step to SwiftUI: 
one for tab roots, one for detail screens. 

`NativeNavComposeView` hosts a tab root (Compose's `NavHost`) and forwards its navigation events:

```swift
@available(iOS 26.0, *)
struct NativeNavComposeView: UIViewControllerRepresentable {
    let topLevelRoute: TopLevelRoute
    let coordinator: TabNavigationCoordinator
    let appCoordinator: AppNavigationCoordinator

    func makeUIViewController(context: Context) -> UIViewController {
        return Main_iosKt.MainViewController(
            topLevelRoute: topLevelRoute,
            onNavigate: { route in self.coordinator.push(route) },
            onGoBack: { self.coordinator.pop() },
            onSet: { route in
                self.coordinator.popToRoot()
                if let topLevel = route as? TopLevelRoute {
                    self.appCoordinator.activateTab(for: topLevel)
                } else {
                    self.coordinator.push(route)
                }
            },
            onActivate: { route in self.appCoordinator.activateTab(for: route) }
        )
    }

    func updateUIViewController(_ uiViewController: UIViewController, context: Context) {
    }
}
```

`DetailComposeView` hosts a single detail screen, one instance for each `NavigationStack` destination:

```swift
@available(iOS 26.0, *)
struct DetailComposeView: UIViewControllerRepresentable {
    let route: AppRoute
    let coordinator: TabNavigationCoordinator
    let appCoordinator: AppNavigationCoordinator

    func makeUIViewController(context: Context) -> UIViewController {
        return Main_iosKt.ScreenViewController(
            route: route,
            onNavigate: { newRoute in self.coordinator.push(newRoute) },
            onGoBack: { self.coordinator.pop() },
            onSet: { route in
                self.coordinator.popToRoot()
                if let topLevel = route as? TopLevelRoute {
                    self.appCoordinator.activateTab(for: topLevel)
                } else {
                    self.coordinator.push(route)
                }
            },
            onActivate: { route in self.appCoordinator.activateTab(for: route) }
        )
    }

    func updateUIViewController(_ uiViewController: UIViewController, context: Context) {
    }
}
```

### Set up navigation within each tab

At the tab level, a `NavigationStack` uses the Compose tab content as its root and renders detail screens as destinations.

Note that `.navigationTitle(title)` must be set on the tab root even when `.navigationBarHidden(true)` is also applied. 
iOS 26 reads this value to label the tab in the floating tab bar, and if it's missing, the label will be blank.

```swift
@available(iOS 26.0, *)
struct TabContentView: View {
    let topLevelRoute: TopLevelRoute
    let coordinator: TabNavigationCoordinator
    let appCoordinator: AppNavigationCoordinator
    let title: String

    var body: some View {
        NavigationStack(path: Binding(
            get: { coordinator.path },
            set: { coordinator.path = $0 }
        )) {
            NativeNavComposeView(
                topLevelRoute: topLevelRoute,
                coordinator: coordinator,
                appCoordinator: appCoordinator
            )
                .ignoresSafeArea(.all)
                .navigationTitle(title)
                .navigationBarHidden(true)
                .navigationDestination(for: RouteWrapper.self) { wrapper in
                    DetailComposeView(
                        route: wrapper.route,
                        coordinator: coordinator,
                        appCoordinator: appCoordinator
                    )
                        .ignoresSafeArea(.all)
                        .navigationTitle(wrapper.route.title ?? "")
                        .navigationSubtitle(wrapper.route.subtitle ?? "")
                        .toolbarTitleDisplayMode(.inline)
                }
        }
    }
}
```

### Build the tab bar

The top-level container is a `TabView` with one `Tab` for each top-level route.
The `.tabBarMinimizeBehavior(.automatic)` modifier makes the tab bar float and minimize on scroll. 
Without it, the tab bar stays fixed at the bottom.
The `.tint(Color(.accent))` modifier applies the app's accent color to selected tabs.

```swift
@available(iOS 26.0, *)
struct NativeNavContentView: View {
    @State private var appCoordinator = AppNavigationCoordinator()

    var body: some View {
        TabView(selection: Binding(
            get: { appCoordinator.selectedTab },
            set: { appCoordinator.selectedTab = $0 }
        )) {
            Tab(String(localized: "Schedule"), systemImage: "clock",
                value: AppNavigationCoordinator.AppTab.schedule) {
                TabContentView(topLevelRoute: ScheduleScreen(),
                               coordinator: appCoordinator.scheduleCoordinator,
                               appCoordinator: appCoordinator, title: String(localized: "Schedule"))
            }
            Tab(String(localized: "Info"), systemImage: "info.circle",
                value: AppNavigationCoordinator.AppTab.info) {
                TabContentView(topLevelRoute: InfoScreen(),
                               coordinator: appCoordinator.infoCoordinator,
                               appCoordinator: appCoordinator, title: String(localized: "Info"))
            }
        }
        .tabBarMinimizeBehavior(.automatic) 
        .tint(Color(.accent))
    }
}
```

`Color(.accent)` resolves to the `AccentColor` asset in your Xcode project's asset catalog.
You can define it either through Xcode's asset catalog editor
(see [Specifying your app's color scheme](https://developer.apple.com/documentation/xcode/specifying-your-apps-color-scheme))
or by creating `Assets.xcassets/AccentColor.colorset/Contents.json`. For the JSON option,
you can use [`Contents.json`](https://github.com/JetBrains/kotlinconf-app/blob/3982334f1c3712fb959f0d20b563d6c8b81e9bbd/app/iosApp/iosApp/Assets.xcassets/AccentColor.colorset/Contents.json)
from the sample project as a starting point and replace the component values with your own colors.

With two tabs, the app renders as follows:

![Two tabs](ios-kotlinconf-two-tabs.png){ width="250" style="block"}

The translucency, depth, and floating tab bar are all applied by iOS 26 — no additional styling code is needed.

### Fall back on older iOS versions

Liquid Glass and the new `TabView` APIs are iOS 26 only. 
On older versions, the app falls back to the previous Compose-driven setup.
`ComposeView` is the SwiftUI wrapper around the no-callback `MainViewController` overload:

```swift
struct ContentView: View {
    var body: some View {
        if #available(iOS 26.0, *) {
            NativeNavContentView()
        } else {
            ComposeView(topLevelRoute: ScheduleScreen())
                .ignoresSafeArea(.all)
        }
    }
}
```

See the complete file: [`ContentView.swift`](https://github.com/JetBrains/kotlinconf-app/blob/3982334f1c3712fb959f0d20b563d6c8b81e9bbd/app/iosApp/iosApp/ContentView.swift).

## Alternative approaches

The migration in this tutorial favors native SwiftUI navigation, 
which gives you Liquid Glass and other system behaviors out of the box. 
If this approach doesn't fit your project, consider one of these alternatives:

* **Compose-driven navigation with native interop controls**. Keep navigation in Compose, but embed native UI controls 
  such as `UITabBar` and `UINavigationBar`, including Liquid Glass styling. The trade-off is some interop limitations 
  between native overlays and Compose content.
* **Compose-only navigation with imitated Liquid Glass effects**. Render everything in Compose and approximate Liquid Glass 
  visually, for example, with libraries like [AndroidLiquidGlass](https://klibs.io/project/Kyant0/AndroidLiquidGlass),
  [Calf](https://klibs.io/project/MohamedRejeb/Calf), or [Liquid](https://klibs.io/project/FletchMcKee/liquid).
  This approach keeps all UI on the Compose side, with the effect visually similar although not identical to system Liquid Glass.

## What's next

* Check out the [Official KotlinConf application](https://github.com/JetBrains/kotlinconf-app/tree/lg-nav) with the Liquid Glass effect applied.
* See [Adopting Liquid Glass](https://developer.apple.com/documentation/TechnologyOverviews/adopting-liquid-glass), Apple's overview of the new material and adoption checklist.
* Refer to [Integration with the SwiftUI framework](compose-swiftui-integration.md) for the official guidance on using Compose Multiplatform inside 
SwiftUI and embedding SwiftUI inside a Compose Multiplatform app.