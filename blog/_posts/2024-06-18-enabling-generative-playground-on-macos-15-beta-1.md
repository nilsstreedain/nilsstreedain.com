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
Last week Apple unveiled their latest software advancements at WWDC, including "Apple Intelligence", their cross-platform suite of AI tools. Including Generative Playground, a dedicated iOS/macOS stable diffusion application for generating images, and one of the more controversial tools.

While none of the announced "Apple Intelligence" are available by default in the initial software betas, through some macOS trickery, the app can be launched and run on some supported Mac devices. Below I will describe the process for enabling this application. 

## ⚠️ SIP Warning
Though this workaround is very interesting, it can have some serious security implications. The method below requires disabling a feature of macOS ‘System Integrity Protection’ (SIP) which is specifically designed to protect macOS system functions and applications from being altered or modified. Keep in mind that the following will boot the system without complete file system protection.

## ⚠️ Beta Warning
Additionally, this workaround requires enabling an intentionally disabled feature within a beta release of software. This could cause significant unintended consequences and is clearly outside of general expectations. Only follow this guide for learning purposes if you are prepared to lose any data stored on your device. Do at your own risk.

## Step 1: Disable macOS SIP
After the initial betas were released, testers quickly noticed that the Generative Playground application binaries appear to be included on macOS 15.0 Beta 1. However when these are run, they only open a very limited version of the app. Though the compiled code is included, it is disabled. To run the full application, it must be modified, which requires disabling macOS SIP. To get started, reboot your mac into recovery mode:

__Note: This guide is only intended of Apple Silicon Devices__

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
Finally, we must modify the app contents to allow the app to launch with these features enabled. To do this, we will be downloading a macOS dynamic library created by [@eveiyneee](https://x.com/eveiyneee) and running it on the installed `GenerativePlayground.app` application.
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
Follow the process in step 1 for entering recovery mode and opening terminal, then use the command `csrutil enable` instead of `csrutil disable`. Finally, reboot your Mac, and ensure the GenerativePlayground application still opens. If not, try each step again.
