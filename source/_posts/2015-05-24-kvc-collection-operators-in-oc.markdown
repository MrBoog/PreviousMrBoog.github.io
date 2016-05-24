---
layout: post
title: "KVC Collection Operators in Oc "
date: 2015-05-24 16:04:31 +0800
comments: true
categories: iOS

---

[KVC的集合运算符](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/KeyValueCoding/Articles/CollectionOperators.html#//apple_ref/doc/uid/20002176-BAJEAIEE)是一些特定的，以@符号作为前缀的字符串，可以作为key path参数传入**valueForKeyPath:**方法中，以完成一些特定的便捷操作，如查找最大值，最小值，平均值等等。

> 集合运算符，除了支持常见的集合NSArray、NSSet、NSDictionary以外，NSPointerArray、NSHashTable、NSMapTable应该也是支持的。

例如，下面是平时我们计算一个数组里Product的price属性的平均值的方法

```
double total = 0.0;
for (Product *product in productsArray) {
  total += [product.price doubleValue];
}
double average = total / [productsArray count];
```

使用KVC的集合运算符，可以用一句话来改写

```
NSNumber *priceAverage = [productsArray valueForKeyPath:@"@avg.price"];

```

---

按照苹果的文档，KVC集合运算符被分为下面三类

### 1.Simple Collection Operators

为集合提供一些便捷运算。返回的是strings, number, 或者 dates。	
共有5种操作：

* @avg : 返回集合中对象某个属性平均值，NSNumber类型

* @count : 返回集合中对象总数，NSNumber类型

* @max、@min : 返回集合中对象某个属性最大、小值，实际是调用**compare: **方法比较，类型根据属性而定

* @sum : 返回集合中对象某个属性之和，每个属性都被转换为double后进行计算，NSNumber类型


> 如果以上集合中的对象不是自定义对象，而是Cocoa提供的类比如NSNumber，我们可以self代替key path，像这样使用：[@[@(1), @(2), @(3)] valueForKeyPath:@"@max.self"]

---

### 2.Object Operators

对象操作符是很有用的操作，获取集合中对象某一个属性组成的数组。返回NSArray。

* @distinctUnionOfObjects : 返回结果去重。

* @unionOfObjects : 返会结果不去重。

例如 获取集合下所有Product的name属性的集合

```
NSArray *names = [productsArray valueForKeyPath:@"@unionOfObjects.name"];

//去重
NSArray *distinctNames = [productsArray valueForKeyPath:@"@distinctUnionOfObjects.name"];
```

---

### 3.Array and Set Operators

类似于Object Operators的作用。不过是作用于嵌套集合。从上面的例子来说，就是支持从包含多个productsArray的数组中获取Product的name属性的集合。返回的是一个数组或者集合。

* @distinctUnionOfArrays : 去重，返回NSArray

* @unionOfArrays : 不去重，返回NSArray

* @distinctUnionOfSets : 去重，返回NSSet。NSSet不能包含重复数据，所以只有这一个方法。

```
NSArray *distinctNames = [@[productsArray1,productsArray2] valueForKeyPath:@"@distinctUnionOfArrays.name"];
```