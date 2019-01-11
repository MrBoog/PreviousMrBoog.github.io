---
layout: post
title: "APNs Token-Based Connection"
date: 2017-02-11 20:17:59 +0800
comments: true
categories: 
---

2016年苹果上线了新的推送验证方式，使用新的方式我们将不再需要之前的 __.p12__或者__.pem__文件，取而代之的是一个__.p8__文件 (并且不用区分dev和release环境)。一些大的推送服务厂商已经开始支持这种新的方式了，比如国外的[Firebase](https://firebase.google.com/docs/cloud-messaging/ios/certs)。这种新的方式优势还是很明显的，比如说



* 速度更快 
*  可以支持同一开发者账号下的多个App 
*  支持多个后台同时使用同一个文件 
*  不需要考虑生产和开发环境
*  不会过期

需要注意的是，虽然新的推送验证永远不会过期，但认证密钥和__.p8__文件需要严格保密。如果发现有泄露风险，建议赶紧revoke然后重新创建。

由于不再需要分别配置dev和release环境的证书，所以测试的流程也会简化。__推荐一个开源的基于Node.js的[推送后台](https://www.npmjs.com/package/apn)。__ 我是只用于测试的，毕竟真正的生产环境大家都还是在用一些大厂的服务，比如谷歌的Firebase或者国内的友盟推送等。这里有个简单用这个Node.js的[Demo](https://github.com/MrBoog/Test-ANPS-with-p8-file), 可以用于开发者自己测试推送使用。