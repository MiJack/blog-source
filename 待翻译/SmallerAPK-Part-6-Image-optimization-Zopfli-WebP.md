这里写中文标题
---

> * 原文链接 : [#SmallerAPK, Part 6: Image optimization, Zopfli & WebP](https://medium.com/@wkalicinski/smallerapk-part-6-image-optimization-zopfli-webp-4c462955647d#.oyffxkv3i)
* 原文作者 : [Wojtek Kaliciński](https://medium.com/@wkalicinski)
* 译文出自 : [MiJack技术博客](http://mijack.github.io)
* 转载声明: 本译文未经允许，不得转载，有意向请联系893380824@qq.com


Every app is bound to include image resources at some point, even if it’s just a few icons. There’s two ways images can contribute to application size. Firstly, we package them in the APK as resources and assets. Secondly, apps download images from servers at runtime and usually keep a disk cache of those images for quicker access. Remember that any optimizations that I cover in this chapter can also be applied on the server side, saving you disk space on the device as well as network bandwidth

First, a some basics. The two most popular compression formats you will use for images are JPEG and PNG. A quick reminder when to use which:

**JPEG** — photographs, backgrounds. No transparency support. Size and quality varies greatly with chosen compression level, so if you’re sticking with JPEG try and experiment with it to lower file size.

**PNG** — logos, graphs, icons, line art; basically anything with sharp edges and contrasts. Supports transparency. Images with large areas of contiguous color compress well, gradients and photographic imagery tend to produce bigger files.
## WebP

There’s another format that you might want to consider for your Android apps and that’s [WebP](https://medium.com/r/?url=https%3A%2F%2Fdevelopers.google.com%2Fspeed%2Fwebp%2F) (pronounced “weppy”). It tends to produce smaller (on average 30%) file sizes and is suitable for replacing both JPEG and PNG in your app, as it handles all kinds of image data.

There are some downsides of course. Lossy WebP support (suitable for replacing most JPEGs and some PNGs) is [guaranteed on Android 4.0+](https://medium.com/r/?url=http%3A%2F%2Fdeveloper.android.com%2Fguide%2Fappendix%2Fmedia-formats.html%23core) devices. Newer WebP features (transparency, lossless, suitable for PNGs) are [supported since Android 4.2.1+](https://medium.com/r/?url=http%3A%2F%2Fdeveloper.android.com%2Fguide%2Fappendix%2Fmedia-formats.html%23core).
>**Note:** WebP images also take a little more CPU power to decompress at runtime, which might mean slightly longer load times.
>**Note 2:** You cannot use WebP for app launcher icons.

## Zopfli-compressed PNGs

If you’re not prepared to adopt WebP just yet, there’s one more avenue you might want to explore when it comes to optimizing PNGs. The cruncher that is part of the Android build system already does some common optimizations like discarding metadata from images and converting color palettes to 8-bit when it can, but we can do better if we pre-optimize our PNGs with Zopfli.

Zopfli is a compression algorithm created by engineers at Google. It’s compatible with the popular DEFLATE specification used for compressing ZIP files and… you guessed it, the image data stream inside PNGs!

Zopfli compressed data can be read by a standard ZIP (or PNG) reader, so no changes will be required in code. You can use one of the several existing tools (AdvPNG, [ZopfliPNG](https://medium.com/r/?url=https%3A%2F%2Fgithub.com%2Fgoogle%2Fzopfli)) that will help you *zopflify* your images before building (you only need to do it once for each image file).

Unfortunately there’s no automatic support for this optimization from Android tools at this time. Moreover, you will have to disable the built-in image cruncher by adding these lines to your build.gradle:

build.gradle

```
android {
    ...
    aaptOptions {
        cruncherEnabled = false
    }
}
```
This is needed because the AAPT tool doesn’t check if the file it produces is smaller than the source file, which might undo some of the space savings from zopfli optimized images.
>Note: The cruncher will still process 9-patch images, as they require special handling at build time

Slim your icons down to a fraction of the [Part 7: Image optimization, Shape and VectorDrawables](../smallerapk-part-7-image-optimization-shape-and-vectordrawables)
