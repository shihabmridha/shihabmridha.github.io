+++
author = "Dijkstra"
categories = ["Flutter"]
date = 2023-05-29T16:54:31Z
description = "Install and configure Flutter on Windows without Android Studio and use Windows Subsystem for Android as emulator"
draft = false
slug = "flutter-setup-without-android-studio-windows-subsystem-for-android"
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