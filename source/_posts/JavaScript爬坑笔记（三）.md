---
layout: '[post.md]'
title: JavaScript爬坑笔记（三）
date: 2016-3-22 16:44:09
tags: ["JavaScript","前端"]
bgimage: http://7xpot0.com1.z0.glb.clouddn.com/public/16-12-8/34326467.jpg
---

这一节介绍一些**JavaScript**的**setTimeout**和**setInterval**方法，以及事件循环处理中的“坑”。

<!--more-->

### 1 setTimeout和setInterval

setTimeout和setInterval方法都是浏览器对象，即BOM提供的方法。这两个方法分别用来进行延时执行，和设置时间间隔执行代码。

#### 1.1 基本用法

setTimeout用来对代码进行一个延时执行，定义为：

``` js
window.setTimeout("javascript function", milliseconds);
```

两个参数分别是执行代码和延时时间，可以用`clearTimeout`来取消定时：

``` js
var timer=setTimeout("alert('hello world')", 5000);
clearTimeout(timer);
```

setInterval方法用来设定一个时间间隔来执行代码，其定义方式和setTimeout方法类似:

``` js
window.setInterval('javascript function', milliseconds)
```

同样的，可以用clearInterval方法来取消setInterval。

#### 1.2 深入原理

举个简单的例子

``` js
function test(){
  setTimeout('console.log(1)',0);
  console.log(2);
}
test();
```

结果是先打印2，再打印1，这不难理解。

> setTimeout中的代码会在单独的线程中执行，不同的线程有优先级的区别。

再看一个特殊的例子

``` js
setTimeout( function(){ while(true){} } , 100);
setTimeout( function(){ console.log(’你好!’); } , 200);
```

这段代码不会打印“你好!”了。

> 这表明js是单线程的运行在浏览器引擎中，只要上面代码没有执行完，下面代码是永远不会执行的。而实际上浏览器提供的两个计时函数，只是告诉浏览器，我要插入回调函数到待执行队列的时间点，而不是执行的时间点。而什么时候执行，则取决于当前执行队列是否空闲。

#### 1.3 一些例子

看下面的例子，

``` js
for (var i = 0; i < 3; i++) {
  setTimeout(function() {
    console.log(i);
  }, 0);
  console.log(i);
}
```

结果是：0 1 2 3 3 3

这个例子中体现了“异步、作用域、闭包”等内容，其实上面的代码等价于：

``` js
var i = 0;
setTimeout(function() {
  console.log(i);
}, 0);
console.log(i);
i++;
setTimeout(function() {
  console.log(i);
}, 0);
console.log(i);
i++;
setTimeout(function() {
  console.log(i);
}, 0);
console.log(i);
i++;
```

又因为setTimeout是单独的线程，需要等待前面的代码执行完成后再执行，继续等价：

``` js
var i = 0;
console.log(i);
i++;
console.log(i);
i++;
console.log(i);
i++;
setTimeout(function() {
  console.log(i);
}, 0);
setTimeout(function() {
  console.log(i);
}, 0);
setTimeout(function() {
  console.log(i);
}, 0);  
//弹出 0 1 2 3 3 3
```

再举个例子：

``` js
function Obj(msg){  
  this.msg=msg;  
  this.shout=function(){  
    alert(this.msg);  
  }  
  this.waitAndShout=function(){  
    setTimeout(this.shout, 2000);  
  }
}
var aa = new Obj("xixi");  
aa.waitAndShout();
```

结果为undefined，这个例子说明是setTimeout里面函数的内部，也就是第一个参数的内部指向window，所以弹出了undefined。

解决的方式也很简单，利用`var that = this`做一下this的绑定即可

``` js
function Obj(msg){  
  this.msg = msg;  
  var that = this;  
  this.shout = function(){  
    alert(that.msg);  
  }  
  this.waitAndShout = function(){  
  setTimeout(this.shout,5000);  
  }  
}  
var obj = new Obj('xixi');  
obj.waitAndShout();  
```

### 2 EVENT LOOP

JavaScript的时间循环机制就是主线程在执行间隙去“任务队列”中读取之前添加好的时间，循环往复。

摘张经典大图[《Help, I'm stuck in an event-loop》](http://vimeo.com/96425312 "event-loop") 

![](http://7xpot0.com1.z0.glb.clouddn.com/public/16-12-9/98423029.jpg)

如上图所示，主线程开启堆和栈区，在栈中执行代码，同时调用外部API在“任务队列”中添加各种事件。栈中的代码执行完毕后，主线程去读取任务队列，依次执行相应的回调函数。
