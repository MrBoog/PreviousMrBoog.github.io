---
layout: post
title: "random in iOS"
date: 2014-06-09 23:10:41 +0800
comments: true
categories: iOS

---


生成随机数		
arc4random(void)//不够平衡	
推荐		
arc4random_uniform(u_int32_t)	

生成 [0 , n )，之间的随机数：

```
NSUInteger foo = arc4random_uniform(n);
```

生成 [minimum , maximum]，之间的随机数：	

```
NSUInteger foo = arc4random_uniform((maximum - minimum) + 1) + minimum;
```

除整数之外，产生(0 , 1)，之前的随机数：		

```
//先种随机种子
srand48(time(0));
double foo = drand48();
```

延伸一下，随机取数组中的元素：

```
//arc4random_uniform(n) 随机范围是[0，n），可直接用于数组下标
if ([array count] > 0) {
  id obj = array[arc4random_uniform([array count])];
}
```

没有使用NSSet，却想打乱顺序：

```
NSMutableArray *mutableArray = [NSMutableArray arrayWithArray:array];
NSUInteger count = [mutableArray count];
// See http://en.wikipedia.org/wiki/Fisher–Yates_shuffle
if (count > 1) {
  for (NSUInteger i = count - 1; i > 0; --i) {
      [mutableArray exchangeObjectAtIndex:i withObjectAtIndex:arc4random_uniform((int32_t)(i + 1))];
  }
}

NSArray *randomArray = [NSArray arrayWithArray:mutableArray];
```
