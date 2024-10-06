---
title: Mobile Security - Fighting with Frameworks
author: Paolo Serra
date: 9999-10-05 10:00:00
categories: [Topic, Mobile Security]
toc: true
author: paoloserra
media_subpath: /images/mobile-security-fighting-with-frameworks
image:
  path: /native/wallpaper.jpeg
---

![Device rooted/jailbroken](device_rooted.png){: .shadow .normal width="35" height="35" }    Device rooted/jailbroken
![Device](device.png){: .shadow .normal width="35" height="35" }    Device

## Intro

There are many resources online about mobile security and how to test the security of native applications. For example, [OWASP MSTG](https://mas.owasp.org/MASTG/) is the best resource, in my opinion. However, I have not found a resource well detailed, organized, and structured to approach security with hybrid mobile development frameworks. 
During my first mobile assessments, I spent a lot of time troubleshooting and figuring out why certain things worked in some applications but not others. Then, I realized the problem was often about the application's nature and the framework used to develop it. It may seem obvious, but it was not for me. That's why I decided to start this series about mobile frameworks. 

This series aims to create another online resource to approach and understand everything about the most popular mobile development frameworks. For each framework, I'll describe all the core concepts and techniques to conduct a security assessment when dealing with that framework: we will explore how to detect if the application has been developed with the framework or natively, how to intercept the traffic, how to implement the certificate pinning, how to debug and reverse engineer the application.

If you believe you can fight against mobile apps using the same techniques and approaches used with native applications, some slightly different mechanisms under the hood can drive you crazy and in the wrong direction.
Here is a list of the frameworks I would like to explore (I will try to keep the list updated): *React Native, Ionic, Cordova, PhoneGap, Xamarin ( nowadays deprecated and replaced by .NET MAUI), Flutter, IBM Worklight, NativeScript, Appcelerator, Corona, Qt, Sencha, Unity3D, 5App, Framework7*. There are others, but we will start with the most famous ones.



### Two words about Network Communications (skip this part if you are confident)

Typically, you can use a proxy to intercept communications between the mobile app and the server. The proxy allows you to inspect and modify the client-server communication and see how the server reacts to unexpected or untrusted data.
However, modern mobile frameworks provide built-in solutions for reducing the possibility of intercepting and proxying network communications. These safeguards primarily consist of certificate pinning and ignoring the system proxy settings. As with all client-side mobile app protections, an attacker can bypass them.

**Important**: *In this guide, I am not going to explain how to install your proxy's certificate in Android or iOS. Google is your friend, so don't be lazy and google it.*


---

## A quick overview to understand the follow-up: Native Applications

A native app is built for a specific platform, usually Android or iOS, as these are the most popular ones. Developers use a programming language designed for a specific platform: Java and Kotlin for Android, Swift and Objective-C for iOS. From a security standpoint, there aren’t many differences with hybrid solutions; you may find the same vulnerability in both. On the other hand, hybrid apps may be similar to web-based applications. The only concern regards the approach: how to test a native or a hybrid application.

|                         | Native Apps                                                        |
| :---------------------- | :----------------------------------------------------------------- |
| **Code**                | Java / Kotlin / Swift / Objective-C                                |
| **Compilation iOS**     | AOT                                                                |
| **Compilation Android** | JIT + AOT                                                          |
| **UI Rendering**        | Native Design Elements                                             |
| **UI Engineering**      | Native or Code Sharing for the cost of native experience (WebView) |

### Detecting app

![](android.png){: .shadow width="35" height="35" } Android
: Every app (native or not) has a codebase written in Java/Kotlin, so don't rely the detection on it. Actually, there are some elements that help you to discover how the app is built with:
- ***Check through Developer Options and layout bounds***: If it's a native app, it will display rectangles in each activity you open, whereas a hybrid app will display a cross line and no rectangles. Consider that this rule does not apply to all frameworks because some of them convert graphic elements into native ones.
- ***Use Webview to check***: If an app uses WebViews most of the time for each screen, it may be hybrid (**uiautomatorviewer**  is your friend). Keep in mind that this is not always the case: native apps can also benefit from a WebView.
- ***Look up explicit frameworks' elements inside the apk***: personally my favourite one and the fastest.

![](ios.png){: .shadow width="30" height="30" } iOS
: - Since I have never found a iOS-related way to detect precisely the kind of an app yet, ***Looking up explicit frameworks elements inside the ipa*** is the only way.


### Intercepting Traffic

![](android.png){: .shadow width="35" height="35" } Android
: - ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  use the local proxy settings (WiFi settings)
- ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  patch the apk to accept ***user*** certificates
- ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  use ***Bettercap*** to set up an ARP Poisoning attack
- ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  use ***NoPE Proxy*** (Burp extension) to carry on a DNS Spoofing attack
- ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  set up an Access Point and connect the iOS device to it
- ![](device_rooted.png){: .shadow width="35" height="35" }  use **ProxyDroid** app
- ![](device_rooted.png){: .shadow width="35" height="35" }  in case of **client isolation** enabled in the network and the iOS device and your laptop are not able to communicate: use SSH remote port forwarding
- ![](device_rooted.png){: .shadow width="35" height="35" } use the `/etc/hosts`{: .filepath} file to make the target domain point to the IP address of your interception proxy.

>When creating your test environment, it is critical to understand which solution you want to adopt and the differences among them. You might think that methods that work for both rooted and unrooted devices are the best options, but what about ignoring proxy settings, network security config integrity checks, or other mechanisms that make your life stressful during a pentest? There are several methods for redirecting traffic to another host (i.e. a proxy). That's where the difference between the **local proxy setting** and **ProxyDroid** takes place. The first does not require a rooted device, it sets up the proxying rules to forward all communication to the proxy server. On the other hand, ProxyDroid requires a rooted device because it uses IPtables (kernel level) to redirect the network traffic to an external host.

Once you have set up one of the solutions listed above, check [here](https://blog.nviso.eu/2020/11/19/proxying-android-app-traffic-common-issues-checklist/) to be sure you are not mad and everything is gonna work.

![](ios.png){: .shadow width="30" height="30" } iOS
: - ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  use the local proxy settings (WiFi settings)
- ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  set up a VPN Server and implement the IPTables rules to forward all incoming traffic on 80 and 443 ports to the proxy host and port. Lastly, download OpenVPN Client on the device.
- ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  use ***Bettercap*** to set up an ARP Poisoning attack
- ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  use ***NoPE Proxy*** (Burp extension) to carry on a DNS Spoofing attack
- ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" }  set up an Access Point and connect the iOS device to it
- ![](device_rooted.png){: .shadow width="35" height="35" }  in case of **client isolation** enabled in the network and the iOS device and your laptop are not able to communicate: use SSH remote port forwarding
- ![](device_rooted.png){: .shadow width="35" height="35" } use the `/etc/hosts`{: .filepath} file to make the target domain point to the IP address of your interception proxy.
 

*iOS doesn't support iptables at kernel level, so either you change the iOS kernel (good luck) or you need to be smart (like setting up a VPN Server).*


### Certificate Pinning

![](android.png){: .shadow width="35" height="35" } Android
: - **Configuration** Approach:
    - [Network Security Config](https://developer.android.com/training/articles/security-config) (**support from Android API 24+*)
- **Programatically** Approach:
    - [OkHttp](https://github.com/square/okhttp) (*support from Android API 17+*): It provides a *CertificatePinner* class that can be added to an *OkHttpClient* instance; basically you implement the pinning with two simple lines of code.
    - [TrustKit for Android](https://github.com/datatheorem/TrustKit-Android) (*support from Android API 17+*): it takes advatange of the network security config from Android API 24+ and that means you need to add some lines in that *.xml* file and initialize the object instance in the source code (check the documentation). 
    - There are probably others, but these two are the most commonly used and recommended.

![](ios.png){: .shadow width="30" height="30" } iOS
: - **Configuration** Approach (+ a few changes in the *native* source code):
    - [TrustKit for iOS](https://github.com/datatheorem/TrustKit):  but recommended only for ***Simple Apps*** and some scenarios, check the docs. It actually use *method swizzling*  (the process of replacing the implementation of a function at runtime) on `NSURLSession` and `NSURLConnection` delegates, for this reason you have to be sure you can do it. This enables TrustKit to be deployed without modifying the App's source code.
- **Programatically** Approach:
    - [TrustKit for iOS](https://github.com/datatheorem/TrustKit): for scenarios where it's not possible using the *Configuration* approach 
    - There are probably others, but Trustkit is the most commonly used and recommended.
    - [AFNetworking]():

![](android.png){: .shadow width="35" height="35" } ![](ios.png){: .shadow width="30" height="30" } WebView (Android and iOS)
: Whether the HTTP requests from a WebView component in a native mobile app follow certificate pinning rules depends on how the certificate pinning is implemented. If certificate pinning is implemented correctly, it should apply to all network requests made by the app, including those made by the WebView component. If the certificate pinning implementation is correctly integrated with the WebView component, the HTTP requests made by the WebView should also be subject to the certificate pinning rules, and will only trust the specified certificates. This typically involves creating a custom WebViewClient that uses the certificate pinning library to validate the server's certificate. If the certificate pinning is not integrated with the WebView, or is not implemented correctly, it is possible for the HTTP requests made by the WebView to bypass the certificate pinning rules, potentially making the app vulnerable to man-in-the-middle attacks. 


### Debugging 

![](android.png){: .shadow width="35" height="35" } Android
:  - ![](device_rooted.png){: .shadow width="35" height="35" }![](device.png){: .shadow width="35" height="35" } App in **Debug mode**: In this case, the app is already built to be debuggable, thus there is nothing special to do: use **Android Studio**, **Jadx Debugger** or any other debugging tool.
- ![](device_rooted.png){: .shadow width="35" height="35" } ![](device.png){: .shadow width="35" height="35" } App in  **Release Mode**: You may want to debug the application during a penetration test, but because it was built in release mode (you rarely receive an apk in debug mode), there is a good chance that the `debuggable` flag in the Manifest is not set to true. Nothing says you can debug the application in this case, so why not force it to be debuggable? The magic words are ***patching the apk***. You can do it yourself (if you are a noob, I recommend this solution so that you can understand how the entire operation should be done) or use a tool that automates everything (i.e. [medusa](https://github.com/Ch0pin/medusa)).

![](ios.png){: .shadow width="30" height="30" } iOS
: -  ![](device_rooted.png){: .shadow width="35" height="35" } Set up ***Debugserver***, a binary included with Xcode for attaching to the running iOS app to debug and run it on the device. After that, use ***lldb*** from your Mac to connect to your device. Setting Debugserver is not trivial as launching a simple tool; in the [References](/posts/mobile-security-fighting-with-frameworks/#references) section, you can find a Medium article. If you can afford ***IDA Pro remote iOS debugger***, you can use it.

#### WebView Debugging

![](android.png){: .shadow width="35" height="35" } Android
:  - WebViews has the `setWebContentsDebuggingEnabled` method to enable debugging on them. In WebView 113.0.5656.0 and later, this is enabled automatically if the app is declared as `android:debuggable="true"` in its manifest; otherwise, the default is **false**. That means to need to patch the application or, if you are lucky, you will find the webview with `setWebContentsDebuggingEnabled(true)`. Then, you can go to `chrome://inspect/#devices` to start debugging your app's webviews (remember to enable *USB Debugging* in **Settings > Developer Options**).


![](ios.png){: .shadow width="30" height="30" } iOS
:  - To enable inspection of web content within `WKWebView` on iOS, you can use the `isInspectable` property starting from , which you set on the `WKWebViewConfiguration` object. `On Safari`: First, on the iOS device, enable **Web Inspector** from `Settings > Safari > Advanced`{: .filepath}. On the mac, open Safari and enable **Show Develop menu in menu bar** under `Safari > Preferences > Advanced`{: .filepath}. Within Safari, select Develop in the toolbar. In the dropdown menu options, you should see the name of your device and app. Hover over the app name and click on localhost.

### Reverse Engineering

![](android.png){: .shadow width="35" height="35" } Android
: Since the codebase is Java or Kotlin, and the application is compiled, you can decompile it to obtain the Java/Kotlin sources. The files tend to be similar to the original ones, but if the developer applied minification or obfuscation techniques, it is harder, but not impossible, to understand the code. Moreover, an Android application could have native libraries developed with the NDK toolkit, which allows you to implement parts of your app by using languages such as C, C++, and even Rust. In this case, the reverse engineering process is as complex as reversing a typical binary. Decompiling an Android application is simple, some tools do it for you: [jadx-gui](https://github.com/skylot/jadx)  or [jd-gui](http://java-decompiler.github.io/). Once decompiled, you will find the native libraries `.so` in the `libs` folder.

![](ios.png){: .shadow width="30" height="30" } iOS
: You need to get your hands dirty, man. It’s not as straightforward as it is for Android; we’re talking about binaries here, so pick one of the following tools and get started: IDA Pro, Ghidra, Radare or Binary Ninja. If you have ever played with binaries and assembly/machine code/unreadable code, you know what I am talking about. Reversing binaries is another area of cybersecurity that you’ll have to deal with sooner or later. The iOS applications are essentially zip files: unzip them and you fill find the `<app_name>.app` binary file inside the `Payload` directory, that's your enemy.

---

## References

General:
- [Android Application Exploitation - Red Team Village](https://av.tib.eu/media/49157)
- [Simulating a Man-In-The-Middle Attack](https://mobile-security.gitbook.io/mobile-security-testing-guide/general-mobile-app-testing-guide/0x04f-testing-network-communication#simulating-a-man-in-the-middle-attack) 
- [Intercepting non-HTTP Network Traffic of Mobile Apps](https://bsddaemonorg.wordpress.com/2021/02/03/intercepting-non-http-network-traffic-of-mobile-apps)
- [Intercepting Network Communication of Mobile Apps](https://www.youtube.com/watch?v=x6yHbCON1u8)
- [Debug third party apps on iOS 12+](https://felipejfc.medium.com/the-ultimate-guide-for-live-debugging-apps-on-jailbroken-ios-12-4c5b48adf2fb)
- [Enabling the Inspection of Web Content in iOS Apps](https://webkit.org/blog/13936/enabling-the-inspection-of-web-content-in-apps/)
