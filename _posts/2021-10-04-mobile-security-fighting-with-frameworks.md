---
title: Mobile Security - Fighting with Frameworks
author: Paolo Serra
date: 9999-04-04 10:00:00
categories: [Topic, Mobile Security]
toc: true
author: paoloserra
img_path: /images/mobile-security-fighting-with-frameworks
image:
  path: wallpaper.webp
  width: 1000   # in pixels
  height: 300   # in pixels
---

![Device rooted/jailbroken](device_rooted.png){: .shadow .normal width="35" height="35" }    Device rooted/jailbroken
![Device](device.png){: .shadow .normal width="35" height="35" }    Device

> Change the date of this post to make it visible.
{: .prompt-danger }

## Intro

There are many resources about Mobile security and how to perform security tests in native applications, but things are changing so quickly these days, and the Mobile world is offering different solutions and technologies to develop applications. We must all keep up with them.

During my first mobile assessments, I spent a lot of time troubleshooting issues and figuring out why some things worked in some applications but not others. Finally, I realized that the issue was often the framework on which the application was built. It might seem obvious, but it wasn't for me at the time, which is why I decided to write this article. My goal is just to give you a panoramic of the most popular mobile development methods and some tips on how to handle mobile applications built using these frameworks.

Specifically, we will concentrate on how to approach a **penetration test** performed against mobile applications built with the most common frameworks. When fighting against native applications, it may appear that you should use the same techniques and approaches, but there are some slightly different concepts that can drive you crazy and into the wrong direction.

Here is a list of the frameworks I'd like to explore (I will try to keep the list updated), but only some of them will be so far: *Xamarin, Flutter, Cordova, PhoneGap, Ionic, React Native, IBM Worklight, NativeScript, Appcelerator, Corona, Qt, Sencha, Unity3D, 5App, Framework7*


|                     |Xamarin                      |Flutter                |Ionic                    |React Native               |NativeScript             |
|:--------------------|:----------------------------|:----------------------|:------------------------|:--------------------------|:------------------------|
|**Code**             |C#                           |Dart                   |HTML,CSS,TypeScript,Javascript                |Javascript                 |Javascript/TypeScript    |
|**Compilation iOS**  |AOT                          |AOT                  |JIT+WKWebView            |Interpreter                |Interpreter              |
|**Compilation Android**|JIT/AOT                      |AOT                  |JIT                      |JIT                        |JIT                      |
|**UI Engineering**   |Native Experience for the cost of Code Sharing| Native                  |Code Sharing for the cost of native experience|Customization with built-in UI components |Code Sharing for the cost of native experience|



### Two words about Network Communications (skip this part if you are confident)

An attacker can use a local proxy to intercept communications between the mobile app and the server to discover and exploit vulnerabilities. A proxy enables an attacker to inspect, modify, repeat, and comprehend how the mobile app communicates with the server, as well as how the server may react to unexpected or untrusted data.
Like Android and iOS themself, also modern frameworks provide built-in solutions for reducing an attacker's ability to intercept proxy communications. These safeguards primarily consist of certificate pinning and/or ignoring system proxy settings. However, as with all client-side mobile app protections, these can be circumvented if an attacker has complete control of the device.

**NB**: *In this guide, I am not going to explain how to install your proxy's certificate in Android or iOS. Google is your friend, so don't be lazy and google it.*


---

## A quick overview to understand the follow-up: Native Applications

A Native app is created for a specific platform, usually Android or iOS, as these are the most popular ones. Developers use a programming language that suits a particular platform: Java and Kotlin for Android, Swift and Objective-C for iOS. From a security standpoint, there aren't many differences with the hybrid solutions; you may find the same vulnerability in both. Keep in mind that hybrid is more like a web-based environment to attack (Webviews' fault). The only problem regards the approach: how to test a native or a hybrid application.

|                                           | Native Apps                                        |
|:------------------------------|:------------------------------------------|
|**Code**                             | Java / Kotlin / Swift / Objective-C  |
|**Compilation iOS**          |AOT                              |
|**Compilation Android**  |JIT                              |
|**UI Rendering**               |Native Design Elements                             |
|**UI Engineering**            |Native or Code Sharing for the cost of native experience (WebView)|

### Detecting app

Android
: Every app (native or not) has a codebase written in Java/Kotlin, so don't rely the detection on it. Actually, there are some elements that help you to discover how the app is built with:
- ***Check through Developer Options and layout bounds***: If it's a native app, it will display rectangles in each activity you open, whereas a hybrid app will display a cross line and no rectangles. Consider that this rule does not apply to all frameworks because some of them convert graphic elements into native ones.
- ***Use Webview to check***: If an app uses WebViews most of the time for each screen, it may be hybrid (**uiautomatorviewer**  is your friend). Keep in mind that this is not always the case: native apps can also benefit from a WebView.
- ***Look up explicit frameworks' elements inside the apk***: personally my favourite one. 

