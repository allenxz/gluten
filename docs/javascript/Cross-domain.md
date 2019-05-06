# 跨域

跨域是一个老生常谈的问题里，基本上面试官在问你的项目的时候都会顺便问到：有没有在开发过程中遇到跨域问题？是怎么解决的？上次阿里电面的时候面试官就问了我，当时我只答了jsonp，然后他继续往深里问，问了CORS，然后就被问倒了...



## 什么是跨域

跨域就是从一个域的网页去请求另一个域里的资源。这是狭义的说法，严格来说，只要两个域的协议、域名、端口有任意一个不同，发起的请求就会被当作跨域。
那为什么跨域的时候会遇到问题？是因为大多数的浏览器都支持同源策略，为了防止遭受XSS、CSRF的攻击。所以网页发起跨域请求的时候浏览器会报错，因为双方不同源。



## 解决方案

解决的方案其实很多，但是我这里就只详细说最主要的两种，其他的如果有兴趣可以上网搜索一下，或者上我博客，我之前有总结过。而下面说的内容参考了阮一峰大大的博客。



### jsonp

jsonp虽然只支持Get请求，但是对旧版本的浏览器兼容性很好，所以应用的很多。

**原理**

它的基本思想是：网页通过插入一个`<script>`元素，向服务器请求json数据，这样是不受同源策略的限制的，因为`<img>`、`<link>`、<script>这三个标签是允许跨域的。服务器收到请求后，把数据放在一个指定名字的回调函数里传回来（即foo(data)），这样就能调用客户端预先写好的回调函数，获得数据。

**具体操作**

```javascript
//插入一个<script>元素
 function addScriptTag(src){
	var script = document.createElement(‘script’);
	script.setAttribute(“type”,”type/javascript”);
	script.src=src;
	document.body.appendChild(script);
}
window.onload = function () {
	addScriptTag(‘http://example.com?callback=foo’);
}
//回调函数
function foo (data){
	console.log(data);
}
```



### CORS
CORS，全称跨域资源共享，是一个W3C标准。它允许浏览器向跨源服务器发出XMLHttpRequest请求，从而克服Ajax只能同源使用的限制。

CORS需要服务器和浏览器同时支持，但是整个通信过程CORS是浏览器自动完成的，所以说实现CORS通信的关键是服务器。只要服务器实现了CORS接口，就能进行跨源通信。

浏览器把CORS请求分为两类，简单请求和非简单请求。
只要满足一下两大条件，就属于简单请求：

1. 请求方法是HEAD、GET、POST中的一种
2. HTTP中的头信息不超过一下几种字段：
   * Accpt Accpt-Language Content-Language Last-Event-ID 
   * Content-Type：只限于三个值application/x-www-form-urlencoded、multipart/form-data、text/plain

不满足以上两个条件的就是非简单请求。



**简单请求**

对于简单请求，浏览器会直接放出请求，请求的头信息中，会增加一个Origin字段，这个字段用来说明这个请求来自哪个源，服务器根据这个值来决定是否同意这次请求。
如果Origin指定的源不在许可范围中，服务器会返回一个正常的Http响应。但是这个响应是没包含Access-Control-Allow-Origin字段的，浏览器发现了就会知道请求出错了，然后抛出一个错误，被XmlHttpRequest的onerror回调函数捕获。

而如果Origin指定的域名在许可范围之内，服务器返回的响应的头信息就会多几个字段：
Access-Control-Allow-Origin（值为Origin字段的值或者*），
Access-Control-Allow-Credentials（表示请求中是否允许携带Cookie）
Access-Control-Expose-Headers（用于表明想要获取的基本字段外的其他字段）



**非简单请求**

非简单请求是那种对服务器有特殊要求的请求，如请求方法为PUT或DELETE，或者Content-Type字段的类型为application/json。

这类请求会在正式通信前，增加一次Http查询请求，称为**预检请求**。

浏览器遇到一个Http请求的方法是PUT，发现这是一个非简单请求，就自动发出一个预检请求。预检请求的请求方法为OPTIONS，表示这个请求是用于询问的。头信息中依然包含了Origin字段，表示请求来自的源。除此之外，预检请求的头信息中还包含了了两个特殊的字段：

* Access-Control-Request-Method（用于表示浏览器的CORS请求会用到哪些HTTP方法）
* Access-Control-Request-Header（该字段是一个以逗号分隔的字符串，指定浏览器CORS请求会额外发送的头字段信息）。



服务器收到预检请求，检查了Origin、Access-Control-Request-Method和Access-Control-Request-Headers字段后，确定是否允许跨源请求，就会做出回应。回应的具体处理和简单请求的服务器回应基本相同。只不过回应中的头信息字段有些不同：

* Access-Control-Allow-Methods（值为逗号分隔的字符串，表明服务器支持的所有跨域请求的方法）
* Access-Control-Allow-Headers（值也是逗号分隔的字符串，表明服务器支持的所有头信息）
* Access-Control-Allow-Credentials（含义和简单请求时的相同）
* Access-Control-Max-Age（用于指定本次预检请求的有效期）

一旦服务器通过预检请求后，以后每次浏览器的正常CORS请求，就跟简单请求一样被处理。



以上的就是CORS的内部工作机制，更多的内容可以去阮一峰大大的博客深入了解。下面有一个CORS的实际应用代码。

```javascript
function createCORSRequest(method, url) {
  var xhr = new XMLHttpRequest();
  if ("withCredentials" in xhr) {

    // "withCredentials"属性是XMLHTTPRequest2中独有的
    xhr.open(method, url, true);

  } else if (typeof XDomainRequest != "undefined") {

    // 检测是否XDomainRequest可用
    xhr = new XDomainRequest();
    xhr.open(method, url);

  } else {

    // 看起来CORS根本不被支持
    xhr = null;

  }
  return xhr;
}

var xhr = createCORSRequest('GET', url);
if (!xhr) {
  throw new Error('CORS not supported');
}
```

