# Moduware Platform Tile
Shared logic for Moduware native tiles.

Logic code to simplify tile development for module platform, provides basic classes for both iOS and Android platforms, tools and predefines tile behaviour that cannot be altered.

## Installation
- **Prefered:** NuGet package (TODO: add link)
- **For customization**: Git Sub-Module
```
cd solution-folder
git submodule add -b master https://github.com/moduware/platform-tile.git
```
- Or just download and copy to sub-folder in your solution

At the end add platfrom tile projects into your solution, reference this projects:
- Types and Shared
- iOS or Droid depending on your target platform

## Standard behaviour
- On start or resume, check for connected gateway and block user actions
- If no connected gateway, then:
    - Warn user
    - Open dashboard app so he can connect
- If connected, but no current gateway-module configuration, then:
    - Send configuration request to app
    - Apply configuration after received
- If connected and config present, unblock user and let use tile
- When gateway disconnected, then:
    - Warn user
    - Open dashboard app so he can connect
- When tile launched from dashboard, then:
    - apply configuration
    - apply target module

## Utilities
- `ShowNoSupportedModuleAlert(callback)`
- `ShowNotConnectedAlert(callback)`
- `OpenDashboard()`

## Android
`class TileActivity : Android.App.Activity`

Your main activity **MUST** have url scheme with Id of your tile:

```csharp
[IntentFilter(new[] { "android.intent.action.VIEW" }, DataScheme = "moduware.tile.speaker", Categories = new[] { "android.intent.category.DEFAULT", "android.intent.category.BROWSABLE" })]
```

Inherit your main activity from `TileActivity`, assign tile id in `OnCreate` method override, two general events available:

```csharp
protected override void OnCreate(Bundle savedInstanceState)
{
    base.OnCreate(savedInstanceState);
    // We need assign Id of our tile here, it is required for proper Dashboard - Tile communication
    TileId = "moduware.tile.speaker";

    // We need to know when core is ready so we can start listening for data from gateways
    CoreReady += (o, e) => ...;
    // And we need to know when we are ready to send commands
    ConfigurationApplied += (o, e) => ...;
}
```

## iOS
`class TileViewController : UIViewController`

Your app **MUST** have url scheme defined in *Info.plist* file:

```xml
<key>CFBundleURLTypes</key>
<array>
    <dict>
        <key>CFBundleURLName</key>
        <string>moduware.xamarin.speaker.urlscheme</string>
        <key>CFBundleURLSchemes</key>
        <array>
            <string>moduware.tile.speaker</string>
        </array>
        <key>CFBundleURLTypes</key>
        <string>Viewer</string>
    </dict>
</array>
```  

Inherit your main controller from `TileViewController`, assign tile Id in `ViewDidLoad` method override, two general events available:

```csharp
public override void ViewDidLoad()
{
    // We need assign Id of our tile here, it is required for proper Dashboard - Tile communication
    TileId = "moduware.tile.speaker";

    // We need to know when core is ready so we can start listening for data from gateways
    CoreReady += (o, e) => ...;
    // And we need to know when we are ready to send commands
    ConfigurationApplied += (o, e) => ...;

    base.ViewDidLoad(); 
}
```

In your *AppDelegate.cs* you need override two methods and pass their data to `TileViewController` handlers:

```csharp
public override bool OpenUrl(UIApplication application, NSUrl url, string sourceApplication, NSObject annotation)
{
    // custom stuff here using different properties of the url passed in
    var viewController = (TileViewController)Window.RootViewController;
    viewController.OnQueryRecieved(url.AbsoluteString);

    return true;
}

public override void OnActivated(UIApplication application)
{
    // Restart any tasks that were paused (or not yet started) while the application was inactive. 
    // If the application was previously in the background, optionally refresh the user interface.
    var viewController = (TileViewController)Window.RootViewController;
    viewController.OnResumeActions();
}
```