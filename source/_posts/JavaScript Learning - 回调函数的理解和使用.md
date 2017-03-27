---
layout: '[post.md]'
title: JavaScript Learning - 回调函数的理解和使用
date: 2015-12-06 14:35:20
tags: ["JavaScript","前端"]
---
回调函数主要体现在函数式编程的编程方式，即使用函数作为参数。函数式编程在编程的进阶领域会有充分的体现，需要进行一段时间的学习和练习。
所幸函数式编程在很多语言规范里都有明确的定义和解释，而函数式编程中主要用到的方法就是回调函数。回调函数的在进阶的JavaScript中常有用到，但其实有一些理解之后，用法和传入普通参数并无太大差异。
<!--more-->

### 1. 回调函数
> 回调函数，是将一个函数作为参数传入另一个函数，并且在这个函数中进行调用的编程方式。回调函数本身是一种编程模式，因此使用回调函数的方式也被称为回调模式。

先来一个简单的jQuery例子
``` js
  $("#btn_1").click(function() {
    alert("Btn 1 Clicked");
  });
```
匿名函数作为参数传入click函数，click函数执行时会执行此函数，这种例子在JavaScript和jQuery种经常被用到。
``` js
  var friends = ["Mike", "Stacy", "Andy", "Rick"];
  friends.forEach(function (eachName, index){
    console.log(index + 1 + ". " + eachName); // 1. Mike, 2. Stacy, 3. Andy, 4. Rick​
  });
```
以上两个例子都是直接传递匿名函数去别的函数进行调用的回调方式，在更复杂的例子之前，有必要先了解一下回调函数的执行原理。

### 2. 工作原理
当使用回调函数时，通过参数传递的是函数的定义，而不是函数的执行功能。简而言之，我们传递的是`callback`，而不是`callback()`。
因此作为函数定义，回调函数可以在另外一个函数中的任何执行时刻被调用。

此外，通过参数传递的回调函数会在另一个函数体内被调用执行，类似于回调函数的定义位于另一个函数内，因此，回调函数本质上是闭包。而闭包可以访问包含函数的作用范围，因此回调函数可以访问包含函数的变量，即使这个变量来自全局作用域。

### 3. 实现回调函数
回调函数的使用不算太难，但在实现方面需要注意一些原则。

- **使用命名或匿名函数作为回调**

上面两个例子使用匿名函数作为回调，声明命名函数，并传递函数名作为参数也可以实现回调。

``` js
  var allUserData = [];

  function logStuff (userData) {
    if (typeof userData === "string")
    {
      console.log(userData);
    }
    else if (typeof userData === "object")
    {
      for (var item in userData) {
        console.log(item + ": " + userData[item]);
      }
    }
  }

  ​function getInput (options, callback) {
    allUserData.push (options);
    callback (options);
  }
  
  getInput ({name:"Rich", speciality:"JavaScript"}, logStuff);
  // name: Rich​
​  // speciality: JavaScript
```

- **在回调函数中使用参数**

回调函数在执行过程中是作为一个常规的函数，因此可以向其传入参数。可以将任何包含函数的属性（或全局属性）传入回调函数的参数。

``` js
  // 全局变量
  var generalLastName = "Clinton";
  
  ​function getInput (options, callback) {
    allUserData.push (options);
    ​// 向回调函数传入全局变量generalLastName和局部变量options
    callback (generalLastName, options);
  }
```

- **先定义，再调用**

在执行回调函数前先对其做一些检查是很较好的编程习惯
``` js
  var generalLastName = "Clinton";
  
  ​function getInput (options, callback) {
    allUserData.push (options);

    if(typeof callback === "function") {
      callback (generalLastName, options);
    }
  }
```
如果没有检查，并且传入参数不当，运行过程中会导致程序出错。

- **回调函数使用this对象**

当回调函数使用`this`对象时，需要注意`this`指代的上下文环境。之前在讨论`apply`和`call`时有个例子

