## Android

(... TODO ...)

Android is an open source platform that can be found nowadays on many devices:

* Mobile Phones and Tablets
* Wearables
* "Smart" devices in general like TVs

It also offers an application environment that supports not only pre-installed applications on the device, but also 3rd party applications that can be downloaded from marketplaces like Google Play.

The software stack of Android comprises of different layers, where each layer is defining certain behavior and offering specific services to the layer above.

![Android Software Stack](https://source.android.com/security/images/android_software_stack.png)

On the lowest level Android is using the Linux Kernel where the core operating system is built up on. The hardware abstraction layer defines a standard interface for hardware vendors. HAL implementations are packaged into shared library modules (.so files). These modules will be loaded by the Android system at the appropriate time. The Android Runtime consists of the core libraries and the Dalvik VM (Virtual Machine). Applications are most often implemented in Java and compiled in Java class files and then compiled again into the dex format. The dex files are then executed within the Dalvik VM. With Android 4.4 the successor of Dalvik VM was introduced, called Android Runtime (ART). Applications are executed in the Android Application Sandbox that enforces isolation of application data and code execution from other applications on the device, that adds an additional layer of security.

The Android Framework is creating an abstraction layer for all the layers below, so developers can implement Android Apps and can utilize the capabilities of Android without deeper knowledge of the layers below. It also offers a robust implementation that offers common security functions like secure IPC or cryptography.

(... TODO ...)

### Inter-Process Communication

As we know, every process on Android has its own sandboxed address space. Inter-process communication (IPC) facilities enable apps to exchange signals and data in a (hopefully) secure way. Instead of relying on the default Linux IPC facilities, IPC on Android is done through Binder, a custom implementation of OpenBinder. A lot of Android system services, as well as all high-level IPC services, depend on Binder.

In the Binder framework, a client-server communication model is used. IPC clients communicate through a client-side proxy. This proxy connects to the Binder server, which is implemented as a character driver (/dev/binder).The server holds a thread pool for handling incoming requests, and is responsible for delivering messages to the destination object. Developers  write interfaces for remote services using the Android Interface Descriptor Language (AIDL).

![Binder Overview](/Document/Images/Chapters/0x04a/binder.jpg)
*Binder Overview. Image source: [Android Binder by Thorsten Schreiber](https://www.nds.rub.de/media/attachments/files/2011/10/main.pdf)*

#### High-Level Abstractions

*Intent messaging* is a framework for asynchronous communication built on top of binder. This framework enables both point-to-point and publish-subscribe messaging. An *Intent* is a messaging object that can be used to request an action from another app component. Although intents facilitate communication between components in several ways, there are three fundamental use cases:

- Explicit intents specify the component to start by name (the fully-qualified class name).

- Implicit intents do not name a specific component, but instead declare a general action to perform, which allows a component from another app to handle it. When you create an implicit intent, the Android system finds the appropriate component to start by comparing the contents of the intent to the intent filters declared in the manifest file of other apps on the device.

An *intent filter* is an expression in an app's manifest file that specifies the type of intents that the component would like to receive. For instance, by declaring an intent filter for an activity, you make it possible for other apps to directly start your activity with a certain kind of intent. Likewise, if you do not declare any intent filters for an activity, then it can be started only with an explicit intent.

For activities and broadcast receivers, intents are the preferred mechanism for asynchronous IPC in Android. Depending on your application requirements, you might use sendBroadcast(), sendOrderedBroadcast(), or an explicit intent to a specific application component.

A BroadcastReceiver handles asynchronous requests initiated by an Intent.

Using Binder or Messenger is the preferred mechanism for RPC-style IPC in Android. They provide a well-defined interface that enables mutual authentication of the endpoints, if required.


(... TODO ... briefly on security implications)

Android’s Messenger represents a reference to a Handler that can be sent to a remote process via an Intent

A reference to the Messenger can be sent via an Intent using the previously mentioned IPC mechanism

Messages sent by the remote process via the messenger are delivered to the local handler. Great for efficient call-backs from the service to the client

#### Security Implications



### Android Application Overview

#### App Folder Structure

Android applications installed (from Google Play Store or from external sources) are located at /data/app/. Since this folder cannot be listed without root, another way has to be used to get the exact name of the apk. To list all installed apks, the Android Debug Bridge (adb) can be used. ADB allows a tester to directly interact with the real phone, e.g., to gain access to a console on the device to issue further commands, list installed packages, start/stop processes, etc.
To do so, the device has to have USB-Debugging enabled (under developer settings) and has to be connected via USB.
Once USB-Debugging is enabled, the connected devices can be viewed with the command

```bash
$ adb devices
List of devices attached
BAZ5ORFARKOZYDFA	device
```

Then the following command lists all installed apps and their locations:

```bash
$ adb shell pm list packages -f
package:/system/priv-app/MiuiGallery/MiuiGallery.apk=com.miui.gallery
package:/system/priv-app/Calendar/Calendar.apk=com.android.calendar
package:/system/priv-app/BackupRestoreConfirmation/BackupRestoreConfirmation.apk=com.android.backupconfirm
```

To pull one of those apps from the phone, the following command can be used:

```bash
$ adb pull /data/app/com.google.android.youtube-1/base.apk
```

This file only contains the “installer” of the application, meaning this is the app the developer uploaded to the market.
The local data of the application is stored at /data/data/PACKAGE-NAME and has the following structure:

```bash
drwxrwx--x u0_a65   u0_a65            2016-01-06 03:26 cache
drwx------ u0_a65   u0_a65            2016-01-06 03:26 code_cache
drwxrwx--x u0_a65   u0_a65            2016-01-06 03:31 databases
drwxrwx--x u0_a65   u0_a65            2016-01-10 09:44 files
drwxr-xr-x system   system            2016-01-06 03:26 lib
drwxrwx--x u0_a65   u0_a65            2016-01-10 09:44 shared_prefs
```

* **cache**: This location used to cache application data on runtime including WebView caches.
* **code_cache**: TBD
* **databases**: This folder stores sqlite database files generated by the application at runtime, e.g. to store user data
* **files**: This folder is used to store files that are created in the App when using the internal storage.
* **lib**: This folder used to store native libraries written in C/C++. These libraries can have file extension as .so, .dll (x86 support). The folder contains subfolders for the platforms the app has native libraries for:
   * armeabi: compiled code for all ARM based processors only
   * armeabi-v7a: compiled code for all ARMv7 and above based processors only
   * arm64-v8a: compiled code for all ARMv8 arm64 and above based processors only
   * x86: compiled code for x86 processors only
   * x86_64: compiled code for x86_64 processors only
   * mips: compiled code for MIPS processors only
* **shared_prefs**: This folder is used to store the preference file generated by application on runtime to save current state of application including data, configuration, session, etc. The file format is XML.

#### APK Structure

An application on Android is a file with the extension .apk. This file is a signed zip-file which contains different files for the bytecode, assets, etc. When unzipped the following directory structure can be identified:

```bash
$ unzip base.apk
$ ls -lah
-rw-r--r--   1 sven  staff    11K Dec  5 14:45 AndroidManifest.xml
drwxr-xr-x   5 sven  staff   170B Dec  5 16:18 META-INF
drwxr-xr-x   6 sven  staff   204B Dec  5 16:17 assets
-rw-r--r--   1 sven  staff   3.5M Dec  5 14:41 classes.dex
drwxr-xr-x   3 sven  staff   102B Dec  5 16:18 lib
drwxr-xr-x  27 sven  staff   918B Dec  5 16:17 res
-rw-r--r--   1 sven  staff   241K Dec  5 14:45 resources.arsc
```

* **AndroidManifest.xml**: Contains the definition of application’s package name, target and min API version, application configuration, application components, user-granted permissions, etc.
* **META-INF**: This folder contains metadata of application:
   * MANIFEST.MF: stores hashes of application resources.
   * CERT.RSA: The certificate(s) of the application.
   * CERT.SF: The list of resources and SHA-1 digest of the corresponding lines in the MANIFEST.MF file.
* **assets**: A directory containing applications assets (files used within the Android App like XML, Java Script or pictures) which can be retrieved by the AssetManager.
* **classes.dex**: The classes compiled in the DEX file format understandable by the Dalvik virtual machine/Android Runtime. DEX is Java Byte Code for Dalvik Virtual Machine. It is optimized for running on small devices.
* **lib**: A directory containting libraries that are part of the APK, for example 3rd party libraries that are not part of the Android SDK.
* **res**: A directory containing resources not compiled into resources.arsc.
* **resources.arsc**: A file containing precompiled resources, such as XML files for the layout.

Since some resources inside the APK are compressed using non-standard algorithms (e.g. the AndroidManifest.xml), simply unzipping the file does not reveal all information. A better way is to use the tool apktool to unpack and uncompress the files. The following is a listing of the the files contained in the apk:

```bash
$ apktool d base.apk
I: Using Apktool 2.1.0 on base.apk
I: Loading resource table...
I: Decoding AndroidManifest.xml with resources...
I: Loading resource table from file: /Users/sven/Library/apktool/framework/1.apk
I: Regular manifest package...
I: Decoding file-resources...
I: Decoding values */* XMLs...
I: Baksmaling classes.dex...
I: Copying assets and libs...
I: Copying unknown files...
I: Copying original files...
$ cd base
$ ls -alh
total 32
drwxr-xr-x    9 sven  staff   306B Dec  5 16:29 .
drwxr-xr-x    5 sven  staff   170B Dec  5 16:29 ..
-rw-r--r--    1 sven  staff    10K Dec  5 16:29 AndroidManifest.xml
-rw-r--r--    1 sven  staff   401B Dec  5 16:29 apktool.yml
drwxr-xr-x    6 sven  staff   204B Dec  5 16:29 assets
drwxr-xr-x    3 sven  staff   102B Dec  5 16:29 lib
drwxr-xr-x    4 sven  staff   136B Dec  5 16:29 original
drwxr-xr-x  131 sven  staff   4.3K Dec  5 16:29 res
drwxr-xr-x    9 sven  staff   306B Dec  5 16:29 smali
```

* **AndroidManifest.xml**: This file is not compressed anymore and can be openend in a text editor.
* **apktool.yml** : This file contains information about the output of apktool.
* **assets**: A directory containing applications assets (files used within the Android App like XML, Java Script or pictures) which can be retrieved by the AssetManager.
* **lib**: A directory containting libraries that are part of the APK, for example 3rd party libraries that are not part of the Android SDK.
* **original**: TBD
* **res**: A directory containing resources not compiled into resources.arsc.
* **smali**: A directory containing the disassembled Dalvik Bytecode in Smali. Smali is a human readable representation of the Dalvik executable.

### References

+ [Android Security](https://source.android.com/security/)
+ [HAL](https://source.android.com/devices/)
+ "Android Security: Attacks and Defenses" By Anmol Misra, Abhishek Dubey
