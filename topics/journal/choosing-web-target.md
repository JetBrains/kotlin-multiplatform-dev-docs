# Choose the right web target for Kotlin Multiplatform

Kotlin Multiplatform (KMP) offers two approaches for web development:

- JavaScript-based (using the Kotlin/JS compiler)

- WebAssembly-based (using the Kotlin/Wasm compiler)

Both options let you share code on the web. However, they differ in important ways, including performance, interoperability, application size, and target browser support. This guide explains when to use each target and how to align your requirements with the appropriate choice.

### Quick guide

The table below summarizes the recommended target based on your application's use case. You can read more about each target in the next sections.



| Use case                              | Recommended target | Reasoning                                                                                                                                                                                                                     |
| ------------------------------------- | ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Sharing business logic, but native UI | JS                 | Offers straightforward interop with JavaScript and minimal overhead.                                                                                                                                                          |
| Sharing both UI and business logic    | Wasm               | Provides faster performance for rendering with [Compose Multiplatform](https://www.jetbrains.com/compose-multiplatform/)                                                                                                      |
| Non-shareable UI                      | JS                 | Allows building UIs with HTML-based frameworks like [Kobweb](https://kobweb.varabyte.com/), [Kilua](https://kilua.dev/), or [React](https://kotlinlang.org/docs/js-react.html), leveraging existing JS ecosystems and tooling |

## When to choose Kotlin/JS

Kotlin/JS provides a great solution if your goal is to:

- [Share business logic with a JavaScript/TypeScript codebase](https://docs.google.com/document/d/1VvRvgpWXwkvvvVdv7sp0buAatz03tvBsrs527zCD2bs/edit?tab=t.0#heading=h.c1gw9tguf9wh)

- [Build non-shareable web apps with Kotlin](https://docs.google.com/document/d/1VvRvgpWXwkvvvVdv7sp0buAatz03tvBsrs527zCD2bs/edit?tab=t.0#heading=h.e2wu8c3bzssl)

### Share business logic with a JavaScript/TypeScript codebase

If you want to share a chunk of Kotlin code (like domain or data logic) with a native JavaScript/TypeScript-based application, JS target offers:

- Straightforward interop with JavaScript/TypeScript.

- Minimal overhead in interoperability (for example, no unnecessary data copying). This helps your code integrate seamlessly into your JS-based workflow.

  
### Build non-shareable web apps with Kotlin

For teams looking to build entire web apps using Kotlin, but without the intention to share it to other platforms (iOS, Android, or Desktop), an HTML-based solution would be a better choice. It provides SEO, accessibility, and seamless browser integration (such as “find on page” functionality or translating a page) by default. In this case, the Kotlin/JS shines and provides severalmultiple options. You can:

- Utilize Compose HTML-based frameworks, such as[ Kobweb](https://kobweb.varabyte.com/) or [Kilua](https://kilua.dev/), to build modern UIs in a familiar Compose Multiplatform architecture.

- Take advantage of React-based solutions with Kotlin wrappers to build [React components in Kotlin](https://kotlinlang.org/docs/js-react.html).

## When to choose Kotlin/Wasm

### Build cross-platform apps with Compose Multiplatform

If you want to share both logic and UI across multiple platforms — including the web — Kotlin/Wasm combined with [Compose Multiplatform](https://www.jetbrains.com/compose-multiplatform/) is the way to go:

- The UI experience is more consistent across platforms.

- You can take advantage of Wasm for improved rendering and smooth, responsive animations.

- Browser support for [WasmGC](https://developer.chrome.com/blog/wasmgc) has matured, allowing Kotlin/Wasm to run on all major modern browsers with near-native performance.

For projects with requirements for older browser version support, you can use a compatibility mode for Compose Multiplatform: build your UI in Wasm for modern browsers, but gracefully fall back to JS on older browsers. And you can share the common logic between both Wasm and JS targets in your project.

——————————————————————————————————————————

Still not sure which route to take? Join our[ Slack community](https://slack-chats.kotlinlang.org) and ask more about key differences, performance considerations, and best practices for choosing the right target.
