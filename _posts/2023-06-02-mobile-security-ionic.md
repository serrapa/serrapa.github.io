---
title: Mobile Security - Fighting with Frameworks - Ionic
author: Paolo Serra
date: 9999-01-02 10:00:00
categories: [Topic, Mobile Security]
toc: true
author: paoloserra
img_path: /images/mobile-security-fighting-with-frameworks
image:
  path: /ionic/
---

![Device rooted/jailbroken](device_rooted.png){: .shadow .normal width="35" height="35" }    Device rooted/jailbroken
![Device](device.png){: .shadow .normal width="35" height="35" }    Device

> The blog post will be kept updated to offer a quality resource. DM me on Twitter or Linkedin for contributions.
{: .prompt-info }

Episode of [Fighting with Frameworks](/posts/mobile-security-fighting-with-frameworks/) season.


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
We mentioned Web Technologies, therefore what are the best tools for debugging them? **Chrome DevTools** is the answer. Because we are dealing with an Ionic App, it is evident that the application logic lies within the Javascript code; thus, we must shift our focus to that side. 

Android
: - `On Chrome` (**Debug & Release Mode**) Go to *Settings > Developer Options* (make sure they're enabled first) and ensure that the *developer options* switch is toggled on. Scroll down to *USB Debugging* and ensure that it is also enabled. Open the Chrome browser and navigate to the URL `chrome://inspect/#devices`. Once a WebView is created or the app starts, something will shows up on the page. Just click on *"inspect"*.

iOS
: - `On Safari` (**Debug & Release Mode**) First, on the iOS device, enable **Web Inspector** from *Settings > Safari > Advanced*. Next, open Safari on a Mac then enable **Show Develop menu in menu bar** under *Safari > Preferences > Advanced*. Within Safari, select Develop in the toolbar. In the dropdown menu options, you should see the name of your device and app. Hover over the app name and click on localhost.
- `On Chrome` : use a [WebKit proxy](https://github.com/google/ios-webkit-debug-proxy)


### Reverse Engineering
 
 It's Javascript. Dive deeply into the source code.

---
## References

- [https://ionicframework.com/docs/](https://ionicframework.com/docs/)
- [https://ionic.io/resources/articles/what-is-apache-cordova](https://ionic.io/resources/articles/what-is-apache-cordova)