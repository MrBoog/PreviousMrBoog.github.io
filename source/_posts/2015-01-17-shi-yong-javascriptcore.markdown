---
layout: post
title: "使用 JavaScriptCore <一>"
date: 2015-01-17 16:06:55 +0800
comments: true
categories: iOS

---

随着iOS7 JavaScript引擎的开放，JS在移动端使用场景越来越多。之前，要想在Oc中调用Js，不可避免要使用webview，JavaScriptCore是可以不依托于webview的。比如当下比较火的 ReactNative、JSPatch 都有应用这个库。大部分时候我们可能不需要手写Oc和JS的交互处理，但至少也应该可以看得懂，这样也方便我们理解一些Hybrid开源框架。
 
JavaScriptCore框架中，两个比较重要的类：

* JSContext 作用域，一段JS的执行环境。

* JSValue JS是弱类型语言(即变量的类型是不确定的,Oc、Swift都是强类型)，JSValue用来处理这种差异，对象转化。

##### Oc to JS
在Oc内部，通过JSContext类，直接调用一段JS，返回值包裹在JSValue类型中。

```

	JSContext *context = [[]JSContext alloc] init];
	// 获取错误信息 便于调试	
	context.exceptionHandler = ^(JSContext *context,JSValue *exception){
		NSLog(@"Error is : %@",exception);
	}
	
	// 执行JS
	[context evaluateScript:@"var num = 2 * 8"];
	[context evaluateScript:@"var colors = ['black','blue','green','red']"];
	[context evaluateScript:@"var triple = function(value) { return value * 3 } "];
	
	// 获取Context中创建的变量的值，包裹在JSValue中
	JSValue *num = [context objectForKeyedSubscript:@"num"];
	JSValue *colors = [context objectForKeyedSubscript:@"colors"];
	JSValue *colorName = [colors objectAtIndexedSubscript:1];
	
	// 调用JS函数，
	JSValue *function = [context objectForKeyedSubscript:@"triple"];
	JSValue *result = [function callWithArguments:@[@16]];
	
	
```


##### JS to Oc

context可设置block，JS通过调用block，来实现JS调用Oc。在使用中值得注意的是内存管理，要避免循环引用。如果在block，还需要调用JS，应使用[JSContext currentContext]取得上下文，不应该直接使用外部的context或者value。

```
	
	context["callback"]	 = ^(NSArray *colorArray){
		NSLog(@"callback : %@",colorArray);
	}
	[context evaluateScript:@" callback( [\"black\",\"blue\",\"green\"] ) "];
	
	
	// 下面情况，注意避免循环引用
	[context evaluateScript:@"var colorsCount = function(value) { return value.length }"];
	
    context[@"callbackAgain"] = ^(NSArray *colorArray){
        
        JSContext *curContext = [JSContext currentContext];
        JSValue *colorsCount = [curContext evaluateScript:@"colorsCount"];
        JSValue *count = [colorsCount callWithArguments:@[colorArray]];
        NSLog(@"%@",count);
    };
    [context evaluateScript:@"callbackAgain( [\"black\",\"red\"] )"];

```

上述方法，传值都是限于 Foundation 类型。那JS种是否可以使用Oc的类的？这就要用到 JSExport 协议了。

##### 使用 JSExport 协议

创建一个CustomObject类，继承 ExportProtocol 协议

```

	@protocol ExportProtocol <JSExport>
	- (void)log:(NSString *)string;
	@end
	
	@interface CustomObject : NSObject <ExportProtocol>
	@end
	
	@implementation CustomObject
	- (void)log:(NSString *)string{
		NSLog(@" js : %@",string);
	}
	@end

```

上面我们创建了一个可以给JS使用的Oc类，具体调用类似JS调用block，代码如下

```
	
	context[@"customObject"] = [[CustomObject alloc] init];
	
	[context evaluateScript:@" customObject.log(\"Hello world\") "];
		
```
这样，通过继承JSExport协议，我们就可以在JS中，调用Oc类的方法了。
