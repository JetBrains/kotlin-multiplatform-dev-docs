[//]: # (title: Kotlin Multiplatform samples)

This is a curated list of cross-platform projects created with Kotlin Multiplatform. Some share almost all their
functionality using Compose Multiplatform for the user interface. Others use native code for the user interface, and share
(for example) only the data model and algorithms. Please note that the best way to get started with a new Kotlin Multiplatform
application is by using the [web wizard](https://kmp.jetbrains.com/).

<table>
  <tr>
    <td>Name</td>
    <td>Description</td>
    <td>What's shared?</td>
    <td>Noteworthy libraries</td>
    <td>User interface</td>
  </tr>
  <tr>
    <td>
      <strong><a href="https://github.com/JetBrains/compose-multiplatform/tree/master/examples/imageviewer">Image Viewer</a></strong>
    </td>
    <td>An application for capturing, viewing, and storing pictures. Includes support for maps. Uses Compose Multiplatform for the UI. Introduced at <a href="https://www.youtube.com/watch?v=FWVi4aV36d8">KotlinConf 2023</a>.</td>
    <td>
      <list>
        <li>UI</li>
        <li>Model</li>
        <li>Networking</li>
        <li>Animation</li>
        <li>Data storage</li>
      </list>
    </td>
    <td>
        <list>
            <li><code>kotlinx-serialization</code></li>
            <li><code>kotlinx-datetime</code></li>
            <li><code>kotlinx-coroutines</code></li>
            <li><code>play-services-maps</code></li>
            <li><code>play-services-locations</code></li>
            <li><code>android-maps-compose</code></li>
            <li><code>accompanist-permissions</code></li>
        </list>
    </td>
    <td>
        <list>
            <li>Jetpack Compose on Android</li>
            <li>Compose Multiplatform on iOS and desktop</li>
        </list>
    </td>
  </tr>
  <tr>
    <td>
      <strong><a href="https://github.com/joreilly/Confetti">Confetti</a></strong>
    </td>
    <td>A showcase of many different aspects of Kotlin Multiplatform and Compose Multiplatform. The use case is an application for fetching and displaying information about conference schedules. Includes support for the Wear and Auto platforms. Uses GraphQL for client-server communications. The architecture is discussed in-depth at <a href="https://www.youtube.com/watch?v=uATlWUBSx8Q">KotlinConf 2023</a>.</td>
    <td>
      <list>
        <li>UI</li>
        <li>Model</li>
        <li>Networking</li>
        <li>Data storage</li>
        <li>Navigation</li>
      </list>
    </td>
    <td>
        <list>
  <li><code>kotlinx-serialization</code></li>
  <li><code>kotlinx-datetime</code></li>
  <li><code>kotlinx-coroutines</code></li>
  <li><code>decompose</code></li>
  <li><code>koin</code></li>
  <li><code>jsonpathkt-kotlinx</code></li>
  <li><code>horologist</code></li>
  <li><code>google-cloud</code></li>
  <li><code>firebase</code></li>
  <li><code>bare-graphql</code></li>
  <li><code>apollo</code></li>
  <li><code>accompanist</code></li>
</list>
    </td>
    <td>
        <list>
            <li>Jetpack Compose on Android, Auto, and Wear</li>
            <li>Compose Multiplatform on iOS and desktop</li>
        </list>
    </td>
  </tr>
<tr>
    <td>
      <strong><a href="https://github.com/xxfast/NYTimes-KMP">NYTimes KMP</a></strong>
    </td>
    <td>A Compose Multiplatform based version of the New York Times application. Allows the user to browse and read articles. Note that to build and run the application, you will need an <a href="https://developer.nytimes.com/">API key from the New York Times</a>.</td>
    <td>
      <list>
        <li>UI</li>
        <li>Model</li>
        <li>Networking</li>
      </list>
    </td>
    <td>
        <list>
  <li><code>kotlinx-serialization</code></li>
  <li><code>kotlinx-datetime</code></li>
  <li><code>kotlinx-coroutines</code></li>
  <li><code>ktor-client</code></li>
  <li><code>molecule</code></li>
  <li><code>decompose</code></li>
  <li><code>horologist</code></li>
</list>
    </td>
    <td>
        <list>
            <li>Jetpack Compose on Android and Wear</li>
            <li>Compose Multiplatform on iOS and desktop</li>
        </list>
    </td>
  </tr>
<tr>
    <td>
      <strong><a href="https://github.com/touchlab/DroidconKotlin">Sessionize / Droidcon</a></strong>
    </td>
    <td>An application for viewing the agenda at Droidcon events using the Sessionize API. Can be customized for use with any event that stores talks in Sessionize. Integrates with Firebase and so requires a Firebase account to run.</td>
    <td>
      <list>
        <li>UI</li>
        <li>Model</li>
        <li>Networking</li>
        <li>Data storage</li>
      </list>
    </td>
    <td>
        <list>
  <li><code>kotlinx-coroutines</code></li>
  <li><code>kotlinx-datetime</code></li>
  <li><code>ktor-client</code></li>
  <li><code>koin</code></li>
  <li><code>multiplatform-settings</code></li>
  <li><code>firebase</code></li>
  <li><code>kermit</code></li>
  <li><code>accompanist</code></li>
  <li><code>hyperdrive-multiplatformx</code></li>
  <li>SQLDelight</li>
</list>
    </td>
    <td>
        <list>
            <li>Jetpack Compose on Android</li>
            <li>Compose Multiplatform on iOS</li>
        </list>
    </td>
  </tr>
<tr>
    <td>
      <strong><a href="https://github.com/JetBrains/compose-multiplatform/tree/master/examples/chat">Chat</a></strong>
    </td>
    <td>A demonstration of how to embed Compose Multiplatform components within a SwiftUI interface. The use case is online messaging.</td>
    <td>
      <list>
        <li>UI</li>
        <li>Model</li>
        <li>Networking</li>
      </list>
    </td>
    <td>
    </td>
    <td>
        <list>
            <li>Jetpack Compose on Android</li>
            <li>Compose Multiplatform on iOS and desktop</li>
            <li>SwiftUI on iOS</li>
        </list>
    </td>
  </tr><tr>
    <td>
      <strong><a href="https://github.com/JetBrains/compose-multiplatform/tree/master/examples/imageviewer">KMM RSS Reader</a></strong>
    </td>
    <td>A sample application for consuming RSS feeds designed to show how Kotlin Multiplatform can be used in production. The UI is implemented natively, but there is an experimental branch showing how Compose Multiplatform could be used on iOS and desktop. Networking is accomplished using the <a href="https://ktor.io/docs/create-client.html">Ktor HTTP Client</a>, while XML parsing is implemented natively. The Redux architecture is used for sharing UI State.</td>
    <td>
      <list>
        <li>Model</li>
        <li>Networking</li>
        <li>UI state</li>
        <li>Data storage</li>
      </list>
    </td>
    <td>
        <list>
  <li><code>kotlinx-serialization</code></li>
  <li><code>kotlinx-coroutines</code></li>
  <li><code>ktor-client</code></li>
  <li><code>voyager</code></li>
  <li><code>coil</code></li>
  <li><code>multiplatform-settings</code></li>
  <li><code>napier</code></li>
  <li>SQLDelight</li>
</list>
    </td>
    <td>
        <list>
            <li>Jetpack Compose on Android</li>
            <li>Compose Multiplatform on iOS and desktop (on experimental branch)</li>
            <li>SwiftUI on iOS</li>
        </list>
    </td>
  </tr>
<tr>
    <td>
      <strong><a href="https://github.com/chrisbanes/tivi">Tivi</a></strong>
    </td>
    <td>An application for tracking television shows via <a href="https://trakt.tv/">Trakt.tv</a>. Uses Coroutines for concurrency and Compose for the UI. Designed to showcase the latest libraries and tools.</td>
    <td>
      <list>
        <li>UI</li>
        <li>Model</li>
        <li>Networking</li>
      </list>
    </td>
    <td>
        <list>
  <li><code>kotlinx.serialization</code></li>
  <li><code>kotlinx.coroutines</code></li>
  <li><code>ktor-client</code></li>
  <li><code>lyricist</code></li>
  <li><code>circuit</code></li>
  <li><code>firebase</code></li>
  <li>SQLDelight</li>
</list>
    </td>
    <td>
        <list>
            <li>Jetpack Compose on Android</li>
            <li>Compose Multiplatform on desktop</li>
        </list>
    </td>
  </tr>
<tr>
    <td>
      <strong><a href="https://github.com/joreilly/PeopleInSpace">People In Space</a></strong>
    </td>
    <td>A showcase of the many different platforms on which Kotlin Multiplatform can run. The use case is to show the number of people currently in space and the position of the International Space Station.</td>
    <td>
      <list>
        <li>Model</li>
        <li>Networking</li>
        <li>Data storage</li>
      </list>
    </td>
    <td>
        <list>
  <li><code>kotlinx-serialization</code></li>
  <li><code>kotlinx-coroutines</code></li>
  <li><code>kotlinx-datetime</code></li>
  <li><code>ktor-client</code></li>
  <li><code>koin</code></li>
  <li><code>multiplatform-settings</code></li>
  <li>SQLDelight</li>
</list>
    </td>
    <td>
        <list>
            <li>Jetpack Compose on Android</li>
            <li>Compose Multiplatform on iOS, desktop, and web</li>
            <li>SwiftUI on iOS, macOS, and watchOSS</li>
        </list>
    </td>
  </tr>
<tr>
    <td>
      <strong><a href="https://github.com/JoelKanyi/FocusBloom">Focus Bloom</a></strong>
    </td>
    <td>A productivity and time management application. Allows users to schedule tasks and provides feedback on their accomplishments.</td>
    <td>
      <list>
        <li>UI</li>
        <li>Model</li>
        <li>Animation</li>
        <li>Data storage</li>
      </list>
    </td>
    <td>
        <list>
  <li><code>kotlinx.serialization</code></li>
  <li><code>kotlinx.coroutines</code></li>
  <li><code>kotlinx.datetime</code></li>
  <li><code>koin</code></li>
  <li><code>voyager</code></li>
  <li><code>multiplatform-settings</code></li>
  <li>SQLDelight</li>
</list>
    </td>
    <td>
        <list>
            <li>Jetpack Compose on Android</li>
            <li>Compose Multiplatform on iOS and desktop</li>
        </list>
    </td>
  </tr>
<tr>
    <td>
      <strong><a href="https://github.com/SEAbdulbasit/recipe-app">Recipe App</a></strong>
    </td>
    <td>A demonstration application for viewing recipes. Showcases the use of animations.</td>
    <td>
      <list>
        <li>UI</li>
        <li>Model</li>
        <li>Data storage</li>
      </list>
    </td>
    <td>
        <p><code>kotlinx-coroutines</code></p>
    </td>
    <td>
        <list>
            <li>Jetpack Compose on Android</li>
            <li>Compose Multiplatform on iOS and desktop</li>
        </list>
    </td>
  </tr>
<tr>
    <td>
      <strong><a href="https://github.com/Kotlin/kmm-basic-sample">Kotlin Multiplatform Mobile Sample</a></strong>
    </td>
    <td>A simple calculator application. Showing how to integrate Kotlin and native code using expected and actual declarations.</td>
    <td><p>Algorithms</p>
    </td>
    <td>
    </td>
    <td>
        <list>
            <li>Android XML layouts</li>
            <li>SwiftUI</li>
        </list>
    </td>
  </tr>
<tr>
    <td>
      <strong><a href="https://github.com/touchlab/KaMPKit">KaMPKit</a></strong>
    </td>
    <td>A collection of code and tools for Kotlin Multiplatform development. Designed to showcase libraries, architectural choices, and best practices when building Kotlin Multiplatform applications. The use case is downloading and displaying information about dog breeds. Introduced in this <a href="https://www.youtube.com/watch?v=EJVq_QWaWXE">video tutorial</a>.</td>
    <td>
      <list>
        <li>Model</li>
        <li>Networking</li>
        <li>ViewModel</li>
        <li>Data storage</li>
      </list>
    </td>
    <td>
        <list>
  <li><code>ktor-client</code></li>
  <li><code>koin</code></li>
  <li><code>multiplatform-settings</code></li>
  <li><code>kermit</code></li>
  <li>SQLDelight</li>
</list>
    </td>
    <td>
        <list>
            <li>Jetpack Compose on Android</li>
            <li>SwiftUI on iOS</li>
        </list>
    </td>
  </tr>
<tr>
    <td>
      <strong><a href="https://github.com/dbaroncelli/D-KMP-sample">D-KMP-sample</a></strong>
    </td>
    <td>A sample application for the <a href="https://danielebaroncelli.medium.com/d-kmp-sample-now-leverages-ios-16-navigation-cebbb81ba2e7">Declarative UIs with Kotlin MultiPlatform architecture</a>. The use case is retrieving and displaying vaccination statistics for different countries.</td>
    <td>
      <list>
        <li>Networking</li>
        <li>Data storage</li>
        <li>ViewModel</li>
        <li>Navigation</li>
      </list>
    </td>
    <td>
        <list>
  <li><code>ktor-client</code></li>
  <li><code>multiplatform-settings</code></li>
  <li>SQLDelight</li>
</list>
    </td>
    <td>
        <list>
            <li>Jetpack Compose on Android</li>
            <li>SwiftUI on iOS</li>
        </list>
    </td>
  </tr>
<tr>
    <td>
      <strong><a href="https://github.com/realm/realm-kotlin-samples/tree/main/Bookshelf">Bookshelf</a></strong>
    </td>
    <td>A demonstration of how to use the Realm database in a Kotlin Multiplatform application.</td>
    <td>
      <list>
        <li>Model</li>
        <li>Networking</li>
        <li>Data storage</li>
      </list>
    </td>
    <td>
        <list>
  <li><code>ktor-client</code></li>
  <li><code>kotlinx.serialization</code></li>
  <li><code>realm-kotlin</code></li>
</list>
    </td>
    <td>
        <list>
            <li>Jetpack Compose on Android</li>
            <li>SwiftUI on iOS</li>
        </list>
    </td>
  </tr>

<tr>
    <td>
      <strong><a href="https://github.com/VictorKabata/Notflix">Notflix</a></strong>
    </td>
    <td>An application that consumes data from <a href="https://www.themoviedb.org/">The Movie Database</a> to display current trending, upcoming, and popular movies and TV shows. Requires that you create an API key with The Movie Database.</td>
    <td>
      <list>
        <li>Model</li>
        <li>Networking</li>
        <li>Caching</li>
        <li>ViewModel</li>
      </list>
    </td>
    <td>
        <list>
  <li><code>kotlinx-coroutines</code></li>
  <li><code>kotlinx-serialization</code></li>
  <li><code>kotlinx-datetime</code></li>
  <li><code>ktor-client</code></li>
  <li><code>multiplatform-settings</code></li>
  <li><code>napier</code></li>
</list>
    </td>
    <td>
        <list>
            <li>Jetpack Compose on Android</li>
            <li>SwiftUI on iOS</li>
        </list>
    </td>
  </tr>
</table>
