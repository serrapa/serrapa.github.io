---
title: Mobile Security - Fighting with Frameworks
author: Paolo Serra
date: 2021-10-04 10:00:00
categories: [Topic, Mobile Security]
toc: true
---

## Intro
In this article we are going to focus on the penetration tests when facing mobile applications built with the most common frameworks. It might seem you have to apply the same techniques when you are fighting against native applications, but there are some concepts a little bit different that can drive you crazy and into the wrong direction.

Here is a list of the frameworks I would like to talk about (I will try to keep the list updated): *Xamarin, Flutter, Cordova, PhoneGap, Ionic, React Native, IBM Worklight, NativeScript, Appcelerator, Corona, Qt, Sencha, Unity3D, 5App, Framework7, etc...*


|                     |Xamarin                      |Flutter                |Ionic                    |React Native               |NativeScript             |
|:--------------------|:----------------------------|:----------------------|:------------------------|:--------------------------|:------------------------|
|**Code**             |C#                           |Dart                   |HTML,CSS,TypeScript,Javascript                |Javascript                 |Javascript/TypeScript    |
|**Compilation iOS**  |AOT                          |xxxxx                  |JIT+WKWebView            |Interpreter                |Interpreter              |
|**Compilation Android**|JIT/AOT                      |xxxxx                  |JIT                      |JIT                        |JIT                      |
|**UI Engineering**   |Native / Code Sharing for the cost of native experience|xxxxx                  |Code Sharing for the cost of native experience|Customization with built-in UI components |Code Sharing for the cost of native experience|




### Network Communications
To discover and exploit vulnerabilities related to the network traffic, an attacker can use a local proxy to intercept communications between the mobile app and the server. A proxy allows the attacker to inspect, modify, repeat and understand how the mobile app communicates with the server and how the server might react to unexpected or untrusted data.
Most modern frameworks provide built-in solutions to limit an attacker’s ability to intercept communications with a proxy. Manly, these protections include Certificate pinning and/or ignoring system proxy settings. As is the case with all client-side mobile app protections however, these protections can be bypassed if an attacker has full control over the device.  

**NB**: *In this guide, I am not going to explain how to install your proxy's certificate in Android or iOS. Google is your friend so not being lazy and google it.*


---

## Native Applications
A Native app is created for a specific platform, usually Android or iOS, as these are the most popular ones. Developers use a programming language that suits a particular platform: Java and Kotlin for Android, Swift and Objective-C for iOS. From a security perspective, there aren't so many differences with the hybrid solutions, you may find the same vulnerability in both of them. Just keep in mind that hybrid is more like a web based environment to attack (Webviews' fault).

### Detecting app

Android
: Every app (native or not) has a codebase written in Java/Kotlin, so don't rely the detection on it. Actually, there are some elements that help you to discover how the app has been built with:
- ***Check through Developer Options and layout bounds***: if it is a native app, it will show rectangles in each activity that you open, while if it's a hybrid app, it will show a cross line and no rectangles. Consider that this rule is not valid for all frameworks, because some of them convert the graphic stuff in native ones.
- ***Use Webview to Check***: a method to check if the app is native or hybrid is by using the WebView method. If an app implements WebViews, most of the time for each screen, it may be hybrid (**uiautomatorviewer** is your friend). Keep in mind that's not always true: native apps can take advantage of a WebView as well.  
- ***Look up explicit frameworks elements inside the apk***: personally my favourite one.

iOS
: Since I have never found a iOS-related way to detect the type of an app yet, it remains for us to ***Look up explicit frameworks elements inside the ipa***.


### Intercepting Traffic
Android
: - *Rooted & Unrooted device*: use the local proxy settings (WiFi settings)
- *Rooted & Unrooted device*: patch the apk to accept "user" certificates
- *Rooted device*: use ProxyDroid App

>When you set your test environment, it is very important to understand which solution you want to adopt and what are the differences among all. You might come up with the idea where the methods which work for both rooted and unrooted are the best ones, but what about ignoring the proxy settings, network security config integrity checks or any other mechanisms that make your life worst during a pentest? Under the woods there are non-identical methods to redirect the traffic to another host (i.e. a proxy) and for this reason there is a difference between the **local proxy setting** and **ProxyDroid**. Let's start from this: the first one doesn't require a rooted device, while ProxyDroid does. Although the expected behaviour of these two solutions is the same, they acts in different ways: Proxydroid uses IPtables (kernel level) to redirect the traffic and that's why it needs root, whereas the local proxy setting sets a proxy host and forward to it all communication that are under these settings.

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
: During a penetration test, you might want to debug the application, but being it built in release mode (hardly ever you receive a debug mode apk), there are many chances that the Debuggable flag in the Manifest is not set to true. In this case, nothing says you can debug the application, so why not force the application to be debuggable? The magic words are ***patching the apk***. You can do it by yourself (if you are a noob I suggest this solution so that you can understand how the entire operation should be done) or by using a tool that automate all.

