这里写中文标题
---

> * 原文链接 : [#SmallerAPK, Part 3: Removing unused resources](https://medium.com/@wkalicinski/smallerapk-part-3-removing-unused-resources-1511f9e3f761#.nw9vbskg2)
* 原文作者 : [Wojtek Kaliciński](https://medium.com/@wkalicinski)
* 译文出自 : [MiJack技术博客](http://mijack.github.io)
* 转载声明: 本译文未经允许，不得转载，有意向请联系893380824@qq.com


There’s two ways you can remove resources that are not used by your app, but get packaged into the APK because they’re in your project folder or dependencies. One method relies on the [minifier/shrinker](../smallerapk-part-2-minifying-code) that we discussed in the previous chapter. Apart from removing unused code, it can also analyze which resources are actually being used and strip those that are never included in your layouts, drawables, code, etc. To enable resource shrinking, add this line to your release build type:
build.gradle

```
android {
    ...
    buildTypes {
        release {
            minifyEnabled true
            shrinkResources true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
}
```

Just like with removing code, automatic tools might sometimes make the wrong call about which resources are actually used. You can tell the build system about resources you want to keep with the special *tools:keep* attribute, similar to how ProGuard configuration files can list classes and methods to keep. You can add it on any *<resources>* tag that’s already in your project, or create a separate file for the shrinker rules:

res/raw/keep.xml
```
<?xml version="1.0" encoding="utf-8"?>
<resources xmlns:tools="http://schemas.android.com/tools"
   tools:keep="@layout/l_used*_c,@layout/l_used_a,@layout/l_used_b*"
/>
```
You can also specify tools:discard to deliberately remove resources that were kept:
res/raw/keep.xml
```
<?xml version="1.0" encoding="utf-8"?>
<resources xmlns:tools="http://schemas.android.com/tools"
    tools:shrinkMode="safe"
    tools:discard="@layout/unused2"
/>
```

You can read more about debugging the resource shrinker by looking at log files and what switching between safe and strict modes means on the [tools site](http://tools.android.com/tech-docs/new-build-system/resource-shrinking).

Of course, it’s good practice to keep your project folder tidy and organized anyway, so if there are any old resources that you know will not be used anymore, you should just remove the files.

## Removing unused configurations with ResConfigs

Lots of libraries come with string resources translated into many languages. That’s the case with the Support Library and Google Play Services for example. You might even have translations that you started working on for your app, but you are not ready to release to markets serving those languages just yet. You can use the *resConfigs* option to restrict the configurations that will be included in the final APK

build.gradle
```
android {
    defaultConfig {
        ...
        resConfigs "en", "fr"
    }
}
```

This will remove any resources that are not meant for these two language configurations.

Please note that resConfigs has slightly other semantics when used with screen densities (you can use only one density), it’s better to use [splits](../smallerapk-part-4-multi-apk-through-abi-and-density-splits) instead. There’s also an example of density resConfig usage in [Part 5: Multi-APK through product flavors](../smallerapk-part-5-multi-apk-through-product-flavors)

## Sparse configurations in resources.arsc

The problem I’m about to describe in this section usually only hits really large apps, with hundreds or thousands of resource strings, styles or other identifiers that go into the resources.arsc file.

If you notice this file takes up an unusually large amount of space in your APK, this might be an indicator that you have too many sparse configurations. Let me explain on a simple example.

Let’s say you have 5 base strings that are defined in a default configuration folder (*values/strings.xml*). The strings themselves will be defined in a string pool, while another area of the resources file will contain a resource config consisting of pointers to those strings. For the sake of this explanation, this is how a simplified *resources.arsc* file might look like:

```
String pool: "My App", "Hello", "Exit", "Settings", "Feature"
                  Default config:
string/myapp      0x00000001
string/hello      0x00000002
string/exit       0x00000003
string/settings   0x00000004
string/feature    0x00000005
```

Now imagine that you’re adding a new feature to your app that works only on API 21+. The feature needs to show a different message when it’s used, so you decide to override a string in *values-v21/strings.xml* and recompile your app.

You would think that you’re adding just one new string value to the string pool and one string pointer for the newly created v21 config. Unfortunately, that’s not how the *resources.arsc* file format works. What you might see instead is:

```
String pool: "My App", "Hello", "Exit", "Settings", "Feature", "New feature"
                  Default config:         -v21 config:
string/myapp      0x00000001              NO_ENTRY
string/hello      0x00000002              NO_ENTRY
string/exit       0x00000003              NO_ENTRY
string/settings   0x00000004              NO_ENTRY
string/feature    0x00000005              0x00000006

                  ==========              ==========
Config size:      20 bytes                20 bytes!
```
As it turns out, each configuration (*-v21*, *-land* or even *-en-land-v21*) reserves space for pointers on every possible resource position. The actual pointers can be null, meaning that there is no value defined in this configuration.

A null pointer still takes up 4 bytes.

Like I mentioned in the beginning, the possible savings here depend mostly on how many strings (or other types of resources) you have in your app. For our example, the overhead of the -v21 config for strings is just 4 * 4 bytes, that is 16 bytes wasted in null entries (plus 4 bytes for a pointer we actually care about).

However, in a real life scenario, an app that defines 3500 strings and has a separate landscape configuration with 1 string translated into 50 languages (so it has folders such as *values-en-land, -pl-land, -de-land, -fr-land*…) will lose:

```
4 bytes * 3500 null entries * 50 languages = 700 kilobytes
```

That is **700 kilobytes that can be saved just by removing one string**, and I’ve seen apps that saved more than **2.5MB** for moving as little as 3 resources around to reduce the number of configurations.

There’s one catch — since you put a resource in a separate config in the first place, that usually means you need it there and it’s actually the proper way to do it in Android. But if you can identify that one or several resources that will save you lots of space, you might consider finding a way to get rid of them altogether.

Another way, although less elegant, is to switch between different versions of the resource in code for that one situation. For example, in your *values/strings.xml* you could have two strings: **string/my_feature** and **string/my_feature_land**, then select the correct one at runtime based on current screen orientation.

Let me know if this technique applied to your app and how you were able to reduce the size of resources.arsc!

[Part 4: Multi-APK through ABI and density splits](../smallerapk-part-4-multi-apk-through-abi-and-density-splits) is one click away.
