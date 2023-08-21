---
title: "Summary of build configurations in Android and iOS"
datePublished: Mon Aug 21 2023 23:22:56 GMT+0000 (Coordinated Universal Time)
cuid: cllli5z8d000c09mj4b469jv7
slug: summary-of-build-configurations-in-android-and-ios

---

### Android

* **Build types** are configurations that are used for different stages in the development lifecycle such as debug and release
    
* **Product flavors** represent different versions of your app that can be released to users such as free and paid.
    
* **Build variant** is a cross-product of build type and product flavor. It is the configuration Gradle uses to build the app. The programmer does not configure build variants directly but does configure the build types and product flavors that form them.
    

### iOS

* **Target** specifies a product to build such as an app, app extension, unit test, etc.
    
* **Build settings** are the configuration of the build process. It includes how Xcode compiles the source files, links executables, packages and distributes code, etc.
    
* **Build configuration** is a plain-text file to specify build settings for a specific build type, target, or project. For instance, you can assign different configuration to Debug or Release build types.
    
* **(Build) Scheme** contains a list of targets to build and any configuration and environment details related to the build.
    
* **Build phase** configuration allows you to specify the files and scripts used to build your target. For instance, when using Firebase authentication, you can write a script to use the right GoogleService-Info.plist
    

references:

* [Android documentation](https://developer.android.com/build)
    
* [iOS documentation](https://developer.apple.com/documentation/xcode/build-system)