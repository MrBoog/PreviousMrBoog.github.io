---
layout: post
title: "Symbolicating crash log"
date: 2017-01-10 17:23:39 +0800
comments: true
categories: iOS
---

1. Create a new folder. Place our crash file and dSYM file in that folder: 

    * myApp.crash
    * myApp.dSYM



2. Find `symbolicatecrash` file  in your Xcode. (which is written by perl) 

   Usually, the path should be: __/Applications/Xcode.app/Contents/SharedFrameworks/DVTFoundation.framework/Versions/A/Resources/symbolicatecrash__

   Or

    ```
    //You could use follow command to Find the path:
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







