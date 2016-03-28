这里写中文标题
---

> * 原文链接 : [#SmallerAPK, Part 2: Minifying code](https://medium.com/google-developers/smallerapk-part-2-minifying-code-554560d2ed40#.9t7eke4bh)
* 原文作者 : [Wojtek Kaliciński](https://medium.com/@wkalicinski)
* 译文出自 : [MiJack技术博客](http://mijack.github.io)
* 转载声明: 本译文未经允许，不得转载，有意向请联系893380824@qq.com

In this chapter you will find advice that applies to almost any app that’s out there. It’s all about keeping your codebase clean, your dependencies in check and giving you the tools to help with those tasks.

## Dex code minification

The first thing you want to do is enable the built-in minifier. It will try to strip any unused classes and class members, as well as rename any identifiers using shorter names. Both operations make the resulting code smaller, but the latter can make debugging cumbersome, so I suggest you only enable minification for your release build type:

build.gradle

```
android {
    ...
    buildTypes {
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
```
The second line provides configuration files for the minifier in [ProGuard rules](http://proguard.sourceforge.net/manual/usage.html) format. The first configuration file ([Sdk/tools/proguard/proguard-android.txt](https://android.googlesource.com/platform/sdk/+/master/files/proguard-android.txt)) is included with the SDK and contains some sane defaults for every Android project. Looking at those rules can help you get familiar with the Proguard configuration syntax. For example, this keeps (in other words prevents from being removed or renamed) any setter or getter methods from View subclasses:
```
# keep setters in Views so that animations can still work.
# see http://proguard.sourceforge.net/manual/examples.html#beans
-keepclassmembers public class * extends android.view.View {
    void set*(***);
    *** get*();
}
```

With any luck, your app will work with the default configuration in place. During the build process, a tool called AAPT also generates the necessary rules for retaining all your Activities and other components that are mentioned in the Manifest, as well as any Views used in XML layouts. Library dependencies should provide their minifier configuration through consumer ProGuard files, though sometimes they only provide the necessary ProGuard rules on their website or manual. In that case, you will be required to copy them to your *app/proguard-rules.pro* file.

It will often be the case, unfortunately, that after enabling minification your app will either not compile or break during runtime, usually by throwing *ClassNotFoundException* for classes removed by the minifier. To fix this, you will need to create a *proguard-rules.pro* file in your *app/* folder and provide rules needed to get rid of compile-time warnings (watch the message log for info). You also have to make sure that you keep any classes and members that are used during runtime, but are getting stripped by the minifier. These are usually parts of your code that are accessed through reflection.

One specific case is when custom attributes in XML layouts take a class name as a String, like setting a *RecyclerView’s layoutManager* for example:
```
<android.support.v7.widget.RecyclerView
    app:layoutManager=”android.support.v7.widget.GridLayoutManager”
    ...
/>
```

In this case, AAPT will not be able to figure out the class usage and generate the necessary ProGuard rule. To apply a fix, you should put the following line in your ProGuard configuration to keep the *GridLayoutManager* class and any of its public and protected methods from being removed or renamed:
```
-keep public class android.support.v7.widget.GridLayoutManager {
    public protected *;
}
```
To check if the configuration rules you write are having the desired effect, you can inspect the *classes.dex* files in the resulting APK with a tool like ClassyShark. It’s also important that you **test your app thoroughly**! The fact that the app opens without crashing is not an indication of a correct minifier config. You **must** test every screen and user flow in your app for crashes. (may I suggest checking out the [Android Testing Support Library and Espresso](https://google.github.io/android-testing-support-library/) for that? :)

## Uploading ProGuard mappings to Play
Analysing exceptions thrown by obfuscated code running on your users’ devices used to be a bit inconvenient until recently. Normally you’d have to copy the stacktrace from Play Developer Console and use a tool on your computer together with a ProGuard mappings file generated at compile time to decode the original class and methods names.

![](https://cdn-images-1.medium.com/max/800/1*3h-9CmwDuPv-sURh4Bx85w.png)

Play Developer Console now has the option to [upload the mappings file](https://support.google.com/googleplay/android-developer/answer/6295281) together with your APK and will show deobfuscated stacktraces right there in the Crashes and ANRs panel. Remember, the mapping file you use has to be from the exact same compilation run as your release APK.
> **Note:** you will find the mappings.txt file in your project folder under this path: <module>/build/outputs/mapping/mapping.txt

## ProGuard configuration for libraries
If you’re building an AAR library to be used in other projects that needs ProGuard rules to work, you should use the consumerProguardFiles option to package a ProGuard configuration file with the AAR. This way, anyone using your library will not have to worry about adding rules manually when enabling the minifier. Also make sure to provide information on any other requirements when compiling a project with your library in the manual.

build.gradle
```
android {
    ...
    defaultConfig {
        consumerProguardFiles “proguard-rules.txt”
    }
}
```

## Using granular dependencies for Google Play Services
![](https://cdn-images-1.medium.com/max/800/0*nmsWPigHxVnyPfHO.)

When using the Google Play Services library in your project, remember to switch to granular dependencies. This means you won’t be pulling in the whole client library if you’re just using a couple of features like Ads, Maps or GCM. There’s a table with all Gradle dependency strings that you can use on [developers.google.com](https://developers.google.com/android/guides/setup). And by the way, ProGuard consumer rules are included with Play services so they should just work if you enable the code minifier for your build.

## Tracking down dependencies

The great thing about developing for Android is that if you have a problem, there is probably someone who’s already solved it. As your project grows, you will usually pull in more and more external dependencies to speed up development time. The most common ones are probably the Android Support Library for backwards compatibility, Play Services, an image loading library, an HTTP client and various other SDKs…
Developers often ask me: which image library should I use? How many dependencies is too much for my project? There are no sure answers to those questions. If you really need to use a library because it solves your problem (and you understand the drawbacks) then go for it. It’s important to have the right tools that help you decide what impact it’s going to have on the size of your project so you can make an informed decision.
## Transitive library dependencies
When you think you’re “just” adding a small helper lib and suddenly your Dex size explodes and your method count goes through the roof, that might be because of transitive dependencies that you don’t normally see in your build.gradle file. Fortunately, there are tools that can help:
```
$ ./gradlew app:dependencies
...
compile — Classpath for compiling the main sources.
+ — — com.android.support:appcompat-v7:23.1.1
| \ — — com.android.support:support-v4:23.1.1
| \ — — com.android.support:support-annotations:23.1.1
+ — — com.android.support:cardview-v7:23.1.1
+ — — com.android.support:design:23.1.1
| + — — com.android.support:appcompat-v7:23.1.1 (*)
| + — — com.android.support:recyclerview-v7:23.1.1
| | + — — com.android.support:support-annotations:23.1.1
| | \ — — com.android.support:support-v4:23.1.1 (*)
| \ — — com.android.support:support-v4:23.1.1 (*)
+ — — com.android.support:recyclerview-v7:23.1.1 (*)
\ — — com.android.support.test.espresso:espresso-idling-resource:2.2.1
```
The *<modulename>:dependencies* command gives you an overview of every library in your project and its dependency tree. An asterisk (*) next to the version number tells you that this particular dependency has been mentioned in the output before and so it would have been included in your project anyway, unless you remove all other instances of it.
> **Note:** *if you are using product flavors and you only need a library for some variants of your app (for example an Ads SDK only for the free version), you can specify dependencies prefixed with your flavor name, e.g.*

> *dependencies { freeCompile ‘…’ }.*

## Inspecting Dex files with ClassyShark
Sometimes, in order to shield the developer from version clashes, a library includes its dependencies directly within its code but with a changed package name, essentially shadowing the original one. It also means that you won’t see this kind of dependency in the Gradle dependency tree.

If you want to get a better view on the exact classes and package names that are packaged in your APK, you can use [ClassyShark](https://github.com/google/android-classyshark) to inspect your Dex files. It’s also great for testing out your ProGuard rules to see exactly what effect they’re having on the final APK.

![](https://cdn-images-1.medium.com/max/800/1*icrlDNJPYDaqb0QuUDV_fw.png)
> **Note:**  *As a bonus, ClassyShark will show method counts for packages, which might help you identify the biggest multidex offenders.*

Read on for [Part 3: Removing unused resources](../smallerapk-part-3-removing-unused-resources)
