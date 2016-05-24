---
layout: post
title: "'super' keyword in Oc"
date: 2015-05-13 20:32:29 +0800
comments: true
categories: iOS

---

我们都知道当使用 self 调用方法时，会从当前类的方法列表中开始找，如果没有，就从父类中再找。当使用 super 时，则从父类的方法列表中开始找，然后调用父类的这个方法。这里 self 代表的是当前调用方法这个类的实例，super 代表父类，但具体是什么，或者是怎么做的呢，我们可以做实验检测一下。

创建一个Oc的类Dog，直接继承于 NSObject，里面有这样一些方法，

```
- (void)doEat:(NSString *)food{}

- (instancetype)init
{
    if (self = [super init]) {
        NSLog(@"%@", [self class]);
        NSLog(@"%@", [super class]);
        NSLog(@"self %@",self);
        //NSLog(@"super %@",super);
    }
    return self;
}
```
在初始化方法里，我们分别输出self和super的类。Dog父类是NSObject，那super输出的会是NSObject么？运行后发现其实二者都是输出Dog。同时，self是一个指针可以看到地址，而当我们把上面注释的那句代码打开，编译器会报一个错误`Use of undeclared identifier 'super'`，可见super跟self不同的是，它不是当前类的一个参数。

在终端中我们使用`clang -rewrite-objc Dog.m`，将上面代码转换成C，看到 - doEat:  转成了下面的C函数

```
	static void _I_Dog_doEat_(Dog * self, SEL _cmd, NSString *food){}
```

在C的方法中可以看到，这个方法里self是作为第一个参数传入的。这也印证了我们的想法，`self 是类的隐藏参数`，在转换成c后，每个类的实例方法中都会有这样一个参数。

初始化中的NSLog方法，转换成C如下

```
	NSLog((NSString *)&__NSConstantStringImpl__var_folders__z_yckwz02x3m1c7_8b6mc8zynm0000gn_T_Dog_ae5b20_mi_0, ((Class (*)(id, SEL))(void *)objc_msgSend)((id)self, sel_registerName("class")));
	NSLog((NSString *)&__NSConstantStringImpl__var_folders__z_yckwz02x3m1c7_8b6mc8zynm0000gn_T_Dog_ae5b20_mi_1, ((Class (*)(__rw_objc_super *, SEL))(void *)objc_msgSendSuper)((__rw_objc_super){(id)self, (id)class_getSuperclass(objc_getClass("Dog"))}, sel_registerName("class")));

```

我们看到 self 和 super 调用方法时，分别使用了如下C函数

```
	id objc_msgSend(id self, SEL op, ...)
	id objc_msgSendSuper(struct objc_super *super, SEL op, ...)
	
```

所以，其实 super 与 self 不同，`不是一个参数，而是一个编译器标识符`。由 super 调用的方法，在编译后被转换成 objc_msgSendSuper，其中第一个参数 [结构体 objc_super](https://developer.apple.com/library/mac/documentation/Cocoa/Reference/ObjCRuntimeRef/#//apple_ref/c/tag/objc_super) 也是编译器在遇到 super 关键字的时候，生成的。定义如下


```
	struct objc_super { id receiver; Class class; };
```

所以编译器在遇到 super 时，大体做了两件事

* 创建 objc_super结构体
* 调用 objc_msgSendSuper，向父类方法发送消息。

