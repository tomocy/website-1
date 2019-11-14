---
title: Adding a Flutter screen to an iOS app
short-title: Add a Flutter screen
description: Learn how to add a single Flutter screen to your existing iOS app.
---

This guide describes how to add a single Flutter screen to an existing iOS app.

## Write code to use FlutterViewController from your host app

The proper place to do this is specific to your host app. Here is an example
that makes sense for the blank screen of the host app generated by Xcode.

First declare your app delegate to be a subclass of `FlutterAppDelegate`. Then define a FlutterEngine property, which helps you to register a plugin without a FlutterViewController instance.

In `AppDelegate.h`:

```objective-c
@import UIKit;
@import Flutter;

@interface AppDelegate : FlutterAppDelegate
@property (nonatomic,strong) FlutterEngine *flutterEngine;
@end
```

This allows `AppDelegate.m` to be simple, unless your host app needs to override
other methods here:

```objective-c
#import <FlutterPluginRegistrant/GeneratedPluginRegistrant.h> // Only if you have Flutter Plugins

#import "AppDelegate.h"

@implementation AppDelegate

// This override can be omitted if you do not have any Flutter Plugins.
- (BOOL)application:(UIApplication *)application
    didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
  self.flutterEngine = [[FlutterEngine alloc] initWithName:@"io.flutter" project:nil];
  [self.flutterEngine runWithEntrypoint:nil];
  [GeneratedPluginRegistrant registerWithRegistry:self.flutterEngine];
  return [super application:application didFinishLaunchingWithOptions:launchOptions];
}

@end
```

If you are writing in Swift, you can do the following in your `AppDelegate.swift`:

```swift
import UIKit
import Flutter
import FlutterPluginRegistrant // Only if you have Flutter Plugins.

@UIApplicationMain
class AppDelegate: FlutterAppDelegate {
  var flutterEngine : FlutterEngine?;
  // Only if you have Flutter plugins.
  override func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
    self.flutterEngine = FlutterEngine(name: "io.flutter", project: nil);
    self.flutterEngine?.run(withEntrypoint: nil);
    GeneratedPluginRegistrant.register(with: self.flutterEngine);
    return super.application(application, didFinishLaunchingWithOptions: launchOptions);
  }

}
```

<details>
<summary>What to do if the app delegate already inherits from somewhere else.</summary>

Make your app delegate implement the `FlutterAppLifeCycleProvider` protocol, for
example:

```objective-c
@import Flutter;
@import UIKit;
@import FlutterPluginRegistrant; // Only if you have Flutter Plugins

@interface AppDelegate : UIResponder <UIApplicationDelegate, FlutterAppLifeCycleProvider>
@property (strong, nonatomic) UIWindow *window;
@property (nonatomic,strong) FlutterEngine *flutterEngine;
@end
```

The implementation should mostly just delegate to a `FlutterPluginAppLifeCycleDelegate`:

