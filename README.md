# background_location_updates

Retrieve periodic location updates, even when the main App is not running. Useful for Navigation Apps to keep a rough idea of where the User is heading, and various other purposes. Please don't be evil though, and tell the User exactly how, when and why you wish to retrieve her location.

The Plugin uses `Significant Location Change` on iOS, and a `WorkManager`-based Periodic Job, combined with a `OnBoot` Broadcast Receiver on Android.

## Getting Started

### Get the Package:

Add the following to your `pubspec.yml`:

```yaml
dependencies:
  background_location_updates: <<UNRELEASED>>
```

### Android Permissions

In your `src/main/app/AndroidManifest.xml`, we'll need to register a couple permissions and a Broadcast Receiver.

```xml
<manifest ...">
    ....
    <!-- Alternatively: ACCESS_COARSE_LOCATION -->
    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED" />

    <application ...>
        .....
        <receiver android:name="io.gjg.backgroundlocationupdates.service.BootBroadcastReceiver">
            <intent-filter>
                <action android:name="android.intent.action.BOOT_COMPLETED" />
            </intent-filter>
        </receiver>
    </application>
</manifest>
```

### iOS Permissions
We'll first have to tell iOS that the app wishes to be started on location updates. Then, we have to justify to the User why we're intending to use their location. Both of these things are taken care of by setting the appropriate keys in `ios/Runner/Info.plist`:

```
<!-- We want to receive location updates in background -->
<key>UIBackgroundModes</key>
<array>
    <string>location</string>
</array>
....

<!-- This key is for iOS 10 and earlier -->
<key>NSLocationAlwaysUsageDescription</key>
<string>Selling it on the black market</string>

<!-- This key is for iOS 11+. Justify here why you need Background Location. -->
<key>NSLocationAlwaysAndWhenInUseUsageDescription</key>
<string>Selling it on the black market</string>
<!-- This key is for iOS 11+. Justify here why you 
need basic Location Services (while the App is running). -->
<key>NSLocationWhenInUseUsageDescription</key>
<string>Selling it on the black market</string>
```

### Requesting Permissions
```dart
await BackgroundLocationUpdates.requestPermission()
```
On both Android and iOS, this will show a Permission Request Alertbox to the User. After the User has consented, further calls to this method will have no effect, and the other API Methods described below will work as intended.

### Starting Location Tracking

After you've asked the User for appropriate permissions (read up on GDPR!), start the Location Tracking.

```dart
await BackgroundLocationUpdates
   .startTrackingLocation(
        BackgroundLocationUpdates.LOCATION_SINK_SQLITE,
        requestInterval: const Duration(seconds: 10));
```

The first argument to `startTrackingLocation` specifies how you which the Location to be persisted. Currently, only `SQLite` is supported. The second argument is a requestInterval, which specifies how often to ask the Operating System for the User's location. This will only be respected on Android, and even there, it will only have the desired effect on Versions below Android O. On Android O and below, this argument is ignored, and you will only receive new locations on the discretion of the Operating System, which is about three times an hour for both platforms.

### Stopping Location Tracking

```dart
await BackgroundLocationUpdates.stopTrackingLocation();
```

### Getting all unread Location Traces
Location traces should be primarily received through this method.

```dart
List<Map<String, double>> traces = await BackgroundLocationUpdates.getUnreadLocationTraces();
```

This will retrieve all location traces that have not been previously marked as read. This means, you can call this method when your App is started in order to receive all Location Updates that happened in the time since your App was last opened. Currently, a Map of the following values is returned. **This will soon change**.


```dart
{
    // ID of the Trace
    "id": 0.0,
    "latitude": 23.2,
    "longitude": 212.2,
    // Recorded altitude. May be zero if it couldn't be measured by the Device.
    "altitude": 0.0,
    // UNIX Timestamp of when the Update occured
    "time": 15124124235324523.0,
    // How often the Trace has been marked as read with markAsRead()
    "readCount": 0.0,
}
```

### Marking Location Traces as read
After you've processed the Unread Location Traces, you can mark them as read, so you won't receive them again by the `getUnreadLocationTraces` call in the future.

```dart
await BackgroundLocationUpdates.markAsRead(
    traces.map((trace) => trace["id"].toInt()).asList()
);
```

### Number of Traces
You can get the count of all traces recorded, or the count of all unread traces.

```dart
int unreadCount = await BackgroundLocationUpdates.getUnreadLocationTracesCount();
int totalCount = await BackgroundLocationUpdates.getLocationTracesCount();
```

### Getting all Location Traces

```dart
List<Map<String, double>> traces = await BackgroundLocationUpdates.getLocationTraces();
```

Receive all Location Traces that have ever been received by the Plugin. It's not recommended to use this method.

For help getting started with Flutter, view our online
[documentation](https://flutter.io/).

For help on editing plugin code, view the [documentation](https://flutter.io/platform-plugins/#edit-code).