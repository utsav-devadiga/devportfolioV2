---
setup: |
  import Layout from '../../layouts/BlogPost.astro'
title: Certificate Pinning in Android
publishDate: 14 Feb 2024
author: Utsav Devadiga
heroImage: "/assets/blog/hello-world.jpeg"
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
This command produces the base64-encoded SHA-256 hash of the server's public key, ready for pinning in your app.

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

