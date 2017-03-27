---
title: WEB跨域的实现
date: 2016-01-08 11:27:43
tags: ["JavaScript","前端"]
bgimage: http://7xpot0.com1.z0.glb.clouddn.com/16-10-10/47252019.jpg
---

在开发现代WEB应用时，常常会遇到以下情形：

1. 客户端使用JavaScript
2. 需要跨域加载服务
3. 如果采用常规的XMLHttpRequest方法，最终会在浏览器终端里显示
```
XMLHttpRequest cannot load [http://external.service/](http://external.service/).
No 'Access-Control-Allow-Origin' header is present on the requested resource.
Origin '[http://my.app](http://my.app/)' is therefore not allowed access.
```

每当WEB应用需要跨域加载API时，往往会遇到这种错误，本文就以此问题入手，记录下WEB跨域的学习笔记。

<!--more-->

### 同源策略（Same origin policy）
同源策略（SOP）是一种WEB安全约定，它的主要目的是抑制来自不同域的文件的相互影响。

一个页面的源由**协议**、**域名**、**端口号**决定，举例，本博客的源地址是（'http', 'littlewin.info', 80）。同源之间的资源文件可以互相访问。如果页面A和页面B拥有相同的源，A页面上的JavaScript代码可以使用HTTP请求访问B的资源，操作B的DOM结构，甚至设置B的cookies。需要注意的是，源是由页面的地址定义的，一个通过CDN引用的JavaScript文件是运行在包含此文件的HTML页面上，而非它的引用地址上。

对于特定的跨域HTTP请求，SOP规定以下一般规则：允许跨域写，禁止跨域读取。这意味着如果A和C是不同的源，A发送的HTTP请求会由C正确地接收（这些就是“写”），但是A中的脚本将无法读取任何数据--甚至来自C返回的响应代码。跨域“读取”会被浏览器屏蔽，导致出现上面的错误。换句话说，**SOP不阻止攻击者向他们源写数据，它只是不允许他们读取来自你的域的数据（cookie, localStorage 或其他）或利用接收到的响应来做任何事。**

SOP策略是是浏览器最核心也最基本的安全功能，如果缺少了同源策略，浏览器很容易受到XSS、CSFR等攻击。

### 执行跨域请求
然而，有时你必须执行跨域请求，而这需要一些额外的操作。合法的跨域请求示例：

- 你必须集成第三方服务（如一个论坛），有一个REST API驻留在不同源。
- 服务器端服务托管在不同的（子）域。
- 客户端逻辑来自不同源而不是服务器端服务端点。
- ...

根据应用的跨服务器规模，可以采用多个选项来启用跨域请求。本文将讨论可能的解决方案：JSONP, 使用服务器端代理和CORS。

当然还有其他选择，使用较广泛的的技术是使用iframes和[window.postMessage](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/postMessage)，这些不在本文讨论。

### 跨域请求示例
有2个网站，一个网站在源（‘http’,'localhost',3000），另一个在（‘http’,'localhost',3001）。它们是不同的源，所以3000请求3001被认为是跨域请求并被浏览器默认屏蔽。

考虑以下场景 - A域的页面想要执行一个GET请求到B域的页面：

浏览器向服务器端发送请求：

```
GET / HTTP/1.1
```

服务器端返回响应：

```
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Content-Length: 57

{
  "response": "This is data returned from the server"
}
```
然而在接收到响应，浏览器屏蔽响应进一步传播，并显示跨域请求错误，如上所示。

### JSONP
JavaScript Object Notation with Padding（JSONP）是一种执行跨域请求的方法，通过利用HTML页面的script标签可以加载来自不同域的API。但这种方式有一些重大的问题：

