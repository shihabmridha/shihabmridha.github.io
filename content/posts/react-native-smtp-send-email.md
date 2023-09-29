+++
author = "Dijkstra"
date = 2019-10-30T09:31:32Z
description = "React native SMTP is a SMTP client for react-native. It can be used to send emails in the background using the SMTP protocol. This module only supports Android."
draft = false
slug = "react-native-smtp-send-email"
summary = "React native SMTP is a SMTP client for react-native. It can be used to send emails in the background using the SMTP protocol. This module only supports Android."
tags = ["react native", "java", "javascript", "smtp"]
title = "React Native SMTP. Send Email in Background (Android Only)"

+++


I am not a big fan of mobile application development. In fact, I don't feel comfortable with anything related to the user interface. It is not like I can not do, it is more like I am less experienced with it.

In my current project, I was writing some back-end stuff for our new IoT platform. Suddenly one of my team asked me to help him with his [react native](https://facebook.github.io/react-native/) project. He had a deadline soon and a lot of work to do. He convinced me to work with him because he knew I had previous experience with the react library. Since react and react-native are similar, I decided to help him. FYI, the app was android only.

He was working on a task where he had to develop a [native module](https://facebook.github.io/react-native/docs/native-modules-android) for the app. In case you didn't know, the native module is a way of using native code (Java for Android, Swift/Objective-c for iOS) or calling a native API from JavaScript. The was for a special kind of android device where he had to implement a scanner to scan something. But, the available scanner library is written in Java. There was no relative react-native module out there. So, he had to write his own Java wrapper for the Java library.

My task was to export those scanned data as CSV to the file system, upload it to an FTP server and send it via email as attachments. Exporting as CSV to the file system was fairly straight forward. No additional module was needed to perform this simple task. After that, I had to implement a feature which uploads file to the FTP server. So, I started looking for a react-native FTP client. Fortunately found one but with poor documentation. Still managed to make it work. Now, it is time to implement a send email feature. To achieve this I needed an SMTP client. Found one. I was happy since things are going well but not for long. The SMTP client couldn't establish a connection with the SMTP server. I tried many ways but failed. Installed another module. This time the app didn't even compile successfully.

I informed my partner. He said once he is done with his native module implementation, he will build the SMTP client as well. I was like okay, let me know when you are done.

Then I decided to give it a try. I had previous experience of working with Java. So, I tried to implement an SMTP client (native module) for our react native application. It took a whole day for me to make it work. But honestly, it was a journey worth taking.

I really love the idea of native module, bridging native code with javascript. Thanks react native team for developing such impressive technology.

I have made a separate NPM module for it and published it to NPM. Check out my first ([react-native-smtp](https://www.npmjs.com/package/react-native-smtp)) NPM module.

If you are an iOS developer and wish to extend the module's support to iOS as well, you can create a pull request for it. I was surprised to see my module is getting downloaded. It feels great!

### Usage

To install the module open up your terminal and run `npm install --save react-native-smtp`.

Then you have to link the module with your project. To achieve this run the following command `react-native link react-native-smtp`.

Now, to send email use the following snippet.

```javascript
import EmailSender from 'react-native-smtp';
 
// Configuration
EmailSender.config({
  host: 'smtp.host.io',
  port: '465', // Optional. Default to 465
  username: 'username',
  password: 'password',
  isAuth: 'true', // Optional. Default to `true`
  tls: 'true' // Optional. Default to `true`
});
 
/*
 * Used react-native-fs module to get file path.
 * Keep this array empty if there is no attachment.
 */
const attachments = [
  RNFS.ExternalStorageDirectoryPath + '/Tracklist/file.txt',
  RNFS.ExternalStorageDirectoryPath + '/Tracklist/file_2.txt',
];
 
// Now send the mail
EmailSender.send(
  {
    from: 'from@email.com',
    to: 'to@email.com,another@email.com',
    subject: 'The subject',
    body: '<h3> Cool Body </h3>';
  },
  attachments, // This second parameter is mandatory. You can send an empty array.
);
```

### Conclusion

This module doesn't handle every tidbits so if you are trying to do something advanced then you might get some error or you might not get the result you expected. But, for basic usage it should work fine.

