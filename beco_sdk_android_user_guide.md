![](https://beco.io/wp-content/themes/beco.io/assets/img/beco_logoblue@2x.png)


# **BECO MOBILE SDK FOR ANDROID™ USER GUIDE**
This document is designed for Beco hardware and cloud services customers looking
to augment their Android applications with location awareness and analytics.

The Beco SDK provides developers a rapid path towards Beco Beacon integration.
While other solutions focus on providing a light wrapper to existing Android
beacon APIs, the Beco SDK for Android has taken the approach of providing a
turn-key solution with a greatly simplified API, designed to let users
leverage the power of the Beco Cloud service while being as easy to integrate
as Beco Beacons are to deploy.

Here, we will cover the fundamentals of configuring and building your project
with our SDK in Android Studio. Be sure to also read our SDK Walkthrough
document, which focuses on the various features and functions of the SDK
itself for the purposes of integration.

Additional resources are available at our online developer site [dev.beco.io](dev.beco.io). Please contact us at the email listed below if you require the password to log in.

>**Questions?**

>[becosupport@convene.com](becosupport@convene.com)

### Version
The current release version of the Beco SDK for Android is v1.10(0) (20190214).

### License
This document, the Beco SDK and the included Android example App are subject to
the Beco SDK license agreement. A reference copy is included in
the [LICENSE.md](https://github.com/becoinc/sdk_resources/blob/master/LICENSE.md) file. The *governing copy* of this agreement is available at [https://www.beco.io/files/sdk/sdk-license-agreement.pdf](https://www.beco.io/files/sdk/sdk-license-agreement.pdf).

## **CONTENTS**
1. Identification
2. Release Notes and Directions
3. Packaging
4. Dependencies
  * Hardware
  * Software
  * Third Party Libraries
  * Environment
5. Special Required Implementation Steps
  * Add String to Ensure SDK Scanning Persistence in a Multi-App Scenario
  * Call `startScan` at Each Instance of App State Transition from Background (or Killed/Swiped) to Foreground
  * Leverage Notification Framework to Enable Background and Killed State Functionality (v1.10.0+)
  * Disable Android Studio's Instant Run Feature
6. Project Configuration
  * Permissions and Features
  * Proguard
7. Build Setup
8. Building the Example App
9. Beco SDK API and Usage
  * Standard API
  * Advanced Performance Settings
  * Main Interface
  * SDK Behavior Details
10. Document Revision Summary
11. Legal
  * Export Statement
  * Trademark Notes

## **IDENTIFICATION**
The SDK is provided to developers as four parts. The first is a distribution
archive containing pre-built binary Android Archive (AAR).
The archive is named in the following format:

`libbecoandroidsdk-release-{date stamp}-{revision}.aar`

The second is a pre-compiled, self-signed Android Package (APK) of the Example Application.

The third is the Java source code for the Android Example Application.

The fourth and final part is this document and the generated SDK javadocs.

The internal version and build of this software are 1.10(x) with a package
identifier of `io.beco.sdk.android.bas`. The "x" build number is the patch
level and is subject to change as minor, compatible changes and fixes are made.
This complies with semantic versioning [(http://semver.org/)](http://semver.org/).

## **RELEASE NOTES AND DIRECTIONS**
Release | Notes and Directions
------------ | -------------
v1.9(16) to v1.10.0 | Added ability for the SDK to run in either the Background or Killed (swiped closed) State. Upgraded to current stable AltBeacon library release v2.16.1 (please update your build.gradle file to use the AltBeacon library v2.16.1 `org.altbeacon:android-beacon-library:2.16.1`. Battery performance optimizations. Overall stability improvements. **Notice:** There are specific implementation steps required for full SDK functionality. See the [Special Required Implementation Steps](#special-required-implementation-steps) section of this document for instructions. **Notice:** There is a known limitation in this release caused by Android OS that prevents Background/Killed State functionality to restart automatically upon a device restart or a device power down and power up cycle. We are currently building a way around this to include in a follow up patch release.
v1.9(15) to v1.9(16) | Incorporated the now stable AltBeacon library release v2.15.4, which now natively includes our indepenent fix for the crash we discovered in the v2.15.3-beta1 (please update your build.gradle file to use the AltBeacon library version 2.15.4 `org.altbeacon:android-beacon-library:2.15.4`). Added support for multiple Beco SDK integrated applications running simultaniously on a single mobile device. **Notice:** There are specific implementation steps that must be followed for this to work. See the "Special Required Implementation Steps" section of this document below for instructions. **Notice:** Because AltBeacon v2.15.4 incorporates our independent fix natively, the special setup requirements for the Beco SDK v1.9(15) release are no longer needed: AltBeacon lib no longer needs to be a local aar (or propped) and the custom gradle line is no longer needed. (customers who have never integrated Beco SDK v1.9(15) may ignore this notice and proceed as normal). **Notice:** Due to a limitation still present in the AltBeacon code, unexpected scanning and Beacon detection behavior occurs when an App returns to the Foreground state from the Background state. See the "Special Required Implementation Steps" section of this document below for instructions on the workaround.
v1.9(14) to v1.9(15) | Fixed multiple memory issues in the SDK. Fixed multiple crashes in the SDK. Forked v2.15.3-beta1 of AltBeacon to incorporate our own fix for a crash discovered in the BeaconService class, as well as incorporate a fix for known memory issue in v2.15.1 (v2.15.2 was found to have incomparable positioning behavior to the v2.15.3-beta1, even though it fixed the memory issue). **Notice:** Because we forked the AltBeacon library to resolve the issue we found, the AltBeacon lib is now going to be a local aar (or propped). It will need to be added to your "libs" folder and you will also need to replace your gradle line with this one: compile files('libs/android-beacon-library-release.aar’). We also discovered a limitation to the current AltBeacon code which creates unexpected scanning and beacon detection behavior when the App returns to the Foreground state from the Background state. Calling startScan at each instance of this state transition resolves the issue as a temporary workaround until this is addressed in their code, and we have provided an example implementation of the fix in the MainActivity of our Example App (included with the other release artifacts). See the "onStart()" method.
v1.9(13) to v1.9(14) | Performed a concurrency refactor to fix an issue where the SDK might cause the host App UI to freeze resulting in an Android "Not Responding" dialog, or the App to crash under certain circumstances. Maintenacnce around altbeacon integration and Beacon detection. Maintenance to improve sugarORM integration. Resolved an issue where startScan may not work on first try after App launch. Fixed an issue where calling stopScan did not hault Beacon scanning on newer versions of Android. Updated build system to Android Gradle v4.6. **Notice:** Please update your build.gradle file to use the AltBeacon library version 2.15.1 `org.altbeacon:android-beacon-library:2.15.1`.  
v1.9(12) to	v1.9(13) | Fixed thread race condition in messaging	code. Code cleanup and dependency update. Please update your build.gradle file to use the updated versions of our third-party dependencies. Refer to the example application `build.gradle`	file for exact version callouts. This release removes support for Android versions prior to 5.1. API level 22. Earlier releases will not work.
v1.8.6 to v1.9.12 | Increase foreground responsiveness and provide initial support for Android 8, Oreo. **Notice:** Please update your build.gradle file to use the AltBeacon library version 2.12.2 `org.altbeacon:android-beacon-library:2.12.2`. **Notice:** This release deprecates support for Android versions prior to 5.0. Earlier releases may work, but are not tested and validated.
v1.7.5 to v1.8.6 | Fix hit stream reporting in the background. **Notice:** Please change your gradle file to point to the altbeacon library version `org.altbeacon:android- beacon-library:2.8.1` because this fixes an issue with Bluetooth stability.
v1.7.4 to v1.7.5 | Fix position stability in Android and upgrade the altbeacon library to the latest stable version of 2.10. **Notice:** Please update your gradle file with this version of the altbeacon library `org.altbeacon:android-beacon-library:2.10`.
v1.7.2 to v1.7.4 | Fix a serialization bug to handle the latest API changes where fields were added. Update build system to Android Gradle stable v2.3.2.
v1.6.1 to v1.7.2 | Update build system to Android Gradle Stable v2.2.3. Fix a serialization bug related to new backend features. **Notice:** Users must upgrade to be compatible with current generation Beco Cloud services.
v1.5.0 to v1.6.0 | Change build system to Android Gradle Stable v2.2.2 and CMake 3.4.1 or later. Update to support Android Studio v2.2.2. Enhance Beco API support to Generation 4 Beco Cloud Services. Earlier releases are not forward compatible with Generation 4. This release is compatible with both Generation 3 and Generation 4 Beco Cloud Services.

## **PACKAGING**
The Beco SDK is a pre-built Android archive compiled from Java 8 code with
native libraries for enhanced performance. We have provided distinct builds
of the SDK, which should be compatible with the majority of Android
application development use cases:

1. Release build for arm, arm7, arm8, mips, mips-64, x86, x86_64

The AAR and APK have been generated as universal binaries, where each
library file has support for multiple platforms and architectures.

## **DEPENDENCIES**

#### Hardware
The Beco SDK requires a Bluetooth 4.0 (BLE, Bluetooth Smart) compatible
 Android device running at least Android Lollipop (5.1), API Level 22+.

We have tested extensively on the following hardware models and Android OS versions:

- Samsung Galaxy S6 (Android 7.0)
- Samsung Galaxy S7 (Android 7.0)
- Samsung Galaxy S8 (Android 8.0)
- Samsung Galaxy S9 (Android 8.0)
- Google Pixel 2 (Android 9.0)

We expect other hardware models and earlier Android OS versions (down to Lollipop 5.1) to work similarly,
but they have not been tested by Beco.

**Notice:** Beco strongly recommends devices running Android 6.0.1
(Marshmallow, API 23) or later due to substantial Bluetooth bugs in
older versions of the operating system. We observe that that Android
does frequently exhibit device and OS specific behavior and interested
parties should contact us for further details.

#### Software
This release of the Beco SDK requires version 23 or later of the Android SDK.
We support Java projects, although Kotlin projects should work.

The SDK was developed using Android Studio 3.2 with the Gradle 4.6 build system.

Both the SDK and the Example Application use the stable Android Gradle plug-in
v3.2.0. The SDK uses an external CMake based build system using the Android NDK.

#### Third Party Libraries
The Beco SDK depends on the following 3rd party libraries:
* `org.springframework.android:spring-android-rest-template:2.0.0.M4`
* `com.fasterxml.jackson.core:jackson-databind:2.3.2`
* `org.altbeacon:android-beacon-library:2.16.1`
* `com.github.satyan:sugar:1.5`

These libraries can be automatically obtained from the standard jcenter repositories via Gradle.

#### Environment
The Beco SDK is designed to work with Beco Beacons exclusively.
Generic BLE beacons or iBeacon devices from other vendors are not supported.

The Beco SDK requires an active Internet connection to obtain necessary
information from the Beco servers, with the exception of users with private
Beco Cloud servers. These users only need to have an active network connection
(typically via WiFi) to their Beco server.

Finally, you must register ("check-in") your Beco Beacons using the Beco
Setup App for iOS prior to using the beacons with the SDK.

## **SPECIAL REQUIRED IMPLEMENTATION STEPS**

>#### Add String to Ensure SDK Scanning Persistence in a Multi-App Scenario

It has been determined that Beacon scanning will not function on Android OS if there are ever two Applications installed on the same mobile device that both have the Beco SDK integrated. 

In order to ensure the continued function of the Beco SDK in your Application should this scenario ever be true for a user, a string must be added within the `defaultConfig` section of your Application's `build.gradle` file where `applicationId` is a variable holding your Application's ID. The following is an example:

```
defaultConfig { 
	... 
	
	resValue 'string', 'io_beco_sdk_containing_app_id', applicationId 
}
```

>#### Call `startScan` at Each Instance of App State Transition from Background (or Killed/Swiped) to Foreground

Currently, there are several user-configurable settings in Android OS that may adversely affect the performance of the Beco Mobile SDK or, in rare cases, cause the integration with your App to break. This is due to a combination of Android OS related limitations as well a limitation we have identified in the AltBeacon library. The specific scenarios are:

- Power Saving Mode (Max) is enabled for any length of time (Samsung devices only)
- The host App's permission to access Location Services is revoked at any time and then granted again (any device)

We are undertaking work with AltBeacon directly to understand the code changes necessary in their lib to resolve for a future release. We are also working to understand how the Beco Mobile SDK can be less impacted when a device exists Samsung's Power Saving Mode (Max) and returns to the normal.

For now, we have identified that calling `startScan` at each instance of App state transition from Background or Killed/Swiped to Foreground maintains expected functionality of the Beco Mobile SDK within your App once the App is brought to the Foreground at least once after either of the two scenarios above.

An example implementation of this in the `MainActivity` --> `onStart()` method is the following:

```
@Override
    protected void onStart()
    {

        super.onStart();
        Log.d( TAG, "Main Activity Starting." );

        final SharedPreferences sp = this.getPreferences( MODE_PRIVATE );
        boolean scanStarted = sp.getBoolean( sfScanStartedKey, false );

        if( scanStarted )
        {
            Log.i( TAG, "Restarting Scan" );
            try
            {
                SharedPreferences.Editor spEdit = sp.edit();
                spEdit.putBoolean( sfScanStartedKey, false ); // make sure the scan restarts.
                spEdit.commit();
                this.becoSdk.startScan();
            }
            catch (CredentialsNotSetException e)
            {
                reportError( BecoSDKErrorCode.CredentialMismatch );
            }
        }

    }
```

>#### Leverage Notification Framework to Enable Background and Killed State Functionality (v1.10.0+)

Please refer to our [Android Integration Supplement](https://github.com/becoinc/sdk_resources/blob/master/beco_sdk_android_integration_supplement.md) for detailed instructions on enabling the full functionality of the Beco Mobile SDK v1.10.0+ in the Background and Killed (swiped closed) States.

>#### Disable Android Studio's Instant Run Feature

We have found that the `Instant Run` feature in Android Studio 2.1.2, and
possibly later versions, has certain bugs which prevent some of our third
party dependencies from initializing correctly. Please disable
the `Instant Run` feature of the IDE in order to properly integrate the Beco SDK.

## **PROJECT CONFIGURATION**
In order for your app to properly integrate the Beco SDK, a
few `AndroidManifest.xml` settings and permissions are required.

#### Permissions and Features
In the main section of your app’s main xml configuration, you need to declare
the following for the Beco SDK (or verify that the build system merged
the provided manifest from the AAR):

XML Tags
![](https://github.com/becoinc/content_images/blob/master/SDK_user_guides/Android_xml_tags.png)

#### Proguard
Certain customers may wish to enable proguard to obfuscate their code.
In order for the Beco SDK to properly function, certain classes must be
excluded from proguard using a `proguard-rules.pro` file:

![](https://github.com/becoinc/content_images/blob/master/SDK_user_guides/Android_proguard_rules.png)

## **BUILD SETUP**
This section discusses setting up Gradle to compile and link against the Beco SDK.

1. Place the SDK AAR file into the *libs/* folder within your main application model.
This is already done in the Example Application.

2. Add a `flatDir` section to your main application `build.gradle` file within
the repositories section. You may put this in your top-level repositories
section, or anywhere else where it is processed during the configuration
phase of the Gradle build. See the Example Application `build.gradle` for details.

3. Within your `build.gradle` file in the top-level dependencies section,
add a compile dependency on the AAR file:
`compile( name: 'libbecoandroidsdk-release', ext: 'aar' )`

## **BUILDING THE EXAMPLE APP**
The Beco SDK for Android Example App comes with a build environment all
set up for you. Simply unpack the distribution tar-ball, change to the
directory `becosdkexampleapp` and run

`./gradlew build`

Assuming you have a live internet connection, this will download the
proper version of Gradle and then download the necessary dependencies
and compile the application to a self-signed APK file for installation
on your Android device. The APKs will be in the *build/outputs/apk* directory
relative to the source code.

Our development systems use Mac OS 10.13.x, however any Unix style OS where
 Gradle 4.6 is supported should work. It may be possible to build on Windows
 as well, but Beco does not test or support this environment for development.

## **BECO SDK API AND USAGE**
In keeping with other aspects of the Beco System, the SDK has been designed
to have a very simple, minimal mandatory interface with a few optional
aspects to provide enhancements.

#### Standard API

![](https://github.com/becoinc/content_images/blob/master/SDK_user_guides/Android_standard_API.png)

#### Advanced Performance Settings

[**NOTE**]: The SDK has pre-set default values for these settings to
 ensure reliable indoor positioning performance. The cases where these
 defaults may need adjustment are extremely rare, and even then should only
 be modified after review with our Support Team in order to preserve system
 performance. Open a ticket via [becosupport@convene.com](becosupport@convene.com)
 if you experience unexpected location performance.
 
 See our SDK Walkthrough document for more details:

![](https://github.com/becoinc/content_images/blob/master/SDK_user_guides/Android_advanced_performance_tuning_calls.png)

#### Main Interface
The main interface between your App and Beco location data is via the
delegate parameter in the `BecoSDKInterface` constructor specified when it
is created by your application.

It is up to your application to define a delegate that implements
the `BecoSDKDelegate` interface. The interface defines a set of methods
that you must implement in order to receive events from the Beco SDK.

Detailed information on each of the interface methods, thread behavior
and other information is provided inline with the interface definition headers
and javadocs included with the Beco SDK. Please refer to the source level
documentation and Sample Application for more information.

It is *required* that SDK customers use the `registerHandset` API call
when initializing the SDK, as per the example application.
Current versions of the Beco Cloud will reject all data from
unregistered handsets.

#### SDK Behavior Details
Once you call `startScan` the SDK will continue to scan for beacons
until `stopScan` is called. The SDK will automatically stay running in the
background and wake the phone up when possible to process beacon information
when permitted by the OS. The SDK is designed to automatically stay
running in the background if the App is swiped closed.

We are aware that currently background behavior on Android is heavily dependent on OS version and the settings a user chooses to enable. Contact us for specific details.


## **DOCUMENT REVISION SUMMARY**
Revision | Summary of Changes
------------ | -------------
1 | Initial document version.
2 | Adjust for hybrid build system and update for SDK v1.1.8.
3 | Update for version SDK v1.3.0.
4 | Update for version SDK v1.6.0.
5 | Update for version SDK v1.7(2).
6 | Update for version SDK v1.7(4).
7 | Update for version SDK v1.7(5).
8 | Update for version SDK v1.8(6).
9 | Update for version SDK v1.9(12).
10 | Update for SDK v1.9(13).
11 | Update for SDK v1.9(14).
12 | Update for SDK v1.9(15).
13 | Update for SDK v1.9(16).
14 | Update for SDK v1.10(0).

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