```objective-c
@interface AppDelegate ()
@property (nonatomic, strong) FlutterPluginAppLifeCycleDelegate* lifeCycleDelegate;
@end

@implementation AppDelegate

- (instancetype)init {
    if (self = [super init]) {
        _lifeCycleDelegate = [[FlutterPluginAppLifeCycleDelegate alloc] init];
    }
    return self;
}

- (BOOL)application:(UIApplication*)application
didFinishLaunchingWithOptions:(NSDictionary*)launchOptions {
    self.flutterEngine = [[FlutterEngine alloc] initWithName:@"io.flutter" project:nil];
    [self.flutterEngine runWithEntrypoint:nil];
    [GeneratedPluginRegistrant registerWithRegistry:self.flutterEngine]; // Only if you are using Flutter plugins.
    return [_lifeCycleDelegate application:application didFinishLaunchingWithOptions:launchOptions];
}

// Returns the key window's rootViewController, if it's a FlutterViewController.
// Otherwise, returns nil.
- (FlutterViewController*)rootFlutterViewController {
    UIViewController* viewController = [UIApplication sharedApplication].keyWindow.rootViewController;
    if ([viewController isKindOfClass:[FlutterViewController class]]) {
        return (FlutterViewController*)viewController;
    }
    return nil;
}

- (void)touchesBegan:(NSSet*)touches withEvent:(UIEvent*)event {
    [super touchesBegan:touches withEvent:event];

    // Pass status bar taps to key window Flutter rootViewController.
    if (self.rootFlutterViewController != nil) {
        [self.rootFlutterViewController handleStatusBarTouches:event];
    }
}

- (void)applicationDidEnterBackground:(UIApplication*)application {
    [_lifeCycleDelegate applicationDidEnterBackground:application];
}

- (void)applicationWillEnterForeground:(UIApplication*)application {
    [_lifeCycleDelegate applicationWillEnterForeground:application];
}

- (void)applicationWillResignActive:(UIApplication*)application {
    [_lifeCycleDelegate applicationWillResignActive:application];
}

- (void)applicationDidBecomeActive:(UIApplication*)application {
    [_lifeCycleDelegate applicationDidBecomeActive:application];
}

- (void)applicationWillTerminate:(UIApplication*)application {
    [_lifeCycleDelegate applicationWillTerminate:application];
}

- (void)application:(UIApplication*)application
didRegisterUserNotificationSettings:(UIUserNotificationSettings*)notificationSettings {
    [_lifeCycleDelegate application:application
didRegisterUserNotificationSettings:notificationSettings];
}

- (void)application:(UIApplication*)application
didRegisterForRemoteNotificationsWithDeviceToken:(NSData*)deviceToken {
    [_lifeCycleDelegate application:application
didRegisterForRemoteNotificationsWithDeviceToken:deviceToken];
}

- (void)application:(UIApplication*)application
didReceiveRemoteNotification:(NSDictionary*)userInfo
fetchCompletionHandler:(void (^)(UIBackgroundFetchResult result))completionHandler {
    [_lifeCycleDelegate application:application
       didReceiveRemoteNotification:userInfo
             fetchCompletionHandler:completionHandler];
}

- (BOOL)application:(UIApplication*)application
            openURL:(NSURL*)url
            options:(NSDictionary<UIApplicationOpenURLOptionsKey, id>*)options {
    return [_lifeCycleDelegate application:application openURL:url options:options];
}

- (BOOL)application:(UIApplication*)application handleOpenURL:(NSURL*)url {
    return [_lifeCycleDelegate application:application handleOpenURL:url];
}

- (BOOL)application:(UIApplication*)application
            openURL:(NSURL*)url
  sourceApplication:(NSString*)sourceApplication
         annotation:(id)annotation {
    return [_lifeCycleDelegate application:application
                                   openURL:url
                         sourceApplication:sourceApplication
                                annotation:annotation];
}

- (void)application:(UIApplication*)application
performActionForShortcutItem:(UIApplicationShortcutItem*)shortcutItem
  completionHandler:(void (^)(BOOL succeeded))completionHandler NS_AVAILABLE_IOS(9_0) {
    [_lifeCycleDelegate application:application
       performActionForShortcutItem:shortcutItem
                  completionHandler:completionHandler];
}

- (void)application:(UIApplication*)application
handleEventsForBackgroundURLSession:(nonnull NSString*)identifier
  completionHandler:(nonnull void (^)(void))completionHandler {
    [_lifeCycleDelegate application:application
handleEventsForBackgroundURLSession:identifier
                  completionHandler:completionHandler];
}

- (void)application:(UIApplication*)application
performFetchWithCompletionHandler:(void (^)(UIBackgroundFetchResult result))completionHandler {
    [_lifeCycleDelegate application:application performFetchWithCompletionHandler:completionHandler];
}

- (void)addApplicationLifeCycleDelegate:(NSObject<FlutterPlugin>*)delegate {
    [_lifeCycleDelegate addDelegate:delegate];
}
@end
```

</details>

`ViewController.m`:

```objective-c
@import Flutter;
#import "AppDelegate.h"
#import "ViewController.h"

@implementation ViewController
- (void)viewDidLoad {
    [super viewDidLoad];
    UIButton *button = [UIButton buttonWithType:UIButtonTypeCustom];
    [button addTarget:self
               action:@selector(handleButtonAction)
     forControlEvents:UIControlEventTouchUpInside];
    [button setTitle:@"Press me" forState:UIControlStateNormal];
    [button setBackgroundColor:[UIColor blueColor]];
    button.frame = CGRectMake(80.0, 210.0, 160.0, 40.0);
    [self.view addSubview:button];
}

- (void)handleButtonAction {
    FlutterEngine *flutterEngine = [(AppDelegate *)[[UIApplication sharedApplication] delegate] flutterEngine];
    FlutterViewController *flutterViewController = [[FlutterViewController alloc] initWithEngine:flutterEngine nibName:nil bundle:nil];
    [self presentViewController:flutterViewController animated:false completion:nil];
}
@end
```

