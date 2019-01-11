---
layout: post
title: "Symbolicating crash log"
date: 2017-01-10 17:23:39 +0800
comments: true
categories: iOS
---

When our App crashes, a crash report is created and stored on the device. But usually, we need to symbolicate the log before analyzing it. So here I write down the steps to symbolicate crash log.

1. Create a new folder. Place our crash file and dSYM file in that folder: 

    * myApp.crash
    * myApp.dSYM



2. Find `symbolicatecrash` file in your Xcode (which is written by perl). Copy it to the folder that you just created.

   Usually, the path should be: 
       __/Applications/Xcode.app/Contents/SharedFrameworks/DVTFoundation.framework/Versions/A/Resources/symbolicatecrash__

   Or, You could find the path by yourself:

    ```
    find /Applications/Xcode.app -name symbolicatecrash -type f
    ```



3. export env var

    ```
    export DEVELOPER_DIR="/Applications/Xcode.app/Contents/Developer"
    ```



4. Final step, symbolicate crash 
 
    ```
    ./symbolicatecrash myApp.crash > SymbolicatedM.crash
    ```

Ok alright, we've already done all of the work. So if the dSYM is matched with the crash file, here should be possible to generate a new file: __SymbolicatedM.crash__. Just open it, after symbolicating it should be easy to read this time.

---





