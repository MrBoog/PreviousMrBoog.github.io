---
layout: post
title: "Hello ReactNative"
date: 2015-11-11 16:09:23 +0800
comments: true
categories: iOS

---


虽然很多开发者都是一直使用Oc和Swift开发iOS应用，但不得不说ReactNative的'learn-once write-anywhere'，对开发者还是有强大的吸引力的。用Js写逻辑，并且使用原生的UI，不依托于浏览器，如今的Js越来越强大。即便是使用原生语言的开发者，也应该详细了解下。

### 配置 React Native

要想在项目中使用[ReactNative](https://facebook.github.io/react-native/docs/getting-started.html#content)，需要先搭建Node.js环境。下面我们使用nvm，下载管理Node.js。


* nvm

官方安装和使用 nvm 的[步骤](https://github.com/creationix/nvm#installation)。简单整理下：

安装：


```
   	curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.30.2/install.sh | bash

	sh ~/.nvm/nvm.sh
	
```

把下面的内容追加到 `~/.bashrc` 和 `~/.profile` 中

例如：vim ~/.bashrc

```
	source ~/.nvm/nvm.sh
```

完成后重启终端

使用nvm下载Node：

```
	//查看支持版本
	nvm ls-remote
		
	nvm install v5.5.0
	
	nvm use v5.5.0
```

或者直接下载最新版本的Node

```
	nvm install node && nvm alias default node
```


* 安装watchman


```
	brew install watchman
```

* 安装 React Native CLI tool

```
	npm install -g react-native-cli
	
```

准备工作基本结束

---

### hello world

下面我们选择一个目录，新建一个ReactNative项目。

```
	react-native init HelloReactNative
```

找到index.ios.js，替换一下下面的代码。找到iOS文件夹下面的工程，运行一下，一个简单用ReactNative实现的helloworld程序，就可以跑起来了。

```
'use strict';

var React = require('react-native');

var styles = React.StyleSheet.create({
  text: {
    color: 'black',
    backgroundColor: 'white',
    fontSize: 30,
    margin: 80
  }
});


// for app itself
class ReactNativeDemoApp extends React.Component {
  render() {
    return React.createElement(React.Text, {style: styles.text}, "Hello World");
  }
}

React.AppRegistry.registerComponent('HelloReactNative', () => ReactNativeDemoApp);
```





