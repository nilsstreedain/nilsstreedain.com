# macOS-DAL-Webcam-Workaround
After macOS 10.15 (aka macOS Catalina), DAL (Device Abstraction Layer) plugins are diabled by default. This can include things like software added webcams, for example I am using the [Canon EOS Webcam Utility](https://www.usa.canon.com/internet/portal/us/home/support/self-help-center/eos-webcam-utility/) which may not work without this workaround to work in some system apps. Third-party apps can specifically enable DAL plugins with the [com.apple.security.cs.disable-library-validation](https://developer.apple.com/documentation/bundleresources/entitlements/com_apple_security_cs_disable-library-validation?language=objc) key in their Info.plist. Modifying third party applications to add this key and re-codesign the app may be an option, however this is not possible for certain system applications such as FaceTime.

## :warning: Warning
Though this workaround is very useful, it can have some serious security implications so be aware when preforming this workaround. The method below requires disabling a feature of macOS's SIP (System Integrity Protection) which is specifically designed to protect macOS system functions and applications from being altered or modified. Keep in mind that the following will boot the system without complete file system protection.

## How To Enable/Disable DAL plugins:
The primary method to enable DAL plugins requires disabling file system protection in macOS SIP. This does not completely disable SIP, just some file system protection however this could still be quite dangerous security wise on your computer.

### Enabling DAL plugins (disabling a SIP feature) can be done with the following steps:
1. Boot your Mac into Recovery Mode (hold Command + R during startup)
2. Choose Language
3. Click the 'Utilities' tab on the top bar
4. Select 'Terminal' to open the Terminal application
5. Type ```csrutil enable --without fs``` and press enter
6. Reboot your Mac

### Disabling DAL plugins (enabling all SIP features) can be done with the following steps:

1. Boot your Mac into Recovery Mode (hold Command + R during startup)
2. Choose Language
3. Click the 'Utilities' tab on the top bar
4. Select 'Terminal' to open the Terminal application
5. Type ```csrutil enable``` and press enter
6. Reboot your Mac

#### Note:
You can also check the status of SIP using the ```csrutil status``` command in Terminal.
