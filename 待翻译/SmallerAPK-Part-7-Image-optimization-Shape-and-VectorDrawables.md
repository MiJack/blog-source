这里写中文标题
---

> * 原文链接 : [#SmallerAPK, Part 7: Image optimization, Shape and VectorDrawables](https://medium.com/@wkalicinski/smallerapk-part-7-image-optimization-shape-and-vectordrawables-ed6be3dca3f#.dhcplmg8a)
* 原文作者 : [Wojtek Kaliciński](https://medium.com/@wkalicinski)
* 译文出自 : [MiJack技术博客](http://mijack.github.io)
* 转载声明: 本译文未经允许，不得转载，有意向请联系893380824@qq.com


Raster image formats like [JPEG, PNG and WebP](../smallerapk-part-6-image-optimization-zopfli-webp) are great for some image types and sometimes even necessary, but they have two major drawbacks — their size and the fact that you need to keep them in your project in many versions for different screen densities, which compounds the problem even further. But some image types, especially icons and UI elements, can be easily recreated with a more concise representation, using XML and path information.
## Shape Drawable
Available in Android from the beginning and usable regardless of platform version, this is a type of generic drawable defined in an XML file. It can only support very basic shapes, like rectangles, ovals, lines and rings, but sometimes it’s enough for simple backgrounds or decorations. Through its attributes you can also create gradients, rounded corners and outline stroke effects.

A thorough guide to shape drawables is [here](https://medium.com/r/?url=http%3A%2F%2Fdeveloper.android.com%2Fguide%2Ftopics%2Fresources%2Fdrawable-resource.html%23Shape). You can use them whenever you would use a bitmap or other types of drawables in your layouts, e.g. in *android:background* or *android:src*.
## VectorDrawables
Since API level 21 (Lollipop), Android supports a new type of drawable for representing vector graphics (perfect for icons!), called [VectorDrawable](https://medium.com/r/?url=http%3A%2F%2Fdeveloper.android.com%2Freference%2Fandroid%2Fgraphics%2Fdrawable%2FVectorDrawable.html).

VectorDrawables are density independent (one file will work for all screens), retain full quality when scaled and they are usually very small in size. **Using VectorDrawables can have a dramatic impact for slimming down your APK.**

The format is similar to SVG in that it uses the same path format, but VectorDrawables have their own XML schema. That means SVGs need to be converted before you can use them, and moreover not all SVG features are supported. You can use the [Vector Asset Studio](https://medium.com/r/?url=http%3A%2F%2Fdeveloper.android.com%2Ftools%2Fhelp%2Fvector-asset-studio.html) within Android Studio for the conversion.

Our Gradle Android plugin can also help if you’d like to keep your app backwards compatible. Add the following lines to your build to choose densities for which PNGs will be generated from *VectorDrawables* found in the *drawable/* folder:
build.gradle
```
android {
    ...
    defaultConfig {
        //if you're using Android Gradle plugin < 2.0.0
        //omit the vectorDrawables block
        vectorDrawables {
            generatedDensities = ["mdpi", "hdpi", "xhdpi"]
        }
    }
}
```
>Note: If you don’t wish to generate PNGs, just set generatedDensities to an empty list: generatedDensities = [].

You can put the vectorDrawables block inside [product flavor definitions if you’re using Multi-APK](../smallerapk-part-5-multi-apk-through-product-flavors) to only generate images for pre-lollipop builds.

## VectorDrawableCompat

AppCompat version 23.2 now includes a backwards compatible implementation of VectorDrawables. Using this approach, with minimal changes to your layouts and code, you can forgo including generated PNGs in all versions of your app (on API 7+).

Chris Banes explains [how to use VectorDrawableCompat in his post](https://medium.com/@chrisbanes/appcompat-v23-2-age-of-the-vectors-91cbafa87c88) or you can check out this and other features in the new Support Library [here](https://medium.com/r/?url=http%3A%2F%2Fandroid-developers.blogspot.co.uk%2F2016%2F02%2Fandroid-support-library-232.html).
Continue reading with [Part 8: Native libraries, open from APK](../smallerapk-part-8-native-libraries-open-from-apk).
