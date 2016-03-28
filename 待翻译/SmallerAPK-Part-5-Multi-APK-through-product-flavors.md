这里写中文标题
---

> * 原文链接 : [#SmallerAPK, Part 5: Multi-APK through product flavors](https://medium.com/@wkalicinski/smallerapk-part-5-multi-apk-through-product-flavors-e069759f19cd#.iklyduonj)
* 原文作者 : [Wojtek Kaliciński](https://medium.com/@wkalicinski)
* 译文出自 : [MiJack技术博客](http://mijack.github.io)
* 转载声明: 本译文未经允许，不得转载，有意向请联系893380824@qq.com


In the [previous article](../smallerapk-part-4-multi-apk-through-abi-and-density-splits), I explained how to create ABI and density based multi-APK by using the splits mechanism in Android Studio. There is another, more flexible way of creating a multi-APK configuration for your app and it’s through the use of product flavors. First, a quick refresher from the [documentation](https://medium.com/r/?url=http%3A%2F%2Fdeveloper.android.com%2Ftools%2Fbuilding%2Fconfiguring-gradle.html%23workBuildVariants):

> The build system uses product flavors to create different product versions of your app. Each product version of your app can have different features or device requirements.

That’s seems exactly like something we need for multi-APK! By making small changes to the Manifest and resources for different flavors, you can essentially get many versions (we call them build variants) of your app built from a single project.

It’s actually possible to recreate splits using flavors. We could create a flavor for each density bucket and use the [resConfigs](https://medium.com/r/?url=http%3A%2F%2Fgoogle.github.io%2Fandroid-gradle-dsl%2Fcurrent%2Fcom.android.build.gradle.internal.dsl.ProductFlavor.html%23com.android.build.gradle.internal.dsl.ProductFlavor%3AresConfigs%28java.lang.String%5B%5D%29) option to specify which drawables should be retained for each flavor.
> **Note:** resConfigs behavior has changed from Build Tools v21 onwards. You can only specify **one** density bucket and the AAPT tool will automatically figure out which drawables need to go into the APK. It is an error to specify more than 1 density bucket for a variant.

Build.gradle

```
android {
    ...
    productFlavors {
        xhdpi {
            resConfigs "xhdpi"
            versionCode 300001
        }
        hdpi {
            resConfigs "hdpi"
            versionCode 200001
        }
        mdpi {
            resConfigs "mdpi"
            versionCode 100001
        }
        anydpi {
            versionCode 1
        }
    }
}
```

Then, for each flavor create a new *AndroidManifest.xml* under *app/src/[flavorname]/AndroidManifest.xml* and fill it in with the correct screen filtering directives. Because we’re doing this manually, we can overcome the lack of support for numeric density buckets in *splits*. You might choose to combine them with the next higher bucket for example, like in this snippet (*xhdpi* is the name for 320 dpi screens):

app/src/xhdpi/AndroidManifest.xml
```
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android">
<compatible-screens>
<screen android:screenSize="small" android:screenDensity="xhdpi” />
<screen android:screenSize="normal" android:screenDensity="xhdpi" />
<screen android:screenSize="large" android:screenDensity="xhdpi" />
<screen android:screenSize="xlarge" android:screenDensity="xhdpi" />
<screen android:screenSize="small" android:screenDensity="280" />
<screen android:screenSize="normal" android:screenDensity="280" />
<screen android:screenSize="large" android:screenDensity="280" />
<screen android:screenSize="xlarge" android:screenDensity="280" /></compatible-screens>
</manifest>
```
Remember to also build and publish a universal (*anydpi* flavor in the example above) APK with no extra Manifest entries and the lowest *versionCode* to support future devices.

## MinSdkVersion-based multi-APK

Many optimizations that I will talk about in the upcoming chapters of the #smallerAPK articles depend on new compression formats or vector images support that were only introduced in more recent Android versions. If we want to take advantage of those while still retaining compatibility with older devices, normally we’d have to include the same resources multiple times.

By using flavors, we can create multi-APK configurations that were not possible with splits. Let’s consider creating a separate version of the app targeting only more recent Android versions and one supporting older devices.

Build.gradle

```
android {
    productFlavors {
        prelollipop {
            versionCode 1
        }
        lollipop {
            minSdkVersion 21
            versionCode 2
        }
    }
}
```
Now, by placing resources in *app/src/prelollipop/res/* or *app/src/lollipop/res* you can control which files will be packaged with each release, so for example you can include your [VectorDrawables](../smallerapk-part-7-image-optimization-shape-and-vectordrawables) only for Android 5.0+ builds and PNG/[WebP](../smallerapk-part-6-image-optimization-zopfli-webp) for pre-Lollipop builds.

If you need product flavors along two or more dimensions, like screen density and minSdkVersion, this is how you would set it up:

Build.gradle

```
android {
    ...
    flavorDimensions “density”, “version”
    productFlavors {
        xhdpi {
            dimension "density"
            resConfigs "xhdpi"
            versionCode 4
        }
        //other densities here...
        anydpi {
            dimension "density"
            versionCode 1
        }
        prelollipop {
            dimension "version"
            versionCode 1
        }
        lollipop {
            dimension "version"
            minSdkVersion 21
            versionCode 2
        }
    }
}
```
This makes it a bit more complicated to set the correct version number, as we need to compute it for each combined flavor. Always remember that the highest versioned APK that can be installed on the device (i.e. matches minSdkVersion and other filters) is always downloaded from the Play Store. You can find a handy script that you can adapt for your needs [here](https://medium.com/r/?url=http%3A%2F%2Ftools.android.com%2Ftech-docs%2Fnew-build-system%2Ftips).
And now, for something completely different… [Part 6: Image optimization, Zopfli & WebP](../smallerapk-part-6-image-optimization-zopfli-webp)
