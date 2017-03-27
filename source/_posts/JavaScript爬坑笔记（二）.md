---
layout: '[post.md]'
title: JavaScript爬坑笔记（二）
date: 2016-3-10 09:16:16
tags: ["JavaScript","前端"]
bgimage: http://7xpot0.com1.z0.glb.clouddn.com/public/16-12-5/39797105.jpg
---
接着前面的路继续“爬坑”，希望能将JS中更多典型的问题分享出来。
<!--more-->

### 一、数据类型判断
JavaScript中有5种基本数据类型和一种复杂数据类型。
 - **Null**, **Undefined**
 特殊值
 - **number**
 所有的数值类型，也包括伪数值**NaN**和**Infinity**
 - **boolean**
 ``true``和``false``
 - **string**
 字符类型

其余类型在JavaScript中都为Object类型，比如**function**和**array**。

#### 1. `typeof`方法
`typeof`方法是JavaScript中返回类型的原生方法。

``` js
typeof undefined // "undefined" 
typeof 0    // "number" 
typeof true // "boolean" 
typeof "foo" // "string" 
typeof {} // "object" 
// 注意下面三行
typeof null  // "object" 
typeof function(){} // "function" 
typeof NaN  // "number"

```
- `typeof null == "object"`算是JS语言中的一个内置Bug。
- 按照语言数据类型定义，Functions应该属于Object类型。
-  `typeof NaN == 'number'`略显滑稽，NaN本来是Not A Number的缩写。

此外
``` js
typeof {}  // 'object'
typeof []  // 'object'
typeof new Date  // 'object'
```
`typeof`无法识别具体的Object类型。

简而言之，`typeof`函数适用于基本类型的判断。同时，由于**Uncaught ReferenceError**的存在，不要用`typeof`函数检测变量的存在性。最佳实践是：

``` js
function f(x) {
 if (typeof x == 'function') {
    ... // in case when x is a function
  } else {
    ... // in other cases
  }
}
```

#### 2. `toString`方法
`toString`方法是最为可靠的类型检测手段，它会将当前对象转换为字符串并输出。`toString`属性定义在`Object.prototype`上，因而所有对象都拥有`toString`方法。 但**Array**, **Date**等对象会重写从`Object.prototype`继承来的`toString`， 所以最好用`Object.prototype.toString`来检测类型。

``` js
toString = Object.prototype.toString;
toString.call(new Date); // [object Date]
toString.call(new String); // [object String]
toString.call(Math); // [object Math]
toString.call(3); // [object Number]
toString.call([]); // [object Array]
toString.call({}); // [object Object]

// Since JavaScript 1.8.5
toString.call(undefined); // [object Undefined]
toString.call(null); // [object Null]
```

对于原生对象和内置数据类型，`toString`方法是非常适用的。

``` js
function Animal(name) { 
  this.name = name
}
var animal = new Animal("Goofy")
var class = {}.toString.apply(animal)
alert(class) // [object Object]
```

#### 3. 检测**自定义类型**

检测自定义类型通常使用`instanceof`方法。

``` js
function Animal(name) { 
  this.name = name
}
var animal = new Animal("Goofy")

alert( animal instanceof Animal ) // true
```

#### 4. Duck Typing
目前前端惯例是使用Duck Typing的方式，比如jQuery是这样判断一个Window的：

``` js
isWindow: function(obj){ 
  return obj && typeof obj === 'object' && "setInterval" in obj;
}
```

### 二、使用匿名函数

