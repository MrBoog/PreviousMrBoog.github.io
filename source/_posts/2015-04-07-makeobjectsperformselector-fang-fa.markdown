---
layout: post
title: "makeObjectsPerformSelector 方法"
date: 2015-04-07 11:25:28 +0800
comments: true
categories: iOS

---

### makeObjectsPerformSelector 方法

当我们需要对NSArray 或者 NSSet里面的对象，执行同一个操作的时候，我们首先想到的可能是 for 循环。其实对于NSArray 和 NSSet 都有个一简便的方法:

`makeObjectsPerformSelector:`，数组里面的元素同时执行某个 selector。`makeObjectsPerformSelector:withObject:`可以带一个参数。例如移除某个视图的所有subview，只要一句话：
 
 ```
 [self.subviews makeObjectsPerformSelector:@selector(removeFromSuperview)];
 ```
 
 ---
 
 既然提到了循环，我们就总结一下，数组中其他几种常用的循环方式：
 
#### for in 
```
for (id obj in array){
}
```
 
#### Block Enumeration
```
[array enumerateObjectsUsingBlock: ^(id obj, NSUInteger idx, BOOL *stop) {
    
}];

//Concurrent:
[array enumerateObjectsWithOptions: NSEnumerationConcurrent usingBlock: ^(id obj, NSUInteger idx, BOOL *stop) {
    
}];
``` 

#### GCD APPLY
```
dispatch_queue_t queue = dispatch_get_global_queue(0, 0);

//sync:
dispatch_apply([array count], queue, ^(size_t idx) {
    id obj = [array objectAtIndex: idx];
    
});

//Async:
dispatch_async(queue, ^(void) {
    dispatch_apply([array count], queue, ^(size_t idx) {
        id obj = [array objectAtIndex: idx];
        
    });
});
``` 
 
 
