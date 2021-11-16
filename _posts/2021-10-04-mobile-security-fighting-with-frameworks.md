---
title: Mobile Security - Fighting with Frameworks
author: Paolo Serra
date: 2021-10-04 10:00:00
categories: [Topic, Mobile Security]
toc: true
---

#### TL;DR
In this article we are going to focus on the penetration tests when facing mobile applications built with the most common frameworks. It might seem you have to apply the same techniques when you are fighting against native applications, but there are some concepts a little bit different that can drive you crazy or, especially, into the wrong direction. Below the frameworks:
- Xamarin
- Flutter
- Cordova
- PhoneGap
- Ionic
- React Native
- IBM Worklight
- NativeScript
- Appcelerator
- Corona
- Qt
- Sencha
- Unity3D
- 5App
- Framework7


|                     |Xamarin                      |Flutter                |Ionic                    |React Native               |NativeScript             |
|:--------------------|:----------------------------|:----------------------|:------------------------|:--------------------------|:------------------------|
|**Code**             |C#                           |Dart                   |HTML,CSS,TypeScript,Javascript                |Javascript                 |Javascript/TypeScript    |
|**Compilation iOS**  |AOT                          |xxxxx                  |JIT+WKWebView            |Interpreter                |Interpreter              |
|**Compilation Android**|JIT/AOT                      |xxxxx                  |JIT                      |JIT                        |JIT                      |
|**UI Engineering**   |Native / Code Sharing for the cost of native experience|xxxxx                  |Code Sharing for the cost of native experience|Customization with built-in UI components |Code Sharing for the cost of native experience|




## Intro
To discover and exploit vulnerabilities related to the network traffic, an attacker can use a local proxy to intercept communications between the mobile app and the server. A proxy allows the attacker to inspect, modify, repeat and understand how the mobile app communicates with the server and how the server might react to unexpected or untrusted data.
Most modern frameworks provide inbuilt solutions to limit an attacker’s ability to intercept communications with a proxy. These protections include Certificate pinning and/or ignoring system proxy settings. As is the case with all client-side mobile app protections however, these protections can be bypassed if an attacker has full control over the device.  

**NB**: *In this guide, I am not going to explain how to install your proxy's certificate in Android or iOS because Google is your friend so not be lazy and type some words, it can take only few seconds, did you know that?*


---

## Native Applications
[Cosa è]
### Detecting app
#### Android
#### iOS
[Come riconoscere queste apps]

### Intercepting Traffic
#### Android
- use the local proxy settings (WiFi settings)
- use ProxyDroid App

>When you are setting your test environment, it is very important to understand which solution you are adopting and what are the differences between each others. Under the woods there are non-identical methods to redirect the traffic to another host (i.e. a proxy) and for this reason a precision has to be done between the **local proxy setting** and **ProxyDroid**. Let's start from this: the first one doesn't require a rooted device, while ProxyDroid does. Although the expected behaviour of these two solutions is the same, they acts in different ways. [continua]

