---
layout: post
title: "模拟低速网络"
date: 2015-05-19 15:09:21 +0800
comments: true
categories: iOS

---


之前一直在终端使用 `ipfw`脚本命令，来模拟低速网络，不过在Yosemite 之后Apple移除了ipfw 命令，[but now  IPFW is gone, Moving to PF](https://discussions.apple.com/thread/6645172?start=0&tstart=0)。


##### GUI工具:

 * Charles 网络上介绍的已经很多了，就不再多说了。

 * 使用官方提供的[ Network Link Conditioner](https://developer.apple.com/downloads/?q=Hardware%20IO%20Tools)。

这是IO工具包，下载后，双击“Network Link Conditioner.prefPane” 进行安装。

启用后，配置一下very bad network，重新启动模拟器，我们就可以使用调整我们的模拟器网络环境了，模拟低速网络。(目前没办法对特定的地址限速，会降低电脑中所有请求的速度)

##### Terminal PF命令：


当然，如果使用终端，还是可以使用 PF命令，来替换 ipfw，下面是一断脚本

```

#!/bin/bash

# Reset dummynet to default config
dnctl -f flush

# Compose an addendum to the default config: creates a new anchor
(cat /etc/pf.conf &&
echo 'dummynet-anchor "my_anchor"' &&
echo 'anchor "my_anchor"') | pfctl -q -f -

# Configure the new anchor
cat <<EOF | pfctl -q -a my_anchor -f -
no dummynet quick on lo0 all
dummynet out proto tcp from any to any pipe 1
EOF

# Create the dummynet queue
dnctl pipe 1 config bw 4100byte/s

# Activate PF
pfctl -E

# to check that dnctl is properly configured: sudo dnctl list
```

检查配置列表

```
sudo dnctl list
```

最后的最后。。使用后别忘记重置。。

```
sudo dnctl -f flush
sudo pfctl -f /etc/pf.conf
sudo dnctl list

```


如果不起作用，可能是PF默认没有开启，使用 `sudo pfctl -E` 开启。

