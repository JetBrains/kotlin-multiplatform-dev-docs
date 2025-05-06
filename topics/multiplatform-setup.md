[//]: # (title: Set up an environment)

Before you create your first Kotlin Multiplatform application, you need to set up an environment for KMP development.

## Install the necessary tools

We recommend that you install the latest stable versions for compatibility and better performance.

<table>
   <tr>
      <td>Tool</td>
      <td>Comments</td>
   </tr>
    <tr>
        <td><a href="https://developer.android.com/studio">Android Studio</a></td>
        <td>You will use Android Studio to create your multiplatform applications and run them on simulated or hardware devices.</td>
    </tr>
    <tr>
        <td>
          <p><a href="https://apps.apple.com/us/app/xcode/id497799835">Xcode</a></p>
          <p>Xcode is required if you want to run iOS applications on a simulated or real device. If you use a different operating system, skip this tool.</p>
        </td>
        <td>
          <p>Launch Xcode in a separate window to accept its license terms and allow it to perform some necessary initial tasks.</p>
          <p>Most of the time, Xcode will work in the background. You will use it to add Swift or Objective-C code to your iOS application.</p>
            <note>
              <p>
                We generally recommend using the latest stable versions for all tools. However, Kotlin/Native sometimes doesn't support the newest Xcode right away. You can check supported versions in the <a href="https://kotlinlang.org/docs/multiplatform-compatibility-guide.html#version-compatibility">compatibility guide</a>, and if necessary, <a href="https://developer.apple.com/download/all/?q=Xcode">install an earlier version of Xcode</a>.
              </p>
            </note>   
      </td>
   </tr>
   <tr>
        <td><a href="https://www.oracle.com/java/technologies/javase-downloads.html">JDK</a></td>
        <td>To check whether Java is installed, run the following command in the Android Studio terminal or your command line: <code style="block"
            lang="bash">java -version</code></td>
   </tr>
   <tr>
        <td><a href="https://kotlinlang.org/docs/multiplatform-plugin-releases.html">Kotlin Multiplatform plugin</a></td>
        <td><p>In Android Studio, open <strong>Settings</strong> (<strong>Preferences</strong>) and find the <strong>Plugins</strong> page. Search the <strong>Marketplace</strong> tab for <i>Kotlin Multiplatform</i>, and then install it.</p>
</td>
   </tr>
   <tr>
        <td><a href="https://kotlinlang.org/docs/releases.html#update-to-a-new-release">Kotlin plugin</a></td>
        <td>
            <p>The Kotlin plugin is bundled and automatically updated with each Android Studio release.</p>
        </td>
   </tr>
</table>

## Check your environment

To make sure everything works as expected, install and run the KDoctor tool:

> KDoctor works on macOS only. If you use a different operating system, skip this step.
>
{style="note"}

1. In the Android Studio terminal or your command-line tool, run the following command to install the tool using Homebrew:

    ```bash
    brew install kdoctor
    ```

   If you don't have Homebrew yet, [install it](https://brew.sh/) or see the KDoctor [README](https://github.com/Kotlin/kdoctor#installation) for other ways to install it.
2. After the installation is completed, call KDoctor in the console: 

    ```bash
    kdoctor
    ```

3. If KDoctor diagnoses any problems while checking your environment, review the output for issues and possible solutions:

   * Fix any failed checks (`[x]`). You can find problem descriptions and potential solutions after the `*` symbol.
   * Check the warnings (`[!]`) and successful messages (`[v]`). They may contain useful notes and tips, as well.
   
   > You may ignore KDoctor's warnings regarding the CocoaPods installation. In your first project, you will use a
   > different iOS framework distribution option.
   >
   {style="tip"}

## Possible issues and solutions

<deflist collapsible="true">
   <def title="Kotlin and Android Studio">
      <list>
         <li>Make sure that you have Android Studio installed. You can get it from its <a href="https://developer.android.com/studio">official website.</a></li>
         <li>You may encounter the <code>Kotlin not configured</code> error. It's a known issue in Android Studio Giraffe 2022.3 that doesn't affect building and running projects. To avoid the error, click <strong>Ignore</strong> or upgrade to Android Studio Hedgehog 2023.1.</li>
      </list>
      <warning>   
        <p>
          Starting with Compose Multiplatform 1.8.0, the UI framework fully transitioned to the K2 compiler.
          So, to share UI code using the latest Compose Multiplatform you should use at least Kotlin 2.1.0 for your projects
          and depend on libraries compiled against at least Kotlin 2.1.0 as well.
        </p>
      </warning>
   </def>
   <def title="Java and JDK">
         <list>
           <li>Make sure that you have JDK installed. You can get it from its <a href="https://www.oracle.com/java/technologies/javase-downloads.html">official website</a>.</li>
           <li>Android Studio uses a bundled JDK to execute Gradle tasks. To configure the Gradle JDK in Android Studio, select <strong>Settings/Preferences | Build, Execution, Deployment | Build Tools | Gradle</strong>.</li>
           <li>You might encounter issues related to <code>JAVA_HOME</code>. This environment variable specifies the location of the Java binary required for Xcode and Gradle. If so, follow KDoctor's tips to fix the issues.</li>
         </list>
   </def>
   <def title="Xcode">
      <list>
         <li>Make sure that you have Xcode installed. You can get it from its <a href="https://developer.apple.com/xcode/">official website</a>.</li>
         <li>If you haven't launched Xcode yet, open it in a separate window. Accept the license terms and allow it to perform some necessary initial tasks.</li>
         <li><p>You may encounter <code>Error: can't grab Xcode schemes</code> or other issues regarding command line tools selection. In this case, do one of the following:</p>
             <list>
               <li><p>In Terminal, run:</p>
                   <code style="block"
                         lang="bash">sudo xcode-select --switch /Applications/Xcode.app</code>
               </li>
               <li>Alternatively, in Xcode, select <strong>Settings | Locations</strong>. In the <strong>Command Line Tools</strong> field, select your Xcode version.
                   <img src="xcode-schemes.png" alt="Xcode schemes" width="500"/>
                   <p>Make sure that the path to <code>Xcode.app</code> is selected. Confirm the action in a separate window if required.</p>
               </li>
             </list>
         </li>
      </list>
   </def>
   <def title="Kotlin plugins">
         <snippet>
            <p><strong>Kotlin Multiplatform plugin</strong></p>
               <list>
                  <li>Make sure that the Kotlin Multiplatform plugin is installed and enabled. On the Android Studio welcome screen, select <strong>Plugins | Installed</strong>. Verify that you have the plugin enabled. If it's not in the <strong>Installed</strong> list, search <strong>Marketplace</strong> for it and install the plugin.</li>
                  <li>If the plugin is outdated, click <strong>Update</strong> next to the plugin name. You can do the same in the <strong>Settings/Preferences | Tools | Plugins</strong> section.</li>
                  <li>Check the compatibility of the Kotlin Multiplatform plugin with your version of Kotlin in the <a href="https://kotlinlang.org/docs/multiplatform-plugin-releases.html#release-details">Release details</a> table.</li>
               </list>
         </snippet>
         <snippet>
            <p><strong>Kotlin plugin</strong></p>
            <p>Make sure that the Kotlin plugin is updated to the latest version. To do that, on the Android Studio welcome screen, select <strong>Plugins | Installed</strong>. Click <strong>Update</strong> next to Kotlin.</p>
         </snippet>
   </def>
   <def title="Command line">
            <p>Make sure you have all the necessary tools installed:</p>
            <list>
              <li><code>command not found: brew</code> — <a href="https://brew.sh/">install Homebrew</a>.</li>
              <li><code>command not found: java</code> — <a href="https://www.oracle.com/java/technologies/javase-downloads.html">install Java</a>.</li>
           </list>
    </def>
   <def title="Still having trouble?">
            <p>Share your problems with the team by <a href="https://kotl.in/issue">creating a YouTrack issue</a>.</p>
   </def>
</deflist>

## Get help

* **Kotlin Slack**. Get an [invite](https://surveys.jetbrains.com/s3/kotlin-slack-sign-up) and join the [#multiplatform](https://kotlinlang.slack.com/archives/C3PQML5NU) channel.
* **Kotlin issue tracker**. [Report a new issue](https://youtrack.jetbrains.com/newIssue?project=KT).
