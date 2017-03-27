---
layout: '[post.md]'
title: JavaScript爬坑笔记（一）
date: 2016-2-28 10:26:10
tags: ["JavaScript","前端"]
---
“坑”也就是“陷阱”。由于JavaScript“弱语言”的性质，使得其在使用过程中异常的宽松灵活，但也极为容易“中招”。这些坑往往隐藏着，只有亲自尝试并爬坑，才能在学习与应用的道路上走的一帆风顺。
平时积累一些觉得比较有意思或是比较有难度的JavaScript题目，顺便附上理解和心得。既是为了将来的面试做准备，也增强印象以防自己掉进坑里。
<!--more-->

### 一、 函数声明的预解析

#### 1.1 函数的声明方式

定义一个函数有两种方式：

``` js
foo() {};                   // 函数声明
var foo = function() {};    // 函数表达式
```

不同之处 - 
 1. 函数表达式后面加括号可以直接执行
 2. 函数声明会提前预解析

#### 1.2 预解析
下面这段代码的输出结果是什么？

``` js
function test(){
  console.log(a);
  console.log(foo());
  var a =1;
  function foo(){
    return 2;
  }
}

test();
```
> undefined和2

换种写法
``` js
function test(){
  console.log(a);
  console.log(foo());
  var a =1;
  var foo = function(){
    return 2;
  }
}

test();
```
> undefined
> 第二条打印会报错 - TypeError: foo is not a function

#### 1.3 变量提升
> JavaScript解析器会在自身作用域内将变量和函数声明提前(hoist)。

也就是说，上面的例子其实被解析器理解解析成了以下形式：

``` js
function test(){
  var a;
  var foo;
  console.log(a);
  console.log(foo());
  a = 1;
  foo = function(){
    return2;
  }
}
test();
```

这样也就可以解释，为什么在函数表达式之前调用函数，会返回错误了，因为它还没有被赋值，只是一个未定义变量，当然无法被执行。

变量声明被提到最前（所以不会报出变量不存在的错误），但赋值没有被提前，所以第一句的输出结果是undefined。

#### 1.4 可能的问题

由于函数声明会被预解析，所以不要使用此种方法来声明不同函数。尝试猜想下面例子的输出结果：

``` js
if(true){
  function aaa(){
    alert('1');
  }  
}
else{
  function aaa(){
    alert('2');
  }
}

aaa();
```

与我们预想的不同，该段代码弹出的是“2”.这是因为两个函数声明在if语句被执行之前就被预解析了，所以if语句根本没有用，调用aaa()的时候直接执行了下面的函数。

#### 1.5 最佳实践
 - 变量和函数声明一定要放在作用域/函数的开头。
 - 函数表达式在赋值后再去使用。
  
  
### 二、作用域

看一下下面的代码：

``` js
(function(){
  var a = b = 5;
})();
console.log(b);
```

> 5

这个问题考查的要点是两个不同的作用域，'a'被var声明成了一个局部变量，但是'b'实际上没有被定义，所以它是一个全局变量。
如果你选择了strict mode，上面的代码就会报Uncaught ReferenceError，因为b没有被定义，它可以帮你检查出代码的一些问题：

``` js
(function(){
  'use strict';
  var a = window.b = 5;
})();
console.log(b);
```

### 三、继承问题

JavaScript是基于对象和实例的语言，它没有类的概念。所以，要想实现继承，必须熟练应用其`prototype`机制、函数构造以及`call`，`apply`等特殊方法。

可以简单的把`prototype`看做是一个模板，新创建的自定义对象都是这个模板（`prototype`）的一个拷贝（实际上不是拷贝而是链接，只不过这种链接是不可见，新实例化的对象内部有一个看不见的`__proto__`指针，指向原型对象）。

#### 3.1 原型链继承

为了让子类继承父类的属性（也包括方法），首先需要定义一个构造函数。然后，将父类的新实例赋值给构造函数的原型。

