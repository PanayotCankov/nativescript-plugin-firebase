<img src="images/firebase-logo.png" width="116px" height="32px" alt="Firebase"/>

<img src="images/features/notifications.png" width="296px" height="124px" alt="Notifications"/>

## Enabling Firebase Cloud Messaging (FCM)
Version 3.3.0 of this plugin added FCM support (which is the successor of GCM).

Although using push messages in your Firebase app is really easy setting it up is not. Traditionally, especially for iOS.

### Android
If you didn't choose this feature during installation you can uncomment `firebase-messaging` in [include.gradle](../platforms/android/include.gradle)

You will not get the title and body if the notification was received while the application was in the background, but you will get the *data* payload.

Add the following services in the `app/App_Resources/Android/AndroidManifest.xml` to enable advanced FCM messaging:
```
<manifest ... >
    <application ... >
        ...
        <service android:name="org.nativescript.plugins.firebase.MyFirebaseInstanceIDService">
            <intent-filter>
                <action android:name="com.google.firebase.INSTANCE_ID_EVENT"/>
            </intent-filter>
        </service>
        <service android:name="org.nativescript.plugins.firebase.MyFirebaseMessagingService">
            <intent-filter>
                <action android:name="com.google.firebase.MESSAGING_EVENT"/>
            </intent-filter>
        </service>
    </application>
</manifest>
```

### iOS
If you didn't choose this feature during installation you can uncomment `Firebase/Messaging` in [Podfile](../platforms/ios/Podfile)

#### Receiving remote notifications in the background
Open `app/App_Resources/iOS/Info.plist` and add this to the bottom:

```xml
<key>UIBackgroundModes</key>
<array>
    <string>remote-notification</string>
</array>
```

#### Provisioning hell
Follow [this guide](https://firebase.google.com/docs/cloud-messaging/ios/certs) to the letter. Once you've done it run `tns run ios` and upon starting the app it should prompt you for notification support. That also works on the simulator, but actually receiving notifications is _only_ possible on a real device.

### Handling a notification
To listen to received notifications while in the foreground or when your app moves from the background to the foreground, add a handler to `init`.

Any pending notifications (while your app was not in the foreground) will trigger the `onMessageReceivedCallback` handler.

```js
  firebase.init({
    onMessageReceivedCallback: function(message) {
      console.log("Title: " + message.title);
      console.log("Body: " + message.body);
      // if your server passed a custom property called 'foo', then do this:
      console.log("Value of 'foo': " + message.foo);
    }
  });
```

You don't _have_ to provide the handler during `init` - you can also do it through a dedicated function.

One scenario where you want to do this is if you don't want the "This app wants to send push notifications" popup during init, but delay it until you call this function.

```js
  firebase.addOnMessageReceivedCallback(
    function(message) {
      // ..
    }
  );
```

### Pushing to individual devices
If you want to send push messages to individual devices, either from your own backend or the FCM console, you need the push token.

Similarly to the message callback you can either wire this through `init` or as a separate function:

```js
  firebase.init({
    onPushTokenReceivedCallback: function(token) {
      console.log("Firebase push token: " + token);
    }
  });
```

.. or:

```js
  firebase.addOnPushTokenReceivedCallback(
    function(token) {
      // ..
    }
  );
```

## Testing
Using the Firebase Console gives you most flexibility, but you can quickly and easily test from the command line as well:

```
curl -X POST --header "Authorization: key=SERVER_KEY" --Header "Content-Type: application/json" https://fcm.googleapis.com/fcm/send -d "{\"notification\":{\"title\": \"My title\", \"text\": \"My text\", \"sound\": \"default\"}, \"data\":{\"foo\":\"bar\"}, \"priority\": \"High\", \"to\": \"DEVICE_TOKEN\"}"
```

* SERVER_KEY: see below
* DEVICE_TOKEN: the one you got in `addOnPushTokenReceivedCallback` or `init`'s `onPushTokenReceivedCallback`

<img src="images/push-server-key.png" width="459px" height="220px" alt="Push server key"/>
