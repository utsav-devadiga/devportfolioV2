---
setup: |
  import Layout from '../../layouts/BlogPost.astro'
title: Certificate Pinning in Android
publishDate: 14 Feb 2024
author: Utsav Devadiga
heroImage: "/assets/blog/sslpin.png"
description: Mastering Certificate Pinning in Android Applications
tags: [java, kotlin,android]
---

# Mastering Certificate Pinning in Android Applications: A Comprehensive Security Guide

In the quest to secure mobile applications from potential threats, Android developers are increasingly turning to certificate pinning as a robust line of defense. This technique, which ensures that an app communicates exclusively with its intended server, is crucial for safeguarding sensitive data against Man-In-The-Middle (MITM) attacks. This guide delves deep into the implementation of certificate pinning, offering detailed code examples, an exploration of Android's Network Security Configuration for pinning, and strategies to counteract bypass techniques such as those employed by Frida.

## Introduction to Certificate Pinning

Certificate pinning involves embedding the known certificate's public key or its hash into your application. This preemptive measure allows the app to verify the server's identity by matching the presented certificate to the one you've pinned. If the certificates match, the connection is deemed secure; if not, the app refuses to connect, effectively neutralizing potential MITM attacks.

## The Necessity of Certificate Pinning

Without certificate pinning, attackers can intercept encrypted communications by exploiting vulnerabilities in SSL/TLS protocols or by compromising Certificate Authorities (CAs). Certificate pinning closes this loophole by mandating the app to trust only the specified certificate or public key.

## Implementing Certificate Pinning in Android

### Step 1: Extracting the Public Key or Certificate Hash

Begin with extracting the hash or public key from your server's SSL certificate using OpenSSL:

```bash
openssl x509 -in mycertificate.pem -pubkey -noout | openssl rsa -pubin -outform der | openssl dgst -sha256 -binary | openssl enc -base64
```
This command produces the base64-encoded **SHA-256** hash of the server's public key, ready for pinning in your app.

### Step 2: Retrofit and OkHttp Configuration

Utilize Retrofit and OkHttp for network operations, configuring OkHttp to enforce certificate pinning:

``` java
OkHttpClient okHttpClient = new OkHttpClient.Builder()
    .certificatePinner(new CertificatePinner.Builder()
        .add("your.domain.com", "sha256/YourPublicKeyHash==")
        .build())
    .build();

Retrofit retrofit = new Retrofit.Builder()
    .baseUrl("https://your.domain.com")
    .client(okHttpClient)
    .addConverterFactory(GsonConverterFactory.create())
    .build();
```
# Addressing Advanced Bypass Techniques
While certificate pinning significantly enhances security, tools like Frida can dynamically modify app behavior to bypass these protections. Implementing runtime checks, using code obfuscation, and detecting debugging tools can mitigate these risks.

### Step 3: Leveraging Network Security Configuration

Android's Network Security Configuration provides a streamlined XML-based approach to certificate pinning, simplifying the management and updating of certificates.

# Setting Up Network Security Configuration

- 1.Define your Network Security Configuration in XML:
```xml
<network-security-config>
    <domain-config>
        <domain includeSubdomains="true">your.domain.com</domain>
        <pin-set expiration="2023-01-01">
            <pin digest="**SHA-256**">base64EncodedPublicKeyHash==</pin>
            <!-- Add more pins if needed / also you can add certificates -->
        </pin-set>
    </domain-config>
</network-security-config>
```
- 2.Reference the configuration in your app's manifest:
```xml
<application
    android:networkSecurityConfig="@xml/network_security_config"
    ... >
    ...
</application>
```

This method allows for easy certificate management, enhancing security without complicating the development process.

### Step 4: Custom Trust Manager for Advanced Scenarios

For scenarios requiring deeper control, like handling self-signed certificates , manually configuring a custom X509TrustManager and SSLContext is advisable. This approach provides a programmable layer for precise security handling.

For advanced scenarios, manually verify the server's certificate by implementing a custom X509TrustManager:

``` java
TrustManager[] trustManagers = new TrustManager[]{
    new X509TrustManager() {
        @Override
        public void checkClientTrusted(X509Certificate[] chain, String authType) {}

        @Override
        public void checkServerTrusted(X509Certificate[] chain, String authType) {
            // we overrrride any exisiting mechanism
        }

        @Override
        public X509Certificate[] getAcceptedIssuers() {
            return new X509Certificate[]{};
        }
    }
};
```
SSL Context Configuration
Configure the SSL context to use your custom trust manager:

```java
SSLContext sslContext = SSLContext.getInstance("TLS");
sslContext.init(null, trustManagers, new SecureRandom());
```

Retrofit and OkHttp Integration
Use the configured SSL context with OkHttp to enforce certificate pinning:

```java
OkHttpClient okHttpClient = new OkHttpClient.Builder()
    .sslSocketFactory(sslContext.getSocketFactory(), (X509TrustManager)trustManagers[0])
    .build();

```


Our security mechanism shines in its manual verification process, facilitated by the CertChecker class. This class compares the **SHA-256** hash of the server's public key against a hardcoded pin—a base64-encoded **SHA-256** hash of the legitimate server's public key:

```  java
class CertChecker {
    companion object {
        fun doesCertMatchPin(pin: String, cert: Certificate): Boolean {
            val certHash = cert.publicKey.encoded.toByteString().sha256()
            return certHash == pin.decodeBase64()
        }
    }
}
l̥```

By integrating CertChecker into the network call flow, we ensure that the app communicates only with the intended server, effectively preventing MITM attacks.

### Step 5: Comprehensive Error Handling
Implement robust error handling mechanisms to address certificate verification failures, ensuring users are promptly informed about potential security issues.

### Visualizing Certificate Pinning
Consider the following simplified diagram to understand the certificate pinning process better:

```scss
[Client App] ---(1. Secure Request)--> [Server]
    |                                     |
    |<---(2. Presents Certificate)--------|
    |
    |---(3. Verifies Certificate)-------->|
    |                                     |
    |<---(4. Secure Communication)--------|

```

- Secure Request: Initiation of a secure request by the app.
- Presents Certificate: The server presents its SSL certificate.
- Verifies Certificate: The app verifies the server'scertificate against the pinned certificate.
- Secure Communication: Upon successful verification, a secure - communication channel is established.

### Conclusion
Certificate pinning is a critical component of mobile app security, ensuring that communications between an app and its server are secure. By incorporating manual methods alongside Android's Network

<div style="text-align:center;margin-top:20px;font-family:Inter;"><strong>Take Care Guys </strong>🤗</div>




