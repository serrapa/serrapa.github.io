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

## References

General:
- [Android Application Exploitation - Red Team Village](https://av.tib.eu/media/49157)
- [https://mobile-security.gitbook.io/mobile-security-testing-guide/general-mobile-app-testing-guide/0x04f-testing-network-communication#simulating-a-man-in-the-middle-attack](https://mobile-security.gitbook.io/mobile-security-testing-guide/general-mobile-app-testing-guide/0x04f-testing-network-communication#simulating-a-man-in-the-middle-attack) 
- [https://bsddaemonorg.wordpress.com/2021/02/03/intercepting-non-http-network-traffic-of-mobile-apps](https://bsddaemonorg.wordpress.com/2021/02/03/intercepting-non-http-network-traffic-of-mobile-apps)
- [https://www.youtube.com/watch?v=x6yHbCON1u8](https://www.youtube.com/watch?v=x6yHbCON1u8)
- [Debug third party apps on iOS 12+](https://felipejfc.medium.com/the-ultimate-guide-for-live-debugging-apps-on-jailbroken-ios-12-4c5b48adf2fb)