---
description: How I Reverse Engineered a Closed 2FA Solution to Work with Apps like Google Authenticator
tags: otp cisco hack hotp google-authenticator workaround bypass duo duo-security duo-mobile
image:
  path: /assets/img/duo-bypass.webp
  alt: Illustration of DUO Mobile lock being opened by a key
last_modified_at: 2024-04-07
featured: true
bluesky_post_uri: https://bsky.app/profile/nilsstreedain.com/post/3lbtfc3ctrs2c
---

In response to the escalating landscape of cybersecurity threats, organizations are increasing measures to authenticate their users securely. Among these measures, 2FA (two-factor authentication) has emerged as a pivotal strategy for many institutions. One of the largest 2FA applications, DUO Mobile, has become the go-to solution for many enterprises, educational institutions, and government agencies seeking to avoid data breaches and minimize cybersecurity threats.

DUO Mobile is a proprietary 2FA application developed by Cisco, a large technology and communications conglomerate. The DUO Mobile app generates unique time-sensitive passcodes and one-time push notifications to confirm user identities before granting them access to sensitive accounts and systems.

![Screenshot of DUO Authentication Page](/assets/img/duo-page.webp)

To achieve this, Cisco has used open standards developed by OATH (Initiative for Open Authentication) and encapsulated them within a proprietary 'application validation' API. This restricts users into their app, rather than allowing use of other open-source or publicly validated apps as OATH originally intended.

