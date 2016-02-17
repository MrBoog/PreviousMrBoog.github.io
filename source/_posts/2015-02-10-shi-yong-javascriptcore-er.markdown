---
layout: post
title: "使用 JavaScriptCore <二>"
date: 2015-02-10 17:04:29 +0800
comments: true
categories: iOS

---


####  当JSExport 遇见 class_addProtocol

之前我们已经大致了解了 JavaScriptCore的基础功能。下面我们想象一个这样的场景：

如果，我需要在JS中设置某个UIView的backgroundColor，那么我可以总么做呢？我们知道以下两点：

* 通过创建继承于 JSExport 的协议，可以让Oc暴漏给JS使用。

* 通过 Runtime 的 class_addProtocol，我们可以给已经存在类添加协议。

这样，我们是不是可以给UIView动态添加继承于 JSExport 的协议，协议里包含backgroundColor属性的setter和getter方法。是不是JS中就可以调用了呢？

代码如下：


```

	#import <objc/runtime.h>
	#import <JavaScriptCore/JavaScriptCore.h>
	
	
	@protocol UIViewExport <JSExport>
		- (void)setBackgroundColor:(UIColor *)backgroundColor;
		- (UIColor *)backgroundColor;
	@end
	
	
	
	- (void)viewDidLoad {
		[super viewDidLoad];
		
		class_addProtocol([UIView class], 	@protocol(UIViewExport));
	

	    JSContext *context = [[JSContext alloc] init];
    	context[@"view"] = self.view;   
	    context[@"color"] = [UIColor blueColor];
    
	    [context evaluateScript:@" view.setBackgroundColor(color) "];
    }
```