- JSONP只能用来执行跨域GET请求。
- 服务器必须明确地支持JSONP请求。
- 你必须绝对地信任服务器提供的JSONP响应。
- 如果服务器被盗用，JSONP可以使你的网站暴露大量的[安全漏洞](http://security.stackexchange.com/questions/23438/security-risks-with-jsonp)。

JSONP实现原理是 - `<script>`标签可以有来自不同域的资源，当浏览器解析`<script>`标签，它会GET请求脚本内容（来自任何源）并在当前的页面中执行。通常，服务器会返回HTML或一些XML或JSON数据。有时向一个启用JSONP的服务器请求时，它会返回一个脚本块，这个脚本块执行一个在页面中定义好的回调函数，并提供参数。以下举例：

源3000的页面想要获取存储在源3001的资源。源3000页面包含下面的script标签：

``` js
<script src='http://localhost:3001?callback=myCallbackFunction'></script>
```

当浏览器解析这个script标签，它将正常的发出GET请求：

```
GET /?callback=myCallbackFunction HTTP/1.1
```

服务器没法返回原生JSON，而是返回一个脚本块，包含函数的调用，函数名在URL中指定，输出的数据作为参数传递。


```
HTTP/1.1 200 OK
Content-Type: application/javascript

myCallbackFunction({'response': 'hello world from JSONP!'});
```

这个脚本块在浏览器接收到后就立即被执行，当前页面定义了回调函数，它使用返回的数据作为参数。

Tips：
- 由于JSONP的工作原理是通过一个script标签加载GET请求，因此它只支持跨域的HTTP GET请求。如果你想使用其他的HTTP请求（像POST, PUT或DELETE），就不能使用JSONP方法。
- 这个方法要求你必须完全地信任服务器。这个服务器可能被盗用，并返回任意代码，将在你的页面中执行（访问你的网站cookies, localStorage等等）。

### 服务器端代理

这种绕过同源策略执行而跨域请求的方法没有任何跨域操作，而是使用一个代理服务器在后端访问外部服务，并把结果返回给客户端。请求代码和代理服务器是在同一个域中，因此并不违反同源策略。

这种机制不需要改变现有的服务器代码，它需要服务器端采用代理服务，且在当前域中在浏览器中运行JavaScript代码。

这次不直接向3001域发送GET请求，而是向自己域的代理服务器发送请求。
```
GET /proxy?urlToFetch=http%3A%2F%2Flocalhost%3A3001 HTTP/1.1
```

服务器将执行实际的GET请求外部服务。服务器端代码可以正常的执行跨域请求而不会发生错误，因此可以成功的调用。代理服务将结果输送给客户：

```
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
  "response": "This is data returned from the server, proxy style!"
}
```
注意这种方法也有一些严重的缺点。如果第三方服务使用cookies进行身份验证，那么你就不能使用这种方法。本地的JavaScript代码是不能访问外部的域的cookies并且也不能发送cookies给你的代理服务，所以包含用户验证信息的cookies无法提供给第三方服务。

### CORS
上述两种方法或多或少存在一些“hack”的感觉，实际上都是绕过浏览器的安全机制而实现。

HTML5引入了一个标准的跨域请求方法 -  [Cross-Origin Resource Sharing](http://www.w3.org/TR/cors/)

CORS为服务器提供了一个准入机制，它告诉浏览器域A读取请求自域B的数据是可以的。这种方法是通过在http响应头中包含一个新的Access-Control-Allow-Origin属性值。当浏览器接收到跨域的响应时，它会检查CORS头。如果响应头中指定的源匹配当前源，它允许读取跨域访问的响应。否则，会得到上述错误信息。

像往常一样进行跨域请求：

```
GET / HTTP/1.1
```

源3001的服务器检查是否这个源可以访问数据，并在响应中增加额外的Access-Control-Allow-Origin头，列出请求源：

```
HTTP/1.1 200 OK
Access-Control-Allow-Origin: http://localhost:3000
Content-Type: application/json; charset=utf-8
Content-Length: 62

{
  "response": "This is data returned from the CORS server"
}
```

当浏览器接收到响应时它比较请求源（3000）和列在Access-Control-Allow-Origin头的源（也是3000）。由于请求源出现在Access-Control-Allow-Origin值中，浏览器允许源3000的请求执行响应。

同样的，这种方法有一些局限性。例如老版本的IE只能[部分支持CORS](http://caniuse.com/#search=CORS)。 同时，对于非简单请求，在正式通信之前，增加了一次HTTP查询请求（参考：[跨域资源共享 CORS 详解](http://www.ruanyifeng.com/blog/2016/04/cors.html)）。

### 总结

对比以上三种方法 - 

JSONP只支持GET请求，服务器代理和CORS支持所有类型的HTTP请求。JSONP的优势在于支持老式浏览器，以及可以向不支持CORS的网站请求数据。Proxy方式，通过强大的后端应用可以提供更灵活更能发挥想象力的服务。CORS是目前W3C支持的方式，在新浏览器上体验更好。