> “The Initiative for Open Authentication (OATH) addresses these challenges with standard, open technology that is available to all. OATH is taking an all-encompassing approach, delivering solutions that allow for strong authentication of all users on all devices, across all networks.” - [OATH](https://openauthentication.org)

In this blog post, we will delve into the inner workings of the DUO Mobile device enrollment process. Along the way, we will learn how to circumvent the DUO Mobile application validation API and activate DUO 2FA codes in third-party applications.

## Background Info - Current Open Standards
First, we'll break down some of the most common 2FA standards, including the one DUO encapsulates in its applciation. Then we’ll work on reverse-engineering DUO Mobile. For the sake of clarity, I will oversimplify to some extent.

Most 2FA methods rely on OTP (One Time Password) systems. These systems provide users with a single-use code to validate their identity alongside their password. When setting up 2FA with an online service, such as your Google account, the service generates a long random string of characters and numbers, referred to as a 'key'. This key is securely shared with your device (like your phone or laptop). Both the server and your device use this key to run an algorithm and produce a short code, typically 6 digits. If the code entered matches the one the server expects, your identity is validated.

To ensure security, it's crucial that the code can only be used once. This prevents attackers from reusing past codes to gain unauthorized access. This is achieved by providing extra information to the algorithm, which varies depending on the implementation. Below, we describe the two most common approaches.

### HOTP (HMAC-based One-Time Password)
- In this approach, the algorithm is provided with a 'counter' that increases with each successful authentication. When the client and server share the same count and key, they will both generate the same code, authenticating the user. However, when the counter is out of sync (i.e., a past code is used), authentication will fail.

### TOTP (Time-based One-Time Password)
- TOTP is a refinement of HOTP. Unlike HOTP which relies on a counter, TOTP generates codes based on the current time. This has multiple benefits, including preventing code desynchronization, often caused by communication problems. Additionally, it allows for validation from multiple devices because any device that knows the time can authenticate the user. This is especially beneficial in situations where a primary device is lost or damaged, ensuring access to important systems.

The vast majority of services (Google, Apple, Amazon, Instagram, etc.) prefer TOTP-based authentication for the benefits listed above. However, DUO uses HOTP.

## DUO Mobile Device Enrollment
In this section, we will go through the entire DUO device enrollment process. As a student at Oregon State University, this demonstration will be conducted using the DUO Managed Devices page that I have access to, with some information redacted for privacy. Note that DUO configurations may vary across institutions.

When adding a device to DUO, various device types can be selected. The OTP flow demonstrated in this blog post is applicable to both the **Mobile phone** and **Tablet** options. However, while these options function very similarly on the backend, the **Mobile phone** option requires adding and verifying a phone number. Because of this, I will be using the **Tablet** option for the remainder of this demonstration.

![Screenshot of DUO Device Type Selection Page](/assets/img/duo-devices.webp)

After selecting the device type, DUO also asks the user to select their operating system (iOS or Android). I have found this to be less important as we will not be utilizing their app, but both have worked in my testing. I have selected Android.

![Screenshot of DUO Operating System Selection Page](/assets/img/duo-os.webp)

Next, we are presented with a QR code that can be scanned into the DUO app; this is where things get interesting. In the image below, the **Continue** button is grayed out. This is because DUO doesn't allow device activation until the DUO Mobile app informs their servers that the code should be activated. We will talk more about this later; first, I want to dig into the QR code we are looking at.

![Screenshot of DUO Activation QR Code](/assets/img/duo-qr.webp)

## Decoding DUO QR Code
It seemed unlikely to me that Cisco would reinvent the wheel, so I thought it most likely they were using a previously developed OTP standard. The first thing I tried was scanning the code into Google Authenticator, but was met with the following error:

![Screenshot of Invalid Barcode Error in Google Authenticator](/assets/img/ga-error.webp)

So it’s not a valid OATH token; let’s dig further into the data contained within this QR code. Using a QR code reader, or the error above, we get the following data:
```
iysYvim1lzExImKx8Sqw-YXBpLTQ2MjE3MTg5LmR1b3NlY3VyaXR5LmNvbQ
```

I also found that this data can be extracted directly from the image URL of the QR code:
```
https://api-46217189.duosecurity.com/frame/qr?value=iysYvim1lzExImKx8Sqw-YXBpLTQ2MjE3MTg5LmR1b3NlY3VyaXR5LmNvbQ
```

At first glance, this seems relatively concerning. This type of request, where data is stored in a URL, is called a GET request. While not inherently less secure than other methods, full URLs are frequently stored and logged by both client devices and servers, meaning this token data could easily end up in the wrong person’s hands. Let’s look further into the data stored here and see if it is really an issue.

The data appears to consist of two separate encoded pieces of text, delimited by a '`-`' character. After running each segment through various text decoders, I found the second string of text (`YXBpLTQ2MjE3MTg5LmR1b3NlY3VyaXR5LmNvbQ`) to be a base64 encoding of `api-46217189.duosecurity.com`. While this does not represent a token, it is noteworthy that it matches the base URL of the QR code image. However, at this stage, I was unable to decode the meaning of `iysYvim1lzExImKx8Sqw`. So it was time to see how the DUO app used the data from this QR code.

## Intercepting DUO Mobile Communication
To monitor communication coming to and from the DUO app on my phone, I set up an internet proxy—a piece of software that routes each internet request from my phone, through my computer, allowing me to dig deeper into each communication the app is making.

With the proxy setup, I scanned the QR code into the DUO Mobile app and captured the following request. Notice the text `iysYvim1lzExImKx8Sqw` in the URL; this must be what the DUO app uses to activate the code, but where is the secret key?
```js
POST https://api-46217189.duosecurity.com/push/v2/activation/iysYvim1lzExImKx8Sqw?customer_protocol=1 HTTP/2.0
User-Agent: okhttp/2.7.5
{
	"jailbroken": "false",
	"architecture": “…”,        // Removed by author
	"region": "US",
	"app_id": "com.duosecurity.duomobile",
	"full_disk_encryption": "true",
	"passcode_status": "true",
	"platform": “…”,            // Removed by author
	"app_version": “…”,         // Removed by author
	"app_build_number": “…”,    // Removed by author
	"version": “…”,             // Removed by author
	"manufacturer": “…”,        // Removed by author
	"language": "en",
	"model": “…”,               // Removed by author
	"security_patch_level": “…” // Removed by author
}
```
At this point, the DUO Device Management webpage updated to show that the device had been activated:
![Screenshot of Activated DUO QR Code](/assets/img/duo-activated.webp)

and the DUO server responded to the client request with the following data. Notably, this data contains the value `hotp_secret`, our key! In the next section, we will attempt to activate HOTP in a third-party app using this secret (aka the key).
```js
{
  "akey": “…”,                  // Removed by author
  "app_status": 0,
  "current_app_version": “…”,   // Removed by author
  "current_os_version": “…”,    // Removed by author
  "customer_logo": “…”,         // Removed by author
  "customer_logo_md5": “…”,     // Removed by author
  "customer_name": “…”,         // Removed by author
  "force_disable_analytics": false,
  "has_backup_restore": true,
  "has_bluetooth_approve": false,
  "has_device_insight": true,
  "has_trusted_endpoints": false,
  "has_trusted_endpoints_permission_flow": false,
  "hotp_secret": "b40ed85d41039fca705293a286cad7ab",
  "instant_restore_status": "enabled_without_key",
  "os_status": 0,
  "pkey": “…”,                  // Removed by author
  "reactivation_token": “…”,    // Removed by author
  "requires_fips_android": true,
  "requires_mdm": 0,
  "security_checkup_enabled": true,
  "urg_secret": “…”             // Removed by author
}
```

## Activating HOTP Token
To activate the HOTP token, I followed the formatting instructions provided in [this archived Google repository](https://github.com/google/google-authenticator/wiki/Key-Uri-Format). This guide outlines the required structure for creating an HOTP URI, which serves as a standardized format for representing authentication tokens.

The HOTP URI includes essential details such as the account name (`Oregon State University`), the secret key, and the issuer (`DUO`). Here is the formatted URI below:
```
otpauth://hotp/Oregon State University?secret=b40ed85d41039fca705293a286cad7ab&issuer=DUO
```
With the URI created, the next step involved converting it into a QR code, to be scanned by Google Authenticator. Using a QR code generator, I converted the text-based URI into a QR code that could be scanned into any standard third-party 2FA app.

![Screenshot of Active OTP in Google Authenticator](/assets/img/ga-success.webp)

Success! Now to make the process easier.

## Creating DUO-Bypass
While informative, going through that entire process manually to use a third-party app is very cumbersome. For those who may want to use a third-party 2FA app without the hassle, I've created [DUO-Bypass](https://duo-bypass.nilsstreedain.com). This site allows anyone to instantly generate a HOTP URI QR Code using only the URL of the initial QR code image. I've also created a simple [bash script](https://github.com/nilsstreedain/duo-bypass/tree/main/script) that automates the same process, for more advanced users who may be uncomfortable with an untrusted site accessing their 2FA credentials. 

## Conclusion
In conclusion, while the importance of robust cybersecurity measures cannot be overstated, it is disheartening to witness the tightening grip of proprietary solutions like DUO Mobile on what was intended to be an open standard. The encapsulation of OATH standards within DUO's proprietary 'application validation' API restricts users to a closed ecosystem, contradicting the ethos of open authentication.

I hope this blog post shed some light on the implications of such proprietary practices. By monopolizing access to 2FA solutions, DUO effectively monetizes what was originally intended to be freely accessible through open standards like OATH. This monetization, however, comes without significant innovation or added value to users. Instead, it perpetuates a cycle of vendor lock-in, where users are beholden to proprietary solutions without meaningful alternatives.

Moreover, the requirement for DUO Mobile to communicate with its servers before codes can be activated introduces unnecessary complexity and dependency. Unlike the straightforward process envisioned by OATH standards, DUO's approach adds an extra layer of vulnerability and reliance on centralized systems. This dependence on server communication not only introduces potential points of failure but also contradicts the essence of decentralized security protocols.

In essence, the locking down of an open standard by DUO highlights broader issues of monopolization, innovation stagnation, and unnecessary complexity in the cybersecurity landscape. Moving forward, it is imperative for industry leaders like Cisco to prioritize interoperability, openness, and user empowerment. Only by embracing these principles can we ensure robust and accessible security solutions that truly serve the needs of users while mitigating evolving cyber threats.
