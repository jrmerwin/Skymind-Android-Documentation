# Prerequisits and Configurations for DL4J in Android
Contents
* [Prerequisites](#head_link1)
* [Required Dependencies](#head_link2)
* [Managing Dependencies with ProGuard](#head_link3)
* [Memory Management](#head_link4)


While training neural networks is typically done on powerful computers running on multiple GPUs, the compatibility of Deeplearning4J with the Android platform makes using DL4J neural networks in android applications a possibility. This tutorial will cover the basics of seeting up android studio for building DL4J applications. Several configurations for dependencies, memory management, and compilation exculations needed to mitigate the limitatiosn of low powered modile device are outlined below. If you just want to get a DL4J app running on your device, you can jump ahead to a simple example application which trains a neural network for Iris flower classification is available example [here](https://github.com/jrmerwin/DL4JIrisClassifierDemo).
## <a name="head_link1">Prerequisites</a>
* Android Studio 2.2 or newer, which can be downloaded [here](https://developer.android.com/studio/index.html#Other). 
* Android Studio version 2.2 and higher comes with the latest OpenJDK embedded; however, it is recommended to have the JDK installed on your own as you are then able to update it independent of Android Studio. Android Studio 3.0 and later supports all of Java 7 and a subset of Java 8 language features. Java JDKs can be downloaded from Oracle's using this [here](https://http://www.oracle.com/technetwork/java/javase/downloads/index.html).
* Within Android studio, the Android SDK Manager can be used to install Android Build tools 24.0.1 or later, SDK platform 24 or later, and the Android Support Repository. 
* l An Android device or an emulator running API level 21 or higher. A minimum of 200 MB of internal storage space free is recommended.
It is also recommended that you download and install IntelliJ IDEA, Maven, and the complete dl4j-examples directory for building and building and training neural nets on your desktop instead of android studio. A quickstart guide for setting up DL4j projects can be found [here](https://deeplearning4j.org/quickstart).
## <a name="head_link2">Required Dependencies</a>
In order to use Deeplearning4J in your Android projects, you will need to add the following dependencies to your app module’s build.gradle file:
``` java
compile 'org.deeplearning4j:deeplearning4j-nn:0.9.1'
compile 'org.nd4j:nd4j-native:0.9.1'
compile 'org.nd4j:nd4j-native:0.9.1:android-x86'
compile 'org.nd4j:nd4j-native:0.9.1:android-arm'
compile 'org.bytedeco.javacpp-presets:systems-platform:1.4'
compile 'org.bytedeco.javacpp-presets:openblas:0.2.19-1.3:android-x86'
compile 'org.bytedeco.javacpp-presets:openblas:0.2.19-1.3:android-arm'
testCompile 'junit:junit:4.12'
```
DL4J depends on ND4J, which is a library that offers fast n-dimensional arrays. ND4J in turn depends on a platform-specific native code library called JavaCPP,therefore you must load a version of ND4J that matches the architecture of the Android device. Both -x86 and -arm types can be included to support multiple device processor types. 

The above dependencies contain several files with identical names which must be handled with the following exclude parameters to your packagingOptions.
```java
packagingOptions {
            exclude 'META-INF/DEPENDENCIES'
            exclude 'META-INF/DEPENDENCIES.txt'
            exclude 'META-INF/LICENSE'
            exclude 'META-INF/LICENSE.txt'
            exclude 'META-INF/license.txt'
            exclude 'META-INF/NOTICE'
            exclude 'META-INF/NOTICE.txt'
            exclude 'META-INF/notice.txt'
            exclude 'META-INF/INDEX.LIST'

            exclude 'org/bytedeco/javacpp/windows-x86/msvcp120.dll'
            exclude 'org/bytedeco/javacpp/windows-x86_64/msvcp120.dll'
            exclude 'org/bytedeco/javacpp/windows-x86/msvcr120.dll'
            exclude 'org/bytedeco/javacpp/windows-x86_64/msvcr120.dll'
        }
 ```       
After addind the above dependencies and exclusions to the build.gradle file, try syncing Gradle with to see if any other exclusions are needed. The error message will identify the file path that should be added to the list of exclusions. An example error message with file path is: *> More than one file was found with OS independent path 'org/bytedeco/javacpp/ windows-x86_64/msvp120.dll'*
Compiling these dependencies involves a large number of files, thus it is necessary to set multiDexEnabled to true in defaultConfig.
```java
multiDexEnabled true
```
Finally, a conflict in the junit module versions will throw the following error: *> Conflict with dependency 'junit:junit' in project ':app'. Resolved versions for app (4.8.2) and test app (4.12) differ*. This can be suppressed by forcing all of the junit modules to use the same version with the following:
``` java
configurations.all {
    resolutionStrategy.force 'junit:junit:4.12'
}
```
## <a name="head_link3">Managing Dependencies with ProGuard</a>
The DL4J dependencies compile a large number of files. ProGuard can be used to minimize your APK file size. ProGuard detects and removes unused classes, fields, methods, and attributes from your packaged app, including those from code libraries. You can learn more about using Proguard [here](https://developer.android.com/studio/build/shrink-code.html).
To enable code shrinking with ProGuard, add minifyEnabled true to the appropriate build type in your build.gradle file.
```java
buildTypes {
    release {
        minifyEnabled true
        proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
    }
}
```
It is recommended to upgrade your ProGuard in the Android SDK to the latest release (5.1 or higher). Upgrading the buildtools or other aspects of the sdk your proguard might reset to the version being shipped with the sdk. In order to force ProGaurd to use a different version of ProGuard than the Android Gradle default you can include this in the buildscript of build.gradle file:
``` java
buildscript {
    configurations.all {
        resolutionStrategy {
            force 'net.sf.proguard:proguard-gradle:5.3.2'
        }
    }
}
```
Proguard optimzes and reduces the amount of code in your Android application in order to make if  smaller and faster. Unfortunately, proguard removes annotations be default, including the @Platform annotation used by javaCV. To make proguard preserve these annotations and keep native methods add the following flags to the progaurd-rules.pro file. 
``` java
# enable optimization
-optimizations !code/simplification/arithmetic,!code/simplification/cast,!field/*,!class/merging/*
-optimizationpasses 5
-allowaccessmodification
-dontwarn org.apache.lang.**

-keepattributes *Annotation*
# JavaCV
-keep @org.bytedeco.javacpp.annotation interface * {*;}
-keep @org.bytedeco.javacpp.annotation.Platform public class *
-keepclasseswithmembernames class * {@org.bytedeco.* <fields>;}
-keepclasseswithmembernames class * {@org.bytedeco.* <methods>;}

-keepattributes EnclosingMethod
-keep @interface org.bytedeco.javacpp.annotation.*,javax.inject.*

-keepattributes *Annotation*, Exceptions, Signature, Deprecated, SourceFile, SourceDir, LineNumberTable, LocalVariableTable, LocalVariableTypeTable, Synthetic, EnclosingMethod, RuntimeVisibleAnnotations, RuntimeInvisibleAnnotations, RuntimeVisibleParameterAnnotations, RuntimeInvisibleParameterAnnotations, AnnotationDefault, InnerClasses
-keep class org.bytedeco.javacpp.** {*;}
-dontwarn java.awt.**
-dontwarn org.bytedeco.javacv.**
-dontwarn org.bytedeco.javacpp.**
# end javacv

# This flag is needed to keep native methods
-keepclasseswithmembernames class * {
 native <methods>;
}

-keep public class * extends android.view.View {
 public <init>(android.content.Context);
 public <init>(android.content.Context, android.util.AttributeSet);
 public <init>(android.content.Context, android.util.AttributeSet, int);
 public void set*(...);
}

-keepclasseswithmembers class * {
 public <init>(android.content.Context, android.util.AttributeSet);
}

-keepclasseswithmembers class * {
 public <init>(android.content.Context, android.util.AttributeSet, int);
}

-keepclassmembers class * extends android.app.Activity {
 public void *(android.view.View);
}

# For enumeration classes
-keepclassmembers enum * {
 public static **[] values();
 public static ** valueOf(java.lang.String);
}

-keep class * implements android.os.Parcelable {
 public static final android.os.Parcelable$Creator *;
}

-keepclassmembers class **.R$* {
 public static <fields>;
}

-keep class android.support.v7.app.** { *; }
-keep interface android.support.v7.app.** { *; }
-keep class com.actionbarsherlock.** { *; }
-keep interface com.actionbarsherlock.** { *; }
-dontwarn android.support.**
-dontwarn com.google.ads.**

# Flags to keep standard classes
-keep public class * extends android.app.Activity
-keep public class * extends android.app.Application
-keep public class * extends android.app.Service
-keep public class * extends android.content.BroadcastReceiver
-keep public class * extends android.content.ContentProvider
-keep public class * extends android.app.backup.BackupAgent
-keep public class * extends android.preference.Preference
-keep public class * extends android.support.v7.app.Fragment
-keep public class * extends android.support.v7.app.DialogFragment
-keep public class * extends com.actionbarsherlock.app.SherlockListFragment
-keep public class * extends com.actionbarsherlock.app.SherlockFragment
-keep public class * extends com.actionbarsherlock.app.SherlockFragmentActivity
-keep public class * extends android.app.Fragment
-keep public class com.android.vending.licensing.ILicensingService
```
Testing your app is the best way to check if any errors are being caused by inappropriately removed code; however, you can also inspect what was removed by reviewing the usage.txt output file saved in <module-name>/build/outputs/mapping/release/.

To fix errors and force ProGuard to retain certain code, add a -keep line in the ProGuard configuration file. For example:
``` java
-keep public class MyClass
```
## <a name="head_link4">Memory Management</a>
It may also be advantageous to increase the allocated memory to your app by adding android:largeHeap="true" to the manifest file. Allocating a larger heap means that you decrease the risk of throwing an OutOfMemoryError during memory intensive operations. 
``` xml
android:largeHeap="true"
```
Practical considerations regarding performance limits are needed when building applications with neural networks. Training a neural network on a device is possible, but should only be attempted with networks with limited numbers of layers, nodes, and iterations. The first Demo app [DL4JIrisClassifierDemo](https://github.com/jrmerwin/DL4JIrisClassifierDemo) is able to train on a standard device in about 15 seconds. Training on a device may be desirable if the neural network is being trained off user input data. 

For larger or more complex neural networks like Convolutional or Reccurrent Neural Networks, training on the device is not a realistic option as long processing times during network training run the risk of generating an OutOfMemoryError and make for a poor user experience. As an alternative, the Neural Network can be built and trained on the desktop and then loaded as a pre-trained model in the application. Using a pre-trained model in you Android application can be achieved with the following steps:
	1. Train the yourModel on desktop and save via modelSerializer.
	2. Create a raw resource folder in the res directory of the application.
	3. Copy yourModel.zip file into the raw folder.
	4. Access it from your resources using an inputStream like so:
``` java
try {
// Load name of model file (yourModel.zip).
        InputStream is = getResources().openRawResource(R.raw.yourModel);

// Load yourModel.zip.
        MultiLayerNetwork restored = ModelSerializer.restoreMultiLayerNetwork(is);
// Use yourModel.
        INDArray results = restored.output(input)
        System.out.println("Results: "+ results );
// Handle the exception error
} catch(IOException e) {
        e.printStackTrace();
    }
```



Sections adapted from [Progur](https://github.com/jrmerwin/DL4JIrisClassifierDemo) 
