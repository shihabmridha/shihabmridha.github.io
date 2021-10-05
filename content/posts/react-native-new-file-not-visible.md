+++
author = "Dijkstra"
categories = ["React native", "JavaScript"]
date = 2019-12-09T09:49:52Z
description = "Newly created file is not visible in computer via USB/MTP in android device. Solution is to use MediaScanner."
draft = false
slug = "react-native-new-file-not-visible"
summary = "Newly created file is not visible in computer via USB/MTP in android device. Solution is to use MediaScanner."
tags = ["React native", "JavaScript"]
title = "React native file system new file is not visible in computer via USB/MTP"

+++


## Problem summary

I was working on a react native project where I had to create a `.csv` file and store it in external/internal storage of the device. So that the user can transfer the file via USB and do someting with it. My target platform was **Android**. Everything was okay untill I tried to access my generate `.csv` file via USB. The file was in the device but I couldn't see it from my computer. Very strange problem. I can view the file from my device's file explorer but can not access it from computer.

After a few minutes of googling I have found that this is a **very known problem of MTP protocol**. I had to restart the device to rescan the file system and access it from computer.

## Solution

Restarting is not a solution. So, the solution was to manually run the [MediaScanner](https://developer.android.com/reference/android/media/MediaScannerConnection) for scanning of the file system. The file [system module for react native](https://github.com/itinance/react-native-fs#android-only-scanfilepath-string-promisestring)  has a function named `scanFile` which is a JavaScript binding.

## Example usage

```javascript
const RNFS = require('react-native-fs');
RNFS.touch(RNFS.DocumentDirectoryPath + '/test.txt')
.then(async path => {
    await RNFS.scanFile(path); // Scanning the newly create file
});
```



