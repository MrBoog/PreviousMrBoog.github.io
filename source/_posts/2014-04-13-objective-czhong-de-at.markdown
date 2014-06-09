---
layout: post	
title: "objective-c中的'@'"	
date: 2014-04-13 13:56:34 +0800		
comments: true	
categories: OC syntax	

---

在学习和使用Objective-c语言的时候，我们随处可见'@'符号，这和其他语言有很大不同。

由于c、c++本身没有'@'，Oc这样设计可能是为了更好地和c、c++混编。正如[这篇文章](http://stackoverflow.com/a/25784)所提到的。不过'@'的用途多种多样，我感觉还是需要整理一下。

网上Steffen Itterheim的[这篇文章](http://www.learn-cocos2d.com/2011/10/complete-list-objectivec-20-compiler-directives/)基本把大部分用到'@'的部分罗列了。

* @class
* @defs
* @protocol @required @optional @end
* @interface @public @package @protected @private @property @end
* @implementation @synthesize @dynamic @end
* @throw @try @catch @finally
* @synchronized @autoreleasepool
* @selector @encode
* @compatibility_alias	（给某个类起别名）
* @"string"
* @'a'	@1  @YES
* @[]
* @{}
* @()

以上涉及的这些，有的十分常见，也有的我们可能这辈子都用不到。下面，我们就给几个不常见的，或者比较重要的并且容易忽视的用法，总结一下。		


1 @compatibility_alias

关于@compatibility_alias的用法，Mattt Thompson大大举过很好的[示例](http://nshipster.com/at-compiler-directives/)。在iOS5下想使用collectionView的效果，无疑[PSTCollectionView](https://github.com/ptshih/PSCollectionView) 是最佳的替代方案，下面的代码，可以为PSTCollectView起个别名。这样在iOS5我们也可以制造出使用UICollectionView的幻觉，重点是以后升级可以省去很多麻烦： 
	
		#if __IPHONE_OS_VERSION_MAX_ALLOWED < 60000	
		@compatibility_alias UICollectionViewController PSTCollectionViewController;		
		@compatibility_alias UICollectionView PSTCollectionView;	
		@compatibility_alias UICollectionReusableView PSTCollectionReusableView;	
		@compatibility_alias UICollectionViewCell PSTCollectionViewCell;	
		@compatibility_alias UICollectionViewLayout PSTCollectionViewLayout;	
		@compatibility_alias UICollectionViewFlowLayout PSTCollectionViewFlowLayout;	
		@compatibility_alias UICollectionViewLayoutAttributes     PSTCollectionViewLayoutAttributes;		
		@protocol UICollectionViewDataSource <PSTCollectionViewDataSource> @end		
		@protocol UICollectionViewDelegate <PSTCollectionViewDelegate> @end				
	 #endif	
 
2 Object Literals
 
 2012年苹果加入了如下字面值，简化了代码。		
 
 NSArray Literal：	@[ ]			
 
  		NSArray * array = @[ @1, @"b", @YES];		
 		id str = array[0];		

 NSDictionary Literal:	@{ }	
 		
		NSDictionary * dict = @{ @"key1" : @"obj1", @"key2" : @"obj2"};	
 		id obj1 = dict[@"key1"];		
 			
 NSNumber Literal:	
 			
 		NSNumber * number = @'a';//([NSNumber numberWithChar:'a'])			
 		NSNumber * number = @1;//([NSNumber numberWithInt:1])		
 		NSNumber * number 	= @12ll;//([NSNumber numberWithLongLong:12ll])
 		NSNumber * number	= @12ul;//([NSNumber numberWithUnsignedLong:12ul])	
 		NSNumber * number	= @12.3f;//([NSNumber numberWithFloat:12.3f])	
 		NSNumber * number = @YES;//([NSNumber numberWithBool:YES])		

NSSet Literal:		
以上应该大部分集合类的literal都有了，不过类似的NSSet Literal还没有提供。			
我们可以用数组初始化的方式:			


	NSSet * set = [NSSet setWithArray:@[@"yes",@YES]];


当然我们也可以通过自己宏定义来实现：		
	
	 #define $(...)  [[NSSet alloc] initWithObjects:__VA_ARGS__, nil]	
	  //或者			
	 #define $(...)  [NSSet setWithObjects:__VA_ARGS__, nil]		
	 //这样就可以用了:		
	 NSSet *set = $(@"hello",@"world",@"!");		
	
Expressions:@( )	
	
@( )，会动态评估里面的表达式。返回值：计算后的表达式值的对象常量。如	

	[@("hello word") class] //返回类型 NSString		
	[@(77 - 88) class] //返回类型 NSNumber。

3 @encode			
返回一个的char *类型的类型编码,[苹果官方文档有介绍](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html)		
const char * objCType = @encode(float);			
printf("%s",objCType);		


 
4 @defs

我们都知道，Oc的类是在c语言的结构体的基础上建立起来的。@defsd会返回Objective-C类相同的布局，例如下述代码，会得到一个相同结构的结构体。


	struct cStructure	
	{	
    @defs(NSObject);	
	} *cStruct;		
		
不过，如果你现在尝试如上代码，你会得到：@defs is not support on this platform now 或者 @defs is no longer supported in new ABI(Application Binary Interface)的错误。因为大概xcode3.2后我们就已经无法在现在的Objective-C中使用它了。


