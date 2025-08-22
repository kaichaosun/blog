---
date: 2023-05-20
title: Javacard quick start
tags: ["Smartcard", "Javacard"]
categories: ["IoT", "Blockchain"]
draft: true
---

## Prerequisites

Install [jdk11](https://www.oracle.com/java/technologies/downloads/#java11-mac), jdk-11.0.19_macos-aarch64_bin.dmg, [GlobalPlatformPro](https://github.com/martinpaljak/GlobalPlatformPro) not work with jdk8

Set JAVA_HOME in ~/.zshrc (choose yours)

```bash
export JAVA_HOME=/Library/Java/JavaVirtualMachines/jdk-11.jdk/Contents/Home
```

## Compile and install applet

Get the souce code of test applet and javacard sdk,

```bash
# clone demo applet
git@github.com:devrandom/javacard-helloworld.git

# clone javacard sdk
git@github.com:martinpaljak/oracle_javacard_sdks.git
```

In javacard-helloworld folder, create file `gradle.properties`, and add its content,

```bash
com.fidesmo.gradle.javacard.home=/path/to/oracle_javacard_sdks/jc304_kit/
```

Build the applet,

```bash
./gradlew convertJavacard -i
```

## Operations with Card

Get the release or compiled version of GlobalPlatformPro,

```bash
java -jar gp.jar -info

java -jar gp.jar -list

# if the card is using a different key other than 404142434445464748494a4b4c4d4e4f
java -jar gp.jar -list -key-enc AC2AD8C8E2E874A4C6B514D7ECD5FBE5 -key-mac 86E6282CE0463C510FD4CB14D2A158EA -key-dek 7C290D97A5F4891F6C16ED7D2BB0A6E1

# change the key
java -jar gp.jar --lock 404142434445464748494A4B4C4D4E4F  -key-enc AC2AD8C8E2E874A4C6B514D7ECD5FBE5 -key-mac 86E6282CE0463C510FD4CB14D2A158EA -key-dek 7C290D97A5F4891F6C16ED7D2BB0A6E1
```

list response:

```
D97A5F4891F6C16ED7D2BB0A6E1
ISD: A000000151000000 (OP_READY)
     Parent:   A000000151000000
     From:     A0000001515350
     Privs:    SecurityDomain, CardLock, CardTerminate, CardReset, CVMManagement, TrustedPath, AuthorizedManagement, TokenVerification, GlobalDelete, GlobalLock, GlobalReg
istry, FinalApplication, ReceiptGeneration

PKG: A0000001515350 (LOADED)
     Parent:   A000000151000000
     Version:  255.255
     Applet:   A000000151535041

PKG: A00000016443446F634C697465 (LOADED)
     Parent:   A000000151000000
     Version:  1.0
     Applet:   A00000016443446F634C69746501

PKG: A0000000620204 (LOADED)
     Parent:   A000000151000000
     Version:  1.0

PKG: A0000000620202 (LOADED)
     Parent:   A000000151000000
     Version:  1.3
```