Once you have set up one of the solutions listed above check [here](https://blog.nviso.eu/2020/11/19/proxying-android-app-traffic-common-issues-checklist/) to be sure you are not mad and doing things as they have to be done.

#### iOS
- use the local proxy settings (WiFi settings)
- set up a VPN Server, download OpenVPN on the device





#### IPTABLES: Android & iOS
I don't feel to say iptables requires a rooted device, because obviously the systems cannot allow you to use this tool as if nothing had happened.
### Certificate Pinning
#### Android
#### iOS
[Come dovrebbe essere implementato il pinning, quali sono le soluzioni comuni e come verificarlo(??).]
### Debugging
#### Android
#### iOS
[Come si debugga l'app]
### Reverse Engineering
#### Android
The code base will be mostly Java or Kotlin so this makes easier to read and statically analyze the application and its source code, unless obfuscation techniques have been implemented. Sometimes you can also deal with applications that use NDK, a toolset that lets you implement parts of your app in native code, using languages such as C and C++.
In general, decompile the Android application is not very difficult (some tools does it for you: [jadx-gui](https://github.com/skylot/jadx) or [jd-gui](http://java-decompiler.github.io/)).
#### iOS

---

## Xamarin
Xamarin, which is a Microsoft product, provides a single platform to develop one application for multiple platforms with **.NET and C#** – iOS and Android in most cases. As it is a framework, it likes getting us more frustrated, but how? Simply ignoring system proxy settings by default. Additionally, Microsoft has already implemented secure coding practices in the Xamarin framework. As such, applications developed with this framework inherit these security controls (uff..). This makes Xamarin–based applications arguably more secure than applications developed from scratch using traditional or no frameworks. Since Xamarin has full access to native APIs and toolkits used by both iOS and Android applications, it not only saves developers time in building and maintaining the application, but also help them in achieving near-native look and performance.
### Detecting app

#### Android
To recognize whether an Android app has been built using Xamarin or not is pretty straighforward, the structure of the apk and **apktool** tool are our friends. Inside it there is a lot of stuff but that help us is the directory containing the assemblies. In fact, going there you will find certain assemblies related to the Mono engine, a free and open-source .NET Framework-compatible software framework now being led by Xamarin that can be run on many software systems.

Let's see just some examples:

|Assembly             |Description                                                                   |
|:--------------------|:-----------------------------------------------------------------------------|
|*mscorlib.dll*         |Silverlight                          |
|*Mono.Android.dll*     |This assembly contains the C# bindings to the Android API                     |
|*Mono.Security.dll*    |Cryptographic APIs                    |
|*System.dll*           |Silverlight, plus types from other namespaces                          |
|*Xamarin.iOS.dll*      |This assembly contains the C# binding to the CocoaTouch API. This is only used in Unified iOS Projects |


Anyway, [here](https://docs.microsoft.com/en-us/xamarin/cross-platform/internals/available-assemblies) you can find a list of all available assemblies with their details.

Furthermore, inside the ***lib*** directory you will find other references related to Xamarin such as the native shared libraries. These are normally going to the the monodroid engine (***libmonodroid.so***) and potentially other plugins that have been used by the developer.

#### iOS
You cannot decompile an iOS application like Android. iOS apps are compiled directly to machine code, with an aggressive optimization pass that tends to destroy a lot of the structure of the original code. To extract [continua]

### Intercepting Traffic
Since any local proxy setting is ignored by Xamarin, the only way to intercept the traffic generated by an application is setting up a VPN Server in another machine, connect the device to it (i.e OpenVPN) and enable IPtables on the remote machine to redirect all the traffic to burp suite (it can stay on the same machine or another one, it doesn't matter at all).
Achieving it for Android and iOS is not the same.

**Android**
: ProxyDroid is enough (so a VPN Server is not needed)

**iOS**
: follow this [guide](https://blog.nviso.eu/2020/06/12/intercepting-flutter-traffic-on-ios/), even though it is for flutter, it works well; you need to create a VPN Server. Otherwise, here another [guide](https://triskelelabs.com/intercepting-xamarin-mobile-app-traffic-2/) for Xamarin.

### Certificate Pinning
#### Android
#### iOS
[Come dovrebbe essere implementato il pinning, quali sono le soluzioni comuni e come verificarlo(??).]
### Debugging
#### Android
#### iOS
[Come si debugga l'app]
### Reverse Engineering
#### Android
#### iOS
[Come fare la reverse]

---

## Flutter
Flutter is Google’s new open source mobile development framework that allows developers to write a single code base and build for Android, iOS, web and desktop. Flutter applications are written in Dart, a language created by Google more than 7 years ago. Do you think is different from Xamarin? No boy, it is also worse! If Xamarin ignores local proxy settings at least, Flutter (actually Dart) doesn’t use the system CA store (so neither system’s proxy settings) but a list of CA’s that’s compiled into the application!
### Detecting app
#### Android
All applications developed in Flutter have some fingerprints that render themselves easy to recognize:
- the entry ***flutter.embedding*** in the Android Manifest
- the folder ***io.flutter***
- Shared Object files (***libflutter.so***) in the **lib** directory
- the release applications have the following files:
  - /resources/lib/arm-64-v8a/libflutter.so
  - /resources/lib/arm-64-v8a/libapp.so
- the debug applications have the following files:
  - /resources/assets/flutter_assets/kernel_blob.bin

#### iOS
[Da scrivere]

### Intercepting Traffic
Since any local proxy setting is ignored by Dart, the only way to intercept the traffic generated by an application is setting up a VPN Server in another machine, connect the device to it (i.e OpenVPN) and enable IPtables on the remote machine to redirect all the traffic to burp suite (it can stay on the same machine or another one, it doesn't matter at all).
Achieving it for Android and iOS is not the same:

**Android**
: ProxyDroid is enough (a VPN Server is not needed)

**iOS**
: follow this [guide](https://blog.nviso.eu/2020/06/12/intercepting-flutter-traffic-on-ios/)


### Certificate Pinning
#### Android
#### iOS
[Come dovrebbe essere implementato il pinning, quali sono le soluzioni comuni e come verificarlo(??).]
### Debugging
#### Android
#### iOS
[Come si debugga l'app]
### Reverse Engineering
#### Android
#### iOS
When you have to reverse a Flutter app, things get more complicated. Production binaries are not easy to reverse because the binary is converted to a Shared Object file (*.so), while for the debug versions is easy to find the source code and get it back.

---

## Cordova
[Cosa è]
### Detecting app
#### Android
#### iOS
[Come riconoscere queste apps]
### Intercepting Traffic
[Come si intercetta il traffico.]
### Certificate Pinning
#### Android
#### iOS
[Come dovrebbe essere implementato il pinning, quali sono le soluzioni comuni e come verificarlo(??).]
### Debugging
#### Android
#### iOS
[Come si debugga l'app]
### Reverse Engineering
#### Android
#### iOS
[Come fare la reverse]

---

## Ionic
[Cosa è]
### Detecting app
#### Android
#### iOS
[Come riconoscere queste apps]
### Intercepting Traffic
[Come si intercetta il traffico.]
### Certificate Pinning
#### Android
#### iOS
[Come dovrebbe essere implementato il pinning, quali sono le soluzioni comuni e come verificarlo(??).]
### Debugging
#### Android
#### iOS
[Come si debugga l'app]
### Reverse Engineering
#### Android
#### iOS
[Come fare la reverse]

---

## React Native
React Native is a cross-platform solution that allows writing native apps using React (JavaScript or TypeScript). The native app executes JavaScript code with the native or custom JavaScript engine in a separate thread. Native JS engine uses the source code stored in .jsbundle file, while custom JS engines can have various behaviours.
### Detecting app
All applications (both Android and iOS) developed with React Native have some fingerprints that render themselves easy to recognize:
#### Android
- ***index.android.bundle*** : a file in the *assets* directory
- ***com.facebook.react***: an entry in the Android Manifest
- ***com.facebook.react.devsupport.DevSettingsActivity***: an activity

#### iOS
- ***main.jsbundle***: a file in the *Payload* directory inside the app folder
- inside the *assets* directory there might be some references to React, for example ***node_modules/@react-navigation*** or ***node_modules/@react-native-****

>*the **bundle** file: it is the "Javascript". In development the bundle likely will come from your *react-native start* development server. That way if your code is changed the server will send a request to the client, through a websocket, to download the new code or update the code on the fly. So, you could say the bundle is dynamically generated from your source code. In production you probably want to use an offline bundle so that your code is already on the device and does not need to be downloaded. [Cit.](https://stackoverflow.com/questions/41960891/what-is-react-natives-bundle-and-its-purpose)

### Intercepting Traffic

#### Android
- use the local proxy settings (WiFi settings)
- use ProxyDroid App

>When you are setting your test environment, it is very important to understand which solution you are adopting and what are the differences between each others. Under the woods there are non-identical methods to redirect the traffic to another host (i.e. a proxy) and for this reason a precision has to be done between the **local proxy setting** and **ProxyDroid**. Let's start from this: the first one doesn't require a rooted device, while ProxyDroid does. Although the expected behaviour of these two solutions is the same, they acts in different ways. [continua]

Once you have set up one of the solutions listed above check [here](https://blog.nviso.eu/2020/11/19/proxying-android-app-traffic-common-issues-checklist/) to be sure you are not mad and doing things as they have to be done.

#### iOS
- use the local proxy settings (WiFi settings)
- set up a VPN Server, download OpenVPN on the device

### Certificate Pinning
#### Android
#### iOS
[Come dovrebbe essere implementato il pinning, quali sono le soluzioni comuni e come verificarlo(??).]
### Debugging
#### Android
#### iOS
[Come si debugga l'app]
### Reverse Engineering
#### Android
#### iOS
[Come fare la reverse]

---

## IBM Worklight
[Cosa è]
### Detecting app
#### Android
#### iOS
[Come riconoscere queste apps]
### Intercepting Traffic
[Come si intercetta il traffico.]
### Certificate Pinning
#### Android
#### iOS
[Come dovrebbe essere implementato il pinning, quali sono le soluzioni comuni e come verificarlo(??).]
### Debugging
#### Android
#### iOS
[Come si debugga l'app]
### Reverse Engineering
#### Android
#### iOS
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
