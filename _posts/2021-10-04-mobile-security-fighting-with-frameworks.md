---
title: Mobile Security - Fighting with Frameworks
author: Paolo Serra
date: 2021-10-04 10:00:00
categories: [Topic, Mobile Security]
toc: true
---

## Intro

There are many resources about Mobile security and how to conduct tests in native applications, but things are changing so quickly these days, and the Mobile world is offering different solutions and technologies to develop applications. We must all keep up with them.

In this article, we will concentrate on penetration tests performed against mobile applications built with the most common frameworks. When fighting against native applications, it may appear that you should use the same techniques and approaches, but there are some slightly different concepts that can drive you insane and in the wrong direction.

Here is a list of the frameworks I'd like to explore (I will try to keep the list updated), but only some of them will be so far: *Xamarin, Flutter, Cordova, PhoneGap, Ionic, React Native, IBM Worklight, NativeScript, Appcelerator, Corona, Qt, Sencha, Unity3D, 5App, Framework7


|                     |Xamarin                      |Flutter                |Ionic                    |React Native               |NativeScript             |
|:--------------------|:----------------------------|:----------------------|:------------------------|:--------------------------|:------------------------|
|**Code**             |C#                           |Dart                   |HTML,CSS,TypeScript,Javascript                |Javascript                 |Javascript/TypeScript    |
|**Compilation iOS**  |AOT                          |xxxxx                  |JIT+WKWebView            |Interpreter                |Interpreter              |
|**Compilation Android**|JIT/AOT                      |xxxxx                  |JIT                      |JIT                        |JIT                      |
|**UI Engineering**   |Native / Code Sharing for the cost of native experience|xxxxx                  |Code Sharing for the cost of native experience|Customization with built-in UI components |Code Sharing for the cost of native experience|




### Let's start from Network Communications

An attacker can use a local proxy to intercept communications between the mobile app and the server to discover and exploit network traffic vulnerabilities. A proxy enables an attacker to inspect, modify, repeat, and comprehend how the mobile app communicates with the server, as well as how the server may react to unexpected or untrusted data.
Like Android and iOS themself, also modern frameworks provide built-in solutions for reducing an attacker's ability to intercept proxy communications. These safeguards primarily consist of certificate pinning and/or ignoring system proxy settings. However, as with all client-side mobile app protections, these can be circumvented if an attacker has complete control of the device.

**NB**: *In this guide, I am not going to explain how to install your proxy's certificate in Android or iOS. Google is your friend so not being lazy and google it.*


---

## A quick overview to understand the follow-up: Native Applications

A Native app is created for a specific platform, usually Android or iOS, as these are the most popular ones. Developers use a programming language that suits a particular platform: Java and Kotlin for Android, Swift and Objective-C for iOS. From a security standpoint, there aren't many differences among hybrid solutions; you may find the same vulnerability in both. Keep in mind that hybrid is more like a web-based environment to attack (Webviews' fault).

### Detecting app

Android
: Every app (native or not) has a codebase written in Java/Kotlin, so don't rely the detection on it. Actually, there are some elements that help you to discover how the app has been built with:
- ***Check through Developer Options and layout bounds***: If it's a native app, it will display rectangles in each activity you open, whereas a hybrid app will display a cross line and no rectangles. Consider that this rule does not apply to all frameworks because some of them convert graphic elements into native ones.
- ***Use Webview to Check***: The WebView method can be used to determine whether an app is native or hybrid. If an app uses WebViews the majority of the time for each screen, it may be hybrid (**uiautomatorviewer**  is your friend). Keep in mind that this is not always the case: native apps can also benefit from a WebView.
- ***Look up explicit frameworks elements inside the apk***: personally my favourite one.

iOS
: Since I have never found a iOS-related way to detect the kind of an app yet, it remains for us to ***Look up explicit frameworks elements inside the ipa***.


### Intercepting Traffic

Android
: - *Rooted & Unrooted device*: use the local proxy settings (WiFi settings)
- *Rooted & Unrooted device*: patch the apk to accept "user" certificates
- *Rooted device*: use ProxyDroid App

