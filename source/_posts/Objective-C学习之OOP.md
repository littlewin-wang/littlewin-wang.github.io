layout: '[post.md]'
title: Objective-C学习之OOP
date: 2016-01-01 15:02:41
tags: ["Objective-C","iOS"]
---

*最近元旦无事在家休息，将这半年来所学记录于此。以后无论事情再多，也要力争在短周期内通过blog上update自己的学习记录和心得，一来督促自己，二来方便日后查阅。*

开发iOS应用需要使用Objective-C语言和Cocoa Touch框架。Objective-C源自C语言，是C语言的扩展。Cocoa Touch框架则是一个Objective-C类的集合。
<!--more--> 
##### 类和对象

Objective-C的类与其他OOP语言基本类似，是一种表示对象类型的结构体。对象通过它隶属的类来获取自身的各种信息，尤其是执行每个操作所要运行的代码。
`@interface`指令将类所提供的数据结构成员告知Objective-C编译器
```
  @interface NewClass : NSObject  
  {
  //fill in instance variable
  }
  //fill in method declaration
  @end
```
`@implementation`部分提供各个数据接口的功能代码
```
  @implementation NewClass
  //fill in implementation
  @end
```
在Objective-C中，几乎所有的类都是NSObject类的子类，因此，可以将这些类也看为对象，对象能够接收的消息列表是保存在它所对应的类中。每一个对象都有一个名为 isa 的指针，指向该对象的类。

NSObject 就是一个包含 isa 指针的结构体，它的定义如下所示
```
  @interface NSObject  { 
      Class isa; 
  } 
```

类是元类(metaclass)的实例，元类的isa指针指向根元类（root metaclass），根元类 (root metaclass) 本身的 isa 指针指向自己，形成一个闭环的继承路径。
![](http://7xpot0.com1.z0.glb.clouddn.com/16-1-1/48692569.jpg)
对象是类的实例，其本身是分配在内存中的一块数据。对象是有生命周期的，首先被创建，然后接受消息，最后在不需要时被释放。向类发alloc消息，会在内存中创建对象并返回对象指针。对新创建的对象，必须先向其发送一个初始化消息，然后它才能正常工作。
```
  NewClass *newClass = [NewClass alloc];
  [newClass init];
```
在对象完成使命后，可以要求程序释放对象
```
  newClass=nil;
```
##### 实例变量和属性
```
  @interface NewClass : NSObject
  {
      NSString *newString;
  }
  @property (nonatomic, retain) NSString *newString;
  @end
```
声明类的实例变量时，需要将相应的声明写在花括号里，并紧跟在类声明的后面。为类添加实例变量后，还要定义存取方法（accessor method），否则无法访问对象的实例变量。
```
  - (void)setNewString:(NSString *)str;
  - (NSString *)newString;
```
```
  - (void)setNewString:(NSString *)str
  {
      _newString = str;
  }

  - (NSString *)newString
  {
      return _newString;
  }
```
`@Property`是声明属性的语法，它可以快速方便的为实例变量创建存取器，并允许我们通过点语法使用存取器。

它们的区别有以下几点
- 直接操作实例变量明显更快。因为不需要经过OC的方法分发，直接从内存中读取。
- 直接写实例变量会绕过属性的内存管理声明，假如你的属性声明为 `copy`，直接写入它的实例变量将不会产生copy操作。
- 直接操作实例变量，不会发送KVO通知。
- 操作属性方便调试，你可以给get/set方法加断点，就能知道谁在什么时候进行了读写操作。

##### 类方法和实例方法
类方法的作用通常是创建对象，或者获取类的某些全局属性。**类方法不会作用在在对象上，也不能存取实例变量**。
```
  + (void)toNewString
  {
      NSLog(@"this is a class method of NewString");
  }
```
实例方法则用来操作类的对象（对象有时也可称为类的一个实例）例如上述存取方法。

在创建一个NewClass对象时，首先向NewClass类发送`alloc`（类方法）消息，然后向使用alloc方法创建的对象newClass发送`init`（实例方法）消息。

##### 继承与复合
继承（is-a关系）
```
  @interface NewClass : NSObject 
```
Objective-c不支持多继承。
Objective-c支持重写方法，同时仍然调用超类的实现方式，这时需要使用`super`关键词来调用父类方法
```
  [super init]；
```
复合（has-a关系）
在objective-c中，复合是通过包含作为实例变量的对象指针实现的，如NewClass可以包含newString和oldString。
```
  @interface NewClass : NSObject 
  { 
      NSString *newString; 
      NSString *oldString;
  }
```
严格来讲，只有对象间的组合才能叫复合，而基本类型都被认为是对象的一部分。
##### 一些没提到的关键字
- `self` 与 `super`
`self` 是一个隐含的指针，指向接收消息的对象的指针。
`super`既不是参数也不是实例变量。向`super`发消息时，实际上是在请求Objective-C向该类的超类发送消息。如果超类中没有定义的消息，Objective-C将按照继承的通常规则在继承链中查找。例如`[super init] `用于向超类的`init`方法发送消息。
- `id`
`id`是指向Objective-C类对象的指针，它可以声明为任何类对象的指针，当在Objective-C中使用id时，编译器会假定你知道，`id`指向哪个类的对象。与`void*`是不同的是，`void*`编译器不知道也不假定指向任何类型的指针。
- `nil`
定义为一个常量，如果一个指针的值为`nil`,代表这个指针没有指向任何对象。