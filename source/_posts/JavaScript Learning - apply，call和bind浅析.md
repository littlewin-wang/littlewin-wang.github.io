layout: '[post.md]'
title: JavaScript Learning - apply，call和bind浅析
date: 2015-11-29 17:06:33
tags: ["JavaScript","前端"]
---
对于Javascript开发者而言，`apply`，`call`和`bind`这三个方法极其重要，因此需要对其形成更深刻的理解。而现实中感觉这三个方法的使用深度较浅，在此稍作深入学习加以记录吧。
<!--more--> 

### 1. 共性
`apply`，`call`和`bind`方法的共同点是它们的第一个参数都是想将方法赋予给的对象。

### 2. apply和call
`apply`和`call`将方法绑定到具体的对象并执行，它们的区别在于第二个参数 - `call`需要把参数按顺序传递进去，而`apply`则是把参数放在数组里。
``` js
  func.call(this, arg1, arg2);
  func.apply(this, [arg1, arg2])
```
#### 2.1 绑定对象
先来一个简单的例子
``` js
  // 全局变量​
  var avgScore = "global avgScore";
  // 全局函数
  function avg (arrayOfScores) {
    // 统计score值得总和
    var sumOfScores = arrayOfScores.reduce (function (prev, cur, index, array) {
      return prev + cur;
    });
    // 这里this会绑定到全局对象，除非使用apply或call
    this.avgScore = sumOfScores / arrayOfScores.length;
  }

  var gameController = {
    scores :[20, 34, 55, 46, 77],
    avgScore:null​
  }

  avg (gameController.scores);
  console.log (window.avgScore); // 46.4​
  console.log (gameController.avgScore); // null​

  // 重置全局变量​
  avgScore = "global avgScore";
  // 使用call绑定对象gameController
  avg.call (gameController, gameController.scores);
  console.log (window.avgScore); //global avgScore
  console.log (gameController.avgScore); // 46.4​
```
再来看一个回调函数的例子
``` js
  var clientData = {
    id: 094545,
    fullName: "Not Set",
    setUserName: function (firstName, lastName) {
      this.fullName = firstName + " " + lastName;
    }
  }

  function getUserInput (firstName, lastName, callback, callbackObj) {
    callback.apply (callbackObj, [firstName, lastName]);
  }
```
`apply`将方法绑定给`callbackObj`对象，不会去影响全局对象。
``` js
  getUserInput ("Barack", "Obama", clientData.setUserName, clientData);
  console.log (clientData.fullName); // Barack Obama​
```
`this`对象在Javascript代码中意义重大，`apply`和`call`可以方便的让我们改变函数执行的`this`环境。在JavaScript代码实现中，这两者将会经常被用到。

#### 2.2 借用函数
`apply`和`call`一般被用来借用函数，常见的用法如下，
- 借用Array数组方法

JavaScript编程中经常会遇到一些伪数组（类似数组的对象，不含负键值），这些伪数组往往需要调用很多原生的数组方法，此时往往可以利用`Array.prototype`中的原型方法加`apply`、`call`方法来实现。

``` js
  // 伪数组对象
  var anArrayLikeObj = {0:"Martin", 1:78, 2:67, 3:["Letta", "Marieta", "Pauline"], length:4 };
  
  // slice
  var newArray = Array.prototype.slice.call (anArrayLikeObj, 0);
  console.log (newArray); // ["Martin", 78, 67, Array[3]]​

  // indexOf
  console.log (anArrayLikeObj.indexOf ("Martin") === -1 ? false : true); // Error: Object has no method 'indexOf
  console.log (Array.prototype.indexOf.call (anArrayLikeObj, "Martin") === -1 ? false : true); // true

  // Reverse
  console.log (Array.prototype.reverse.call (anArrayLikeObj)); // {0: Array[3], 1: 67, 2: 78, 3: "Martin", length: 4}​

  // Pop​
  console.log (Array.prototype.pop.call (anArrayLikeObj));
  console.log (anArrayLikeObj); // {0: Array[3], 1: 67, 2: 78, length: 3}​

  // Push
  console.log (Array.prototype.push.call (anArrayLikeObj, "Jackie"));
  console.log (anArrayLikeObj); // {0: Array[3], 1: 67, 2: 78, 3: "Jackie", length: 4}​
```
`arguments `是JavaScript中函数的属性，用来表示参数，这就是一个伪数组，通常我们利用`apply`和`call`方法调用数组方法检验每一个参数。

