这里写中文标题
---

> * 原文链接 : [#SmallerAPK, Part 4: Multi-APK through ABI and density splits](https://medium.com/@wkalicinski/smallerapk-part-4-multi-apk-through-abi-and-density-splits-477083989006#.u0o6jrx5y)
* 原文作者 : [Wojtek Kaliciński](https://medium.com/@wkalicinski)
* 译文出自 : [MiJack技术博客](http://mijack.github.io)
* 转载声明: 本译文未经允许，不得转载，有意向请联系893380824@qq.com

Android is a diverse ecosystem, with devices ranging from phones to tablets and even TVs, each having its own hardware characteristics such as screen size, pixel density and CPU architecture.

Although in the [documentation](http://developer.android.com/google/play/publishing/multiple-apks.html) we encourage you to create a single package to support all these devices, sometimes the biggest space savings can come from breaking up your app into multiple APKs. If you prepare your app this way, a user on a phone with an ARM processor will not have to download native code for x86 CPUs or someone with a medium density screen won’t be storing extra-high density assets and wasting space and bandwidth.

You can find details about how Multi-APK support works, what kind of filters are supported on the Play Store and some important rules about version numbering [here](http://developer.android.com/google/play/publishing/multiple-apks.html#HowItWorks). It’s crucial to understand the theory behind Multi-APK, so that you as a developer can decide if you want to add to the complexity of your release process and if it’s something that will benefit your users.

The simplest way to add Multi-APK support to your app is through an Android Studio (or more precisely the Android Gradle plugin) configuration option called [splits](http://tools.android.com/tech-docs/new-build-system/user-guide/apk-splits). Splits is a section you can add in your *build.gradle* file that enables the creation of separate APKs for different screen densities or ABIs (i.e. CPU architectures) as part of the usual build process.

## Density splits

If you need a sample application to play around with before you start adding splits to your app, I suggest you check out [Topeka on Github](https://github.com/googlesamples/android-topeka) which already includes image resources for many densities — perfect for trying out splits!

Open the *app/build.gradle* file and add the following lines in the android section:

build.gradle

```
android {
    ...
    splits {
        density {
            enable true
            exclude 'ldpi', 'tvdpi', 'xxxhdpi'
//alternatively use the following two lines to only include:
//            reset()
//            include 'mdpi', 'hdpi', 'xhdpi', 'xxhdpi'
            compatibleScreens 'small', 'normal', 'large', 'xlarge'
        }
    }
}
```

The configuration options are pretty straightforward. First, you enable splitting by screen density, then you can exclude (or include) splits for certain configurations from being created and finally specify what screens your app is compatible with, which should usually be all four buckets — from small to extra large.

By default, a universal APK with all densities included will be created as well. It’s **vital** that you publish this APK on the Play Store with a lower version number than all other density-specific packages. This is necessary for 2 reasons:

1. The mechanism used to target specific devices is the *<compatible-screens>* section in the Manifest which is not future-proof. It requires listing every supported density explicitly and there are no catch-all buckets. When new devices come out, sometimes new screen densities are introduced (as it [happened](http://android-developers.blogspot.co.uk/2015/09/android-marshmallow-ready-for-devices.html) with the Nexus 5X and 6P) and these devices would not be able to download your app without the universal APK fallback.
2. Unfortunately, only named densities currently work with the include/exclude statements for now, so you can’t create an APK that targets 280/360/420/480/560 dpi devices. I’ve filed a bug for that [here](https://code.google.com/p/android/issues/detail?id=198393).

**Update:** I have figured out a way to fix the above mentioned problem (2) by adding missing densities to the generated AndroidManifest.xml using a Gradle script. It might not be elegant, but it does the job while support is missing from the Android Gradle plugin. In this example, 280dpi devices will get the *xhdpi* APK, 420/400/360 dpi devices will get *xxhdpi* etc.

build.gradle

```

ext.additionalDensities = ['xhdpi': ['280'], 'xxhdpi': ['420', '400', '360'], 'xxxhdpi': ['560']]
import com.android.build.OutputFile

android.applicationVariants.all { variant ->
    // assign different version code for each output
    variant.outputs.each { output ->
        if (output.getFilter(OutputFile.DENSITY) != null && project.ext.additionalDensities.containsKey(output.getFilter(OutputFile.DENSITY))) {
            output.processManifest.doFirst {
                def manifestFile = new File(project.buildDir, "intermediates" + File.separator + "manifests" + File.separator + "density" + File.separator + output.getFilter(OutputFile.DENSITY) + File.separator + variant.buildType.name + File.separator + "AndroidManifest.xml")
                def manifestText = manifestFile.text
                for (String density : project.ext.additionalDensities.get(output.getFilter(OutputFile.DENSITY))) {
                    manifestText = manifestText.replaceAll("</compatible-screens>", "<screen android:screenSize=\"small\" android:screenDensity=\"${density}\" />\n" +
                            "<screen android:screenSize=\"large\" android:screenDensity=\"${density}\" />\n" +
                            "<screen android:screenSize=\"xlarge\" android:screenDensity=\"${density}\" />\n" +
                            "<screen android:screenSize=\"normal\" android:screenDensity=\"${density}\" />\n </compatible-screens>")
                }
                manifestFile.text = manifestText
            }
        }
    }
}
```

You can inspect the Manifest entries that are created by the build system in intermediate files located in your project’s folder:

```
/app/build/intermediates/manifests/density/hdpi/debug/AndroidManifest.xml

```
They’re partial Manifest files and contain only the *<compatible-screens>* part that gets merged with your main Manifest file:

```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android" package="">
 <compatible-screens>
 <screen android:screenSize="small" android:screenDensity="hdpi" />
 <screen android:screenSize="normal" android:screenDensity="hdpi" />
 <screen android:screenSize="large" android:screenDensity="hdpi" />
 <screen android:screenSize="xlarge" android:screenDensity="hdpi" />
 </compatible-screens>
</manifest>
```

This will be used by the Play Store to filter out devices that are not in the defined density bucket and to deliver the correct APK to a device which is requesting to install your app.
> **Note:** if there are any images that you wish to always include in all available densities so that they are not stripped away in the split process, you should put them in the mipmap resource folder. This is normally used for the app icon, because some launchers may use an icon from a higher density bucket.

Savings for the Topeka app when splits are enabled are substantial. Here are the APK sizes:

```
Original (universal) APK:   3.5 MB
MDPI devices:               1.1 MB (2.4 MB savings)
HDPI devices:               1.2 MB (2.3 MB savings)
XHDPI devices:              1.3 MB (2.2 MB savings)
```
Of course this will vary from app to app depending on the size of image resources.
## ABI splits
Similar to density based splits, you can also configure ABI splits for devices with different CPU architectures:
```
splits {
    abi {
        enable true
        reset()
        include 'x86', 'armeabi-v7a', 'mips'
        universalApk false
    }
}
```
You can control if you wish to build a universal APK but this shouldn’t normally be needed. Remember to give the x86_64 and x86 higher version numbers than ARM, as many x86 devices can run ARM code through an emulation layer, although with lower performance. By setting up the version numbering correctly you can make sure they get the optimal library.

**Update:** If you don’t wish to manage too many APKs (one for every ABI, potentially multiplied by the number of density splits), there is a way to leverage the universal APK. If you have any analytics data for your app, look at how many users are on which ABIs. Target the most popular ones (usually ARM and maybe x86) with a split APK and serve a universal APK to everyone else. That way you can make sure that > 90% of users get the optimal version, while others will still be able to download and use your app.

Of course, it would be best to serve every user the optimal version, but as with everything — you need to decide how much time you want to spend as a developer maintaining releases and how many users will benefit from the optimization.
## Setting version codes
Here’s a code snippet that will help you with setting different version codes for the output APKs so that you can upload them to Play Store as a single listing.

```
// map for the version codes
ext.versionCodes = ['mdpi':1, 'hdpi':2, 'xhdpi':3].withDefault {0}
import com.android.build.OutputFile
android.applicationVariants.all { variant ->
// assign different version code for each output
    variant.outputs.each { output ->
        output.versionCodeOverride = project.ext.versionCodes.get(output.getFilter(OutputFile.DENSITY)) * 1000000 + android.defaultConfig.versionCode
    }
}
```
This example uses 3 density splits and the version numbering is calculated as `density versionCode * 1000000 + app’s versionCode`. So the version numbers will go like this:
```
Universal:       1,      2,       3 ...
MDPI:      1000001,1000002, 1000003 ...
HDPI:      2000001,2000002, 2000003 ...
XHDPI:     3000001,3000002, 3000003 ...
```
That way, only devices who don’t fit any density-specific bucket will get the Universal APK. You can use similar approach to set version codes for ABI splits, by changing *OutputFile.DENSITY* to *OutputFile.ABI* and using ABI names like ‘x86’ and ‘armeabi-v7a’ instead of ‘mdpi’, ‘hdpi’ and ‘xhdpi’ in the code.

If you need more flexibility for your Multi-APK setup, check out [Part 5: Multi-APK through product flavors](../smallerapk-part-5-multi-apk-through-product-flavors)
