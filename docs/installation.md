# Installation

First, download the library from npm:

```
npm install react-native-maps --save
```

## 1. Get a Google Maps API key

To use Google Maps, you will need an API key. Otherwise, Google Maps will not render anything, and maps will not work at all on Android.

#### 1. Create a new project for your app in the Google Developer console: https://console.cloud.google.com/projectcreate

* _(if you already have an existing Google Cloud project for your app you can skip this step.)_

#### 2. Enable the iOS and/or Android SDKs, as required:

* iOS: https://console.cloud.google.com/apis/library/maps-ios-backend.googleapis.com/
* Android: https://console.cloud.google.com/apis/library/maps-android-backend.googleapis.com/
* _⚠️ Make sure the project you just created is selected in the nav bar at the top of the screen on Google Cloud Platform._

#### 3. Create an API key: https://console.cloud.google.com/apis/credentials

* _You can choose to use the same API key for both iOS and Android, or you can choose to restrict the key and use different keys for each platform. This is up to you._

Keep this key ready - you will need to use it further along in the installation process.

**⚠️ Make sure not to upload your API keys to public GitHub repos, especially if you have billing enabled on your account.**

## 2. Install native dependencies

### iOS

**⚠️ Do not use `react-native link` before you have set up the Podfile.** If you accidentally ran `react-native link`,

#### 1. Install CocoaPods

* `gem install cocoapods`
* _You can skip this step if you already have CocoaPods installed_

#### 2. Create/edit your Podfile

* If you don't have a Podfile yet, create a new file in `ios/Podfile`.
* Replace `_YOUR_PROJECT_TARGET_` with the name of your project target. It's usually the same as the name of your XCode project.
* Add the following to your Podfile:

```ruby
target '_YOUR_PROJECT_TARGET_' do
  rn_path = '../node_modules/react-native'
  rn_maps_path = '../node_modules/react-native-maps'

  # See http://facebook.github.io/react-native/docs/integration-with-existing-apps.html#configuring-cocoapods-dependencies
  pod 'yoga', path: "#{rn_path}/ReactCommon/yoga/yoga.podspec"
  pod 'React', path: rn_path, subspecs: [
    'Core',
    'CxxBridge',
    'DevSupport',
    'RCTActionSheet',
    'RCTAnimation',
    'RCTGeolocation',
    'RCTImage',
    'RCTLinkingIOS',
    'RCTNetwork',
    'RCTSettings',
    'RCTText',
    'RCTVibration',
    'RCTWebSocket',
  ]

  # React Native third party dependencies podspecs
  pod 'DoubleConversion', :podspec => "#{rn_path}/third-party-podspecs/DoubleConversion.podspec"
  pod 'glog', :podspec => "#{rn_path}/third-party-podspecs/glog.podspec"
  # If you are using React Native <0.54, you will get the following error:
  # "The name of the given podspec `GLog` doesn't match the expected one `glog`"
  # Use the following line instead:
  #pod 'GLog', :podspec => "#{rn_path}/third-party-podspecs/GLog.podspec"
  pod 'Folly', :podspec => "#{rn_path}/third-party-podspecs/Folly.podspec"

  # react-native-maps dependencies
  pod 'react-native-maps', path: rn_maps_path
  pod 'react-native-google-maps', path: rn_maps_path  # Remove this line if you don't want to support GoogleMaps on iOS
  pod 'GoogleMaps'  # Remove this line if you don't want to support GoogleMaps on iOS
  pod 'Google-Maps-iOS-Utils' # Remove this line if you don't want to support GoogleMaps on iOS
end

post_install do |installer|
  installer.pods_project.targets.each do |target|
    if target.name == 'react-native-google-maps'
      target.build_configurations.each do |config|
        config.build_settings['CLANG_ENABLE_MODULES'] = 'No'
      end
    end
    if target.name == "React"
      target.remove_from_project
    end
  end
end
```

#### 3. Install native dependencies using CocoaPods

* First, make sure that your CocoaPods repo is up to date:

  ```
  pod repo update
  ```

* Ensure that you're in the `ios/` directory, and run

  ```
  pod install
  ```

