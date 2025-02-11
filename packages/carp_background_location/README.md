# CARP Background Location Plugin

A background location plugin for Android and iOS which works even when the app is in the background. This is a simple wrapper to the [`background_locator`](https://pub.dev/packages/background_locator) plugin. Refer to the background_locator's [wiki](https://github.com/rekab-app/background_locator/wiki) page for install and setup instruction.

## Android setup

Add the following permissions to your manifest:

```xml
    <uses-permission android:name="android.permission.INTERNET" />
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />
    <uses-permission android:name="android.permission.WAKE_LOCK" />
    <uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"/>
```

Afterwards, include the following entries within the application tag, as follows:

```xml
<application
       ...
        <receiver
                android:name="rekab.app.background_locator.LocatorBroadcastReceiver"
                android:enabled="true"
                android:exported="true"
        />

        <receiver android:name="rekab.app.background_locator.BootBroadcastReceiver"
                  android:enabled="true">
            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED"/>
            </intent-filter>
        </receiver>

        <service
                android:name="rekab.app.background_locator.LocatorService"
                android:permission="android.permission.BIND_JOB_SERVICE"
                android:exported="true"
        />
        <service
                android:name="rekab.app.background_locator.IsolateHolderService"
                android:permission="android.permission.FOREGROUND_SERVICE"
                android:exported="true"
        />
    </application>
```

**NOTE** In Android 11 location permissions cannot be set automatically by the app (via the manifest file). Google writes on [this page](https://developer.android.com/training/location/permissions#request-background-location) that:


> On Android 11 (API level 30) and higher [...] the system dialog doesn’t include the **Allow all the time** option. Instead, users must enable background location on a settings page, as shown in figure 7.

Hence, for this plugin (and the example app) to work, you need to go to this settings screen and **manually** set the permission to "Alow all the time".

## iOS setup

Add the following entries to your `Info.plist` file

```xml
<key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
<string>This app needs access to location when open and in the background.</string>
<key>NSLocationAlwaysUsageDescription</key>
<string>This app needs access to location when in the background.</string>
<key>NSLocationWhenInUseUsageDescription</key>
<string>This app needs access to location when open.</string>
<key>UIBackgroundModes</key>
<array>
	<string>location</string>
</array>
```

Next, overwrite your AppDelegate.swift in the XCode project with:

```swift
import UIKit
import Flutter

import background_locator

func registerPlugins(registry: FlutterPluginRegistry) -> () {
    if (!registry.hasPlugin("BackgroundLocatorPlugin")) {
        GeneratedPluginRegistrant.register(with: registry)
    }
}


@UIApplicationMain
@objc class AppDelegate: FlutterAppDelegate {
  override func application(
    _ application: UIApplication,
    didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?
  ) -> Bool {
    GeneratedPluginRegistrant.register(with: self)
    BackgroundLocatorPlugin.setPluginRegistrantCallback(registerPlugins)
    return super.application(application, didFinishLaunchingWithOptions: launchOptions)
  }
}
```

## Usage

```dart
  Stream<LocationDto> locationStream;
  StreamSubscription<LocationDto> locationSubscription;
 
  @override
  void initState() {
    super.initState();

    LocationManager().interval = 1;
    LocationManager().distanceFilter = 0;
    LocationManager().notificationTitle = 'CARP Location Example';
    LocationManager().notificationMsg = 'CARP is tracking your location';
    locationStream = LocationManager().locationStream;
  }

  void onData(LocationDto dto) {
    setState(() {
      if (_status == LocationStatus.UNKNOWN) {
        _status = LocationStatus.RUNNING;
      }
      lastLocation = dto;
      lastTimeLocation = DateTime.now();
    });
  }


  void start() async {
    locationSubscription?.cancel();
    locationSubscription = locationStream?.listen(onData);
    await LocationManager().start();
    setState(() {
      _status = LocationStatus.RUNNING;
    });
  }

  void stop() {
    locationSubscription?.cancel();
    LocationManager().stop();
    setState(() {
      _status = LocationStatus.STOPPED;
    });
  }
```

See the example app for a complete example of usage.

## Features and bugs

Please file feature requests and bug reports at the [issue tracker][tracker].

[tracker]: https://github.com/cph-cachet/flutter-plugins/issues

## License

This software is copyright (c) [Copenhagen Center for Health Technology (CACHET)](https://www.cachet.dk/) at the [Technical University of Denmark (DTU)](https://www.dtu.dk).
This software is available 'as-is' under a [MIT license](https://github.com/cph-cachet/flutter-plugins/blob/master/packages/carp_background_location/LICENSE).


