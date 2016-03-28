这里写中文标题
---

> * 原文链接 : [#SmallerAPK, Part 8: Native libraries, open from APK](https://medium.com/@wkalicinski/smallerapk-part-8-native-libraries-open-from-apk-fc22713861ff#.z22z9v1ou)
* 原文作者 : [Wojtek Kaliciński](https://medium.com/@wkalicinski)
* 译文出自 : [MiJack技术博客](http://mijack.github.io)
* 转载声明: 本译文未经允许，不得转载，有意向请联系893380824@qq.com


Adding even a single native library can contribute quite significantly to your app’s size. If your app uses any native libraries (i.e. there are .so files in your APK), this article is for you.

There are various techniques and flags for C/C++ compilation that can make your .so files smaller, but I will not be discussing native builds in this article as it’s a broad and very advanced topic, with each particular case requiring special treatment. Besides, it might happen that you get compiled .so files only and without access to the source code there’s not much you can do about their size. Instead, I will show you this one weird trick that cuts disk space usage in half! No, really… read on :)

In Part 1 of this series, I [mentioned](../smallerapk-part-1-anatomy-of-an-apk) that the biggest waste of space from having native libraries comes from the fact that compressed .so files are copied out from the APK at install time into a user’s */data* partition. The originals are never deleted from the APK (because we can’t delete files from signed APKs, remember?). Thus, they take up twice the amount of space on the device.

Starting with Android 6.0 (Marshmallow), there is a new flag that you can set on your *<application>*:
AndroidManifest.xml

```
<application
   android:extractNativeLibs=”false”
   ...
>
```
This will essentially prevent the system from creating a second copy of the .so files and fix the System.loadLibrary call so it’s able to find and open native libs straight from the APK, no code changes on your part required.

There are some important preconditions for that to work though and that’s where things get more complicated:

1. The .so files inside the APK **cannot be compressed** — they must be stored.
2. The .so files **must be page aligned** using *zipalign -p 4*

Unfortunately, [there is no way currently](https://medium.com/r/?url=https%3A%2F%2Fcode.google.com%2Fp%2Fandroid%2Fissues%2Fdetail%3Fid%3D172630) to have this done automatically as part of the build in Android Studio or Gradle. You will have to do it manually or write custom scripts, but regardless of how you wish to proceed, the recipe for success is more or less as follows:

1. Delete the .so files from the APK archive

2. Add the .so files back to the archive, with the ZIP compression level set to 0 (zero, sometimes also named STORE).
Remember that you have to add the libraries on the same paths inside the archive as they were before.
3. Run *zipalign -p 4 <APK file>*

If you’re on Linux, you can use a script similar to this one:

fix_native_libs.sh
```
FILENAME=$(basename "$1" .apk)
OUT_FILENAME=${FILENAME}-noextract.apk
OUT_FILENAME2=${FILENAME}-noextract-aligned.apk
unzip "$1" "lib/*.so"
cp "$1" $OUT_FILENAME
zip -n ".so" -r $OUT_FILENAME lib
zipalign -p 4 $OUT_FILENAME $OUT_FILENAME2
rm $OUT_FILENAME
```
Let’s look at what effect on file and download sizes keeping native libraries uncompressed has:
### Initial download size
Should be roughly the same. How is that possible? The Play Store actually uses its own compression when sending APK files to users’ devices. Depending on the compressibility of your particular library, the download size might be **roughly the same** or just slightly larger.
## Update size
Should be **smaller**. Play Store uses a mechanism called “delta updates” to deliver new versions of apps to existing users. It calculates the difference between the new APK and the APK that’s installed on a device and only sends the binary difference needed to reconstruct the final APK on the device.

Having the native libraries uncompressed actually helps the algorithms used by Play calculate a more optimal (i.e. smaller) delta. These delta patch files will additionally be compressed for delivery.
## On-device size
Total size on-device will be **smaller on Marshmallow and above**. However, older Android versions will not recognize the android:extractNativeLibs flag and always copy out the .so files. The amount of additional space wasted because of using this technique is the difference between a compressed and uncompressed .so file.
