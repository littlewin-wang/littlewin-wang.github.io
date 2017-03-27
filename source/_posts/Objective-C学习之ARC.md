layout: '[post.md]'
title: Objective-C学习之ARC
date: 2016-01-03 21:27:33
tags: ["Objective-C","iOS"]
---

内存管理在程序开发中有着至关重要的作用，基本上每个C系的程序员都会遇到与内存相关的错误，这种错误是灾难性的。Java或脚本语言的内存管理是自动进行的，就像父母会给孩子打扫房间一样。对于C系程序开发，我们必须合理分配内存，在内存结束使用后释放它们，避免内存泄漏（leak memory）。

<!--more--> 

##### iOS内存管理机制
iOS开发中，内存中的对象主要有两类，一类是值类型，比如`int`、`float`、`struct`等基本数据类型；另一类是引用类型，也就是继承自NSObject类的所有的OC对象。

值类型会被放入栈中，他们依次紧密排列，在内存中占有一块连续的内存空间，遵循先进后出的原则。栈内存中的值类型系统会随着其生命周期自动管理。

引用类型会被放到堆中，当给对象分配内存空间时，会随机的从内存当中开辟空间，设备内存使有限的，需要我们在释放对象后将其占据的内存归还堆空间，使之可以再被利用。

在Apple引入自动引用计数（Automatic Reference Counting, ARC）之前，应用只能通过手动引用计数（Mannul Reference Counting, MRC）来管理内存，具体的原则也很简单
>如果需要持有一个对象，那么对其发送`retain`如果之后不再使用该对象，那么需要对其发送`release`（或者`autorealse`）。每一次对`retain`，`alloc`或者`new`的调用，需要对应一次`release`或`autorealse`调用。

大多数情况下，可以依靠ARC来自动的完成需要的内存管理功能。但是，理解其工作原理，可以便于我们调试应用或对一些场景做特殊处理。

##### 指针变量与对象所有权
>当某个方法（或函数）有一个指向某个对象的局部变量时，可以称该变量拥有该变量所指向的对象。
>当某个对象有一个指向其他对象的实例变量时，可以称该对象拥有该实例变量所指向的对象。

例如
![](http://7xpot0.com1.z0.glb.clouddn.com/16-1-3/76239862.jpg)
```
  @interface NewClass : NSObject
  {
      NSString *newString;
  }
  @property (nonatomic, retain) NSString *newString;
  @end
```
```
  NewClass *newClass = [NewClass alloc];
  [newClass init];
  newClass.newString = @"littlewin"
```
`newClass`对象拥有其实例变量`newString`所指向的对象`@"littlewin"`。
 
对象的所有权可以帮助我们决定释放对象和回收内存的时机。
> 如果某个对象没有拥有者，就应该将其释放掉。
如果某个对象有一个或多个拥有者，就必须保留不能释放。

下列情况会使对象失去拥有者：
- 当指针变量指向另一个对象时
![](http://7xpot0.com1.z0.glb.clouddn.com/16-1-3/11277658.jpg)