* Once this has succeeded, you will have a new file ending in `.xcworkspace`.
  **Make sure you use this file from now on if you want to open the project in Xcode** - if you open the existing `.xcproject`, Xcode will not be able to find the dependencies installed through CocoaPods, and your build will fail.

#### 4. Enable Google Maps in your iOS app

_You can skip this step if you don't want to use Google Maps on iOS._

To use Google Maps, you have to provide an API key.

* Make the following changes to `ios/_YOUR_PROJECT_NAME_/AppDelegate.m`:
  * _⚠️ Make sure you change `_YOUR_API_KEY_` to the Google API key you set up earlier._

```objc
// 1. add this line
@import GoogleMaps;

@implementation AppDelegate
// ...

- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions
{
    // 2. add this line - replace _YOUR_API_KEY_ with your Google Maps API key.
    [GMSServices provideAPIKey:@"_YOUR_API_KEY_"];
// ...
```

#### 5. Run your project

If everything went well, you should now be able to build your project in Xcode.

You should now be able to add a `MapView` component anywhere in your app, and it should successfully render a map. See the [troubleshooting](#troubleshooting) section if you are still having issues.

If you want to run your project on a real device, you can follow the steps described in [the official React Native "Running on Device" documentation](https://facebook.github.io/react-native/docs/running-on-device.html), but make sure you open the `.xcworkspace` file instead of the `.xcodeproj` file.

### Android

## Troubleshooting

### I get errors after running `react-native link`.

**This library can currently not be installed using `react-native link`.** Please follow the [installation instructions above](#ios).

> If you have any experience with creating `prelink`/`postlink` scripts, we would appreciate any contribution that would enable installation with `react-native link`.

If you ran `react-native link` by mistake:

1.  delete `node_modules/`
2.  delete `ios/Pods`
3.  delete `ios/Podfile.lock`
4.  open Xcode and delete `AIRMaps.xcodeproj` from Libraries if it exists
5.  in _Build Phases -> Link Binary With Libraries_ delete `libAIRMaps.a` if it exists
6.  delete `ios/build` directory
7.  follow the [installation steps above](#ios)

### I get other build errors in Xcode

If you use Xcode with version less than 9 you may get `use of undeclared identifier 'MKMapTypeMutedStandard'` or `Entry, ":CFBundleIdentifier", Does Not Exist` errors. In this case you have to update your Xcode.

---

## Android

1.  In your `android/app/build.gradle` add:

    ```groovy
    ...
    dependencies {
      ...
      compile project(':react-native-maps')
    }
    ```

If you've defined _[project-wide properties](https://developer.android.com/studio/build/gradle-tips.html)_ (**recommended**) in your root `build.gradle`, this library will detect the presence of the following properties:

    ```groovy
    buildscript {...}
    allprojects {...}

    /**
     + Project-wide Gradle configuration properties
     */
    ext {
        compileSdkVersion   = 26
        targetSdkVersion    = 26
        buildToolsVersion   = "26.0.2"
        supportLibVersion   = "26.1.0"
        googlePlayServicesVersion = "11.8.0"
        androidMapsUtilsVersion = "0.5+"
    }
    ```

If you do **not** have _project-wide properties_ defined and have a different play-services version than the one included in this library, use the following instead (switch 10.0.1 for the desired version):

```groovy
...
dependencies {
    ...
    compile(project(':react-native-maps')){
        exclude group: 'com.google.android.gms', module: 'play-services-base'
        exclude group: 'com.google.android.gms', module: 'play-services-maps'
    }
    compile 'com.google.android.gms:play-services-base:10.0.1'
    compile 'com.google.android.gms:play-services-maps:10.0.1'
}
```

2.  In your `android/settings.gradle` add:

    ```groovy
    ...
    include ':react-native-maps'
    project(':react-native-maps').projectDir = new File(rootProject.projectDir, '../node_modules/react-native-maps/lib/android')
    ```

3.  Specify your Google Maps API Key:

    Add your API key to your manifest file (`android/app/src/main/AndroidManifest.xml`):

    ```xml
    <application>
        <!-- You will only need to add this meta-data tag, but make sure it's a child of application -->
        <meta-data
          android:name="com.google.android.geo.API_KEY"
          android:value="Your Google maps API Key Here"/>
    </application>
    ```

    > Note: As shown above, com.google.android.geo.API_KEY is the recommended metadata name for the API key. A key with this name can be used to authenticate to multiple Google Maps-based APIs on the Android platform, including the Google Maps Android API. For backwards compatibility, the API also supports the name com.google.android.maps.v2.API_KEY. This legacy name allows authentication to the Android Maps API v2 only. An application can specify only one of the API key metadata names. If both are specified, the API throws an exception.
    > Source: https://developers.google.com/maps/documentation/android-api/signup

4)  Add `import com.airbnb.android.react.maps.MapsPackage;` and `new MapsPackage()` in your `MainApplication.java` :

    ```
    import com.airbnb.android.react.maps.MapsPackage;
    ...
    @Override
         protected List<ReactPackage> getPackages() {
             return Arrays.<ReactPackage>asList(
                     new MainReactPackage(),
                     new MapsPackage()
             );
         }
    ```

5.  Ensure that you have Google Play Services installed:

* For Genymotion you can follow [these instructions](https://www.genymotion.com/help/desktop/faq/#google-play-services).
* For a physical device you need to search on Google for 'Google Play Services'. There will be a link that takes you to the Play Store and from there you will see a button to update it (do not search within the Play Store).

## Troubleshooting

If you have a blank map issue, ([#118](https://github.com/airbnb/react-native-maps/issues/118), [#176](https://github.com/airbnb/react-native-maps/issues/176), [#684](https://github.com/airbnb/react-native-maps/issues/684)), try the following lines :

### On Android:

1.  Be sure to have `new MapsPackage()` in your `MainApplication.java` :

    ```
    import com.airbnb.android.react.maps.MapsPackage;
    ...
    @Override
         protected List<ReactPackage> getPackages() {
             return Arrays.<ReactPackage>asList(
                     new MainReactPackage(),
                     new MapsPackage()
             );
         }
    ```

1.  Set this Stylesheet in your map component

    ```
    import MapView from 'react-native-maps';
    ...
    const styles = StyleSheet.create({
      container: {
        ...StyleSheet.absoluteFillObject,
        height: 400,
        width: 400,
        justifyContent: 'flex-end',
        alignItems: 'center',
      },
      map: {
        ...StyleSheet.absoluteFillObject,
      },
    });

    export default class MyApp extends React.Component {
      render() {
        const { region } = this.props;
        console.log(region);

        return (
          <View style ={styles.container}>
            <MapView
              style={styles.map}
              region={{
                latitude: 37.78825,
                longitude: -122.4324,
                latitudeDelta: 0.015,
                longitudeDelta: 0.0121,
              }}
            >
            </MapView>
          </View>
        );
      }
    }
    ```

1.  Run "android" and make sure all packages are up-to-date.
1.  If not installed yet, you have to install the following packages :
    * Extras / Google Play services
    * Extras / Google Repository
    * Android 6.0 (API 23) / Google APIs Intel x86 Atom System Image Rev. 19
    * Android SDK Build-tools 23.0.3
1.  Go to [Google API Console](https://console.developers.google.com/flows/enableapi?apiid=maps_android_backend) and select your project, or create one.
    Then, once enabled, select `Go to credentials`.
    Select `Google Maps Android API` and create a new key.
    Enter the name of the API key and create it.

1.  Clean the cache :

```
 watchman watch-del-all
 npm cache clean
```

1.  When starting emulator, make sure you have enabled `Wipe user data`.

1.  Run `react-native run-android`

1.  If you encounter `com.android.dex.DexException: Multiple dex files define Landroid/support/v7/appcompat/R$anim`, then clear build folder.

```
 cd android
 ./gradlew clean
 cd ..
```

1.  If you are using Android Virtual Devices (AVD), ensure that `Use Host GPU` is checked in the settings for your virtual device.

1.  If using an emulator and the only thing that shows up on the screen is the message: `[APPNAME] won't run without Google Play services which are not supported by your device.`, you need to change the emulator CPU/ABI setting to a system image that includes Google APIs. These may need to be downloaded from the Android SDK Manager first.