``` js
  function transitionTo (name) {
    var args = Array.prototype.slice.call (arguments, 1);
    console.log (args);
  }

  transitionTo ("contact", "Today", "20"); // ["Today", "20"]​
```

- 借用String对象方法

String对象一般不可变，所以一些固定的数组可以借用String对象方法。

- 借用其他自定义方法

``` js
  var gameController = {
    scores: [20, 34, 55, 46, 77],
    avgScore: null,
    players: [
      {name:"Tommy", playerID:987, age:23}
      {name:"Pau", playerID:87, age:33}
    ]
  }
  
  var appController = {
    scores: [900, 845, 809, 950],
    avgScore: null,
    avg: function () {
      var sumOfScores = this.scores.reduce (function (prev, cur, index, array) {
        return prev + cur;
      });
      this.avgScore = sumOfScores / this.scores.length;
    }
  }

  appController.avg.apply(gameController);
  console.log(gameController.avgScore); // 46.4​
  console.log(appController.avgScore);  // null​
```

### 3. bind
> bind()方法会创建一个新函数，称为绑定函数，当调用这个绑定函数时，绑定函数会以创建它时传入 bind()方法的第一个参数作为 this，传入 bind() 方法的第二个以及以后的参数加上绑定函数运行时本身的参数按照顺序作为原函数的参数来调用原函数。

bind方法和apply、call方法很类似，也可以改变函数体内this的指向。在常见的单体模式中，通常我们会使用 _this , that , self 等保存 this ，这样我们可以在改变了上下文之后继续引用到它。

``` js
  var foo = {
    bar : 1,
    eventBind: function(){
      var _this = this;
      $('.someClass').on('click',function(event) {
        /* Act on the event */
        console.log(_this.bar);    //1
      });
    }
  }
```
由于 Javascript 特有的机制，上下文环境在 eventBind:function(){ } 过渡到 $(‘.someClass’).on(‘click’,function(event) { }) 发生了改变，上述使用变量保存 this 这些方式都是有用的，也没有什么问题。当然使用 bind() 可以更加优雅的解决这个问题。
``` js
  var foo = {
    bar : 1,
    eventBind: function(){
      $('.someClass').on('click',function(event) {
        /* Act on the event */
        console.log(this.bar);    //1
      }.bind(this));
    }
  }
```
在上述代码里，bind创建了一个函数，当这个click事件绑定在被调用的时候，它的 this 关键词会被设置成被传入的值（这里指调用bind()时传入的参数）。因此，这里我们传入想要的上下文this(其实就是 foo)，到bind函数中。然后，当回调函数被执行的时候， this便指向foo对象。

有个有趣的问题，如果连续 bind() 两次，亦或者是连续 bind() 三次那么输出的值是什么呢？

``` js
  var bar = function(){
    console.log(this.x);
  }
  var foo = {
    x:3
  }
  var sed = {
    x:4
  }
  var func = bar.bind(foo).bind(sed);
  func();    //?
 
  var fiv = {
    x:5
  }
  var func = bar.bind(foo).bind(sed).bind(fiv);
  func();    //?
```
答案是，两次都仍将输出 3 ，而非期待中的 4 和 5 。原因是，在Javascript中，多次 bind() 是无效的。更深层次的原因， bind的实现，相当于使用函数在内部包了一个call / apply，第二次 bind() 相当于再包住第一次bind，故第二次以后的 bind 是无法生效的。

### 4.区别

``` js
  var obj = {
    x: 81,
  };
 
  var foo = {
    getX: function() {
      return this.x;
    }
  }
 
  console.log(foo.getX.bind(obj)());    //Note the () after bind
  console.log(foo.getX.call(obj));
  console.log(foo.getX.apply(obj)); 
```
当你希望改变上下文环境之后并非立即执行，而是回调执行的时候，使用bind() 方法；而 apply/call 则会立即执行函数。
