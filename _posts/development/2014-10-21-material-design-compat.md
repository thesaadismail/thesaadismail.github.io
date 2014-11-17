---
layout: post
title: Android 5.0 Compat Libraries
---

![My helpful screenshot]({{ site.url }}/resources/images/layout_structure_toolbars1.png)

Many Android developers have been itching to get their hands on Android 5.0 compatibility libraries to integrate it into their applications. I am working on integrating it into [Spottr](http://getspottr.com) (a data usage monitoring application). In this post I will discuss my process of integrating Material Design and the items that the app compatiblity libraries have to offer.

### Integrating Compatibility Libraries

AppCompat v21, a library that backports Android 5.0 libraries to previous verisons of Android has been released. A mini sidebar on compatiblity libraries. There are multiple android support libraries that backport a different feature sets in Android. The v4 Support Library backports the largest set of APIs (related with Fragments, Pagers, Accessibility, and much more). The v7 compatibility libraries support ActionBar APIs back to Android API Level 7 (Android 2.1). You can check out the rest of the support libraries [here](http://developer.android.com/tools/support-library/features.html#v7-appcompat).

To begin integrating the support libraries, you must add them to your build.gradle file.

    compile "com.android.support:support-v4:21.0.+"
    compile "com.android.support:appcompat-v7:21.0.+"

The project will need to compiled against Android 5.0 Libraries (API Level 21) to take advantage of the latest APIs. In your build.gradle file, you will need to set the compileSdkVersion to 21 and the buildToolsVersion to '21.0.1'. Note this does not limit your app only be compatible with v21, it just compiles against v21. The minSdkVersion specifies the lowest API that your app is compatible with. 

### Working with Compatibility Libraries

##### App Theme
To start off, we need to update the theme of the application. This would be in your themes.xml or styles.xml. Your app theme's parent should be based off of Theme.AppCompat. In the example below I have chosen to use Theme.AppCompat.Light as my parent theme. Make sure to remove all other instances of AppTheme in your styles.xml and themes.xml files. 

_App Theme in themes.xml_
{% highlight xml %}
<?xml version="1.0" encoding="utf-8"?>
<resources>
    <style name="AppTheme" parent="Theme.AppCompat.Light">
        <!-- Set AppCompat’s actionBarStyle -->
        <item name="actionBarStyle">@style/BlueActionBar</item>

        <!-- Set AppCompat’s color theming attrs -->
        <item name="colorPrimary">@color/primary_color_blue</item>
        <item name="colorPrimaryDark">@color/primary_darker_color_blue</item>
    </style>
</resources>
{% endhighlight %}

In your main application theme, you will need to set the colorPrimary and the colorPrimaryDark. You can set these in your colors.xml file. 

![Color Attributes]({{ site.url }}/resources/images/color_attribs.png)

_App Colors in colors.xml_
{% highlight xml %}
    <color name="primary_color_blue">#08519c</color>
    <color name="primary_darker_color_blue">#08306b</color>
{% endhighlight %}

Next, make sure your activities extend ActionBarActivity instead of Activity. 
{% highlight java %}
public class MyActivity extends ActionBarActivity
{% endhighlight %}

##### Toolbar

Right now if you run your application, your actionbar will be styled with the primary color. If you are testing this on Android Lollipop, your status bar will also be colored. However, this is not using the new Toolbar APIs in the 5.0 and Compatibility Libraries. It is using the old ActionBar APIs. The reason to use the Toolbar apis are to have the ToolBar directly in your layouts. This will allow developers to interact with the ToolBar as any other view, allowing animations and etc. It is also possible to set the height of the ToolBar to various sizes to follow the new Material Design Guidelines.

To implement Toolbar, some changes will need to be made within your layouts and in your activities. The layout for the default activity would look like this:

{% highlight xml %}
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/container"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MyActivity"
    tools:ignore="MergeRootFrame" />
{% endhighlight %}

It contains the FrameLayout to allow the use of Fragments in the activity. Adding a Toolbar is not complicated. Since the toolbar is now part of our view, we will have the toolbar at the top of the screen withour view elements below it. To achieve this this, a vertical LinearLayout is needed as the parent element, and the ToolBar and FrameLayout inside the linear layout:

{% highlight xml %}
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
xmlns:tools="http://schemas.android.com/tools"
android:layout_width="match_parent"
android:layout_height="match_parent"
android:orientation="vertical"
tools:context=".MainActivity"
tools:ignore="MergeRootFrame" >

	<android.support.v7.widget.Toolbar
	    xmlns:app="http://schemas.android.com/apk/res-auto"
	    android:id="@+id/my_awesome_toolbar"
	    android:layout_height="wrap_content"
	    android:layout_width="match_parent"
	    android:minHeight="?attr/actionBarSize"
	    android:background="?attr/colorPrimary"/>

	<FrameLayout
	    android:layout_width="match_parent"
	    android:layout_height="match_parent"
	    android:id="@+id/container" />

</LinearLayout>
{% endhighlight %}

One additional change to be made is letting your activity know that you will be using the toolbar instead of the application bar. In your onCreate method, set your actionbar to your Toolbar in your layout:

{% highlight java %}
Toolbar toolbar = (Toolbar) findViewById(R.id.my_awesome_toolbar);
setSupportActionBar(toolbar);
{% endhighlight %}

Now, when you run the application, you might get an error (displayed below). If you get these errors, be sure to set windowActionBar to false in your AppTheme. This will tell your application, you will no longer be using ActionBar and will be using your own toolbar. If your application will have a mix of ActionBars and Toolbars (which I would not reccommend), you can use different themes with different settings of windowActionBar to achieve this type of functionality.

_windowActionBar error_
	
	java.lang.RuntimeException: Unable to start activity ComponentInfo{com.squarestaq.compattest/com.squarestaq.compattest.MyActivity}: java.lang.IllegalStateException: This Activity already has an action bar supplied by the window decor. Do not request Window.FEATURE_ACTION_BAR and set windowActionBar to false in your theme to use a Toolbar instead.

	Caused by: java.lang.IllegalStateException: This Activity already has an action bar supplied by the window decor. Do not request Window.FEATURE_ACTION_BAR and set windowActionBar to false in your theme to use a Toolbar instead.

_fix_
{% highlight xml %}
<style name="AppTheme" parent="Theme.AppCompat.Light">
	...
    <item name="windowActionBar">false</item>
    ...
</style>
{% endhighlight %}

##### Toolbar Shadow
_Unstyled Toolbar_
![Unstyled Toolbar]({{ site.url }}/resources/images/app-compat-toolbar-not-styled.png)

You will notice a slight change in your Toolbar, the font should be somewhat differet as well as the "flatness" of the Toolbar. Now you may be asking your self, where is my shadow?! A shadow will not exist under a toolbar for pre-lollipop devices due to the elevation attribute not being supported. The elevation attribute will only work on lollipop devices. 

To add a shadow to your toolbar, you will need to add it manually by using a 9 patch drawable. The Google I/0 app does exactly this to achieve a shadow under the toolbar [link](https://github.com/google/iosched/blob/master/android/src/main/res/layout/activity_my_schedule_narrow.xml#L51). Download the shadow 9-patch drawable [here](https://github.com/google/iosched/blob/master/android/src/main/res/drawable-xxhdpi/bottom_shadow.9.png) and add it to your FrameLayout in your activity as the windowForegorund attribute.
	
{% highlight xml %}
<FrameLayout
    android:id="@+id/container"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:foreground="@drawable/bottom_shadow" />
{% endhighlight %}

##### Toolbar Theme
We also need to set the theme of our toolbar so our text is the right color. This is most noticable when the primary color is dark and the toolbar text colors need to be changed to white. To do this, we need to create a separate theme for our Toolbar:

{% highlight xml %}
<style name="BaseToolbarStyle" parent="ThemeOverlay.AppCompat.ActionBar">
    <item name="android:textColorPrimary">#FFFFFF</item>
    <item name="android:textColorSecondary">#FFFFFF</item>
</style> 
{% endhighlight %}

_Styled Toolbar_
![My helpful screenshot]({{ site.url }}/resources/images/app-compat-toolbar-styled.png)

##### Toolbar Overflow Menu Theme
Our Toolbar Overflow Menu is currently white text on a black background. If we needed to change this, we need to specify a new popup theme for the toolbar. Create a BaseToolbarPopupStyle in your styles.xml. By setting the background, and textColorPrimary attributes we can control the toolbar's overlflow menu's background color and text color.

{% highlight xml %}
<style name="BaseToolbarPopupStyle" parent="Theme.AppCompat">
    <item name="android:background">#FFFFFF</item>
    <item name="android:textColorPrimary">#000000</item>
</style>
{%endhighlight %}

Also, remember to specify the toolbar theme and the toolbar popup in your toolbar tag: 

 <android.support.v7.widget.Toolbar
        xmlns:app="http://schemas.android.com/apk/res-auto"
        ...
        app:theme="@style/BaseToolbarStyle"
        app:popupTheme="@style/BaseToolbarPopupStyle"/>

##### Accent Colors

It is also possible set the accent colors of widgets. (tinting the widgets). To do this set the colorAccent attribute in your Toolbar's theme.

<item name=”colorAccent”>@color/accent</item>

On earlier versions of Android AppCompat will only tint a subset of UI Widgets:

* Everything provided by AppCompat’s toolbar (action modes, etc)
* EditText
* Spinner
* CheckBox
* RadioButton
* Switch (use the new android.support.v7.widget.SwitchCompat)
* CheckedTextView

##### Misc

There are also other ways of using the Toolbar, such as on the bottom or part of the screen. See this [link](http://android-developers.blogspot.com/2014/10/appcompat-v21-material-design-for-pre.html) to find out more information.

AppCompat also provides material design theme widgets. I was unable to find a complete list of widgets that applied the material design theme, however it seemsl ike any widgets that support tinting also implement the material design themes.

* EditText
* Spinner
* CheckBox
* RadioButton
* Switch (use the new android.support.v7.widget.SwitchCompat)
* CheckedTextView
  
### Troubleshooting

#### Duplicate Resources Error
If you run into this issue (detailed below) them make sure the AppTheme does not exist anywhere else in your other resource files. 

    Error: Duplicate resources: {path-to-app}/app/src/main/res/values/themes.xml:style/AppTheme, {path-to-app}/app/src/main/res/values/styles.xml:style/AppTheme

### Resources
* http://android-developers.blogspot.com/2014/10/appcompat-v21-material-design-for-pre.html
* http://android-developers.blogspot.com/2014/10/material-design-on-android-checklist.html
* http://android-developers.blogspot.com/2014/10/implementing-material-design-in-your.html
* http://www.murrayc.com/permalink/2014/10/28/android-changing-the-toolbars-text-color-and-overflow-icon-color/
* http://antonioleiva.com/material-design-everywhere/
