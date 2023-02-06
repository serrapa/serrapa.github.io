---
title: Mobile Security - Fighting with Frameworks - React Native
author: Paolo Serra
date: 9999-01-02 10:00:00
categories: [Topic, Mobile Security]
toc: true
author: paoloserra
img_path: /images/mobile-security-fighting-with-frameworks
image:
  path: /react-native/
---

![Device rooted/jailbroken](device_rooted.png){: .shadow .normal width="35" height="35" }    Device rooted/jailbroken
![Device](device.png){: .shadow .normal width="35" height="35" }    Device

> Change the date of this post to make it visible.
{: .prompt-danger }

## React Native

React Native is a cross-platform solution that allows writing native apps using React (JavaScript or TypeScript). The native app executes JavaScript code with the native or custom JavaScript engine in a separate thread. Native JS engine uses the source code stored in the *.jsbundle* file, while custom JS engines can have various behaviours.

|                                           | React Native                                     |
|:------------------------------|:-----------------------------------------|
|**Code**                             |Javascript  + Java / Objective-C / Swift |
|**Compilation iOS**          |Interpreter or ATO (with Hermes)        |
|**Compilation Android**  |JIT or / ATO (with Hermes)               |
|**UI Rendering**               |Native UI Controllers                      |
|**UI Engineering**            |Customization with built-in UI components |

### Detecting app

All React Native applications (both Android and iOS) have unique fingerprints that make them easy to identify:

Android
: - `index.android.bundle` : a file in the *assets* directory
- `com.facebook.react`: an entry in the Android Manifest
- `com.facebook.react.devsupport.DevSettingsActivity`: an activity

iOS
: - `main.jsbundle`: a file in the `Payload`{: .filepath} directory inside the app folder
- inside the `assets`{: .filepath} directory there might be some references to React, for example ***node_modules/@react-navigation*** or ***node_modules/@react-native-****

>*the **bundle** file: it's the "Javascript". In development the bundle likely will come from your *react-native start* development server. That way if your code is changed the server will send a request to the client, through a websocket, to download the new code or update the code on the fly. So, you could say the bundle is dynamically generated from your source code. In production you probably want to use an offline bundle so that your code is already on the device and does not need to be downloaded. [Cit.](https://stackoverflow.com/questions/41960891/what-is-react-natives-bundle-and-its-purpose)

### Intercepting Traffic

Android
: - ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  use the local proxy settings (WiFi settings)
- ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  patch the apk to accept ***user*** certificates
- ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  use ***Bettercap*** to set up an ARP Poisoning attack
- ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  use ***NoPE Proxy*** (Burp extension) to carry on a DNS Spoofing attack
- ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  set up an Access Point and connect the iOS device to it
- ![](device_rooted.png){: .shadow width="35" height="35" }  use ProxyDroid App
- ![](device_rooted.png){: .shadow width="35" height="35" }  in case of “client isolation” activated in the Wireless network and the iOS device and your laptop are not able to communicate: use SSH remote port forwarding
- ![](device_rooted.png){: .shadow width="35" height="35" } use the `/etc/hosts`{: .filepath} file to make the target domain point to the IP address of your interception proxy.


iOS
: - ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  use the local proxy settings (WiFi settings)
- ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  set up a VPN Server and implement the IPTables rules to forward all incoming traffic on 80 and 443 ports to the proxy host and port. Lastly, download OpenVPN Client on the device
- ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  use ***Bettercap*** to set up an ARP Poisoning attack
- ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  use ***NoPE Proxy*** (Burp extension) to carry on a DNS Spoofing attack
- ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  set up an Access Point and connect the iOS device to it
- ![](device_rooted.png){: .shadow width="35" height="35" }  in case of “client isolation” activated in the Wireless network and the iOS device and your laptop are not able to communicate: use SSH remote port forwarding
- ![](device_rooted.png){: .shadow width="35" height="35" } use the `/etc/hosts`{: .filepath} file to make the target domain point to the IP address of your interception proxy.

