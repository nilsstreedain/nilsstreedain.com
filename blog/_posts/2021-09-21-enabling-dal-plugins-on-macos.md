---
title: Enabling DAL plugins on macOS
description: How to enable DAL (Device Abstraction Layer) plugins on macOS for webcams and other devices.
tags: macos sip dal workaround webcam macos-sip macos-webcam
image:
  path: /assets/img/dal-plugins.webp
  alt: Mac logo and illustration of a webcam
last_modified_at: 2022-03-03
featured: true
---
After macOS 10.15 (Catalina), 'Device Abstraction Layer' (DAL) plugins are disabled by default. DAL plugins are used across many applications including software defined webcams, device printers, and more. This guide was created after failing to setup [Canon EOS Webcam Utility](https://www.usa.canon.com/internet/portal/us/home/support/self-help-center/eos-webcam-utility/) which may not work in some system apps on macOS Catalina. Third-party apps can specifically enable DAL plugins with the [com.apple.security.cs.disable-library-validation](https://developer.apple.com/documentation/bundleresources/entitlements/com_apple_security_cs_disable-library-validation?language=objc) key in their Info.plist. Modifying third party applications to add this key and re-codesign the app may be an option, however this is not possible for certain system applications such as FaceTime.

## ⚠️ Warning
Though this workaround is very useful, it can have some serious security implications so be aware when preforming this workaround. The method below requires disabling a feature of macOS 'System Integrity Protection' (SIP) which is specifically designed to protect macOS system functions and applications from being altered or modified. Keep in mind that the following will boot the system without complete file system protection.

## How To Enable/Disable DAL plugins:
The primary method to enable DAL plugins requires disabling file system protection in macOS SIP. This does not completely disable SIP, just some file system protection however this could still be quite dangerous security wise.

### Enabling DAL plugins (disabling a SIP feature) can be done with the following steps:
1. Boot your Mac into Recovery Mode (hold Command + R during startup)
2. Choose Language
3. Click the 'Utilities' tab on the top bar
4. Select 'Terminal' to open the Terminal application
5. Type `csrutil enable --without fs` and press enter
6. Reboot your Mac

### Disabling DAL plugins (enabling all SIP features) can be done with the following steps:
1. Boot your Mac into Recovery Mode (hold Command + R during startup)
2. Choose Language
3. Click the 'Utilities' tab on the top bar
4. Select 'Terminal' to open the Terminal application
5. Type `csrutil enable` and press enter
6. Reboot your Mac

#### Note:
You can also check the status of SIP using the `csrutil status` command in Terminal.