Or, using Swift:

`ViewController.swift`:

```swift
import UIKit
import Flutter

class ViewController: UIViewController {
  override func viewDidLoad() {
    super.viewDidLoad()
    let button = UIButton(type:UIButton.ButtonType.custom)
    button.addTarget(self, action: #selector(handleButtonAction), for: .touchUpInside)
    button.setTitle("Press me", for: UIControl.State.normal)
    button.frame = CGRect(x: 80.0, y: 210.0, width: 160.0, height: 40.0)
    button.backgroundColor = UIColor.blue
    self.view.addSubview(button)
  }

  @objc func handleButtonAction() {
    if let flutterEngine = (UIApplication.shared.delegate as? AppDelegate)?.flutterEngine {
      let flutterViewController = FlutterViewController(engine: flutterEngine, nibName: nil, bundle: nil)
      self.present(flutterViewController, animated: false, completion: nil)
    }
  }
}
```

You should now be able to build and launch MyApp on the Simulator or on a device.
Pressing the button should bring up a full-screen Flutter view with the standard
Flutter Demo counting app. You can use routes to show different widgets at different
places in your app, as described in the Android section above. To set the route, call

- Objective-C:

```objective-c
[flutterViewController setInitialRoute:@"route1"];
```

- Swift:

```swift
flutterViewController.setInitialRoute("route1")
```

immediately after construction of the `FlutterViewController` (and before presenting
it).

You can have the Flutter app dismiss itself by calling `SystemNavigator.pop()`
in the Dart code.

## Building and running your app

You build and run MyApp using Xcode in exactly the same way that you did before you
added the Flutter module dependency. The same goes for editing, debugging, and
profiling your iOS code.

## Hot restart/reload and debugging Dart code

Connect a device or launch a Simulator. Then make Flutter CLI tooling listen
for your app to come up:

```bash
$ cd some/path/my_flutter
$ flutter attach
Waiting for a connection from Flutter on iPhone X...
```

Launch `MyApp` in debug mode from Xcode. Navigate to an area of the app that uses
Flutter. Then turn back to the terminal, and you should see output similar to the
following:

```text
Done.
Syncing files to device iPhone X...                          4.7s

🔥  To hot reload changes while running, press "r". To hot restart (and rebuild state), press "R".
An Observatory debugger and profiler on iPhone X is available at: http://127.0.0.1:54741/
For a more detailed help message, press "h". To quit, press "q".
```

You can now edit the Dart code in `my_flutter`, and the changes can be hot
reloaded by pressing `r` in the terminal. You can also paste the URL above into
your browser to use the Dart Observatory for setting breakpoints, analyzing
memory retention and other debugging tasks.

### Debugging specific instances of Flutter

It's possible to add multiple instances of Flutter (`root isolates`) to an app. `flutter attach` connects to all of the available isolates by default. Any commands sent from the attached CLI are then forwarded to each of the attached isolates.

List all attached isolates by typing `l` from an attached `flutter` CLI tool. If unspecified, isolate names are automatically generated from the dart entry point file and function name.

Example `l` output for an application that's displaying two Flutter isolates simultaneously:

```text
Connected views:
  main.dart$main-517591213 (isolates/517591213)
  main.dart$main-332962855 (isolates/332962855)
```

Attach to specific isolates instead in two steps:

1. Name the Flutter root isolate of interest in its Dart source.

```dart
// main.dart
import 'dart:ui' as ui;

void main() {
  ui.window.setIsolateDebugName("debug isolate");
  // ...
}
```

2. Run `flutter attach` with the `--isolate-filter` option.

```bash
$ flutter attach --isolate-filter='debug'
Waiting for a connection from Flutter...
Done.
Syncing files to device...      1.1s

🔥  To hot reload changes while running, press "r". To hot restart (and rebuild state), press "R".
An Observatory debugger and profiler is available at: http://127.0.0.1:43343/
For a more detailed help message, press "h". To detach, press "d"; to quit, press "q".

Connected view:
  debug isolate (isolates/642101161)
```

You can check out
[`commit 93573de`](https://github.com/flutter/flutter/commit/93573de2165c750fdeefcd2d620e2b8bd494fed6)
for a more detailed example.