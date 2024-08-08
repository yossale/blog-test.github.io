---
title: Adding badge icon to ionic Android app

---
#### TL;DR
You have an Ionic application, and you want to use icon badges on Android. You've found the [cordova badge plugin](https://github.com/katzer/cordova-plugin-badge), but it only works if the the application is running in the foreground/background, not if it's closed. This is the [patched cordova-push-plugin](https://github.com/silo-co/phonegap-plugin-push) that will work.

#### The long story
This is the [official cordova push plugin](https://github.com/phonegap/phonegap-plugin-push), which handles push notifications on Android and iOS. If you're using only this, you'll have badges on iOS (because it's natively supported there), but not on Android.

This is the [cordova plugin](https://github.com/katzer/cordova-plugin-badge) that enables to add badge icons to Android. 
However, it only works if the app is in running in the background, becausae it's triggered from the application.js itself, which only runs when the app is alive. 

These are two Android libraries that enable you to add badge icons to Android apps (they have nothing to do with cordova - plain Java code)

- [Badges](https://github.com/arturogutierrez/Badges)
- [ShortcutBadger](https://github.com/leolin310148/ShortcutBadger)

So currently it seems like the only way to have badge work on Android the same way the work on iOS - with a push notification - is to add one of the above Android libraries to the original cordova plugin, and run the code from one of the classes that runs when the push notification is parsed. 

The class that is being called when a push notification is parsed is the ```GCMIntentService```, so this is the one we want to patch - specifically, the ```onMessageReceived``` function. 

So the first thing we need to do is add the dependency to the ShortcutBadger class inside the cordova badge plugin. For that we need to do 2 things: 

1 - Add the dependency in the build.gradle file (if you don't have any, create one it the top directory)

```groovy
repositories {
    mavenCentral()
}
dependencies {
    compile 'me.leolin:ShortcutBadger:1.1.4@aar'
}
```

2 - Add the reference to the build.gradle file to the plugin.xml

```xml
<framework src="build.gradle" custom="true" type="gradleReference" />
```

##### Now to the code itself 
The code I've added is very simple - if a push arives with 0 on null count value, the icon badge is cleared. Otherwise, the badge icon is set to the recieved value. 

```java
private void updateBadge(Context context, String badge) {
        int count = badge == null ? 0 : Integer.parseInt(badge);
        if (count > 0) {
            ShortcutBadger.applyCount(context, count);
        } else if (count == 0) {
            ShortcutBadger.removeCount(context);
        }
    }
```

c'est tout!












