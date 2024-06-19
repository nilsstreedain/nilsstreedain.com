---
title: Enabling Generative Playground on macOS 15.0 Beta 1
description: How to Enable Apple Intelligence Generative Playground on macOS 15.0 Beta 1
tags: apple macos ios macos15 ios18 generativeplayground ai beta intelligence appleintelligence generative playground
image:
  path: /assets/img/generative-playground.webp
  alt: Screenshot of Generative Playground running on macOS 15.0 Beta 1
last_modified_at: 2024-07-18
featured: true
---
Last week at WWDC, Apple unveiled their latest software advancements, showcasing 'Apple Intelligence'—their cross-platform suite of AI tools. Among these tools is Generative Playground, a dedicated iOS/macOS application for generating images, which has sparked some controversy.

While none of the announced "Apple Intelligence" features are available by default in the initial software betas, the Generative Playground app can be launched and run on some supported Mac devices through a series of macOS adjustments. Below, I will outline the process for enabling this application.

## ⚠️ SIP Warning
Although this workaround is intriguing, it carries serious security implications. The method described involves disabling a feature of macOS called System Integrity Protection (SIP), which is specifically designed to safeguard macOS system functions and applications from unauthorized modifications. Disabling SIP temporarily compromises the security of your macOS system.

## ⚠️ Beta Warning
Furthermore, this workaround requires enabling a feature intentionally disabled in beta software releases. This could lead to unexpected consequences and data loss. Proceed only if you understand the risks involved and are prepared for potential issues.

## Step 1: Disable macOS SIP
After the initial betas were released, testers noted that Generative Playground application binaries are included in macOS 15.0 Beta 1, albeit in a limited state. To fully enable the application, SIP must be disabled. Here's how to begin:

__Note: This guide is specifically for Apple Silicon devices.__

### Recovery Mode:
1. On your Mac, click the **Apple logo** on the upper left corner and select **Shut Down**.
  - Wait for your Mac to shut down completely. A Mac is completely shut down when the screen is black and any lights (including in the Touch Bar and keyboard) are off.
2. Press and hold the power button on your Mac until the system volume and the **Options** button appear.
3. Click the **Options** button, then click **Continue**.
4. If asked, select a volume to recover, then click **Next**.
5. Select an administrator account, then click **Next**.
6. Enter the password for the administrator account, then click **Continue**.
  - When the Recovery app appears in the menu bar, you can choose any of the available options in the window or the menu bar.

### System Integrity Protection Configuration
1. Select the **Utilities** tab in the macOS menu bar.
2. Choose **Terminal** from the dropdown list.
3. Disable SIP by entering the following command, then pressing enter:
```
csrutil enable
```
4. Reboot your Mac.

__Note: You can also check the status of SIP using the `csrutil status` command in Terminal.__

## Step 2: Enable arm64e
To run the application on Apple Silicon Macs, we must enable the arm64e architecture for the application. Run the following commands to enable arm64e, and then reboot your Mac.
```
sudo nvram boot-args="amfi_get_out_of_my_way=1 -arm64e_preview_abi"
```

```
sudo reboot
```

## Step 3: Modify App Contents
After disabling SIP and enabling the necessary architecture, the next step involves modifying the Generative Playground application itself. This modification allows the app to fully utilize its features, which are initially disabled in the beta release. To do this, we will be downloading a macOS dynamic library created by [@eveiyneee](https://x.com/eveiyneee) and running it on the installed `GenerativePlayground.app` application.
1. Download the [libplainhook.dylib](/assets/files/libplainhook.dylib) file.
2. Open Terminal and run the following command to resign the library.
```
xattr -sc Downloads/libplainhook.dylib
```
3. Use the following command to run the library on the application binary.
```
DYLD_INSERT_LIBRARIES=./Downloads/libplainhook.dylib /System/Applications/GenerativePlayground.app/Contents/MacOS/GenerativePlayground
```
5. Control-click the application icon in the Dock and select **Options** > **Keep in Dock**
4. Close Terminal and GenerativePlayground, then open the application normally by selecting it in the dock

## Step 4: Re-Enable macOS SIP
After completing the modifications and ensuring the app functions correctly, the final step is to re-enable macOS System Integrity Protection (SIP). This restores the security protections to your macOS system that were temporarily disabled for the modification process.

Follow the process in step 1 for entering recovery mode and opening terminal, then use the command `csrutil enable` instead of `csrutil disable`. Finally, reboot your Mac, and ensure the GenerativePlayground application still opens. If not, try each step again.
