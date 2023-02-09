---
title: Mobile Security - Fighting with Frameworks - React Native
author: Paolo Serra
date: 9999-02-06 10:00:00
categories: [Topic, Mobile Security]
toc: true
author: paoloserra
img_path: /images/mobile-security-fighting-with-frameworks
image:
  path: /react-native/wallpaper.jpeg
---

![Device rooted/jailbroken](device_rooted.png){: .shadow .normal width="35" height="35" }    Device rooted/jailbroken
![Device](device.png){: .shadow .normal width="35" height="35" }    Device

> The blog post will be kept updated to offer a quality resource. DM me on Twitter or Linkedin for contributions.
{: .prompt-info }

Episode of [Fighting with Frameworks](/posts/mobile-security-fighting-with-frameworks/) season.

## React Native

React Native is a cross-platform framework that allows the development of mobile apps using only one codebase in JavaScript or TypeScript. React Native has come a long way in terms of performance, and in many cases, it can provide a level of performance that is comparable to that of a native app. 

The native app executes JavaScript code with the native or custom JavaScript engine in a separate thread. Native JS engine uses the source code stored in the `.jsbundle`{: .filepath} file, while custom JS engines can have various behaviours. React Native has come a long way in terms of performance, and in many cases, it can provide a level of performance that is comparable to that of a native app.

|                                           | React Native                                     |
|:------------------------------|:-----------------------------------------|
|**Code**                             |(Javascript/Typescript)  + (Java / Objective-C / Swift) |
|**Compilation iOS**          |JIT (with JavaScriptCore, until iOS 6)  or ATO (with Hermes)     |
|**Compilation Android**  |JIT (with JavaScriptCore) or ATO (with Hermes)               |
|**UI Rendering**               |Native UI Controllers                      |
|**UI Engineering**            |Customization with built-in UI components |

*The compilation is a little bit complicated, because there are many options. Generally, you will see Hermes in action, but keep in mind that also the v8 engine is used (when debugging with Chrome)*

### Detecting app

All React Native applications (both Android and iOS) have unique fingerprints that make them easy to identify:

Android
: - `index.android.bundle`: a **minified** javascript file in the `assets`{: .filepath} directory
- `com.facebook.react`: an entry in the `AndroidManifest.xml`{: .filepath}
- `com.facebook.react.devsupport.DevSettingsActivity`: an activity

