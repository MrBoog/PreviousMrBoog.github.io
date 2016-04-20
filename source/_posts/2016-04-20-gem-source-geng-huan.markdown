---
layout: post
title: "gem source 更换"
date: 2016-04-20 11:43:30 +0800
comments: true
categories: 

---



换电脑重新装pods的时候报错，其实是gem报错了。

在执行 `gem install cocoapods ` 的时候，遇到了下面的错误，

```
Unable to download data from https://ruby.taobao.org/ - SSL_connect returned=1 errno=0 state=SSLv3 read server certificate B: certificate verify failed (https://rubygems-china.oss-cn-hangzhou.aliyuncs.com/latest_specs.4.8.gz)
```
感觉像是阿里提供的源出了问题，在github上提了一个[issue](https://github.com/alibaba/ruby.taobao.org/issues/48)，才知道原来是原作者不再维护了。根据作者提供的[链接](https://ruby-china.org/topics/29250)，我们更换一下源就可以了。

```
gem sources --remove https://ruby.taobao.org/

gem sources --add https://gems.ruby-china.org/

```