>When creating your test environment, it is critical to understand which solution you want to use and the differences among them. You might think that methods that work for both rooted and unrooted devices are the best, but what about ignoring proxy settings, network security config integrity checks, or any other mechanisms that make your life stressful during a pentest? There are non-identical methods for redirecting traffic to another host (i.e. a proxy) under the the woods, and this is why there is a difference between the **local proxy setting** and **ProxyDroid**. To begin, the first does not require a rooted device, whereas ProxyDroid does.

Although their expected behavior is the same, these two solutions behave differently: Proxydroid uses IPtables (kernel level) to redirect traffic, which is why it requires root, whereas the local proxy setting sets up a proxy host and forwards all communication to it.

Once you have set up one of the solutions listed above, check [here](https://blog.nviso.eu/2020/11/19/proxying-android-app-traffic-common-issues-checklist/) to be sure you are not mad and everything is gonna work.

iOS
: - *Jailbroken & Unjailbroken device*: use the local proxy settings (WiFi settings)
- *Jailbroken & Unjailbroken device*: set up a VPN Server, download OpenVPN Client on the device

*iOS doesn't support iptables at kernel level, so either you change the iOS kernel (good luck) or you are screwed.*


### Certificate Pinning

Android
: .....

iOS
: .....

[Come dovrebbe essere implementato il pinning, quali sono le soluzioni comuni e come verificarlo(??).]

### Debugging

Android
: You may want to debug the application during a penetration test, but because it was built in release mode (you rarely receive a debug mode apk), there is a good chance that the `debuggable` flag in the Manifest is not set to true. Nothing says you can debug the application in this case, so why not force it to be debuggable? The magic words are ***patching the apk***. You can do it yourself (if you are a noob, I recommend this solution so that you can understand how the entire operation should be done) or use a tool that automates everything.

iOS
: LLDB + debugserver


### Reverse Engineering

Android
: Unless obfuscation techniques are used, the code base will be mostly Java or Kotlin, making it easier to read and statically analyze the application and its source code. You may also encounter applications that use the NDK, a toolkit that allows you to implement parts of your app in native code using languages such as C and C++. In general, decompiling an Android application is simple (some tools does it for you: (some tools does it for you: [jadx-gui](https://github.com/skylot/jadx)  or [jd-gui](http://java-decompiler.github.io/)).

iOS
: Eh eh... You need to get your hands dirty, man. It's not as straightforward as it is for Android; we're talking about binaries here, so pick one of the following tools and get started: IDA Pro, Ghidra, Radare or Binary Ninja. If you have ever played with binaries and assembly / machine code / unreadable code, you know what I am talking about, otherwise, it's another area of cybersecurity that you'll have to deal with sooner or later.

---

## Xamarin

Xamarin, which is a Microsoft product, provides a single platform to develop one application for multiple platforms with **.NET** and **C#** – iOS and Android in most cases. As it is a framework, it likes getting us more frustrated, but how? Simply ignoring system proxy settings by default. Additionally, Microsoft has already implemented secure coding practices in the Xamarin framework. As such, applications developed with this framework inherit these security controls (uff..). This makes Xamarin–based applications arguably more secure than applications developed from scratch using traditional or no frameworks. Since Xamarin has full access to native APIs and toolkits used by both iOS and Android applications, it doesn't only save developers time in building and maintaining the application, but also help them in achieving near-native look and performance.

### Detecting app

Android
: To recognize whether an Android app has been built using Xamarin or not is pretty straighforward. Inside the structure of the apk (**apktool** is your friend) there is a lot of stuff but that help us is the directory containing the assemblies. In fact, going there you will find certain assemblies related to the Mono engine, a free and open-source .NET Framework-compatible software framework now being led by Xamarin that can be run on many software systems.
Let's see some examples:

|Assembly             |Description                                                                   |
|:--------------------|:-----------------------------------------------------------------------------|
|*mscorlib.dll*         |Silverlight                          |
|*Mono.Android.dll*     |This assembly contains the C# bindings to the Android API                     |
|*Mono.Security.dll*    |Cryptographic APIs                    |
|*System.dll*           |Silverlight, plus types from other namespaces                          |
|*Xamarin.iOS.dll*      |This assembly contains the C# binding to the CocoaTouch API. This is only used in Unified iOS Projects |

Anyway, [here](https://docs.microsoft.com/en-us/xamarin/cross-platform/internals/available-assemblies) you can find a list of all available assemblies with their details.

In addition, the ***lib*** directory contains additional Xamarin-related references, such as native shared libraries. These are often directed to the monodroid engine, `libmonodroid.so` , and maybe some other plugins used by the developer.

iOS
: You cannot decompile an iOS application like Android. iOS apps are compiled directly to machine code, with an aggressive optimization pass that tends to destroy a lot of the structure of the original code. To extract [continua]

### Intercepting Traffic

Since any local proxy setting is ignored by Xamarin, the only way to intercept the traffic generated by an application is setting up a VPN Server in another machine, connect the device to it (i.e OpenVPN) and enable IPtables on the remote machine to redirect all the traffic to burp suite (it can stay on the same machine or another one, it doesn't matter at all).
Achieving it for Android and iOS is not the same.

Android
: ProxyDroid is enough (so a VPN Server is not needed)

iOS
: follow this [guide](https://blog.nviso.eu/2020/06/12/intercepting-flutter-traffic-on-ios/), even though it is for flutter, it works well; you need to create a VPN Server. Otherwise, here another [guide](https://triskelelabs.com/intercepting-xamarin-mobile-app-traffic-2/) for Xamarin.

### Certificate Pinning

Android
: .....

iOS
: .....

[Come dovrebbe essere implementato il pinning, quali sono le soluzioni comuni e come verificarlo(??).]

### Debugging

Android
: .....

iOS
: .....

[Come si debugga l'app]

### Reverse Engineering

Android
: .....

iOS
: .....

[Come fare la reverse]

---

## Flutter

Flutter is Google’s new open source mobile development framework that allows developers to write a single code base and build for Android, iOS, web and desktop. Flutter applications are written in Dart, a language created by Google more than 7 years ago. Do you think is different from Xamarin? No boy, it is also worse! If Xamarin ignores local proxy settings at least, Flutter (actually Dart) doesn’t use the system CA store (so neither system’s proxy settings) but a list of CA’s that’s compiled into the application!

### Detecting app

Android
: All applications developed in Flutter have some fingerprints that render themselves easy to recognize:

  - the entry `flutter.embedding` in the Android Manifest
  - the folder `io.flutter`
  - Shared Object files (`libflutter.so`) in the **lib** directory
  - release-mode applications have the following files:
    - /resources/lib/arm-64-v8a/libflutter.so
    - /resources/lib/arm-64-v8a/libapp.so
  - debug-mode applications have the following files:
    - /resources/assets/flutter_assets/kernel_blob.bin

iOS
: .....


### Intercepting Traffic

Since any local proxy setting is ignored by Dart, the only way to intercept the traffic generated by an application is setting up a VPN Server in another machine, connect the device to it (i.e OpenVPN) and enable IPtables on the remote machine to redirect all the traffic to burp suite (it can stay on the same machine or another one, it doesn't matter at all).
Achieving it for Android and iOS is not the same:

Android
: ProxyDroid is enough (a VPN Server is not needed)

iOS
: follow this [guide](https://blog.nviso.eu/2020/06/12/intercepting-flutter-traffic-on-ios/)


### Certificate Pinning

Android
: .....

iOS
: .....

[Come dovrebbe essere implementato il pinning, quali sono le soluzioni comuni e come verificarlo(??).]

### Debugging

Android
: .....

iOS
: .....

[Come si debugga l'app]

### Reverse Engineering

When you have to reverse a Flutter app, things get more complicated. Production binaries are not easy to reverse because the binary is converted to a Shared Object file (*.so), while for the debug versions is easy to find the source code and get it back.

Android
: .....

iOS
: .....



---

## Cordova (aka PhoneGap)

Apache Cordova (aka PhoneGap) is an open-source mobile development framework. It allows you to use standard web technologies - HTML5, CSS3, and JavaScript for cross-platform development. Applications execute within wrappers targeted to each platform and rely on standards-compliant API bindings to access each device's capabilities such as sensors, data, network status, etc. Cordova doesn't offer any UI

### Detecting app

Android
: .....

iOS
: .....

[Come riconoscere queste apps]

### Intercepting Traffic

[Come si intercetta il traffico.]

### Certificate Pinning

Android
: .....

iOS
: .....

[Come dovrebbe essere implementato il pinning, quali sono le soluzioni comuni e come verificarlo(??).]

### Debugging

Android
: .....

iOS
: .....

[Come si debugga l'app]

### Reverse Engineering

Android
: .....

iOS
: .....

[Come fare la reverse]

---

## Ionic

Ionic is an open source UI toolkit for building performant, high-quality mobile and desktop apps using web technologies — HTML, CSS, and JavaScript — with integrations for popular frameworks like Angular, React, and Vue. Actually, if you don't want to mess things up you can move alone with a simple *script include* without using any frontend framework. That means that if you don't find any references to Angular, React or Vue, there might be more vulnerabilities to discover because it's all up to developers (blink!).  You may wonder how exactly Ionic works, but in nutshell it uses Capacitor or Cordova to deploy natively, because they give access to Native SDKs, and Web Views (provided by both iOS and Android SDKs) to render any Ionic app. 

|                                           | Ionic                                                    |
|:------------------------------|:-----------------------------------------|
|**Code**                             |HTML, CSS, TypeScript, Javascript    |
|**Compilation iOS**          |JIT + WKWebView                              |
|**Compilation Android**  |JIT                                                        |
|**UI Rendering**               |HTML, CSS                                           |
|**UI Engineering**             |Code Sharing for the cost of native experience   |

>***Capacitor*** is an open source *cross-platform app runtime* (like Cordova) that allows web-based apps to run natively on iOS, Android, Electron and the Web. Capacitor was created and is actively developed/supported by Ionic (while Cordova by Apache). Although they are similar to each other (think that Capacitor has been built on the fundamentals of Cordova), there are some differences that a pentester should know about:
- **Native Project Management**: when developers adotp Capacitor, configuration changes are made by editing the appropriate platform-specif configuration files directly, such as AndroidManifest.xml and Info.plist. 
- **Plugin Management**: Cordova copies plugin source code to the app before building, while in Capacitor all plugins are build as *frameworks*  on iOS and *libraries* on Android. Additionally, Capacitor doesn't modify native source code, in fact, any necessary native project settings (such as permissions on AndroidManifest and Info.plist) must be added manually. One major difference is the way plugins handle the JavaScript code they need in order to be executed from the WebView. Cordova requires plugins to ship their own JavaScript and manually call *exec()*. Capacitor, in contrast, registers and exports all JavaScript for each plugin based on the native methods it detects at runtime, so all plugin methods are available as soon as the WebView loads.

### Detecting app

Android
: .....

iOS
: .....

[Come riconoscere queste apps]

### Intercepting Traffic

[Come si intercetta il traffico.]

### Certificate Pinning

Android
: .....

iOS
: .....

[Come dovrebbe essere implementato il pinning, quali sono le soluzioni comuni e come verificarlo(??).]

### Debugging

Android
: .....

iOS
: .....

[Come si debugga l'app]

### Reverse Engineering

Android
: .....

iOS
: .....

[Come fare la reverse]

---

## React Native

React Native is a cross-platform solution that allows writing native apps using React (JavaScript or TypeScript). The native app executes JavaScript code with the native or custom JavaScript engine in a separate thread. Native JS engine uses the source code stored in the *.jsbundle* file, while custom JS engines can have various behaviours.

|                                           | React Native                                     |
|:------------------------------|:-----------------------------------------|
|**Code**                             |Javascript  + Java / Objective-C / Swift |
|**Compilation iOS**          |Interpreter                             |
|**Compilation Android**  |JIT                                                        |
|**UI Rendering**               |Native UI Controllers                                           |
|**UI Engineering**            |Customization with built-in UI components |

### Detecting app

All React Native applications (both Android and iOS) have unique fingerprints that make them easy to identify:

Android
: - `index.android.bundle` : a file in the *assets* directory
- `com.facebook.react`: an entry in the Android Manifest
- `com.facebook.react.devsupport.DevSettingsActivity`: an activity

iOS
: - `main.jsbundle`: a file in the *Payload* directory inside the app folder
- inside the ***assets*** directory there might be some references to React, for example ***node_modules/@react-navigation*** or ***node_modules/@react-native-****

>*the **bundle** file: it's the "Javascript". In development the bundle likely will come from your *react-native start* development server. That way if your code is changed the server will send a request to the client, through a websocket, to download the new code or update the code on the fly. So, you could say the bundle is dynamically generated from your source code. In production you probably want to use an offline bundle so that your code is already on the device and does not need to be downloaded. [Cit.](https://stackoverflow.com/questions/41960891/what-is-react-natives-bundle-and-its-purpose)

### Intercepting Traffic

Android
: - *Rooted & Unrooted device*: use the local proxy settings (WiFi settings)
- *Rooted & Unrooted device*: patch the apk to accept "user" certificates
- *Rooted device*: use ProxyDroid App


iOS
: - *Jailbroken & Unjailbroken device*: use the local proxy settings (WiFi settings)
- *Jailbroken & Unjailbroken device*: set up a VPN Server, download OpenVPN Client on the device

### Certificate Pinning

Android
: .....

iOS
: .....

[Come dovrebbe essere implementato il pinning, quali sono le soluzioni comuni e come verificarlo(??).]

### Debugging

Android
: .....

iOS
: .....

[Come si debugga l'app]

### Reverse Engineering

Android
: .....

iOS
: .....

[Come fare la reverse]

---


## References

General:
- [Android Application Exploitation - Red Team Village](https://av.tib.eu/media/49157)

Xamarin:
- [https://triskelelabs.com/intercepting-xamarin-mobile-app-traffic-2/](https://triskelelabs.com/intercepting-xamarin-mobile-app-traffic-2/)
- [https://mobile-security.gitbook.io/mobile-security-testing-guide/general-mobile-app-testing-guide/0x04f-testing-network-communication](https://mobile-security.gitbook.io/mobile-security-testing-guide/general-mobile-app-testing-guide/0x04f-testing-network-communication)
- [https://docs.microsoft.com/en-us/xamarin/cross-platform/internals/available-assemblies?context=xamarin/android](https://docs.microsoft.com/en-us/xamarin/cross-platform/internals/available-assemblies?context=xamarin/android)
- [https://www.codeguru.com/csharp/introduction-to-mono-for-android/](https://www.codeguru.com/csharp/introduction-to-mono-for-android/)

Flutter:
- [https://blog.nviso.eu/2019/08/13/intercepting-traffic-from-android-flutter-applications/](https://blog.nviso.eu/2019/08/13/intercepting-traffic-from-android-flutter-applications/)

Ionic & Cordova:
- [https://ionicframework.com/docs/](https://ionicframework.com/docs/)
- [https://ionic.io/resources/articles/what-is-apache-cordova](https://ionic.io/resources/articles/what-is-apache-cordova)

React Native:
- [https://www.cossacklabs.com/blog/react-native-app-security.html](https://www.cossacklabs.com/blog/react-native-app-security.html)
- [https://book.hacktricks.xyz/mobile-apps-pentesting/android-app-pentesting/react-native-application](https://book.hacktricks.xyz/mobile-apps-pentesting/android-app-pentesting/react-native-application)
