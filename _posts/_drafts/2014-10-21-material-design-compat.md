---
layout: post
title: Android 5.0 Compat Libraries
---

![My helpful screenshot]({{ site.url }}/resources/images/layout_structure_toolbars1.png)

Many Android developers have been itching to get their hands on Android 5.0 compatibility libraries to integrate it into their applications. I have gotten the chance to integrate it into Spottr (a data usage monitoring application). In this post I will discuss my process of integrating Material Design and the items that the app compatiblity has to offer.

### Integrating Compatibility Libraries

AppCompat v21, a library that backports Android 5.0 libraries to previous verisons of Android have been released. A mini sidebar on compatiblity libraries. There are multiple android support libraries that backport a different feature sets in Android. The v4 Support Library backports the largest set of APIs (related with Fragments, Pagers, Accessibility, and much more). The v7 compatibility libraries support ActionBar APIs back to Android API Level 7 (Android 2.1). There are many other support libraries that backport various features, you can checkthem out here [here](http://developer.android.com/tools/support-library/features.html#v7-appcompat)

To begin integrating the support libraries, you must add them to your build.gradle file.

    compile "com.android.support:support-v4:21.0.+"
    compile "com.android.support:appcompat-v7:21.0.+"

Also the project needs to compiled against Android 5.0 Libraries (API Level 21). In your build.gradle file, you will need to set the compileSdkVersion to 21 and the buildToolsVersion to '21.0.1'. Note this does not make your app only compatible with v21, it just compiles against v21. The minSdkVersion specifies the lowest API that your app is compatible with. 

### Working with Compatibility Libraries

