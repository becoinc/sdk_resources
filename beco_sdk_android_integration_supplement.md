![](https://beco.io/wp-content/themes/beco.io/assets/img/beco_logoblue@2x.png)

# **BECO MOBILE SDK FOR ANDROID™: INTEGRATION SUPPLEMENT**

This document is a supplement to integration efforts involving all versions of the Beco Mobile SDK for Android beginning with v1.10.0. The v1.10.0 release introduced the ability for the Beco Mobile SDK to continue to scan for and detect the closest Beco Beacon, as well as post that location update to the Beco Cloud, even when the host App is in the Background or Killed (swiped closed) State. This major update allows Android devices to join iOS devices in creating a continuous Beco data stream throughout the day, enhancing both your in-App functionality reliant on the real-time data as well as your long term spatial analytics.

Given the underlaying architecture of Android OS, in particular Android 8 and above, providing services in the Background/Killed State is achieved by utilizing the Android notification framework similar to how a music player App does so. Therefore, a successful integration of the Beco Mobile SDK for Android must also include your App having a persistent notification in the App Bar.

You can create your own customizable notification and notification ID, or you can use your App's existing notification if you already have one as shown here:

```
 if(becoSdk == null) {

          /* Create your own customizable notification and notification ID.
            Or you can use your application's existing notification if you already have one.  */
          Notification notification = createNotification(); //Create your own customizable notification
          int notificationID = 1234; //Set your notification ID

          /* Then pass your notification and notification ID into the BecoSDKInterface constructor. */
          becoSdk = new BecoSDKInterface( this, this, notification, notificationID);
          becoSdk.setCredentials( mServerHostname, mServerUsername, mServerPassword );

  }
```

>**NOTE:** If you pass the Beco Mobile SDK's notification through to an existing notification, be sure you also have a Notification Channel to assign it to that would be acceptable, from a UX perspective, to persist indefinitely. Convene recommends that your App has a dedicated Notification Channel specifically for the SDK in order to minimize user impact and provide greater flexibility (see the "Notification Channel" section below for details).

If you don't already have a notification in the App Bar, the following sections cover each individual step involved in creating one as it pertains to integrating the Beco Mobile SDK for Android.

This supplement, together with our Android Example App and source code (included in the Beco Mobile SDK installation package), provide comprehensive guidance for a successful integration and full functionality.

>**NOTE:** For the purposes of clear illustration, our Android Example App uses language such as "SDK" and "Beacons." You will want to adapt the language and terminology used in the relevant places to be appropriate for your App and functionality.

> Questions?

> [becosupport@convene.com](becosupport@convene.com)

### License
This document, the Beco Mobile SDK, and the included Android Example App are subject to the Beco Mobile SDK license agreement. A reference copy is included in the [LICENSE.md](https://github.com/becoinc/sdk_resources/blob/master/LICENSE.md) file. The governing copy of this agreement is available at [https://www.beco.io/files/sdk/sdk-license-agreement.pdf](https://www.beco.io/files/sdk/sdk-license-agreement.pdf).

## **CONTENTS**
1. Complete Notification Integration Logic
2. Notification Channel
3. Notification
4. Options in Earlier OS Versions
  - Interacting with the Lock Screen Notification
  - Manually Blocking/Hiding Notifications
5. Options in Android 9+
  - Interacting with the Lock Screen Notification
  - Automatically Blocking/Hiding Notifications
6. Legal

## **COMPLETE NOTIFICATION INTEGRATION LOGIC**

Before covering each piece of the integration individually, the full code of the required notification integration in the Beco Mobile SDK is presented below for review:

```
  private static final String NOTIFICATION_CHANNEL_ID = "Your channel ID here";

  private Notification createNotification() {

      createNotificationChannel(); //Create your own notification channel on API 26+

      Notification.Builder builder = new Notification.Builder(this);
      if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
          builder.setChannelId(NOTIFICATION_CHANNEL_ID);
      }
      builder.setSmallIcon(android.R.drawable.btn_star); //Set your custom app icon here
      builder.setContentTitle("Your customizable text here");

      Intent intent = new Intent(this, MainActivity.class);
      PendingIntent pendingIntent = PendingIntent.getActivity(
              this, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT
      );
      builder.setContentIntent(pendingIntent);

      return builder.build();

  }

  private void createNotificationChannel() {

      /* Create the NotificationChannel, but only on API 26+ because
         the NotificationChannel class is new and not in the support library. */
      if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {

          CharSequence name = "Your channel name here";
          String description = "Your channel description here";
          int importance = NotificationManager.IMPORTANCE_DEFAULT;
          NotificationChannel channel = new NotificationChannel(NOTIFICATION_CHANNEL_ID, name, importance);
          channel.setDescription(description);
          channel.setSound(null,null);
          /* Register the channel with the system; you can't change the importance
             or other notification behaviors after this. */
          NotificationManager notificationManager = getSystemService(NotificationManager.class);
          notificationManager.createNotificationChannel(channel);

      }

  }
```

## **NOTIFICATION CHANNEL**

```
  private void createNotificationChannel() {

      /* Create the NotificationChannel, but only on API 26+ because
         the NotificationChannel class is new and not in the support library. */
      if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {

          CharSequence name = "Your channel name here";
          String description = "Your channel description here";
          int importance = NotificationManager.IMPORTANCE_DEFAULT;
          NotificationChannel channel = new NotificationChannel(NOTIFICATION_CHANNEL_ID, name, importance);
          channel.setDescription(description);
          channel.setSound(null,null);
          /* Register the channel with the system; you can't change the importance
             or other notification behaviors after this. */
          NotificationManager notificationManager = getSystemService(NotificationManager.class);
          notificationManager.createNotificationChannel(channel);

      }

  }
```

The first step in a successful integration is creating a dedicated Notification Channel within your App specific to the Beco Mobile SDK. For details regarding the concept of Notification Channels first required in Android 8, please refer to the "Create and Manage Notification Channels" portion of the official Android Developer Documentation available at this link: [https://developer.android.com/training/notify-user/channels](https://developer.android.com/training/notify-user/channels).

>**NOTE:** Be sure you have reviewed the official Android Documentation on Notification Channels. There are extensive details and options available that may be important based on your specific implementation and App use case that are out of scope for this document beyond how they relate to integrating the Beco Mobile SDK.

Our Android Example App highlights what you can customize when creating a specific Channel for the Beco Mobile SDK, including the **channel name**, **channel ID**, and **channel description** shown in the above snippet and the Example App screenshots below. In the screenshots of the Example App, note where you can see the channel name and the channel description you create (BecoSdkExampleApp and its icon would be replaced with your App name and icon as you are creating the notification channel within your App, not our Android Example App).

*Android 8: settings-->notifications-->becosdkexampleapp*

![](https://github.com/becoinc/content_images/blob/master/android_integration_supplement/device_8_notification_menu.png)

*Android 8: settings-->notifications-->becosdkexampleapp-->your channel name here*

![](https://github.com/becoinc/content_images/blob/master/android_integration_supplement/device_8_channel_menu.png)

*Android 9: settings-->apps and notifications-->see all-->becosdkexampleapp*

![](https://github.com/becoinc/content_images/blob/master/android_integration_supplement/device_9_notification_menu.png)

*Android 9: settings-->apps and notifications-->see all-->becosdkexampleapp-->notifications*

![](https://github.com/becoinc/content_images/blob/master/android_integration_supplement/device_9_channel_menu.png)

>**NOTE:** By creating a Notification Channel specifically for notifications coming from the Beco Mobile SDK, you give your users the option to hide the notification either automatically (Android 9+--see the "Options in Android 9+" section) or manually (see the "Options in Earlier OS Versions" section) which creates a less invasive UX. Additionally, should users opt to hide the notification, they will *only* be hiding the notification associated with the Beco Mobile SDK. All other notifications that your App may deliver via another Channel(s) will continue to be delivered and available unless otherwise opted out by the user.

## **NOTIFICATION**

```
private Notification createNotification() {

      createNotificationChannel(); //Create your own notification channel on API 26+

      Notification.Builder builder = new Notification.Builder(this);
      if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.O) {
          builder.setChannelId(NOTIFICATION_CHANNEL_ID);
      }
      builder.setSmallIcon(android.R.drawable.btn_star); //Set your custom app icon here
      builder.setContentTitle("Your customizable text here");

      Intent intent = new Intent(this, MainActivity.class);
      PendingIntent pendingIntent = PendingIntent.getActivity(
              this, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT
      );
      builder.setContentIntent(pendingIntent);

      return builder.build();

  }
```

Once a dedicated Notification Channel has been created within your App, the next step is creating a persistent notification which will include an App Bar icon and a user-intractable lock screen notification.

As with a music player App, a persistent notification is a key element to how the Beco Mobile SDK within your App can continue to deliver consistent services even when your App is not in the Foreground State. However, *unlike* a music player App, in order to ensure a consistent data stream to power location-driven features as users move around your physical spaces, we have engineered the Beco Mobile SDK with the ability to run even if your App is put in to the Killed State (swiped closed) and in this state the notification will continue to display on the lock screen as well as on the App Bar.

*Lock screen notification for our Android Example App*

![](https://github.com/becoinc/content_images/blob/master/android_integration_supplement/device_8_lockscreen.png)

The App Bar icon is completely customizable by you and can be anything you select to match the existing branding of your App. In our Android Example App, we use a star icon. This icon will appear on the lock screen notification and as part of the Always On Display UI (if that feature is enabled by the user).

On the lock screen, a user can interact with the notification and so the text and the content of the notification is customizable by you - including how the user might hide or mute the notification (see the following two sections on OS version specific options, as this dictates what users can and can't do from an interactivity perspective with muting/hiding the notification depending on the version they are running).

## **OPTIONS IN EARLIER OS VERSIONS**

#### Interacting with the Lock Screen Notification

In Android 8, you have the ability to nest a Notification Channel allow/disallow toggle, among other things, in the actual lock screen notification UI as shown here:

*Android 8. Note the customizable description and the toggle to allow/disallow notifications from the Channel.*

![](https://github.com/becoinc/content_images/blob/master/android_integration_supplement/device_8_lockscreen_2.png)

In Android 7 and earlier, as this pre-dates the requirement of assigning notification(s) to a channel(s), tapping the lock screen notification routes the user to the settings for the App's notifications as shown here:

*Android 7. Note that pressing the notification takes you to settings where you can allow/disallow notification in the pre-Channel.*

![](https://github.com/becoinc/content_images/blob/master/android_integration_supplement/android_7_notification_menu.png)

#### Manually Blocking/Hiding Notifications

In Android 8 and earlier, there is not an ability to give users a one-tap way to mute/hide the notification from a specific Channel. Instead, using a combination of the lock screen UI that is available in the version Android the user is running, as well as by navigating to settings, the user is able to manually do this.

>**NOTE:** It is expected from our testing that once the notification coming from the Beco Mobile SDK is muted/hidden, there may be a system notification generated that replaces it. The user can hide this notification as well using the same methodology.

## **OPTIONS IN ANDROID 9+**

#### Interacting with the Lock Screen Notification

As in Android 8, you have the ability to nest a Notification Channel allow/disallow toggle, among other things, in the actual lock screen notification UI as shown here:

![](https://github.com/becoinc/content_images/blob/master/android_integration_supplement/device_9_lockscreen_2.png)

Note that the option you see here is different than in Android 8. Please see the next section for details.

#### Automatically Blocking/Hiding Notifications

Beginning in Android 9, a user has the ability to automatically hide a notification(s) on a per Channel basis with a single selection, in addition to the existing Channel-specific notification preferences introduced in Android 8. This presents an opportunity to give users a choice, via UI, on whether they actually see the notification within the Channel you create for the Beco Mobile SDK.

Our Android Example App provides a demonstration of giving the user a choice to hide the notification from that Channel and guiding them to the appropriate settings window to do so. By clicking on the info (i) icon within the Android Example App (see screenshots below), a popup appears asking users if they would like to hide notifications from the SDK. If "Yes" is selected, a deep link flow first provides instructions and then auto-routes the user to the correct settings sub-menu for the appropriate Notification Channel:

*Tap the info (i) icon*

![](https://github.com/becoinc/content_images/blob/master/android_integration_supplement/device_9_i_1.png)

*Select "yes"*

![](https://github.com/becoinc/content_images/blob/master/android_integration_supplement/device_9_i_2.png)

*Select "ok"*

![](https://github.com/becoinc/content_images/blob/master/android_integration_supplement/device_9_i_3.png)

*User deep linked to the sub menu for the appropriate Notification Channel*

![](https://github.com/becoinc/content_images/blob/master/android_integration_supplement/device_9_i_4.png)

Note how pressing the info (i) icon on a device running Android 8 or earlier results in the following message:

![](https://github.com/becoinc/content_images/blob/master/android_integration_supplement/device_8_i.png)

If you would like to simply re-create the same high-level flow as seen in the Android Example App, replacing the (i) icon with your own implementation, you can use the following line of code to do so:

```
/* questionDialogText is a customizable text String for the initial question to the user asking if they would like to hide this application's notification.
informativeDialogText is a customizable text String for instructions as to how to turn off notifications once directed to the Notification Channel's setting screen.
null can be passed for both values to use default text. */
BecoSDKInterface.displayNotificationSettingsInfoDialog(activity, questionDialogText, informativeDialogText, NOTIFICATION_CHANNEL_ID);
```

If you would rather create a completely custom experience, the following line can be used to direct the user to the Notification Channel's settings screen wherever and however you would like that to occur within your App:

```
BecoSDKInterface.directUserToNotificationSettings(context, NOTIFICATION_CHANNEL_ID);
```  

>NOTE: This is where the concept of having distinct Notification Channels within your App becomes important and useful, as previously noted above: you can create a flow that guides the user to hide only notifications from the specific channel you have dedicated to the SDK.

## **LEGAL**

#### Export Statement

You understand that the Software may contain cryptographic functions that may
be subject to export restrictions, and you represent and warrant that you are
not located in a country that is subject to United States export restriction or
embargo, including Cuba, Iran, North Korea, Sudan, Syria or the Crimea region,
and that you are not on the Department of Commerce list of Denied Persons,
Unverified Parties, or affiliated with a Restricted Entity.

You agree to comply with all export, re-export and import restrictions and
regulations of the Department of Commerce or other agency or authority of the
United States or other applicable countries. You also agree not to transfer,
or authorize the transfer of, directly or indirectly, the Software to any
prohibited country, including Cuba, Iran, North Korea, Sudan, Syria or the
Crimea region, or to any person or organization on or affiliated with the
Department of Commerce lists of Denied Persons, Unverified Parties or
Restricted Entities, or otherwise in violation of any such restrictions or
regulations.

#### Trademark Notes

* Android is a trademark of Google Inc.