iOS
: Since I have never found a iOS-related way to detect precisely the kind of an app yet, it remains for us to ***Look up explicit frameworks elements inside the ipa***.


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

>When creating your test environment, it is critical to understand which solution you want to use and the differences among them. You might think that methods that work for both rooted and unrooted devices are the best, but what about ignoring proxy settings, network security config integrity checks, or any other mechanisms that make your life stressful during a pentest? There are non-identical methods for redirecting traffic to another host (i.e. a proxy) under the the woods, and this is why there is a difference between the **local proxy setting** and **ProxyDroid**. To begin, the first does not require a rooted device, whereas ProxyDroid does.
Although their expected behavior is the same, these two solutions behave differently: Proxydroid uses IPtables (kernel level) to redirect traffic, which is why it requires root, whereas the local proxy setting sets up a proxy host and forwards all communication to it.

Once you have set up one of the solutions listed above, check [here](https://blog.nviso.eu/2020/11/19/proxying-android-app-traffic-common-issues-checklist/) to be sure you are not mad and everything is gonna work.

iOS
: - ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  use the local proxy settings (WiFi settings)
- ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  set up a VPN Server and implement the IPTables rules to forward all incoming traffic on 80 and 443 ports to the proxy host and port. Lastly, download OpenVPN Client on the device
- ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  use ***Bettercap*** to set up an ARP Poisoning attack
- ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  use ***NoPE Proxy*** (Burp extension) to carry on a DNS Spoofing attack
- ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  set up an Access Point and connect the iOS device to it
- ![](device_rooted.png){: .shadow width="35" height="35" }  in case of “client isolation” activated in the Wireless network and the iOS device and your laptop are not able to communicate: use SSH remote port forwarding
- ![](device_rooted.png){: .shadow width="35" height="35" } use the `/etc/hosts`{: .filepath} file to make the target domain point to the IP address of your interception proxy.
 

*iOS doesn't support iptables at kernel level, so either you change the iOS kernel (good luck) or you need to be smart (like setting up a VPN Server).*


### Certificate Pinning

Android
: - **Configuration** Approach:
    - [Network Security Config](https://developer.android.com/training/articles/security-config) (**support from Android API 24+*)
- **Programatically** Approach:
    - [OkHttp](https://github.com/square/okhttp) (*support from Android API 17+*): It provides a *CertificatePinner* class that can be added to an *OkHttpClient* instance; basically you implement the pinning with two simple lines of code.
    - [TrustKit for Android](https://github.com/datatheorem/TrustKit-Android) (*support from Android API 17+*): it takes advatange of the network security config from Android API 24+ and that means you need to add some lines in that *.xml* file and initialize the object instance in the source code (check the documentation). 
    - There are probably others, but these two are the most commonly used and recommended.

iOS
: - **Configuration** Approach (+ a few changes in the *native* source code):
    - [TrustKit for iOS](https://github.com/datatheorem/TrustKit):  but recommended only for ***Simple Apps*** and some scenarios, check the docs. It actually use *method swizzling*  (the process of replacing the implementation of a function at runtime) on `NSURLSession` and `NSURLConnection` delegates, for this reason you have to be sure you can do it. This enables TrustKit to be deployed without modifying the App's source code.
- **Programatically** Approach:
    - [TrustKit for iOS](https://github.com/datatheorem/TrustKit): for scenarios where it's not possible using the *Configuration* approach 
    - There are probably others, but Trustkit is the most commonly used and recommended.

WebView (Android and iOS)
: [TO BE DONE]


### Debugging 

Android
:  - ![](device_rooted.png){: .shadow width="35" height="35" }![](device.png){: .shadow width="35" height="35" } App in **Debug mode**: In this case, the app is already built to be debuggable, thus there is nothing special to do: Use Android Studio, Jadx Debugger, or any other debugging tool
- ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" } App in  **Release Mode**: You may want to debug the application during a penetration test, but because it was built in release mode (you rarely receive a debug mode apk), there is a good chance that the `debuggable` flag in the Manifest is not set to true. Nothing says you can debug the application in this case, so why not force it to be debuggable? The magic words are ***patching the apk***. You can do it yourself (if you are a noob, I recommend this solution so that you can understand how the entire operation should be done) or use a tool that automates everything (i.e. medusa).

iOS
: -  ![](device_rooted.png){: .shadow width="35" height="35" } Setup ***Debugserver***, a binary included with Xcode for attaching to the running iOS app to debug, and run it on the device. After that, just use ***lldb*** from your Mac to connect to your device. If you are rich, you can also use ***IDA Pro remote iOS debugger***. Setting Debugserver is not trivial as launching a simple tool, in the [References](/posts/mobile-security-fighting-with-frameworks/#references) section you find a useful Medium article.

WebView (Android and iOS)
: [to do]


### Reverse Engineering

Android
: Unless obfuscation techniques are used, the code base will be mostly Java or Kotlin, making it easier to read and statically analyze the application and its source code. You may also encounter applications that use the NDK, a toolkit that allows you to implement parts of your app in native code using languages such as C and C++ (reversing these is more challenging, same as iOS). You will find the native libraries `.so` in the `libs` folder.
In general, decompiling an Android application is simple. (some tools does it for you: [jadx-gui](https://github.com/skylot/jadx)  or [jd-gui](http://java-decompiler.github.io/)).

iOS
: Eh eh... You need to get your hands dirty, man. It's not as straightforward as it is for Android; we're talking about binaries here, so pick one of the following tools and get started: IDA Pro, Ghidra, Radare or Binary Ninja. If you have ever played with binaries and assembly / machine code / unreadable code, you know what I am talking about, otherwise, it's another area of cybersecurity that you'll have to deal with sooner or later.

---

## Xamarin

Xamarin, which is a Microsoft product, provides a single platform to develop one application for multiple platforms with **.NET** and **C#** – iOS and Android in most cases. As it is a framework, it likes getting us more frustrated, but how? Simply ignoring system proxy settings by default. Additionally, Microsoft has already implemented secure coding practices in the Xamarin framework. As such, applications developed with this framework inherit these security controls (uff..). This makes Xamarin–based applications arguably more secure than applications developed from scratch using traditional or no frameworks. Since Xamarin has full access to native APIs and toolkits used by both iOS and Android applications, it doesn't only save developers time in building and maintaining the application, but also help them in achieving near-native look and performance.

|                                           | Xamarin                                                    |
|:------------------------------|:-----------------------------------------|
|**Code**                             |C#    |
|**Compilation iOS**          |AOT                              |
|**Compilation Android**  |JIT/AOT                                                        |
|**UI Rendering**               |Native Design Elements                                           |
|**UI Engineering**            |Native / Code Sharing for the cost of native experience   |

### Detecting app

Android
: It is rather simple to determine whether an Android app was created with Xamarin or not. There is a lot of data inside the apk structure (**apktool** is your friend), but the directory holding the assemblies is what we need.. In fact, going there you will find certain assemblies related to the Mono engine, a free and open-source .NET Framework-compatible software framework now being led by Xamarin that can be run on many software systems.
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
: You cannot decompile an iOS application like Android. iOS apps are compiled directly to machine code, with an aggressive optimization pass that tends to destroy much of the original code's structure. To extract [continua]

### Intercepting Traffic

Since any local proxy setting is ignored by Xamarin, the only way to intercept the traffic generated by an application is setting up a VPN Server in another machine, connect the device to it (i.e OpenVPN) and enable IPtables on the remote machine to redirect all the traffic to **Burp Suite** (it can stay on the same machine or another one, it doesn't matter at all). It is not the same to achieve it on Android and iOS.

Android
: - ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  ProxyDroid is enough (so a VPN Server is not needed)
- ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  use ***Bettercap*** to set up an ARP Poisoning attack
- ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  use ***NoPE Proxy*** (Burp extension) to carry on a DNS Spoofing attack
- ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  set up an Access Point and connect the iOS device to it
- ![](device_rooted.png){: .shadow width="35" height="35" } use the `/etc/hosts`{: .filepath} file to make the target domain point to the IP address of your interception proxy.

iOS
: - ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  Follow this [guide](https://blog.nviso.eu/2020/06/12/intercepting-flutter-traffic-on-ios/), even though it is for flutter, it works well; you need to create a VPN Server. Otherwise, here another [guide](https://triskelelabs.com/intercepting-xamarin-mobile-app-traffic-2/) for Xamarin.
- ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  use ***Bettercap*** to set up an ARP Poisoning attack
- ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  use ***NoPE Proxy*** (Burp extension) to carry on a DNS Spoofing attack
- ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  set up an Access Point and connect the iOS device to it
- ![](device_rooted.png){: .shadow width="35" height="35" } use the `/etc/hosts`{: .filepath} file to make the target domain point to the IP address of your interception proxy.


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

|                                           | Flutter                                        |
|:------------------------------|:------------------------------------------|
|**Code**                             | Dart  / C + C++ (Graphic Engine) |
|**Compilation iOS**          |AOT                              |
|**Compilation Android**  |AOT                                  |
|**UI Rendering**               |Native Design Elements                             |
|**UI Engineering**            |Native  |

### Detecting app
All applications developed in Flutter have some fingerprints that render themselves easy to recognize.

Android
: - the entry `flutter.embedding` in the Android Manifest
  - the folder `io.flutter`{: .filepath}
  - Shared Object files (`libflutter.so`) in the **lib** directory
  - **release-mode** applications have the following files:
    - `/resources/lib/arm-64-v8a/libflutter.so`{: .filepath}
    - `/resources/lib/arm-64-v8a/libapp.so`{: .filepath}
  - **debug-mode** applications have the following files:
    - `/resources/assets/flutter_assets/kernel_blob.bin`{: .filepath}
  - to make sure Dart is in place, look at the AOT snapshot of all the Dart application stored as binary blobs in the `assets`{: .filepath} directory (*isolate_snapshot_data, isolate_snapshot_instr,vm_snapshot_data* and *vm_snapshot_instr*)

iOS
: - the folder `Flutter.framework`{: .filepath} inside the `Payload\appname.app\Frameworks`{: .filepath} directory
- any other flutter package called: *flutter_package_name.framework*
- to make sure Dart is in place, look at the AOT snapshot of all the Dart application in the `Frameworks/App.framework/App`{: .filepath} binary (*kDartVmSnapshotData, kDartVmSnapshotInstructions, kDartIsolateSnapshotData* and *kDartIsolateSnapshotInstructions*)
- as always, the `assets`{: .filepath} directory may reveal precious information


### Intercepting Traffic

Since any local proxy setting is ignored by Dart, the only way to intercept the traffic generated by an application is setting up a VPN Server in another machine, connect the device to it (i.e OpenVPN) and enable IPtables on the remote machine to redirect all the traffic to burp suite (it can stay on the same machine or another one, it doesn't matter at all).
Achieving it for Android and iOS is not the same:

Android
: - ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  ProxyDroid is enough (so a VPN Server is not needed)
- ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  use ***Bettercap*** to set up an ARP Poisoning attack
- ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  use ***NoPE Proxy*** (Burp extension) to carry on a DNS Spoofing attack
- ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  set up an Access Point and connect the iOS device to it
- ![](device_rooted.png){: .shadow width="35" height="35" } use the `/etc/hosts`{: .filepath} file to make the target domain point to the IP address of your interception proxy.

iOS
: - ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  Follow this [guide](https://blog.nviso.eu/2020/06/12/intercepting-flutter-traffic-on-ios/)
- ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  use ***Bettercap*** to set up an ARP Poisoning attack
- ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  use ***NoPE Proxy*** (Burp extension) to carry on a DNS Spoofing attack
- ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  set up an Access Point and connect the iOS device to it
- ![](device_rooted.png){: .shadow width="35" height="35" } use the `/etc/hosts`{: .filepath} file to make the target domain point to the IP address of your interception proxy.




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
- cit: Here notice that Debug mode builds use a Dart virtual machine to run Dart code in order to enable stateful hot reload.

### Reverse Engineering

When you have to reverse a Flutter app, things get more complicated. Production binaries are not easy to reverse because the binary is converted to a Shared Object file (*.so), while for the debug versions is easy to find the source code and get it back.

Android
: .....

iOS
: .....



---

## Cordova (aka PhoneGap)

Apache Cordova (aka PhoneGap) is an open-source mobile development framework. It allows you to use standard web technologies - HTML5, CSS3, and JavaScript for cross-platform development. Applications execute within wrappers targeted to each platform and rely on standards-compliant API bindings to access each device's capabilities such as sensors, data, network status, etc. Cordova doesn't offer any UI

|                                           | Cordova                                                    |
|:------------------------------|:-----------------------------------------|
|**Code**                             |HTML, CSS, Javascript    |
|**Compilation iOS**          |JIT + WKWebView                              |
|**Compilation Android**  |JIT                                                        |
|**UI Rendering**               |HTML, CSS                                           |
|**UI Engineering**            |Code Sharing for the cost of native experience   |

### Detecting app

Android
: .....

iOS
: .....

[Come riconoscere queste apps]

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
 
 It's Javascript. Dive deeply into the source code.

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
: .....

iOS
: .....

[Come dovrebbe essere implementato il pinning, quali sono le soluzioni comuni e come verificarlo(??).]

### Debugging
We mentioned Web Technologies, therefore what are the best tools for debugging them? **Chrome DevTools** is the answer. Because we are dealing with an Ionic App, it is evident that the application functionality lies within the Javascript code; thus, we must shift our focus to that side. 

Android
: - `On Chrome` (**Debug & Release Mode**) Go to *Settings > Developer Options* (make sure they're enabled first) and ensure that the *developer options* switch is toggled on. Scroll down to *USB Debugging* and ensure that it is also enabled. Open the Chrome browser and navigate to the URL `chrome://inspect/#devices`. Once a WebView is created or the app starts, something will shows up on the page. Just click on *"inspect"*.

iOS
: - `On Safari` (**Debug & Release Mode**) First, on the iOS device, enable **Web Inspector** from *Settings > Safari > Advanced*. Next, open Safari on a Mac then enable **Show Develop menu in menu bar** under *Safari > Preferences > Advanced*. Within Safari, select Develop in the toolbar. In the dropdown menu options, you should see the name of your device and app. Hover over the app name and click on localhost.
- `On Chrome` : use a [WebKit proxy](https://github.com/google/ios-webkit-debug-proxy)


### Reverse Engineering
 
 It's Javascript. Dive deeply into the source code.

---

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

General:
- [Android Application Exploitation - Red Team Village](https://av.tib.eu/media/49157)
- [https://mobile-security.gitbook.io/mobile-security-testing-guide/general-mobile-app-testing-guide/0x04f-testing-network-communication#simulating-a-man-in-the-middle-attack](https://mobile-security.gitbook.io/mobile-security-testing-guide/general-mobile-app-testing-guide/0x04f-testing-network-communication#simulating-a-man-in-the-middle-attack) 
- [https://bsddaemonorg.wordpress.com/2021/02/03/intercepting-non-http-network-traffic-of-mobile-apps](https://bsddaemonorg.wordpress.com/2021/02/03/intercepting-non-http-network-traffic-of-mobile-apps)
- [https://www.youtube.com/watch?v=x6yHbCON1u8](https://www.youtube.com/watch?v=x6yHbCON1u8)
- [Debug third party apps on iOS 12+](https://felipejfc.medium.com/the-ultimate-guide-for-live-debugging-apps-on-jailbroken-ios-12-4c5b48adf2fb)

Xamarin:
- [https://triskelelabs.com/intercepting-xamarin-mobile-app-traffic-2/](https://triskelelabs.com/intercepting-xamarin-mobile-app-traffic-2/)
- [https://mobile-security.gitbook.io/mobile-security-testing-guide/general-mobile-app-testing-guide/0x04f-testing-network-communication](https://mobile-security.gitbook.io/mobile-security-testing-guide/general-mobile-app-testing-guide/0x04f-testing-network-communication)
- [https://docs.microsoft.com/en-us/xamarin/cross-platform/internals/available-assemblies?context=xamarin/android](https://docs.microsoft.com/en-us/xamarin/cross-platform/internals/available-assemblies?context=xamarin/android)
- [https://www.codeguru.com/csharp/introduction-to-mono-for-android/](https://www.codeguru.com/csharp/introduction-to-mono-for-android/)

Flutter:
- [https://blog.nviso.eu/2019/08/13/intercepting-traffic-from-android-flutter-applications/](https://blog.nviso.eu/2019/08/13/intercepting-traffic-from-android-flutter-applications/)
- [https://medium.com/jay-tillu/3-flutter-compilation-process-8fd18630ba7f](https://medium.com/jay-tillu/3-flutter-compilation-process-8fd18630ba7f)
- [https://medium.com/flutter/flutters-ios-application-bundle-6f56d4e88cf8](https://medium.com/flutter/flutters-ios-application-bundle-6f56d4e88cf8)

Ionic & Cordova:
- [https://ionicframework.com/docs/](https://ionicframework.com/docs/)
- [https://ionic.io/resources/articles/what-is-apache-cordova](https://ionic.io/resources/articles/what-is-apache-cordova)

React Native:
- [https://www.cossacklabs.com/blog/react-native-app-security.html](https://www.cossacklabs.com/blog/react-native-app-security.html)
- [https://book.hacktricks.xyz/mobile-apps-pentesting/android-app-pentesting/react-native-application](https://book.hacktricks.xyz/mobile-apps-pentesting/android-app-pentesting/react-native-application)
- [React Documentation - Debugging](https://reactnative.dev/docs/debugging)
