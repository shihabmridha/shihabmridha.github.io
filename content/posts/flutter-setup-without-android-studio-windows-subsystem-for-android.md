+++
author = "Dijkstra"
categories = ["Flutter"]
date = 2023-05-26T16:54:31Z
description = "Install and configure Flutter on Windows without Android Studio and use Windows Subsystem for Android as emulator"
draft = false
slug = "flutter-setup-without-android-studio-windows-subsystem-for-android"
summary = "Install and configure Flutter on Windows without Android Studio and use Windows Subsystem for Android as emulator"
tags = ["Flutter"]
title = "Configure Flutter without Android Studio on Windows Subsystem for Android"

+++

### Prerequisite
- Flutter Windows SDK downloaded and added to the environment variable.
- Install the latest Java JDK (ex: 20 as of writing it) and add `JAVA_HOME` environment variable.

### Download android command line tool
This command line tools will be used to install the Android SDK and other build tools. Download the latest version from [here](https://developer.android.com/studio#command-tools:~:text=Command%20line%20tools%20only). The documentation of the SDK manager commands can be found [here](https://developer.android.com/studio/command-line/sdkmanager).

In `C:\` drive, create a directory named `android-sdk/cmdline-tools/latest` and extract the downloaded CLI tool there. The reason we are creating such a folder structure is that when we are going to download the build-tools, platforms, etc they are going to be downloaded in the android-sdk directory automatically. We do not have to specify any path while downloading.

Open a terminal and go to `android-sdk/cmdline-tools/latest/bin` directory. Now we will run a few commands to download the SDK and necessary build tools.

Run the following command to see all the items that can be downloaded via the SDK manager.
```bash
> sdkmanager.bat --list
```

We need to download platform-tools, build-tools, and an android platforms. The command is:
```bash
# Assuming we are downloading android platform-33 and build-tools 33.0.2
# We can get all the available options from the --list command above

sdkmanager.bat "platform-tools" "build-tools;33.0.2" "platforms;android-33"
```

Add the following paths to the [environment variable](https://learn.microsoft.com/en-us/previous-versions/office/developer/sharepoint-2010/ee537574(v=office.14)).
```bash
C:\android-sdk\platform-tools
C:\android-sdk\platforms
```

Close the terminal and open it. Check if we have access to adb command by checking its version.
```bash
> adb --version

Android Debug Bridge version 1.0.41
Version 34.0.1-9979309
Installed as C:\android-sdk\platform-tools\adb.exe
Running on Windows 10.0.22621
```

### Prepare WSA emulator for running the app
Open Windows Subsystem for Android settings and go to “Developer” section. Turn on the “Developer mode”. If you do not see any IP address, then click on the “Manage developer settings”.
![flutter setup without android studion wsa](images/flutter-setup-1.png)

It should open the WSA emulator’s “Developer options” page. Now, close the Windows Subsystem for Android settings window and re-open it and go to “Developer” section again.  Now you should see the IP and the port.
![flutter setup without android studion wsa](images/flutter-setup-2.png)

Run the following command to connect to the WSA emulator so that we can run our app there.
```bash
> adb connect 127.0.0.1:58526
```
ou might get a prompt saying something like “do you want to allow access”. Press “Allow/Yes/Okay/Whatever positive”.
Run the following command to check if your WSA is recognized as a device.
```bash
> flutter devices
```
![flutter setup without android studion wsa](images/flutter-setup-3.png)

### Create and run a Flutter app
Create a new Flutter app using the following command.
```bash
> flutter create fake_app
```
Go to the fake_app directory and run the app using the command below.
```bash
> flutter run

or, you can mention the device
> flutter run -d Subsystem

```

Now, there is a chance that you will get a Gradle build failed error saying there is a version mismatch. To fix that open the `fake_caller\android\gradle\wrapper\gradle-wrapper.properties` file. Change the `distributionUrl` URL to a Gradle version that is supported by your installed JDK. I am using JDK 20 so I will be using 8.1 or later. My file would look like this.
```bash
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
distributionUrl=https\://services.gradle.org/distributions/gradle-8.1-all.zip
```
Execute the flutter run command again to run the app. This time it should run successfully and open the app in a WSA window. For the first run it might take some time.

### Note:
I have noticed that when I ran the `flutter run` command it downloaded the “Android Emulator” for some reason. Probably it is required for debugging purpose. But, I do not exactly know why.
