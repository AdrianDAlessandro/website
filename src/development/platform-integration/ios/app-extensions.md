---
title: Adding iOS app extensions
description: Learn how to add app extensions to your Flutter apps
---

iOS App extensions allow you to expand functionality
outside your app. Your app could appear as a home screen widget,
or you can make portions of your app available within other apps.

To learn more about app extensions, check out
[Apple's documentation][].

## How do you add an app extension to your Flutter app?
To add an app extension to your Flutter app,
add the extension point *target* to your Xcode project.

1. Open the default Xcode workspace in your project by running
   `open ios/Runner.xcworkspace` in a terminal window from your
   Flutter project directory.
1. In Xcode, select **File -> New -> Target** from the menu bar. 

    <figure class="site-figure {{include.class}}">
    <div class="site-figure-container">
        <img src='/assets/images/docs/development/platform-integration/app-extensions/xcode-new-target.png'
        height='300'>
    </div>
    </figure>
1. Select the app extension you intend to add.
   This selection generates extension-specific code 
   within a new folder in your project.
   To learn more about the generated code and the SDKs for each
   extension point, check out the resources in
   [Apple's documentation][].

## How do Flutter apps interact with App Extensions? 
Flutter apps interact with app extensions using the same
techniques as UIKit or SwiftUI apps.
The containing app and the app extension don't communicate directly.
The containing app may also not be running while the extension 
is being used.
Instead, the app and the extension read and write to shared resources or
use higher-level APIs to communicate with each other.

### Using higher-level APIs
Some extensions have APIs. For example, 
the [Core Spotlight][] framework indexes your app 
allowing users to search from Spotlight and Safari. The
[WidgetKit][] framework can trigger an update of your home screen
widget.

Flutter plugins make it easier for your app to communicate 
with extensions by wrapping these APIs. 
To find plugins that wrap extension APIs,
check out [Leveraging Apple's System APIs and Frameworks][] or
search [pub.dev][].

### Sharing resources
To share resources between your Flutter app and your app extension, put
the `Runner` app target and the extension target in the same
[App Group][].

To add a target to an App Group:

1. Open the target settings in Xcode.
1. Navigate to the **Signing & Capabilities** tab.
1. Select **+ Capability** then **App Groups**.
1. Select or create the App Group.

{{site.alert.note}}
  You must be signed in to your Apple Developer account.
{{site.alert.end}}

{% include docs/app-figure.md
image="development/platform-integration/app-extensions/xcode-app-groups.png" %}

When two targets belong to the same App Group, they can read and write
data to the same container using one of the following
options:

- **Key/value:** Use the [`shared_preference_app_group`][]
  plugin to read or write to `UserDefaults` within the same App Group.
- **File:** Use the App Group container path from the
  [`path_provider`][] plugin to [read and write files][].
- **Database:** Use the App Group container path from
  the [`path_provider`][] plugin to create a database with the
  [`sqflite`][] plugin.

### Background updates
Background tasks provide a means to update your extension through code
regardless of the status of your app.

To schedule background work from your Flutter app, use the
[`workmanager`][] plugin.

### Deep linking
You might want to direct users from an app extension to a
specific page in your Flutter app.
To have a URL open a specified route in your app, you can use
[Deep Linking][].

## Creating app extension UIs with Flutter
Some app extensions display a user interface.
For example, iMessage extensions allow users to access your app's
content directly from the **Messages** app.

<figure class="site-figure {{include.class}}">
    <div class="site-figure-container">
        <img src='/assets/images/docs/development/platform-integration/app-extensions/imessage-extension.png'
        height='300'>
    </div>
</figure>

The Flutter command line tool does **not** support 
building Flutter UI for app extensions. 
To create the UI for 
an app extension using Flutter, you must compile 
a custom engine and embed the `FlutterViewController`
as described in the following section.

{{site.alert.note}}
  This requires a custom build of the Flutter engine and
  is only for advanced users.
  To learn more, check out [Compiling the engine][].
{{site.alert.end}}

<!-- TO DO: replace link with a ghost PR -->
1. Create a custom build of the Flutter engine that removes uses of
   `sharedApplication` and corrects the path for the bundle.
   Check out an [example from the community on GitHub][].

1. Open the Flutter app project settings in Xcode to share build
   configurations. 

   In the **Info** tab,
   under the **Configurations** expandable group,
   expand the **Debug**, **Profile**, and **Release** entries.
   For each, select the same value from the drop-down menu for the
   extension target as the entry selected for the normal app target. 

    <figure class="site-figure {{include.class}}">
        <div class="site-figure-container">
            <img src='/assets/images/docs/development/platform-integration/app-extensions/xcode-configurations.png'
            height='300'>
        </div>
    </figure>

1. If the generated extension code includes a storyboard file, delete it. 
    In the **Info.plist** file delete the **NSExtensionMainStoryboard** property, 
    and add add the **NSExtensionPrincipalClass** property with a value that matches the 
    name of your `ViewController`. For example, in an iMessage extension you would use 
    `MessageViewController`. 
    
    <figure class="site-figure {{include.class}}">
        <div class="site-figure-container">
            <img src='/assets/images/docs/development/platform-integration/app-extensions/extension-info.png'
            height='300'>
        </div>
    </figure>


1. Embed the `FlutterViewController` as described in
   [Adding a Flutter Screen][]. For example, you can display a 
   specific route in your Flutter app within an iMessage extension.

    ```swift
    //This attribute tells the compiler that this piece of Swift code can be accessed from Objective-C.
    @objc(MessagesViewController)

    class MessagesViewController: MSMessagesAppViewController {
        override func viewDidLoad() {
            super.viewDidLoad()
            showFlutter()
        }
    
        @objc func showFlutter() {
            // Create a FlutterViewController with an implicit FlutterEngine
            let flutterViewController = FlutterViewController(project: nil, initialRoute: "/ext", nibName: nil, bundle: nil)
            present(flutterViewController, animated: true, completion: nil)
        }
    ```

{{site.alert.important}}
Commenting out `sharedApplication` disables
many features in the Flutter framework.
This hasn't been tested in App Store submissions
and might not work for some app extensions.
For example, home screen Widgets can't use some
lower-level APIs needed to draw Flutter UI.
{{site.alert.end}}

## Tutorials
For step-by-step instruction for using app extensions with your
Flutter iOS app, check out the 
[Adding Home Screen Widgets to your Flutter app][] codelab.

[Apple's documentation]: https://developer.apple.com/app-extensions/
[Core Spotlight]: https://developer.apple.com/documentation/corespotlight
[WidgetKit]: https://developer.apple.com/documentation/widgetkit
[Leveraging Apple's System APIs and Frameworks]: {{site.url}}/development/platform-integration/ios/apple-frameworks
[pub.dev]: {{site.pub-pkg}}
[App Group]: https://developer.apple.com/documentation/xcode/configuring-app-groups
[Adding a Flutter Screen]: {{site.url}}/development/add-to-app/ios/add-flutter-screen?tab=vc-uikit-swift-tab#alternatively---create-a-flutterviewcontroller-with-an-implicit-flutterengine
[`shared_preference_app_group`]: {{site.pub-pkg}}/shared_preference_app_group
[Compiling the Engine]: https://github.com/flutter/flutter/wiki/Compiling-the-engine
[`path_provider`]: {{site.pub-pkg}}/path_provider
[`sqflite`]: {{site.pub-pkg}}/sqflite
[`workmanager`]: {{site.pub-pkg}}/workmanager
[read and write files]: {{site.url}}/cookbook/persistence/reading-writing-files
[example from the community on GitHub]: {{site.github}}/tomduncalf/engine/pull/1/files
[Deep Linking]:{{site.url}}/development/ui/navigation/deep-linking

<!--  TO DO: add links when published -->
[Adding Home Screen Widgets to your Flutter app]: {{site.codelabs}}/flutter