### Certificate Pinning

Android
: - [React Native OkHttp](https://github.com/facebook/react-native/blob/main/ReactAndroid/src/main/java/com/facebook/react/modules/network/OkHttpClientProvider.java): This *Provider* is similar to the default one, but it's provided by React and exposes an `OkHttpClientBuilder` to which you may provide your own CertificatePinner instance. Unlike iOS (as we will see), the method required for implementing Pinning is actually exposed in Android.

iOS
: - [TrustKit for iOS](https://github.com/datatheorem/TrustKit): We already mentioned ***method swizziling*** in the Native Apps section and even for React Native we will take advantage of it (remember: it is not always possible using it, see scenarios where it's not inside the ***Trustkit*** doc). Since we don't have access to React Native's delegates `NSURLConnection` and `NSURLSession` as far, this is the only solution. For this reason, the method required for implementing Pinning is not exposed like in Android.


### Debugging
We mentioned Web Technologies, therefore what are the best tools for debugging them? **Chrome DevTools** is the answer. Because we are dealing with an Ionic App, it is evident that the application functionality lies within the Javascript code; thus, we must shift our focus to that side. 

Android
: - **Debug Mode**: use [Chrome](https://reactnative.dev/docs/debugging#chrome-developer-tools) and select "Debug JS Remotely" from the Developer Menu. This will open a new tab at http://localhost:8081/debugger-ui. Shake your device or run `adb shell input keyevent 82` to acces the Developer Menu. Once done, select `Tools → Developer Tools` from the Chrome Menu to open the Developer Tools.
- **Release Mode**: The Developer Menu is disabled in release (production) builds.

To debug the JavaScript code in Chrome:
- `On Chrome` (**Debug & Release Mode**) Go to *Settings > Developer Options* (make sure they're enabled first) and ensure that the *developer options* switch is toggled on. Scroll down to *USB Debugging* and ensure that it is also enabled. Open the Chrome browser and navigate to the URL `chrome://inspect/#devices`. Once a WebView is created or the app starts, something will shows up on the page. Just click on *"inspect"*.

Android (native code)
: - When working with native code, such as when writing native modules, you can launch the app from Android Studio or Xcode and take advantage of the native debugging features (setting up breakpoints, etc.) as you would in case of building a standard native app.


iOS
: - **Debug Mode**: [React Native Debugger](https://reactnative.dev/docs/debugging) Shake your device or by selecting "Shake Gesture" inside the Hardware menu in the iOS Simulator.
- **Release Mode**: The Developer Menu is disabled in release (production) builds.
- `On Safari` (**Debug & Release Mode**) First, on the iOS device, enable **Web Inspector** from *Settings > Safari > Advanced*. Next, open Safari on a Mac then enable **Show Develop menu in menu bar** under *Safari > Preferences > Advanced*. Within Safari, select Develop in the toolbar. In the dropdown menu options, you should see the name of your device and app. Hover over the app name and click on localhost.
- `On Chrome` : use a [WebKit proxy](https://github.com/google/ios-webkit-debug-proxy)





### Reverse Engineering
Most of the time, the app will be optimized using Meta's Hermes engine, an open-source JavaScript engine optimized for React Native. This engine is not very pleasant for us because, in order to enhance efficiency, it converts Javascript directly into bytecode at build time. That means we'll have to deal with things not particulary similar to readable code.

Android
: .....

iOS
: .....

[Come fare la reverse]

---

## References

- [https://www.cossacklabs.com/blog/react-native-app-security.html](https://www.cossacklabs.com/blog/react-native-app-security.html)
- [https://book.hacktricks.xyz/mobile-apps-pentesting/android-app-pentesting/react-native-application](https://book.hacktricks.xyz/mobile-apps-pentesting/android-app-pentesting/react-native-application)
- [React Documentation - Debugging](https://reactnative.dev/docs/debugging)
