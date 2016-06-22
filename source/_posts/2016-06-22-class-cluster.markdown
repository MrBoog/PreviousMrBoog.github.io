---
layout: post
title: "Class cluster"
date: 2016-06-22 20:29:41 +0800
comments: true
categories: iOS

---



Class cluster，类簇，这是苹果对抽象工厂模式一种应用，在苹果的Framework中使用广泛，是定义了相同的接口，并提供相同功能的一组类的集合。类簇会有一个公共抽象超类，比如NSString,NSArray,NSNumber,NSDictionary等等，来完成实例化一个具体的私有子类。对于使用者来说，只需要知道有公共超类各种API的作用就可以，不需要知道，超类背后子类的实际实现。

##### self = [super init]

我们在初始化的时候，为什么super init调用过后，会赋值给self呢？因为系统有很多时候使用了类簇，所以这样做防止产生的私有类和self不匹配。`实例的isa指针终究应该指向这个私有类，而不是当前的超类。`	
这样在发送消息的时候，self才能找到真正的需要执行的私有类的方法。

```
id objc_msgSend(id self, SEL op, ...)
```

##### superclass

```
NSArray *array = @[];
NSLog(@"%@",array.superclass);               //NSArray
NSLog(@"%@",NSStringFromClass(array.class)); //__NSArrayI
```

对于类簇中的超类，比如NSArray，调用superclass就会发现，他的superclass才是真正的NSArray。

##### isMemberOfClass

```
NSLog(@"%d",[array isMemberOfClass:[NSArray class]]); //0
NSLog(@"%d",[array isKindOfClass:[NSArray class]]);   //1
```

通过上面的输出同样证明了苹果的类簇利用了多态，通过子类化公共类来实现。
私有子类的实现，隐藏在簇的内部，不能被直接调用。这样公共类在使用中，实际上存放在内存中的是隐藏在类簇中的某个私有实例。	
这样的好处很明显，针对类簇，调用者只需要简单的使用超类提供的API即可。这样NSNumber根据数据不同的类型，而有多种不同的子类化的NSNumber，NSArray根据存储策略不同也有不同的NSArray，而调用者却可以根本不用注意这些。