iOS
: - `index.ios.bundle` or `main.jsbundle`: a **minified** javascript file in the the `Payload`{: .filepath} directory inside the app folder TODO
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
: - [TrustKit-Android](https://github.com/datatheorem/TrustKit-Android): it makes use of the Network Security Config.
- [React Native OkHttp](https://github.com/facebook/react-native/blob/main/ReactAndroid/src/main/java/com/facebook/react/modules/network/OkHttpClientProvider.java): This *Provider* is similar to the default one, but it's provided by React and exposes an `OkHttpClientBuilder` to which you may provide your own `CertificatePinner` instance. Unlike iOS (as we will see), the method required for implementing Pinning is actually exposed in Android.


iOS
: - [TrustKit for iOS](https://github.com/datatheorem/TrustKit): We already mentioned ***method swizziling*** in the Native Apps section and even for React Native we will take advantage of it (remember: it is not always possible using it, see scenarios where it's not inside the ***Trustkit*** doc). Since we don't have access to React Native's delegates `NSURLConnection` and `NSURLSession` as far, this is the only solution. For this reason, the method required for implementing Pinning is not exposed like in Android.

Plugins
: - [react-native-ssl-pinning](https://github.com/MaxToyberman/react-native-ssl-pinning#)
- [react-native-pinch](https://github.com/localz/react-native-pinch)




### Debugging
We mentioned Web Technologies, so what are the best tools for debugging them? **Chrome DevTools** is the answer. Because we are dealing with a React Native App, it is evident that the application logic lies within the Javascript code; thus, we must shift our focus to that side. 

Android
: - **Debug Mode**: use the [In-App Developer Menu](https://reactnative.dev/docs/debugging#chrome-developer-tools)  (Android uses Chrome). Shake your device or run `adb shell input keyevent 82` to access the Developer Menu. Select "Debug JS Remotely" from the Developer Menu. This will open a new tab at *http://localhost:8081/debugger-ui*.  Once done, select `Tools → Developer Tools`{: .filepath} from the Chrome Menu to open the Developer Tools. In this scenario, the Javascript code runs in your computer, and then the results are sent to the device via WebSockets.
- **Release Mode**: The Developer Menu is disabled in release (production) builds. TO-DO

iOS
: - **Debug Mode**: use the [In-App Developer Menu](https://reactnative.dev/docs/debugging#chrome-developer-tools)  (Android uses Chrome). Shake your device or by selecting "Shake Gesture" inside the Hardware menu in the iOS Simulator. Select "Debug JS Remotely" from the Developer Menu. This will open a new tab at *http://localhost:8081/debugger-ui*. Once done, select `Tools → Developer Tools`{: .filepath} from the Chrome Menu to open the Developer Tools. In this scenario, the Javascript code runs in your computer, and then the results are sent to the device via WebSockets. 
- **Release Mode**: The Developer Menu is disabled in release (production) builds. So we have two options:
    - `On Safari`: First, on the iOS device, enable **Web Inspector** from `Settings > Safari > Advanced`{: .filepath}. On the mac, open Safari and enable **Show Develop menu in menu bar** under `Safari > Preferences > Advanced`{: .filepath}. Within Safari, select Develop in the toolbar. In the dropdown menu options, you should see the name of your device and app. Hover over the app name and click on localhost.
    -  `On Chrome` : use a [WebKit proxy](https://github.com/google/ios-webkit-debug-proxy)

>To debug **javascript** code in Chrome: on your device, go to `Settings > Developer Options`{: .filepath} (make sure they're enabled first) and ensure that the *developer options* switch is toggled on. Scroll down to *USB Debugging* and ensure that it is also enabled. Open the Chrome browser in you pc or mac and navigate to the URL `chrome://inspect/#devices`. Once a WebView is created or the app starts, something will shows up on the page. Just click on *"inspect"*.

>To debug **native code**: when working with native code, such as when writing native modules, you can launch the app from Android Studio or Xcode and take advantage of the native debugging features (setting up breakpoints, etc.) as you would in case of building a standard native app.



### Reverse Engineering
Reverse engineering of React Native mobile applications involves analyzing the compiled code to understand its structure and functions. It can be performed on both debug and release mode builds, although reverse engineering a release mode build is often more challenging because it is obfuscated and optimized.
To reverse a React Native app, first, we need to know how the framework works and what javascript engine it uses. When using React, you have two options:

- [Hermes](https://hermesengine.dev/): it is a JavaScript engine developed by Facebook that is optimized for running React Native apps on Android and iOS devices. Starting from React Native 0.70, it is enabled by default. Since React Native 0.64, Hermes also runs on iOS.

- [JavaScriptCore](https://trac.webkit.org/wiki/JavaScriptCore):  it is the built-in JavaScript engine for WebKit, the web browser engine used by Safari, Mail, App Store, and many other apps on macOS, iOS, and Linux. On iOS 7+, JavaScriptCore does not use JIT due to the absence of writable executable memory in iOS apps.

##### Hermes vs JavaScriptCore
The main difference between JavaScriptCore and Hermes is the optimizations they provide for their respective platforms. JavaScriptCore is optimized for running JavaScript code within the WebKit framework, while Hermes is optimized for running React Native apps on Android. Additionally, Hermes includes many performance improvements over other JavaScript engines, including faster startup time, smaller app size, and improved memory usage.


##### Hermes Reverse Engineering

Most of the time, the app will be optimized using Meta's Hermes engine. This engine is not very pleasant for us because it converts Javascript directly into bytecode at build time to enhance efficiency. That means we will have to deal with things not similar to readable code. However, from a security overview, Hermes does not do anything to prevent code tampering or to obfuscate the code by default.

When an app is compiled with Hermes, it is transformed from its original source code into machine code, making it difficult to retrieve the original source code. However, it is possible to decompile the app and retrieve a representation of the original source code using reverse engineering tools. Decompiling a React Native app compiled with Hermes can reveal the structure and functionality of the app, as well as any hardcoded strings or secrets that may be present in the code.

A useful tool for decompiling Hermes bytecode is [hbctool](https://github.com/bongtrop/hbctool). In case the Hermes version of the bytecode is not supported by the latest version of the tool, remember to check the issues tab on GitHub to make sure there is a beta version. Sometimes the community may surprise you.
Instead, if you want to disassemble the bytecode, here we have [hbcdump](https://github.com/facebook/hermes/tree/main/tools/hbcdump).

---

## References

- [React Native Documentation](https://reactnative.dev/docs)
- [React Native App Security](https://www.cossacklabs.com/blog/react-native-app-security.html)
- [Securing React Native Applications](https://blog.jscrambler.com/securing-react-native-applications)
- [On Hermes and Mattermost](https://developers.mattermost.com/blog/on-hermes-and-mattermost/)
- [Hacktricks - React Native](https://book.hacktricks.xyz/mobile-apps-pentesting/android-app-pentesting/react-native-application)