匿名函数就是指没有分配名称，直接通过函数表达式定义的函数。关于函数表达式和函数声明的区别在[上节](http://littlewin.info/2016/02/28/JavaScript%E7%88%AC%E5%9D%91%E7%AC%94%E8%AE%B0%EF%BC%88%E4%B8%80%EF%BC%89/ "JavaScript爬坑笔记（一）")

``` js
// function expression
var sayHello = function (){
  alert("hi");
};

// in an object
var Person = {
  sayHello: function(){
    alert("hi");
  }
};

// event handler
$("p").click = function(){
  alert("hello");
}; 

// ajax callback from http://api.jquery.com/jQuery.ajax/
$.ajax({
  url: "test.html",
  context: document.body,
  success: function(){
    $(this).addClass("done");
  }
});

// Self-evoking anonymous functions
((function () {
  alert("hi");
})());
```

#### 使用匿名函数

匿名函数有如下好处：

 - 代码简洁性，通常匿名函数会在回调函数和事件处理函数中被调用。
 - 控制作用域，匿名函数可以创建私有的作用域，不会干扰其他作用域。
 - 匿名函数可以便于实现闭包和递归函数。

##### 以jQuery为例

``` js
 (function( window, undefined ) {

  var jQuery = (function() {
  });

  // Expose jQuery to the global object
  window.jQuery = window.$ = jQuery;

  //
})(window);
```

上面是个立即执行的匿名函数，其中定义的变量作用范围都是匿名函数以内，不会影响全局域。而且在函数执行完后，其作用域链也会及时清理。

##### 在递归函数中也常常使用匿名函数

通常会配合`arguments.callee`方法一起使用，避免函数名称变更引起的递归错误。

``` js
function f(n) {
  
  if (n < 2)
    return n;

  if (f.answers[n] != null)
    return f.answers[n];

  f.answers[n] = arguments.callee(n - 1) + arguments.callee(n - 2);

  return f.answers[n];
}

f.answers ={};

var fibo = f;
alert(fibo(10)); //55
alert(fibo(11)); //89
```

这种机制确保即使将函数赋值给另外的变量，也能正常实现函数递归；同时，将每次递归的计算值保存，避免了多次重复的计算过程。

##### 在闭包中的使用

各种专业文献上的"闭包"（closure）定义非常抽象，很难看懂。我的理解是，闭包就是能够读取其他函数内部变量的函数。

由于在Javascript语言中，只有函数内部的子函数才能读取局部变量，因此可以把闭包简单理解成"定义在一个函数内部的函数"。

所以，在本质上，闭包就是将函数内部和函数外部连接起来的一座桥梁。而匿名函数和闭包基本上是相辅相成的。

``` js
$('#foo').click( function() {
  alert('User clicked on "foo."');
});
```

在闭包中使用循环常常出现错误，比如

``` js
for (var i = 1; i <= 10; i++) {
  $('#Div' + i).click(function () {
    alert('#Div' + i + " is kicked");
    return false;
  });
}
```

上述代码会输出10次‘#Div10  is kicked’，而利用匿名函数做下修改，

``` js
for (var i = 1; i <= 10; i++) {
  (function(v){
    $('#Div' + v).click(function () {
      alert('Div' + v + " is kicked");
      return false;
    })
  })(i);
}
```

此时，代码的输出会如我们预期。

更多闭包的细节将在下一章中展开。

##### A quiz

``` js
var foo = {
  bar: function() { return this.baz; },
  baz: 1
};

(function(){
  return typeof arguments[0]();
})(foo.bar);  
```

> 返回值是“undefined”

关注**this**的指向是理解这个过程的重点，开始调用`foo.bar()`时，**this**指向**foo**，而在匿名函数中调用`arguments[0]()`,**this**指向**arguments**，显然`arguments.baz`是**undefined**。

### 闭包

要理解闭包，首先必须理解Javascript特殊的变量作用域。

变量的作用域无非就是两种：全局变量和局部变量。

#### 闭包的定义

> 闭包就是能够读取其他函数内部变量的函数。

闭包这个概念引入了三个作用域链，访问其自己的作用域，访问外部作用域和访问全局作用域。

``` js
function showName (firstName, lastName) {
  var nameIntro = "Your name is ";
  ​function makeFullName () {
    return nameIntro + firstName + " " + lastName;
  }

  return makeFullName ();
}

showName("Michael", "Jackson"); // Your name is Michael Jackson
```

``` js
$(function() {
  var selections = []; 
  $(".inners").click(function() {
    // this closure has access to the selections variable​
    selections.push (this.prop("name")); 
    // update the selections variable in the outer function's scope​
  });
});
```

#### 闭包的使用原则和注意事项

> 闭包可以在外部函数返回后访问其变量

``` js
function celebrityName (firstName) {
  var nameIntro = "This celebrity is ";
  // this inner function has access to the outer function's variables, including the parameter​
  function lastName (theLastName) {
    return nameIntro + firstName + " " + theLastName;
  }
  return lastName;
}
​
​var mjName = celebrityName("Michael"); // At this juncture, the celebrityName outer function has returned.​
​
​// The closure (lastName) is called here after the outer function has returned above​
​// Yet, the closure still has access to the outer function's variables and parameter​
mjName ("Jackson"); // This celebrity is Michael Jackson
```

**celebrityName**返回后，实际上相当于在全局域上创建了一个函数。

``` js
mjName = function lastName(theLastName) {
  var firstName = "Michael";
  return nameIntro + firstName + " " + theLastName;
}
```

> 闭包储存了外部函数变量的引用

``` js
function celebrityID () {
  var celebrityID = 999;
  // We are returning an object with some inner functions​
  // All the inner functions have access to the outer function's variables​
  return {
    getID: function ()  {
      // This inner function will return the UPDATED celebrityID variable​
      // It will return the current value of celebrityID, even after the changeTheID function changes it​
      return celebrityID;
    },
    setID: function (theNewID)  {
      // This inner function will change the outer function's variable anytime​
      celebrityID = theNewID;
    }
  }
}
var mjID = celebrityID(); // At this juncture, the celebrityID outer function has returned
mjID.getID(); // 999
mjID.setID(567); // Changes the outer function's variable
mjID.getID(); // 567: It returns the updated celebrityId variable
```

> 闭包失效

参考[在闭包中的使用](#在闭包中的使用 "在闭包中的使用")，返回闭包时牢记的一点就是：返回函数不要引用任何循环变量，或者后续会发生变化的变量。如果一定要引用循环变量怎么办？方法是再创建一个函数，用该函数的参数绑定循环变量当前的值，无论该循环变量后续如何更改，已绑定到函数参数的值不变。

本文完，以上。