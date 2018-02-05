# Prerequisits and Configurations for DL4J in Android
While training neural networks is typically done on powerful computers running on multiple GPUs, the compatibility of Deeplearning4J with the Android platform makes using DL4J neural networks in android applications a possibility. This tutorial will cover the basics of seeting up android studio for building DL4J applications. Several configurations for dependencies, memory management, and compilation exculations needed to mitigate the limitatiosn of low powered modile device are outlined below. If you just want to get a DL4J app running on your device, you can jump ahead to a simple example application which trains a neural network for Iris flower classification is available example [here](https://github.com/jrmerwin/DL4JIrisClassifierDemo).
## Prerequisites
* Android Studio 2.2 or newer, which can be downloaded [here](https://developer.android.com/studio/index.html#Other). 
* Android Studio version 2.2 and higher comes with the latest OpenJDK embedded; however, it is recommended to have the JDK installed on your own as you are then able to update it independent of Android Studio. Android Studio 3.0 and later supports all of Java 7 and a subset of Java 8 language features. Java JDKs can be downloaded from Oracle's using this [here](https://http://www.oracle.com/technetwork/java/javase/downloads/index.html).
* Within Android studio, the Android SDK Manager can be used to install Android Build tools 24.0.1 or later, SDK platform 24 or later, and the Android Support Repository. 
* l An Android device or an emulator running API level 21 or higher. A minimum of 200 MB of internal storage space free is recommended.
It is also recommended that you download and install IntelliJ IDEA, Maven, and the complete dl4j-examples directory for building and building and training neural nets on your desktop instead of android studio. A quickstart guide for setting up DL4j projects can be found [here](https://deeplearning4j.org/quickstart).
## Required Dependencies
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
