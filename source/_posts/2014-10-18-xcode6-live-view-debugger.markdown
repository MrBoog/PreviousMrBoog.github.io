---
layout: post
title: "Xcode6 live view debugger"
date: 2014-10-18 11:36:18 +0800
comments: true
categories: iOS

---

之前查看视图层级，一直使用reveal。每次新的工程要想使用reveal，都要引入它的framework，用起来还是挺麻烦的。

这次Xcode 6带来了自己的 live view debugger，支持iOS8模拟器。除非想做逆向相关的处理，查看其它app的层级。不然debug自己项目的话，已经够用了。


just select ` Debug\View  Debugging\Capture View Hierarchy `

当然相比于Reveal缺点也有，比如不支持iOS7，比如抓取视图的时候模拟器不能继续其他操作等等。

了解更多[点击这里](http://www.raywenderlich.com/98356/view-debugging-in-xcode-6)