iOS
: LLDB + debugserver


### Reverse Engineering

Android
: The code base will be mostly Java or Kotlin so this makes easier to read and statically analyze the application and its source code, unless obfuscation techniques have been implemented. Sometimes you can also deal with applications that use NDK, a toolset that lets you implement parts of your app in native code, using languages such as C and C++.
In general, decompiling the Android application is not very difficult (some tools does it for you: [jadx-gui](https://github.com/skylot/jadx) or [jd-gui](http://java-decompiler.github.io/)).

iOS
: iOS is the funny part for the Reverse Engineering.

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

Furthermore, inside the ***lib*** directory you will find other references related to Xamarin such as the native shared libraries. These are normally going to the the monodroid engine (***libmonodroid.so***) and potentially other plugins that have been used by the developer.

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

  - the entry ***flutter.embedding*** in the Android Manifest
  - the folder ***io.flutter***
  - Shared Object files (***libflutter.so***) in the **lib** directory
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

## Cordova
Apache Cordova is an open-source mobile development framework. It allows you to use standard web technologies - HTML5, CSS3, and JavaScript for cross-platform development. Applications execute within wrappers targeted to each platform, and rely on standards-compliant API bindings to access each device's capabilities such as sensors, data, network status, etc.

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
[Cosa è]

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

### Detecting app
All applications (both Android and iOS) developed with React Native have some fingerprints that render themselves easy to recognize:

Android
: - ***index.android.bundle*** : a file in the *assets* directory
- ***com.facebook.react***: an entry in the Android Manifest
- ***com.facebook.react.devsupport.DevSettingsActivity***: an activity

iOS
: - ***main.jsbundle***: a file in the *Payload* directory inside the app folder
- inside the *assets* directory there might be some references to React, for example ***node_modules/@react-navigation*** or ***node_modules/@react-native-****

>*the **bundle** file: it is the "Javascript". In development the bundle likely will come from your *react-native start* development server. That way if your code is changed the server will send a request to the client, through a websocket, to download the new code or update the code on the fly. So, you could say the bundle is dynamically generated from your source code. In production you probably want to use an offline bundle so that your code is already on the device and does not need to be downloaded. [Cit.](https://stackoverflow.com/questions/41960891/what-is-react-natives-bundle-and-its-purpose)

### Intercepting Traffic

Android
: - use the local proxy settings (WiFi settings)
- use ProxyDroid App

[da mettere i riferimenti alla parte Native application]

iOS
: - use the local proxy settings (WiFi settings)
- set up a VPN Server, download OpenVPN on the device

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

## IBM Worklight
[Cosa è]

### Detecting app

Android
: .....

iOS
: .....

[Come riconoscere queste apps]

### Intercepting Traffic

Android
: .....

iOS
: .....

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

#### References
General:
- [Android Application Exploitation - Red Team Village](https://av.tib.eu/media/49157)

Xamarin:
- [https://triskelelabs.com/intercepting-xamarin-mobile-app-traffic-2/](https://triskelelabs.com/intercepting-xamarin-mobile-app-traffic-2/)
- [https://mobile-security.gitbook.io/mobile-security-testing-guide/general-mobile-app-testing-guide/0x04f-testing-network-communication](https://mobile-security.gitbook.io/mobile-security-testing-guide/general-mobile-app-testing-guide/0x04f-testing-network-communication)
- [https://docs.microsoft.com/en-us/xamarin/cross-platform/internals/available-assemblies?context=xamarin/android](https://docs.microsoft.com/en-us/xamarin/cross-platform/internals/available-assemblies?context=xamarin/android)
- [https://www.codeguru.com/csharp/introduction-to-mono-for-android/](https://www.codeguru.com/csharp/introduction-to-mono-for-android/)

Flutter:
- [https://blog.nviso.eu/2019/08/13/intercepting-traffic-from-android-flutter-applications/](https://blog.nviso.eu/2019/08/13/intercepting-traffic-from-android-flutter-applications/)

React Native:
- [https://www.cossacklabs.com/blog/react-native-app-security.html](https://www.cossacklabs.com/blog/react-native-app-security.html)
- [https://book.hacktricks.xyz/mobile-apps-pentesting/android-app-pentesting/react-native-application](https://book.hacktricks.xyz/mobile-apps-pentesting/android-app-pentesting/react-native-application)
