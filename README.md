# fcm_demo

A new Flutter project.

## Getting Started

This project is a starting point for a Flutter application.

A few resources to get you started if this is your first Flutter project:

- [Lab: Write your first Flutter app](https://docs.flutter.dev/get-started/codelab)
- [Cookbook: Useful Flutter samples](https://docs.flutter.dev/cookbook)
- - [firebase_core](https://pub.dev/packages/firebase_core), which is required to use any Firebase service with Flutter
- [firebase_messaging](https://pub.dev/packages/firebase_messaging), which is used for receiving notifications in the app
- [overlay_support](https://pub.dev/packages/overlay_support), which builds overlay UI

For help getting started with Flutter development, view the
[online documentation](https://docs.flutter.dev/), which offers tutorials,
samples, guidance on mobile development, and a full API reference.


What are push notifications?

    If you use a smartphone, you almost certainly encounter push notifications daily.
    Push notifications are clickable pop-up messages that appear on your users’ devices,
    regardless of whether they are using that particular app at the time.
    Even when the device is idle or the user is using another app,
    a user will receive push notifications as long as the device is online and notification permissions are granted.
    Push notifications can be used to inform a user of status updates, message requests, reminders, alerts, and more.

In this , we’ll use Firebase Cloud Messaging to send push notifications.

Setting up Firebase

    1.To start using Firebase, you have to create new Firebase project.
      Log into your Google account, navigate to the Firebase console, and click Add project:
    2.Enter a project name and click Continue:
    3.Disable Google Analytics; we don’t need it for our sample project. Then, click Create project:
    4.After the project initializes, click Continue:
    5.This will take you to the Project Overview screen. Here,
      you’ll find options to integrate the Firebase project with your Android and iOS app:


Integrate Firebase with Flutter: Android

To integrate your Firebase project with the Android side of the app, first,
click the Android icon on the project overview page:

    You should be directed to a form. First, enter the Android package name.
    You can find this in your project directory → android → app → src → main → AndroidManifest.xml. On the second line,
    you’ll see your package name. Just copy and paste it into the form.

    Optionally, you can choose a nickname for your app. If you leave this field empty,
    an auto-generated app name will be used:

    You’ll have to enter the SHA-1 hash. Just hover over the help icon ? and click on See this page,
    which will take you to the Authenticating Your Client page:

    From here, you’ll get the command to generate the SHA-1 hash. Paste this command into your terminal,
    then just copy and paste the generated SHA-1 hash into the form. Click Register app, which will take you to the next step.

    Download the google-services.json file, drag and drop it into your project directory → android → app, then,
    click Next:

    Follow the instructions and add the code snippets in the specified position. Then, click Next:

    Finally, click Continue to console: With this, you’ve completed the Firebase setup for the Android side of your app.

Integrate Firebase with Flutter: iOS

To integrate your Firebase project with the iOS side of your app, first,
click the Add app button on the project overview page, then select iOS:

    Enter the iOS bundle ID and your App nickname. Then, click Register app.
    You can leave the App store ID blank for now; you’ll get this when you deploy your app to the iOS App Store:

    You can find the bundle ID inside ios → Runner.xcodeproj → project.pbxproj by searching for PRODUCT_BUNDLE_IDENTIFIER:

    Next, select Download GoogleService-Info.plist:
 
    Open the ios folder of the project directory in Xcode.
    Drag and drop the file you downloaded into the Runner subfolder.
    When a dialog box appears, make sure the Copy items if needed of the Destination property is checked and Runner is selected in the Add to targets box.
    Then, click Finish:

Adding push notification functionality with Firebase Cloud Messaging

Now, it’s time for us to add the functionality for our push notifications.
To start using the Firebase Cloud Messaging service, first, define a variable for 

FirebaseMessaging:

      late final FirebaseMessaging _messaging;

Create a method called registerNotification() inside the HomePage class.
registerNotification() will initialize the Firebase app, request notification access,
which is required only on iOS devices, and finally, configure the messaging to receive and display push notifications:

    void registerNotification() async {
     // 1. Initialize the Firebase app
             await Firebase.initializeApp();
     // 2. Instantiate Firebase Messaging
           _messaging = FirebaseMessaging.instance;
    // 3. On iOS, this helps to take the user permissions
          NotificationSettings settings = await _messaging.requestPermission(
             alert: true,
             badge: true,
             provisional: false,
             sound: true,
           );

          if (settings.authorizationStatus == AuthorizationStatus.authorized) {
                 print('User granted permission');
            // TODO: handle the received notifications
           } else {
           print('User declined or has not accepted permission');
          }
         }


In the code above, we first initialized the Firebase app, without which we wouldn’t be able to access any Firebase services inside the app.
After that, we instantiated Firebase Messaging. The requestPermission() method takes user consent on iOS devices. If the app is being run on an Android device,
this is ignored.

To receive push notifications that arrive on the device and perform a UI change according to the notification, use the following code:
    
       PushNotification? _notificationInfo;

       void registerNotification() async {
         if (settings.authorizationStatus == AuthorizationStatus.authorized) {
              print('User granted permission');
                 // For handling the received notifications
                  FirebaseMessaging.onMessage.listen((RemoteMessage message) {
                    // Parse the message received
               PushNotification notification = PushNotification(
                   title: message.notification?.title,
                   body: message.notification?.body,
               );

             setState(() {
                _notificationInfo = notification;
               });
            });
         } else {
          print('User declined or has not accepted permission');
             }
           }


The PushNotification is a model class for storing the notification content.

The PushNotification model class looks like the code below:

        class PushNotification {
             PushNotification({
             this.title,
             this.body,
           });
         String? title;
         String? body;
        }

Then, use the showSimpleNotification() method to display the notification inside the app:

      void registerNotification() async {
           if (settings.authorizationStatus == AuthorizationStatus.authorized) {
              FirebaseMessaging.onMessage.listen((RemoteMessage message) {
               if (_notificationInfo != null) {
              // For displaying the notification as an overlay
                 showSimpleNotification(
               Text(_notificationInfo!.title!),
                   leading: NotificationBadge(totalNotifications: _totalNotifications),
                   subtitle: Text(_notificationInfo!.body!),
                   background: Colors.cyan.shade700,
                   duration: Duration(seconds: 2),
                  );
                }
               });
            } else {
           print('User declined or has not accepted permission');
            }
          }

If you try to put the app in the background, you’ll still receive the notification.
Since we haven’t yet configured how to handle background notifications,
you won’t see any change in the UI as you tap on the notification to open the app:

Handling background notifications

To handle background notifications, we have to define a top-level function called _firebaseMessagingBackgroundHandler() and pass it to the onBackgroundMessage() inside the registerNotification() method.

You can define the _firebaseMessagingBackgroundHandler() function as follows:

     Future _firebaseMessagingBackgroundHandler(RemoteMessage message) async {
     print("Handling a background message: ${message.messageId}");
      }

Keep in mind that you have to define this as a top-level function, which means it should be outside of any class.

Call the onBackgroundMessage() method:

      void registerNotification() async {
          await Firebase.initializeApp();
           _messaging = FirebaseMessaging.instance;
          // Add the following line
            FirebaseMessaging.onBackgroundMessage(_firebaseMessagingBackgroundHandler);
          }

If you just define this, you won’t be able to retrieve and show data within the app.

To handle the action when the app is in the background and the notification is tapped,
you have to add the following code to the initState() method:


         @override
       void initState() {
          // For handling notification when the app is in background
         // but not terminated
            FirebaseMessaging.onMessageOpenedApp.listen((RemoteMessage message) {
              PushNotification notification = PushNotification(
              title: message.notification?.title,
              body: message.notification?.body,
            );
          setState(() {
         _notificationInfo = notification;
         _totalNotifications++;
           });
         });
          super.initState();
         }

But, the initState() method won’t be enough to retrieve the information if the app is in terminated state and is brought back by tapping on the notification.
Define a method called checkForInitialMessage() and add the following code to it:

       // For handling notification when the app is in terminated state
          checkForInitialMessage() async {
          await Firebase.initializeApp();
          RemoteMessage? initialMessage =
          await FirebaseMessaging.instance.getInitialMessage();
          if (initialMessage != null) {
          PushNotification notification = PushNotification(
          title: initialMessage.notification?.title,
          body: initialMessage.notification?.body,
          );
        setState(() {
        _notificationInfo = notification;
          });
          }
         }

Call checkForInitialMessage() from the initState() method:

                 @override
                       void initState() {
                            // Call here
                       checkForInitialMessage();
                       super.initState();
                 }