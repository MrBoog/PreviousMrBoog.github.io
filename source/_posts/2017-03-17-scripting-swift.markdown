---
layout: post
title: "Scripting Swift"
date: 2017-03-17 17:19:06 +0800
comments: true
categories: iOS
---

作为iOS开发者，Swift断断续续也用了几年了，不得不说还真是很好用的。但其实swift不仅可以用来写 iOS or Mac OS，只要装了Xcode，__Swift 在Mac系统下写起脚本来一样很好用的。__

Well... Long story short,  Let's Get Started~~

### Hello world

首先，最简单的，我们先来在终端用swift输出hello world试试。
输入 **swift** into the terminal然后回车，输入任何swift语句比如打印hello world，使用__Ctrl-D__退出：

```
$ swift
Welcome to Apple Swift. Type :help for assistance.
1> print(“Hello World”)
2>
```

这就像Python一样，swift命令行的交互模式叫做 [__Swift REPL (read eval print loop)__](https://developer.apple.com/swift/blog/?id=18)。
既然是按照脚本执行，肯定就存在这对应的解析执行脚本的程序。先看看我们平时用的bash：

```
$ cat /etc/shells
# List of acceptable shells for chpass(1).
# Ftpd will not allow users to connect who are not using
# one of these shells.

/bin/bash
/bin/csh
/bin/ksh
/bin/sh
/bin/tcsh
/bin/zsh
```

这里当然没有了，不过如果我们执行type命令可以看到如下输出：

```
$ type swift
swift is hashed (/usr/bin/swift)
```

注意到**/usr/bin/swift**了吗，事实上这就是我们要找的了。

在swift脚本文件最上面加上**#!/usr/bin/swift** (或者 _#!/usr/bin/env swift_) 就启动swift解释器了, 后面的语句就会在swift环境下面编译。无需多说，这基本和我们以前使用bash时 要加入 **#!/bin/bash**一样。

接下来我们就创建一个demo.swift，做个最简单的测试：

```
$touch demo.swift
$vim demo.swift
```

编辑demo.swift文件：

```
#!/usr/bin/swift

import Foundation 
print("hello world")
```

编辑保存后，给它可执行的权限后，**./demo.swift**运行肯定就看到输出效果了。这样就是最简单的swift_脚本了_：

```
$chmod +x demo.swift
$./demo.swift
hello world
```


### Demo time: 

下面简单写一个可以传参数的demo，假设我们要做一个可以一键备份我的博客的功能。

按照之前的步骤，创建一个新的 backup.swift文件，用来提交我的博客到git服务器同时需要提交相应描述。
这里我们要解决两个问题。

1. 获取命令行传入的参数
2. 要在swift里面调用其他命令行工具，比如语句: git add .

要解决第一个问题，我们需要引用一个新的类__CommandLine __(或者`C_ARGC` and `C_ARGV`)。
CommandLine.arguments是个数组，可以直接通过它拿到想要的参数。
至于第二个问题，我们需要另一个在Swift脚本中很重要类__Process__。Process可以混合 swift和bash。
下面直接看下编写后的backup.swift：

```
#!/usr/bin/swift

import Foundation

func backup() {
let args = CommandLine.arguments
guard args.count > 1 else {
return print("Backup Failed! : \n    Detail: Missing commit message whick should be surrounded by quotation mark.")
}
let arg = args[1]
shell("git", "pull")
shell("git", "commit", "-am", arg)
shell("git", "push", "origin", "HEAD:source")
}

@discardableResult
func shell(_ args: String...) -> Int32 {
let task = Process()
task.launchPath = "/usr/bin/env"
task.arguments = args
task.launch()
task.waitUntilExit()
return task.terminationStatus
}

backup()
```

把这个脚本放在我的博客git目录下，运行就会自动开始将内容Push到git服务器了。

```
$ ./backup.swift "Blog: Backup"
remote: Enumerating objects: 139, done.
remote: Counting objects: 100% (139/139), done.
remote: Compressing objects: 100% (33/33), done.
remote: Total 149 (delta 53), reused 139 (delta 53), pack-reused 10
Receiving objects: 100% (149/149), 61.23 KiB | 61.00 KiB/s, done.
...
```

