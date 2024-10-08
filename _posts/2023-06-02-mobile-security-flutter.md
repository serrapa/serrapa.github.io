---
title: Mobile Security - Fighting with Frameworks - Flutter
author: Paolo Serra
date: 9999-01-02 10:00:00
categories: [Topic, Mobile Security]
toc: true
author: paoloserra
media_subpath: /images/mobile-security-fighting-with-frameworks
image:
  path:  /flutter/wallpaper.jpeg
---

![Device rooted/jailbroken](device_rooted.png){: .shadow .normal width="35" height="35" }    Device rooted/jailbroken
![Device](device.png){: .shadow .normal width="35" height="35" }    Device

> The blog post will be kept updated to offer a quality resource. DM me on Twitter or Linkedin for contributions.
{: .prompt-info }

Episode of [Fighting with Frameworks](/posts/mobile-security-fighting-with-frameworks/) season.


## Flutter

Flutter is Google’s new open source mobile development framework that allows developers to write a single code base and build for Android, iOS, web and desktop. Flutter applications are written in Dart, a language created by Google more than 7 years ago. Do you think is different from Xamarin? No boy, it is also worse! If Xamarin ignores local proxy settings at least, Flutter (actually Dart) doesn’t use the system CA store (and neither system’s proxy settings) but a list of CA’s that’s compiled in the application!

|                         | Flutter                               |
| :---------------------- | :------------------------------------ |
| **Code**                | Dart  / C + C++ (Graphic Engine)      |
| **Compilation iOS**     | AOT (production) / JIT (development)  |
| **Compilation Android** | AOT  (production) / JIT (development) |
| **UI Rendering**        | Native Design Elements                |
| **UI Engineering**      | Native                                |

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
## References

- [https://blog.nviso.eu/2019/08/13/intercepting-traffic-from-android-flutter-applications/](https://blog.nviso.eu/2019/08/13/intercepting-traffic-from-android-flutter-applications/)
- [https://medium.com/jay-tillu/3-flutter-compilation-process-8fd18630ba7f](https://medium.com/jay-tillu/3-flutter-compilation-process-8fd18630ba7f)
- [https://medium.com/flutter/flutters-ios-application-bundle-6f56d4e88cf8](https://medium.com/flutter/flutters-ios-application-bundle-6f56d4e88cf8)