``` js
function Super() {
  this.belong = "human";
}

function Sub(age) {
  this.age = age;
}

Sub.prototype = new Super();

```

简单的说法就是拿父类的实例来充当子类的原型对象。

优点是实现方式相当简单，易于实现。

缺点之一是来自原型对象的引用属性是所有实例共享的，会导致覆盖的属性篡改；二是创建子类实例时，无法向父类构造函数传参。

#### 3.2 借用构造函数

原型链够简单，但缺点也很明显。于是出现了借用构造函数的方式。

``` js
function Super(val){
  this.val = val;
  this.arr = [1];
  
  this.fun = function(){
    // ...
  }
}

function Sub(val){
  Super.call(this, val);   // 核心
  // ...
}

var sub1 = new Sub(1);
var sub2 = new Sub(2);
sub1.arr.push(2);
alert(sub1.val);    // 1
alert(sub2.val);    // 2

alert(sub1.arr);    // 1, 2
alert(sub2.arr);    // 1

alert(sub1.fun === sub2.fun);   // false
```

借父类的构造函数来增强子类实例，等于是把父类的实例属性复制了一份给子类实例装上了（完全没有用到原型）。

优点：
- 解决了子类实例共享父类引用属性的问题
- 创建子类实例时，可以向父类构造函数传参

缺点：
- 无法实现函数复用，每个子类实例都持有一个新的fun函数，太多了就会影响性能，内存溢出。


#### 3.3 组合继承

``` js
function Super(){
  // 只在此处声明基本属性和引用属性
  this.val = 1;
  this.arr = [1];
}
//  在此处声明函数
Super.prototype.fun1 = function(){};
Super.prototype.fun2 = function(){};
//Super.prototype.fun3...
function Sub(){
  Super.call(this);   // 核心
  // ...
}

Sub.prototype = new Super();    // 核心

var sub1 = new Sub(1);
var sub2 = new Sub(2);
alert(sub1.fun === sub2.fun);   // true
```

把实例函数都放在原型对象上，以实现函数复用。同时还要保留借用构造函数方式的优点，通过`Super.call(this)`;继承父类的基本属性和引用属性并保留能传参的优点；通过`Sub.prototype = new Super()`;继承父类函数，实现函数复用。

优点：
- 不存在引用属性共享问题
- 可传参，并且函数可复用

缺点：
- 子类原型上有一份多余的父类实例属性，因为父类构造函数被调用了两次，生成了两份，而子类实例上的那一份屏蔽了子类原型上的。。。

#### 3.4 寄生组合继承

``` js
function beget(obj){   // 其实只需要原型的一份copy
  var F = function(){};
  F.prototype = obj;
  return new F();
}
function Super(){
  // 只在此处声明基本属性和引用属性
  this.val = 1;
  this.arr = [1];
}
//  在此处声明函数
Super.prototype.fun1 = function(){};
Super.prototype.fun2 = function(){};
//Super.prototype.fun3...
function Sub(){
  Super.call(this);   // 核心
  // ...
}
var proto = beget(Super.prototype); // 核心
proto.constructor = Sub;            // 核心
Sub.prototype = proto;              // 核心

var sub = new Sub();
alert(sub.val);
alert(sub.arr);
```

用beget(Super.prototype)切掉了原型对象上多余的那份父类实例属性。
暂时看起来是集合了上述几种方法的优点，使用起来会稍微麻烦一点。

#### 3.5 关于`beget`函数

``` js
function beget(obj){   // 其实只需要原型的一份copy
  var F = function(){};
  F.prototype = obj;
  return new F();
}
```

beget函数的意义在于不必为了指定子类型的原型而调用超类型的构造函数，我们所需要的无非就是超类型原型的一个副本而已。
这个函数其实在IE9+、Firefox 4+、Safari 5+、Opera 12+和Chrome中就是`Object.create()`方法。

此外JS中还有原型式继承和寄生式继承，它们也有明显的缺点，适用于特点的场景，使用不频繁，在此不赘述了。