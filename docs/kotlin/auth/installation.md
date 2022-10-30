# Installation

Kotlin implementation of WalletConnect v2 Auth protocol for Android applications.

![Maven Central](https://img.shields.io/maven-central/v/com.walletconnect/auth)

## Requirements

* Android min SDK 23
* Java 11

## Installation

root/build.gradle.kts:

```gradle
allprojects {
 repositories {
    mavenCentral()
 }
}
```

app/build.gradle

```gradle
implementation("com.walletconnect:auth:release_version")
```
