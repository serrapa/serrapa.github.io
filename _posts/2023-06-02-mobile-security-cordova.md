---
title: Mobile Security - Fighting with Frameworks - Cordova
author: Paolo Serra
date: 9999-01-02 10:00:00
categories: [Topic, Mobile Security]
toc: true
author: paoloserra
img_path: /images/mobile-security-fighting-with-frameworks
image:
  path: /cordova/
---

![Device rooted/jailbroken](device_rooted.png){: .shadow .normal width="35" height="35" }    Device rooted/jailbroken
![Device](device.png){: .shadow .normal width="35" height="35" }    Device

> The blog post will be kept updated to offer a quality resource. DM me on Twitter or Linkedin for contributions.
{: .prompt-info }

Episode of [Fighting with Frameworks](/posts/mobile-security-fighting-with-frameworks/) season.


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
## References

- [https://ionic.io/resources/articles/what-is-apache-cordova](https://ionic.io/resources/articles/what-is-apache-cordova)