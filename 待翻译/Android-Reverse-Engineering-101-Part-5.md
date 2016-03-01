# Android Reverse Engineering 101 – Part 5

In our introduction journey in the Android reverse engineering world, so far we’ve seen [what is an APK and its format](http://www.fasteque.com/android-reverse-engineering-101-part-1/), how we can extract useful information about an application using [aapt](http://www.fasteque.com/android-reverse-engineering-101-part-2/), which is provided by the Android SDK, how we can [convert the DEX](http://www.fasteque.com/android-reverse-engineering-101-part-3/) bytecode to a more readable and easily editable format and how we can actually [decompile and modify an Android application](http://www.fasteque.com/android-reverse-engineering-101-part-4/) (source code and resource).

In this last part of the series, we’ll have a look at [**Androguard**](https://github.com/androguard/androguard), a full python tool to play with Android files.

According to the official website, **Androguard** let us play with:

>* DEX, ODEX
>* APK
>* Android’s binary xml
>* Android resources
>* Disassemble DEX/ODEX bytecodes
>* Decompiler for DEX/ODEX files

The complete list of features is quite impressive and even though on GitHub is not fully reported, it can be found [here](https://code.google.com/p/androguard/#Features).



**Androguard** is available for Linux, OS X and Windows.

## INSTALLATION

The installation procedure is described in great details in [the old project home](https://code.google.com/p/androguard/wiki/Installation), while on the new GitHub page the documentation doesn’t provide any updated information.

What is important is to have Python installed on our machine (`2.6` or later versions), and this let us decompile/disassemble APK files only. For more advanced features it’s necessary to install more modules: on the installation page we can find the complete list with a short explanation about why each module is required.

It’s just important to remember to run the following command from the main **Androguard** folder:

`sudo python setup.py install`

After the installation, we can directly run the tools from the main directory of **Androguard**.


Specially if the operating system of the machine is Windows, there’s also the possibility to run **Androguard** (actually many other reverse engineering tools) from **ARE** – Android Reverse Engineering virtual machine: download and installation procedures can be found at the official [wiki](https://redmine.honeynet.org/projects/are/wiki) page. We just need to rely on [VirtualBox](https://www.virtualbox.org/) to run the image.

All the examples of the article will be run using this **Androguard** version: `2.0`. It can be downloaded directly [here](https://github.com/androguard/androguard/releases/tag/v2.0), otherwise there’s also the chance to pick a specific version from the [stable releases page](https://github.com/androguard/androguard/releases).

The first thing to do, is to check if our installation is properly working, so let’s move to the main **Androguard** folder and type the following command:

`androlyze.py -s`
  ![](http://i2.wp.com/www.fasteque.com/wp-content/uploads/2015/12/Screen-Shot-2015-12-04-at-18.14.40.png)

We should get an output like this:

The interactive console (with commands history support) is ready to accept the first inputs.

It’s now necessary to set the APK to analyze and the decompiler type. In fact, 3 different decompilers are supported: **DAD**, **DED** and **dex2jar** + **jed**. Specific details are available [here](https://code.google.com/p/androguard/wiki/Decompiler) and for the following examples, we’ll use DAD because is the default decompiler, so nothing in specific must be done at installation time to enable it.

`a,d,dx = AnalyzeAPK("FILENAME.apk", decompiler="dad")``

![](http://i2.wp.com/www.fasteque.com/wp-content/uploads/2015/12/Screen-Shot-2015-12-04-at-18.30.51.png?resize=1024%2C118)
It could take some time depending on the specific APK being decompiled: when the procedure is done, we’ll be able to run other commands from the prompt.

We can now type `a.` and then press TAB key the get the full list of APK related commands directly from the prompt.


## APK files

It’s possible to list all the files contained in the APK package (which is essentially a ZIP file):

`a.get_files()`

![](http://i1.wp.com/www.fasteque.com/wp-content/uploads/2015/12/Screen-Shot-2015-12-05-at-14.52.30.png?resize=1024%2C587)
Application Activities

With the following command, we can extract the list of Activities in the application:

`a.get_activities()`

## Application permissions

With the following command, we can extract the list of permissions requested in the manifest file by the application:

`a.get_permissions()`

![](http://i2.wp.com/www.fasteque.com/wp-content/uploads/2015/12/Screen-Shot-2015-12-05-at-14.47.29.png)

It’s even possible to get a detailed description for each permission:

`a.get_requested_aosp_permissions_details()`
![](http://i1.wp.com/www.fasteque.com/wp-content/uploads/2015/12/Screen-Shot-2015-12-05-at-15.09.31.png)


An important feature to highlight about **Androguard**, is the possibility to know which part of the application code is requesting permissions to be declared in the manifest.

That’s very important because it can help us to:

* know if we’re missing any permission declaration in the manifest
* know if we’re asking for more permissions than required in the manifest
* review the pieces of code where permissions are required and double check if we must add a missing annotation in case we’re using them.

`show_Permissions(dx)`
![](http://i1.wp.com/www.fasteque.com/wp-content/uploads/2015/12/Screen-Shot-2015-12-05-at-17.09.55.png)

## Application Content providers

With the following command, we can extract the list of Content providers declared by the application (if any):

`a.get_providers()`
![](http://i1.wp.com/www.fasteque.com/wp-content/uploads/2015/12/Screen-Shot-2015-12-05-at-14.42.07.png)

## Application signature

It’s also possible to know where the signature of the APK is stored and the signature itself:

`a.get_signature_name()`

`a.get_signature()`


[Here](http://doc.androguard.re/html/index.html) is available the documentation with the full list of commands that can be used: it’s for version `1.9` of the tool, but it’s a good reference anyway.


## Code dumping

Using the **androdd** tool, we can get the source code all the classes packaged in the APK (application code + dependencies): in this case, rather than getting `.class` files as we do using [dex2jar](http://www.fasteque.com/android-reverse-engineering-101-part-3/), in output we get Java classes.

`androdd.py -i FILENAME.apk -o OUTPUT_DIR`
![](http://i2.wp.com/www.fasteque.com/wp-content/uploads/2015/12/Screen-Shot-2015-12-05-at-15.44.22.png?resize=1024%2C430)

In the output directory, Java classes are organized by package name as expected.

Of course, what we get is not exactly the same code as the original one implemented by the developer, but it’s anyway readable and very easy to understand. Please bear in mind that if the original application code has been obfuscated, methods and classes names could have been renamed and the tool is not able to restore the original names.

Here below an example of one Fragment class: the original source code is available [here](https://github.com/fasteque/rgb-tool/blob/master/android-rgb-tool/src/main/java/com/fastebro/androidrgbtool/fragments/SelectPictureDialogFragment.java).

![](http://i0.wp.com/www.fasteque.com/wp-content/uploads/2015/12/Screen-Shot-2015-12-05-at-15.52.47.png?resize=1024%2C390)
As we already seen during this series, don’t expect to see resource names, such as strings, layouts, arrays, colors, but what we get is the integer assigned to a specific resource at build time and stored in the `R.java` file. Here it’s represented as a decimal value, while in the original class file is in hexadecimal.

## XML manifest

Using the next tool, we can extract the manifest file, which is stored in binary format inside the APK file, in plain XML, so easily readable and viewable.

`androaxml.py -i FILENAME.apk -o OUTPUT.xml`

If we open the output XML file, we can see the complete manifest file of the APK, nearly identical to the [original one](https://github.com/fasteque/rgb-tool/blob/master/android-rgb-tool/src/main/AndroidManifest.xml).

![](http://i1.wp.com/www.fasteque.com/wp-content/uploads/2015/12/Screen-Shot-2015-12-05-at-16.10.44.png)

## Comparing two APKs

One important tool provided by **Androguard** is **androsim**, which essentially compares two applications files. That’s an important feature because we can compare the APK of the original application against one which could be potentially altered to add malware. Of also simply to see the different between two different releases of the same application.

`danielealtomare$ androsim.py -i FILENAME_1.apk FILENAME_2.apk -c ZLIB -n`
That’s the output for comparing two identical APKs:

![](http://i1.wp.com/www.fasteque.com/wp-content/uploads/2015/12/Screen-Shot-2015-12-05-at-16.26.17.png)

While the following is the result for comparing two different releases of the same application:

![](http://i1.wp.com/www.fasteque.com/wp-content/uploads/2015/12/Screen-Shot-2015-12-05-at-16.56.22.png)

Adding `--help` we can get all the options available for this tool, so the analysis can be even more precise and focused on specific aspects of the application.

A course there’s a lot more we can do with **Androguard**, but the aim of this article is just to present the tool and from that start any kind of experiment and analysis.

## MENTIONS

Before concluding the series, there are a couple of tools which are worth a mention.

The first one is [Bytecode Viewer](https://bytecodeviewer.com/), and it’s actually a complete suit of reverse engineering tools: it allow us to easily edit APKs via smali/baksmali integration and more in general open Android APK and DEX files. **dex2jar** and **Apktool** are integrated too. A bit more about it can be found [here](https://the.bytecode.club/wiki/index.php?title=Bytecode_Viewer).

The second one is [ClassyShark](https://github.com/google/android-classyshark), which is a handy browser for Android executables. It has clients for both Android (APK) and Desktop (JAR). With ClassyShark we can open APK/Zip/Class/Jar files and analyze their contents.

This ends the series: as already highlighted, this was just an introduction to the Android reverse engineering world and the main tools involved in this practice. Of course it really depends on what we need to do, so we may find some tools more suitable while others could be not helpful at all.
