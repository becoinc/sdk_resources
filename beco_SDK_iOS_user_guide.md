![](https://beco.io/wp-content/themes/beco.io/assets/img/beco_logoblue@2x.png)


# **BECO MOBILE SDK FOR iOS USER GUIDE**
This document is designed for Beco hardware and cloud services customers looking to augment their iOS applications with location awareness and analytics.

The Beco Mobile SDK provides developers a rapid path towards Beco Beacon integration. While other solutions focus on providing a light wrapper to existing iOS beacon APIs, the Beco Mobile SDK for iOS has taken the approach of providing a turn-key solution with a greatly simplified API, designed to let users leverage the power of the Beco Cloud service while being as easy to integrate as Beco Beacons are to deploy.

Here, we will cover the fundamentals of configuring and building your project with the Beco Mobile SDK in Xcode. Be sure to also read our SDK Walkthrough document, which focuses on the various features and functions of the SDK itself for the purposes of integration.

Additional resources are available at our online developer site [dev.beco.io](http://dev.beco.io).

>**Questions?**

>[becosupport@convene.com](becosupport@convene.com)

### Version
The current Swift 5 release version of the Beco SDK for iOS is v3.7.3 (20190603).

### License
This document, the Beco SDK and the included sample iOS Apps are subject to the Beco SDK license agreement. A reference copy is included in the [LICENSE.md](./LICENSE.md) file. The *governing copy* of this agreement is available at [https://www.beco.io/files/sdk/sdk-license-agreement.pdf](https://www.beco.io/files/sdk/sdk-license-agreement.pdf).

## **CONTENTS**
1. Identification
2. Release Notes and Directions
3. Packaging
4. Dependencies
  * Hardware
  * Software
  * Third Party Libraries
  * Environment
4. Project Configuration
  * Capabilities
  * Info.plist
5. Build Setup
6. Building the Example App
7. Beco SDK API and Usage
  * Standard API
  * Advanced Performance Settings
  * Main Interface
  * SDK Behavior Details
8. Document Revision Summary
9. Legal
  * Export Statement
  * Trademark Notes

## **IDENTIFICATION**
The Beco Mobile SDK is provided to developers as two parts. The first is a distribution archive containing pre-built binary iOS Frameworks. The archive is named in the following format:

`beco_ios_sdk_v3_7_3-{date stamp}-{revision}.tar.gz`

The second part is this document.

The internal version and build of this software are 3.7(x) with a bundle identifier of `com.beco.BecoSDK`. The (x) build number is the patch level and is subject to change as minor, compatible changes and fixes are made. This complies with semantic versioning [(http://semver.org/)](http://semver.org/).

## **RELEASE NOTES AND DIRECTIONS**
Release | Notes and Directions
------------ | -------------
**Notices** | SDK Users who wish to submit apps to the Apple App Store will need to add the following shell script execution to their build process: `bash"${BUILT_PRODUCTS_DIR}/${FRAMEWORKS_FOLDER_PATH}/BecoSDK.framework/strip-frameworks.sh"` (see the Build Setup section below). In iOS 11/Xcode 9 and newer, several additional privacy keys are required in the Info.plist (see  Project Configuration section below).
v3.7(2) to v3.7(3) | Updated SDK to Swift 5 for compatibility with Xcode 10.2(1) and ABI stability moving forward. Minor syntax changes based on requirements in Swift 5 (language internationalization and new hash handling). No further code or performance changes were made in this release; code and performance/feature set are identical to v3.7(2).
v3.7(2) Xcode 10.0/Swift 4.2 Compatibility Update. | Updated SDK to Swift 4.2 for compatibility with Xcode 10.0. No feature or performance changes or enhancements were made, and so the Beco SDK version number has been retained to reflect this. This release is purely about adding support to the existing SDK version for Apple's most recent tooling. **NOTE: Given that Swift is not yet ABI stable, if you are using Xcode 9.4 and Swift 4.1 in your build, please use the previous release of v3.7(2) date stamped 20180825 in the SDK release archive and referenced below.**
v3.6(4) to v3.7(2) | Code build process changes: Make 'BecoSDKDelegate' a weak reference. Remove XCGLogger dependency.    Changes to decrease power consumption when app is in the background: Change maximum wake time from 14s to 11s. Allow the app to sleep sooner. Adjust sleep timer when position isn't moving. Add new internal variable 'scanInterval' which adjusts how aggressively the SDK wakes the device in the background. The default value for 'scanInterval' is 4 with a range of 0 to 300, with higher numbers being less aggressive. Xcode 9.4 and Swift 4.1 compatibility.
v3.5(15) to v3.6(4) | Consider "grouped" beacons as one "place" when registering hits. Support for battery powered beacons. Save 'userData' from server in 'Place' and 'Location' object returned in callback. Fixed a crash bug.
v3.5(9) to v3.5(15) | Improved beacon detection. Make 'username' case-insensitive. Enhancements to support future server upgrades. Cleanup '@objC' runtime warnings.
v3.4(0) to v3.5(9) | Battery life improvements. Improved tracking accuracy. Assign a new HSID with each 'userName'. Add ".authorizedWhenInUse" to valid states to run. iOS 11.x / Xcode 9.2 / Swift 4.0.3 compatibility.
v3.2(0) to v3.4(0) | Battery life and general performance improvements. iOS 11 / Xcode 9 / Swift 3.2 - 4.0 compatibility.
v3.1(6) to v3.2(0) | Improve behavior in spotty internet conditions (SDK-60). Correct App Store submission problems (SDK-61).
v3.1(5) to v3.1(6) | Battery life improvements.
v3.1(4) to v3.1(5) | Ensure delegate is called on location status changes (SDK-55).
v2.8(0) to v3.1(4) | Remove Realm and replace with CoreData/SQLite (SDK-43). Fix memory and storage leaks, update XCGLogger (SDK-42, SDK-41). Code cleanup and maintenance (SDK-47): **Minor Breaking Changes:** The Beco SDK Delegate protocol changed to include an additional method. startScan() in BecoSDKInterface no longer requires a closure parameter. You must call `self.sdk.hostname = hostname` before any calls to `setCredentials()`. In almost all cases, hostname will be defined similar to `let hostname = “api.beco.io”`. This is a limitation for this release and will be relaxed in a future release. Refer to the Example Applications for details. **Notice:** Users upgrading are strongly encouraged to review this entire document as the linked libraries and frameworks are different. **Update Quick Start Directions:** **1.** Remove `Realm.framework` and `RealmSwift.framework` from the “General” section of your application. **2.** Remove the `Realm strip-paths.sh` script from your “Build Scripts” section. **3.** Update XCGLogger. **4.** Add `ObjcExceptionBridging.framework` and the upgraded XCGLogger.![](https://github.com/becoinc/content_images/blob/master/SDK_user_guides/iOS_add_XCGLogger.png)
v2.6(2) to v2.8(0) | Release support for Swift 3.1, Xcode 8.3.1 and Realm 2.5.1 and Bitcode. Fix issue with SDK startup where in some cases when a user is prompted for Location Services authorization, scanning would not resume after user authorized access until the SDK was re-started.
v2.5(2) to v2.6(2) | Beta release with Swift 3.1, Xcode 8.3.1 and Realm 2.5.1 support (SDK-38). Enable support for Bitcode. **Users will need to upgrade their Realm Frameworks to 2.5.1 from 2.1.0 and their Xcode to 8.3.x with Swift 3.1 support.**
v2.2(1) to v2.5(2) | Documentation and example updates to clarify proper SDK integration procedures with respect to person identifiers. (GIA-36). Behavior consistency enhancements during long running backgrounded operation. Support enhanced features of new Generation Beacon Hardware. New Hardware is backwards compatible.

## **PACKAGING**
The Beco SDK is a pre-built iOS Framework compiled from Swift 4 code. We have provided two distinct builds for each version of the SDK, which should be compatible with the majority of iOS applications development use cases:

1. Debug build for arm64, armv7, i386, x86_64
2. Release build for arm64, armv7, i386, x86_64

**Future Deprecation Warning:** Apple has indicated that 32-bit support (devices prior to the iPhone 5s) will be dropped from iOS as of iOS 11. We will discontinue armv7 *and* iOS 9 support in a future release.

The frameworks have been generated as universal binaries, where each library file has support for multiple platforms and architectures.

The Beco Mobile SDK supports both iPhone and Simulator environments, although BLE devices have limited CoreLocation support when in simulation. We’ve found the simulator useful for testing UI code, but not usable for testing the Beco System.

These are inside the tar archive in folders named using
`$(CONFIGURATION)-universal`.
Where `$(CONFIGURATION)` is the environment variable set by Xcode for the current build type.

For example, the Debug build of the Beco Mobile SDK for the iPhone Simulator will be in the `Debug-universal/BecoSDK.Framework` directory that is present after extracting the archive. Bitcode generation is supported in this release. The Beco Mobile SDK includes embedded bitcode.

## **DEPENDENCIES**

#### Hardware
The Beco Mobile SDK requires a Bluetooth 4.0 (BLE, Bluetooth Smart) compatible iOS device. We have tested extensively on the following hardware models and iOS versions: 

- iPhone 6 (iOS 11.2.1)
- iPhone 7 (iOS 11.3 and iOS 12)
- iPhone 7 Plus (iOS 12)
- iPhone 8 (iOS 11.4 and iOS 12)
- iPhone X (iOS 11.4)
- iPhone XS (iOS 12.3)
- iPhone XS Max (iOS 12)

We expect other iOS devices to work similarly, but they have not been tested by Convene and are not officially supported.

#### Software
This release of the Beco Mobile SDK for iOS requires Xcode 10.2(1). We support Swift 5 and Objective-C projects. Headers are provided for Objective-C use, and Beco provides an example Objective-C based application using the Beco Mobile SDK. Internally we are using Xcode 10.2(1) and Swift 5.

The `Deployment Target` setting is set to 9.0, thus requiring iOS 9.0 or newer. 

Convene officially tests with and supportes iOS 11.x and iOS 12.X in order to stay current with Apple's release cycle and user adoption. We expect earlier versions of iOS back to iOS 9.x to work similarly, but they are no longer tested or officially supported. 

The Beco Mobile SDK has Universal (iPhone and iPad) support.

#### Third Party Libraries
The previous release of the Beco Mobile SDK, v3.7(2), removed dependancy on third party libraries.

#### Environment
The Beco Mobile SDK is designed to work with Beco Beacons exclusively. Generic BLE beacons or iBeacon devices from other vendors are not supported.

The Beco Mobile SDK requires an active Internet connection to obtain necessary information from the Beco servers, with the exception of users with private Beco Cloud servers. These users only need to have an active network connection (typically via WiFi) to their Beco server.

Finally, you must register ("check-in") your Beco Beacons using the Beco Setup App for iOS prior to using the Beacons with the Beco Mobile SDK.

## **PROJECT CONFIGURATION**
In order for your app to properly integrate the Beco Mobile SDK, a few Info.plist settings and permissions are required.

#### Capabilities
In the capabilities section of your app’s main target configuration, you need to declare the following for the Beco Mobile SDK:

![](https://github.com/becoinc/content_images/blob/master/SDK_user_guides/iOS_capabilities.png)

#### Info.plist
In order for iOS to properly enable your app with the Beco Mobile SDK, the following keys must be set in the Info.plist. This is typically done via Xcode in the “Info” section of the target configuration:

![](https://github.com/becoinc/content_images/blob/master/SDK_user_guides/iOS_info_plist_keys_UI.png)

Note that the text shown above for each privacy key is sample only and should be configured as is most appropriate for your App and users.

In text form (viewed as source code) these keys are as follows:

![](https://github.com/becoinc/content_images/blob/master/SDK_user_guides/iOS_info_plist_keys_source_view.png)

## **BUILD SETUP**
This section discusses setting up Xcode to compile and link against the Beco Mobile SDK for a Swift Project. *Setup for Objective-C projects is similar and extra steps are noted inline*.

#### Step 1
Extract the Beco Mobile SDK into a folder. We recommend that you place it in version control with any other third-party libraries/frameworks.

#### Step 2
Select the App project in Xcode. On the “Info” screen, add `BecoSDK` to the `Embedded Binaries` section.

![](https://github.com/becoinc/content_images/blob/master/SDK_user_guides/iOS_build_setup_step2-1.png)

Click the "**+**" in the `Embedded Binaries` section.

![](https://github.com/becoinc/content_images/blob/master/SDK_user_guides/iOS_build_setup_step2-2.png)

Use "Add Other."

![](https://github.com/becoinc/content_images/blob/master/SDK_user_guides/iOS_build_setup_step2-3.png)

Browse to the location of the BecoSDK.

![](https://github.com/becoinc/content_images/blob/master/SDK_user_guides/iOS_build_setup_step2-4.png)

Create the link between the project and the frameworks.

**Important:** Ensure `Copy items if needed` is *checked*.

#### Step 3
Verify you set the proper Info.plist settings as described in the "Project Configuration" section.

#### Step 4-1
Import the BecoSDK swift module into your source code and begin using the SDK as described below.

#### Step 4-2 (OBJECTIVE-C ONLY)
If incorporating the Beco Mobile SDK into an `Objective-C` App project, set the `Always Embed Swift Standard Libraries` Build Flag for the App Target:

  ![](https://github.com/becoinc/content_images/blob/master/SDK_user_guides/iOS_swift_libraries_build_flag.png)
**WARNING:** Do not set this flag for Framework Targets, *only* the top-level app target. Otherwise, your app will fail app-store upload validation checks due to multiple copies of the Swift Runtime support libraries.

#### Step 5
**Notice:** In order to work around an app store submission bug, documented in Radar (http://www.openradar.me/radar?id=6409498411401216), you need to add the following shell script execution to the “Build Phases” section of your app build:

`bash" ${BUILT_PRODUCTS_DIR}/${FRAMEWORKS_FOLDER_PATH}/BecoSDK.framework/strip-frameworks.sh"`

![](https://github.com/becoinc/content_images/blob/master/SDK_user_guides/iOS_shell_script_execution.png)

## **BUILDING THE EXAMPLE APP**

The Beco Mobile SDK for iOS Example App comes with a build environment all set up for you. Simply double check that all build settings are enabled/added as outlined in the "Project Configuration" section above, and add the appropriate Beco Service Account credentials to the `setCredentials` flow.

## **BECO SDK API AND USAGE**

In keeping with other aspects of the Beco System, the Beco Mobile SDK has been designed to have a very simple, minimal mandatory interface with a few optional aspects to provide enhancements.

#### Standard API

![](https://github.com/becoinc/content_images/blob/master/SDK_user_guides/iOS_standard_API.png)

#### Advanced Performance Settings

[**NOTE**]: The Beco Mobile SDK has pre-set default values for these settings to ensure reliable indoor positioning performance. The cases where these defaults may need adjustment are extremely rare, and even then should only be modified after review with our Support Team in order to preserve system performance. Open a ticket via [becosupport@convene.com](becosupport@convene.com) if you experience unexpected location performance. See our SDK Walkthrough document for more details.

![](https://github.com/becoinc/content_images/blob/master/SDK_user_guides/iOS_advanced_performance_tuning_calls.png)

#### Main Interface
The main interface between your App and Beco location data is via the delegate field in the `BecoSDKInterface` instance created by your application.

It is up to your application to define a delegate that implements the `BecoSDKDelegate` protocol and then set the `delegate` field to that object. The protocol defines a set of methods that you must implement in order to receive events from the Beco SDK.

Detailed information on each of the protocol delegate methods, thread behavior and other information is provided inline with the protocol definition headers and Swift modules included with the Beco Mobile SDK. Please refer to the source level documentation, Quick Help and Example Apps for more information.

#### SDK Behavior Details
Once you call `startScan` the Beco Mobile SDK will continue to scan for Beco Beacons until `stopScan` is called. The Beco Mobile SDK will automatically stay running in the background and wake the phone up when possible to process beacon information when permitted by iOS. It will also automatically stay running in the Background State or in the Killed (Swiped Closed) State, or after a phone reboot (as long as the phone has been unlocked at least once after reboot).

In order for the Beco Mobile SDK to function properly, you should call `startScan()` as soon as possible after your App has launched.

Additionally, the Beco Mobile SDK does not invoke `CBCentralManager` to detect the current phone Bluetooth enabled/disabled state. This is done to minimize user annoyance with OS pop- ups. In certain circumstances iOS will prompt the user to enable Bluetooth for Location services. If SDK customers wish to remind their App users about enabling Bluetooth, then they should invoke `CBCentralManager` as appropriate. **No beacons will be detected if Bluetooth is disabled.**

## **DOCUMENT REVISION SUMMARY**
Revision | Summary of Changes
------------ | -------------
1 | Initial document version.
2 | Updated for Object and Delegate based SDK enhancements.
3 | Updated to include v1.2.x SDK release with Realm.
4 | Incorporate editorial changes.
5 | Update for Realm v1.0(0).
6 | Adjust UIBackgroundModes Info.plist, update version.
7| Update versions for v1.4(0).
8 | Update for v1.5(3).
9 | Update example app screen shots.
10 | Update for v1.6(2).
11 | Update for v1.7(0).
12 | Update for v1.8(8).
13 | Update for v2.1(0).
14 | Update for v2.2(1).
15 | Update for v2.5(2).
16 | Update for Beta Build with Swift 3.1 support.
17 | Update for v2.8(0).
18 | Update for v3.1(4).
19 | Update for v3.1(5).
20 | Update for v3.1(6) to v3.2(0).
21 | Update for v3.4(0).
22 | Update for v3.5(9).
23 | Update for v3.5(15).
23 | Update for v3.6(4).
24 | Update for v3.7(2).
25 | Update for v3.7(2) with Xcode 10.0/Swift 4.2 compatibility.
26 | Update for v3.7(3) with Xcode 10.2(1)/Swift 5 compatibility.

## **LEGAL**

#### Export Statement
You understand that the Software may contain cryptographic functions that may be subject to export restrictions, and you represent and warrant that you are not located in a country that is subject to United States export restriction or embargo, including Cuba, Iran, North Korea, Sudan, Syria or the Crimea region, and that you are not on the Department of Commerce list of Denied Persons, Unverified Parties, or affiliated with a Restricted Entity.

You agree to comply with all export, re-export and import restrictions and regulations of the Department of Commerce or other agency or authority of the United States or other applicable countries. You also agree not to transfer, or authorize the transfer of, directly or indirectly, the Software to any prohibited country, including Cuba, Iran, North Korea, Sudan, Syria or the Crimea region, or to any person or organization on or affiliated with the Department of Commerce lists of Denied Persons, Unverified Parties or Restricted Entities, or otherwise in violation of any such restrictions or regulations.

#### Trademark Notes

* IOS is a trademark or registered trademark of Cisco in the U.S. and other countries and is used under license.