``` js
  var clientData = {
    id: 094545,
    fullName: "Not Set",
    setUserName: function (firstName, lastName) {
      this.fullName = firstName + " " + lastName;
    }
  }

  function getUserInput (firstName, lastName, callback) {
    callback(firstName, lastName);
  }

  getUserInput ("Barack", "Obama", clientData.setUserName);
  console.log (clientData.fullName); // "Not Set"
```
下面代码将不会修改`clientData`的`fullName`属性，而是将操作执行到全局作用域，因为`getUserInput`函数定义在了全局域，如果要修改回调函数的执行上下文，就要用到`apply`和`call`方法。
重新定义`getUserInput`函数
``` js
  function getUserInput (firstName, lastName, callback, callbackObj) {
    callback.apply (callbackObj, [firstName, lastName]);
  }
  
  getUserInput ("Barack", "Obama", clientData.setUserName, clientData);
  console.log (clientData.fullName); // Barack Obama​
```
- **多个回调函数**

就像JavaScript中函数可以传入多个参数一样，函数可以调用多个回调函数。下面给一个jQuery中的ajax例子。

``` js
  function successCallback() {
    // Do stuff before send​
  }

  ​function successCallback() {
    // Do stuff if success message received​
  }

  ​function completeCallback() {
    // Do stuff upon completion​
  }

  ​function errorCallback() {
    // Do stuff if error received​
  }

  $.ajax({
    url: "http://fiddle.jshell.net/favicon.png",
    success: successCallback,
    complete: completeCallback,
    error: errorCallback
  });
```

- **"Call Hell"问题和解决方法**

相信每一个JS程序员都曾被或者正在被回调地狱所折磨，特别是写过Nodejs代码的程序员。
``` js
  asyncFun1(function(err, a) {
    // do something with a in function 1
    asyncFun2(function(err, b) {
      // do something with b in function 2
      asyncFun3(function(err, c) {
          // do something with c in function 3
      });
    });
  });
```

JS的后续传递风格（回调）是这门语言的优点也是这门语言的缺点，优点之一是我们可以很轻易的写出异步执行的代码，而缺点也是由异步引起的，当太多的异步步骤需要一步一步执行，或者一个函数里有太多的异步操作，这时候就会产生大量嵌套的回调，使代码嵌套太深而难以阅读和维护，即所谓的回调地狱。

最基本的解决此问题的两种思路一是使用具名函数，并保持代码层级不要太深。

``` js
  function fun3(err, c) {
    // do something with a in function 3
  }
  function fun2(err, b) {
    // do something with b in function 2 
    asyncFun3(fun3);
  }
  function fun1(err, a) {
    // do something with a in function 1
    asyncFun2(fun2);
  }
  asyncFun1(fun1);
```
二是做好模块划分，保持模块功能独立性。
进阶一些的做法有ES6推行的Promise和Generator、async等辅助库等，这个以后再介绍吧。

### 4. 实际用例

- 资源加载：动态加载js文件后执行回调，加载iframe后执行回调，ajax操作回调，图片加载完成执行回调，AJAX等等。

- DOM事件及Node.js事件基于回调机制(Node.js回调可能会出现多层回调嵌套的问题)。

- setTimeout的延迟时间为0，这个hack经常被用到，settimeout调用的函数其实就是一个callback的体现。

- 链式调用：链式调用的时候，在赋值器(setter)方法中(或者本身没有返回值的方法中)很容易实现链式调用，而取值器(getter)相对来说不好实现链式调用，因为你需要取值器返回你需要的数据而不是this指针，如果要实现链式方法，可以用回调函数来实现。

- setTimeout、setInterval的函数调用得到其返回值。由于两个函数都是异步的，即：他们的调用时序和程序的主流程是相对独立的，所以没有办法在主体里面等待它们的返回值，它们被打开的时候程序也不会停下来等待，否则也就失去了setTimeout及setInterval的意义了，所以用return已经没有意义，只能使用callback。callback的意义在于将timer执行的结果通知给代理函数进行及时处理。