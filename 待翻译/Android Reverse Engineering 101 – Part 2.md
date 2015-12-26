Android Reverse Engineering 101 – Part 2

In the first article of this series I have explained the format of APK and AAR bundles.

As already mentioned, applications available on Google Play Store (or more generally any application installed on a device) are packaged in a file with the APK extension. In this second part, I will show you how you can get valuable information from an APK file using aapt.

AAPT

If you have the Android SDK installed, you already have aapt: in fact, the Android Asset Packaging Tool is part of the Build Tools and you can find it here for example:

ANDROID_SDK_HOME/build-tools/23.0.2

Please notice that you will find a separate folder for each version of the Build Tools: when you install a new version from the Android SDK Manager, the existing version is not overwritten, but a separate folder is created. This gives you the possibility to switch between different versions of the tools.

From Android Studio, you have to remember to set the version of the Build Tools you want to use in the  build.gradle script of your module.

This tool is part of the Android Build System and it allows you to view, create, and update Zip-compatible archives (zip, jar, apk). It can also compile resources into binary assets.

Details about how the Android Build System works are beyond this article, but aapt is mainly used in the process to:

Generate the R.java file, so basically processing the resources of your application.
Create the initial APK file, assembling the Android manifest, resources, assets, …
Add to the APK file the compiled classes, which have been already converted to the dex format by the dx tool.
A detailed look at the build process is provided by the official documentation here.

Anyway, aapt can be also used to extract important data from an APK file.

If you would like to try the same commands, you just need to get one APK file as I have already explained in the first article.

PACKAGE CONTENT

To get the content of an APK file listed, simply issue the following command:

aapt list FILENAME.apk

Adding the  -v option, you can obtain more information, such as file size, creation date and time, CRC-32, …

aapt list -v
aapt list -v
PACKAGE DETAILS

With the  dump command you can get more fine grained data.

The  badging option prints details such as package name, version name, version code, permissions, supported screens, launcher  Activity , application name and icon file, …

aapt dump badging FILENAME.apk

Screen Shot 2015-11-14 at 14.26.44
aapt dump badging
aapt dump badging
aapt dump badging
 

The  permissions option prints the permissions required by the application (referred with the package name defined in the Android manifest file). Please notice that only explicit permissions declared in the manifest will be listed by this command.  android.permission.WRITE_EXTERNAL_STORAGE , for example, implicitly requires  android.permission.READ_EXTERNAL_STORAGE too, but this is not listed as you can see in the screenshot here below.  aapt dump badging command will give you also this information though.

aapt dump permissions FILENAME.apk

aapt dump permissions
aapt dump permissions
 

The  configurations options prints the configurations in the APK file:

aapt dump configurations FILENAME.apk

aapt dump configurations
aapt dump configurations
 

The  resources command prints the resource table from the APK file. Thus, what you get is the list of all resources used and referenced by the the application: attributes, strings, dimens, layouts, styles, menus, drawables, …

You also get resources from the dependencies of the application: for example, if you declare the  appcompat-v7 library as a dependency, its resources are listed as well.

aapt dump resources FILENAME.apk

aapt dump resources
aapt dump resources
 

The last command which is worth a mention is  xmltree: it prints the compiled XMLs in the given asset. As I mentioned in the first article, XML files packaged in an APK file are in binary format, so you cannot open and read them with an editor or a viewer. Anyway, using this command, you get a more readable version of the file.

aapt dump xmltree FILENAME.apk RESOURCE.xml

aapt dump xmltree
aapt dump xmltree
These are the most important commands and options provided by aapt, but I encourage you to take a look at it and at all the options it provides.

As you have seen, with few simple commands it’s possible to get several details about an application, but it’s just a read-only process: in fact, you can’t actually change anything about the APK file.

In this article I was also supposed to speak about dex2jar and how you can use it to decompile Android applications, but I think there are already enough information to digest, so this is postponed to the next post.