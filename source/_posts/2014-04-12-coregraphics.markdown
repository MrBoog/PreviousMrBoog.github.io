---
layout: post	
title: "CoreGraphics之图形上下文"	
date: 2014-04-12 21:54:18 +0800	
comments: true	
categories: iOS	
published: false

---

CoreGraphics框架在iOS开发中，与UIKit相依为命，占着十分重要的位置。所以对于开发者而言，学习和理解CoreGraphics框架肯定是必不可少啦。我们知道，在一个图形上下文之外进行绘图操作是不可能的。要进行绘制，首先要准备图形上下文。接下来我们就讨论一下几种使用graphics context的情况。	
	
---

一般来讲，我们会建立一个UIView的子类，重载其drawRect:方法，执行该方法的时候系统会为我们准备一个图形上下文。	


	UIGraphics.h:
	
	UIGraphicsGetCurrentContext()
	
	//在drawRect中直接获得图形上下文：CGContextRef context = UIGraphicsGetCurrentContext();

---

不过，在某种情况下。比如对图片的处理，截图，缩放等，需要我们手动创建图形上下文。


先来了解一下获得图形上下文的相关方法:

	UIGraphics.h中：
	
	UIGraphicsBeginImageContextWithOptions(CGSize size, BOOL opaque, CGFloat scale)
	参数：
	void UIGraphicsBeginImageContextWithOptions(
	   CGSize size,
	   BOOL opaque,//否不透明。YES则忽略alpha通道
	   CGFloat scale//缩放因子。我的理解：传1.0，真对于普通屏一个点一个像素；传2.0，针对Retina一个点四个像素。传0.0，根据当前设备匹配[UIScreen mainScreen].scale；
	);

	UIGraphicsBeginImageContext(CGSize size)
	参数：
	void UIGraphicsBeginImageContext (
	   CGSize size
	);
	//这个相当于UIGraphicsBeginImageContextWithOptions (CGSize size,NO,1.0). 所以不推荐大家用UIGraphicsBeginImageContext(CGSize size)。多使用UIGraphicsBeginImageContextWithOptions()。
 

	

---

创建上下文，并将绘制的图保存到UIImage中		
//创建上下文
UIGraphicsBeginImageContextWithOptions(size,NO,[[UIScreen mainScreen] scale]);		
…绘图…		
//获取结果		
UIImage *result = UIGraphicsGetImageFromCurrentImageContext();	
//清除上下文		
UIGraphicsEndImageContext();

---	

将绘制的图保存到UIImage中		
CGColorSpaceRef colorSpace = CGColorSpaceCreateDeviceRGB();	
//创建上下文		
CGContextRef context = CGBitmapContextCreate(NULL,width,height,8,width*4,colorSpace,kCGImageAlphaPremultipliedLast);	
//将创建的上下文设置为当前的上下文		
UIGraphicsPushContext(context);			
…绘图…		
//获取结果			
CGImageRef image = CGBitmapContextCreateImage(ctx);	
UIImage *result = [[UIImage alloc] initWithCGImage:image];	
//清理图片上下文			
UIGraphicsPopContext();			
CGImageRelease(image);			
CGContextRelease(context);			
CGColorSpaceRelease(colorSpace);		


---


大部分情况下，我们绘制的时候，都是子类化UIView，在drawRect:方法中绘制，并通过 setNeedsDisplay刷新。

学习几个绘制常用的属性		

* path
* shadow
* stroke
* line width
* clip path
* file color
