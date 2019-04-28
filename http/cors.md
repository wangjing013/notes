[TOC]



# 跨域

因为浏览器为了安全，引入的同源策略。

## 一：同源

------

​	在计算中，同源策略是Web应用程序安全模型中的一个重要概念。根据该策略，Web浏览器允许第一个Web页面中包含的脚本访问第二个Web页面中的数据，但前提是两个Web页面具有相同的*源*。

### 1.1  何为同源 ？ 

- ​	相同协议（http）
- ​	相同域名
- ​	相同端口

同时满足以上三点即可； 

举例来说： `http://www.example.com/dir/page.html` 这个网址，协议是 `http://` ，域名 `www.example.com` ，端口是 `80` (默认端口可以省略). 它的同源情况如下。

| URL                                                      | Outcome | Reason                                    |
| -------------------------------------------------------- | ------- | ----------------------------------------- |
| http://www.example.com/dir/page2.html                    | 同源    | Same protocol, host and port              |
| http://www.example.com/dir2/other.html                   | 同源    | Same protocol, host and port              |
| http://username:password@www.example.com/dir2/other.html | 同源    | Same protocol, host and port              |
| http://www.example.com:81/dir/other.html                 | 不同源  | Same protocol and host but different port |
| https://www.example.com/dir/other.html                   | 不同源  | Different protocol                        |
| http://en.example.com/dir/other.html                     | 不同源  | Different host                            |
| http://example.com/dir/other.html                        | 不同源  | Different host (exact match required)     |
| http://v2.www.example.com/dir/other.html                 | 不同源  | Different host (exact match required)     |

同源策略主要适用于脚本的数据访问(XMLHttpRequrest)；对于通过HTML标签（例如图像、CSS、动态加载脚本）不受同源策略的约束。 

**特殊情况**：@font-face Web字体会受到相同域的限制（字体文件必须与使用它们的页面位于同一域中）

### 1.2 目的

同源政策的目的，是为了保证用户信息的安全，防止恶意的网站窃取数据。

设想这样一种情况：A网站是一家银行，用户登录以后，又去浏览其他网站。如果其他网站可以读取A网站的 Cookie，会发生什么？

很显然，如果 Cookie 包含隐私（比如存款总额），这些信息就会泄漏。更可怕的是，Cookie 往往用来保存用户的登录状态，如果用户没有退出登录，其他网站就可以冒充用户，为所欲为。因为浏览器同时还规定，提交表单不受同源政策的限制。

由此可见，"同源政策"是必需的，否则 Cookie 可以共享，互联网就毫无安全可言了。

### 1.3 限制范围

随着互联网的发展，"同源政策"越来越严格。目前，如果非同源，共有三种行为受到限制。

1.  Cookie、LocalStorage 和 IndexDB 无法读取。
2. DOM 无法获得。
3. AJAX 请求不能发送。

虽然这些限制是必要的，但是有时很不方便，合理的用途也受到影响。下面，将详细介绍，如何规避上面三种限制。



## 二 . 如何共享 Cookie

------

 Cookie 是服务器写入浏览器的一小段信息，只有同源的网页才能共享。但是，两个网页一级域名相同，只是二级域名不同，浏览器允许通过设置 `Document.domain` 来共享Cookie.

举例来说，A网页是`http://w1.example.com/a.html`，B网页是`http://w2.example.com/b.html`，那么只要设置相同的`document.domain`，两个网页就可以共享Cookie。

```javascript
document.domain = 'example.com';
```

现在，A网页通过脚本设置一个 Cookie。

```javascript
document.cookie = "test1=hello";
```

B网页就可以读到这个 Cookie。

```javascript
var allCookie = document.cookie;
```

**注意** 这种方法只适用于 Cookie 和 iframe 窗口，LocalStorage 和 IndexDB 无法通过这种方法规避同源政策，而要使用下文介绍的PostMessage API。

另外，服务器也可以在设置Cookie的时候，指定Cookie的所属域名为一级域名，比如`.example.com`。

```http
Set-Cookie: key=value; domain=.example.com; path=/
```

这样的话，二级域名和三级域名不用做任何设置，都可以读取这个Cookie。

## 三.  如何解决跨域窗口的通信？ 例如 iframe、window.open()   

------

如果两个网页不同源，就无法拿到对方的DOM。典型的例子是`iframe` 窗口和`window.open`方法打开的窗口，它们与父窗口无法通信。

比如，父窗口运行下面的命令，如果`iframe`窗口不是同源，就会报错。

```javascript
document.getElementById("myIFrame").contentWindow.document
```

上面命令中，父窗口想获取子窗口的DOM，因为跨源导致报错。

反之亦然，子窗口获取主窗口的DOM也会报错。

```javascript
window.parent.document.body
```

如果两个窗口一级域名相同，只是二级域名不同，那么设置上一节介绍的`document.domain`属性，就可以规避同源政策，拿到DOM。

对于完全不同源的网站，目前有三种方法，可以解决跨域窗口的通信问题。

1. 片段识别符（fragment identifier）

2. window.name

3. 跨文档通信API（Cross-document messaging）


### 3.1 片段识别符

------

片段标识符（fragment identifier）指的是，URL的`#`号后面的部分，比如`http://127.0.0.1:5000/parent.html#fragment`的`#fragment`。如果只是改变片段标识符，页面不会重新刷新。

父窗口可以把信息，写入子窗口的片段标识符。

```javascript
var src = originURL + '#' + data;
document.getElementById('myIFrame').src = src;
```

子窗口通过监听`hashchange`事件得到通知。

```javascript
window.onhashchange = checkMessage;

function checkMessage() {
  var message = window.location.hash;
  // ...
}
```

同样的，子窗口也可以改变父窗口的片段标识符。

```javascript
parent.location.href= target + "#" + hash;
```

### 3.2  window.name

------

浏览器窗口有`window.name`属性.  这个属性的最大特点是，无论是否同源，只要在同一个窗口里，前一个网页设置了这个属性，后一个网页可以读取它。

例如现在三个页面： `parent.html`、`parent-sibling.html`（http://127.0.0.1:5500）,  `childhtml`(http://127.0.0.1:8888);    

父窗口(`parent.html`) 先打开一个子窗口(打开方式可以是 `iframe` 或 `window.open`)，载入一个不同源的网页( `child.html `)，该网页将信息写入`window.name`属性。

```javascript
window.name = data;
```

接着，子窗口跳回一个与主窗口同域的网址。

```javascript
location = 'http://127.0.0.1:5500/parent-sibling.html';
```

然后，主窗口就可以读取子窗口的`window.name`了。

```javascript
var data = document.getElementById('myFrame').contentWindow.name;
// window.open 可以通过
var prop = window.open('http://127.0.0.1:8888/window-name-child.html', 'title')   
var data = prop.name;
```

这种方法的优点是，`window.name`容量很大，可以放置非常长的字符串；缺点是必须监听子窗口`window.name`属性的变化，影响网页性能。

### 3.3  window.postMessage

------

上面两种方法都属于破解，HTML5为了解决这个问题，引入了一个全新的API：跨文档通信 API（Cross-document messaging）。

这个API为`window`对象新增了一个`window.postMessage`方法，允许跨窗口通信，不论这两个窗口是否同源。

举例来说，父窗口`http://127.0.0.1:5500/parent.html`向子窗口`http://127.0.0.1:8888/child.html`发消息，调用`postMessage`方法就可以了。

```javascript
// parent.html

var popup = window.open('http://127.0.0.1:8888/child.html', 'title');  // 打开一个新的窗口

popup.postMessage('Hello World!', 'http://127.0.0.1:8888');
// 往 http://bbb.com 发送消息
```

``postMessage`` 方法的第一个参数是具体的信息内容， 第二个参数是接收消息的窗口的源（origin），即 "协议 + 域名 + 端口". 也可以设为 ``*`` ,表示不限制域名，向所有窗口发送。

子窗口向父窗口发送消息的写法类似。

```javascript
// child.html

window.opener.postMessage('Nice to see you', 'http://127.0.0.1:5500');
```

``window.opener``  在这里返回的是父窗口的引用。

父窗口和子窗口都可以通过`message`事件，监听对方的消息。

```javascript
window.addEventListener('message', function(e) {
  console.log(e.data);
},false);
```

`message`事件的事件对象`event`，提供以下三个属性。

- `event.source`:   发送方的[窗口](https://developer.mozilla.org/en-US/docs/DOM/window)对象的引用; 您可以使用此来在具有不同origin的两个窗口之间建立双向通信。
- `event.origin`:   发送方窗口的 [origin](https://developer.mozilla.org/en-US/docs/Origin) .  这个字符串由 协议、“://“、域名、“ : 端口号”拼接而成。
- `event.data`:       消息内容

下面例子中是，子窗口通过 ``event.source`` 属性引用父窗口，然后发送消息

```javascript
window.addEventListener('message', receiveMessage);
function receiveMessage(event) {
  event.source.postMessage('Nice to see you!', '*');
}
```

``event.origin`` 属性可以过滤不是发给本窗口的消息

```javascript
window.addEventListener('message', receiveMessage);

function receiveMessage(event) {
  if (event.origin !== 'http://127.0.0.1:5500') return;
  if (event.data === 'Hello World') {
      event.source.postMessage('Hello', event.origin);
  } else {
    console.log(event.data);
  }
}
```

#### `postMessage` 安全问题

**如果您不希望从其他网站接收message，请不要为message事件添加任何事件侦听器。** 这是一个完全万无一失的方式来避免安全问题。

如果您确实希望从其他网站接收message，请**始终使用origin和source属性验证发件人的身份**。 

#### 兼容性

​	现在浏览器都支持外，其中 `IE8` 和 `IE9` 仅支持窗口与 [`frame`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/frame) 和 [`iframe`](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/iframe) 之间的通信。

### 3.4  LocalStorage

------

通过`window.postMessage`，读写其他窗口的 `LocalStorage` 也成为了可能。

下面是一个例子，主窗口写入`iframe`子窗口的`localStorage`。

父窗口 (http://127.0.0.1:5500/parent.html) 发送消息的代码如下。

​	`parent.html 中的内容`

```html
 <iframe src="http://127.0.0.1:8888/child.html" frameborder="0" id="myFrame"></iframe> 
```

```javascript

 window.onload = function(){
      var refs = document.getElementById('myFrame').contentWindow;
      var obj = { name: 'Jack' };
      // 存储对象
      refs.postMessage(JSON.stringify({key: 'storage', data: obj}), 'http://127.0.0.1:8888');
  }
```

子窗口将父窗口发来的消息，写入自己的 `LocalStorage`。

```javascript
 window.onmessage = function(e) {
     // 先判断消息是不是 127.0.0.1:5500 发来的， 不是话拒绝
     if (e.origin !== 'http://127.0.0.1:5500') {
         return;
     }
     var payload = JSON.parse(e.data);
     localStorage.setItem(payload.key, 			  JSON.stringify(payload.data));
 };
```

加强版的父窗口发送消息代码如下。

```javascript
 window.onload = function () {
     var win = document.getElementById('myFrame').contentWindow;
     var obj = {
         name: 'Jack'
     };
     // 存储对象
     win.postMessage(JSON.stringify({
         key: 'storage',
         data: obj,
         method: 'set'
     }), 'http://127.0.0.1:8888');

     // 读取对象
     win.postMessage(JSON.stringify({
         key: 'storage',
         method: "get"
     }), "*");
     window.onmessage = function (e) {
         if (e.origin != 'http://127.0.0.1:8888') return;
         // "Jack"
         console.log(JSON.parse(e.data).name);
     };
 }
```

 加强版本的子窗口接收消息

```javascript
window.onmessage = function (e) {
     if (e.origin !== 'http://127.0.0.1:5500') return;
     var payload = JSON.parse(e.data);
     // 根据不同method参数来执行不同操作  
     switch (payload.method) {
         case 'set':
             localStorage.setItem(payload.key, JSON.stringify(payload.data));
             break;
         case 'get':
             var data = localStorage.getItem(payload.key);
             e.source.postMessage(data, e.origin);
             break;
         case 'remove':
             localStorage.removeItem(payload.key);
             break;
     }
 };
```



## 四、AJAX

同源政策规定，AJAX请求只能发给同源的网址，否则就报错。

除了架设服务器代理（浏览器请求同源服务器，再由后者请求外部服务），有三种方法规避这个限制。

- JSONP

- WebSocket

- CORS


### 4.1  JSONP (JSON  Padding)

------

JSONP是服务器与客户端跨源通信的常用方法。最大特点就是简单适用，老式浏览器全部支持，服务器改造非常小。

它的基本思想是，网页通过添加一个`<script>`元素，向服务器请求JSON数据，这种做法不受同源政策限制；服务器收到请求后，将数据放在一个指定名字的回调函数里传回来。

首先，网页动态插入`<script>`元素，由它向跨源网址发出请求。

```javascript
 function addScriptTag(src) {
     var script = document.createElement('script');
     script.setAttribute("type", "text/javascript");
     script.src = src;
     document.body.appendChild(script);
 }
window.onload = function () {
    addScriptTag('http://127.0.0.1:3000?callback=foo');
}
function foo(data) {
    // 获取用户信息- 姓名：王小二 年龄：30
    console.log("获取用户信息- 姓名：" + data.name + " 年龄：" + data.age);
};
```

上面代码通过动态添加`<script>`元素，向服务器`example.com`发出请求。注意，该请求的查询字符串有一个`callback`参数，用来指定回调函数的名字，这对于JSONP是必需的。

服务器收到这个请求以后，会将数据放在回调函数的参数位置返回。

``nodejs`` 服务端代码为例：

```javascript
const http = require('http');
const querystring  = require('querystring');
 http.createServer(function(req,res){
    let params = querystring.parse(req.url.replace(/\/?\?/, ""));
    let data = {"name": "王小二", "age": 30};
    res.writeHead(200, {"Content-Type":"application/json;charset=utf-8"});
    res.end(params.callback + "("+JSON.stringify(data)+")");
}).listen(3000);
```

由于`<script>`元素请求的脚本，直接作为代码运行。这时，只要浏览器定义了`foo`函数，该函数就会立即调用。作为参数的 ``JSON`` 数据被视为JavaScript对象，而不是字符串，因此避免了使用`JSON.parse`的步骤。



### 4.2  WebSocket

------

``WebSocket`` 是一种通信协议，使用`ws://`（非加密）和`wss://`（加密）作为协议前缀。该协议不实行同源政策，只要服务器支持，就可以通过它进行跨源通信。

下面是一个例子，浏览器发出的 WebSocket 请求的头信息。

```http
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9,en;q=0.8
Cache-Control: no-cache
Connection: Upgrade
Host: 127.0.0.1:5500
Origin: http://127.0.0.1:5500
Pragma: no-cache
Sec-WebSocket-Extensions: permessage-deflate; client_max_window_bits
Sec-WebSocket-Key: MDRTbLPaWSotkJt3zwp8cQ==
Sec-WebSocket-Version: 13
Upgrade: websocket
User-Agent: Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/69.0.3497.100 Safari/537.36
```

上面代码中，有一个字段是`Origin`，表示该请求的请求源（origin），即发自哪个域名。

正是因为有了`Origin`这个字段，所以`WebSocket`才没有实行同源政策。因为服务器可以根据这个字段，判断是否许可本次通信。如果该域名在白名单内，服务器就会做出如下回应。

```javascript
 var socket = new WebSocket('ws://127.0.0.1:3001');
 	 socket.addEventListener('open', function (event) {
  	 socket.send('Hello Server!');
 });
 socket.addEventListener('message', function (event) {
  	var data =  JSON.parse(event.data);
  	console.log("获取用户信息- 姓名：" + data.name + " 年龄：" + data.age);
 });
```

**Nodejs 服务端代码**

```javascript
var WebSocketServer = require('ws').Server,
    socket = new WebSocketServer({
        port: 3001
    });
socket.on('connection', function (ws) {
    ws.on('message', function (message) {
        console.log('client connected');
    });
    // string, Buffer, ArrayBuffer, Array, or array-like object.
    ws.send(JSON.stringify({"name":'王小二', age: 30}));
});
```



### 4.3  CORS

------

`CORS`是跨源资源分享（Cross-Origin Resource Sharing）的缩写。它是`W3C`标准，是跨源AJAX请求的根本解决方法。相比`JSONP`只能发`GET`请求，`CORS`允许任何类型的请求。

#### 注意

**`CORS`** 需要浏览器和服务器同时支持。目前，所有浏览器都支持该功能，IE浏览器不能低于 **`IE10`.**

整个`CORS` 通信过程，都是**浏览器自动**完成，不需要用户参与。对于开发者来说，`CORS` 通信与同源的AJAX通信没有差别，代码完全一样。浏览器一旦发现AJAX请求跨源，就会自动添加一些附加的头信息，有时还会多处一次附加的请求，但用户不会有感觉。

因此，**实现`CORS`通信的关键是服务器。只要服务器实现了`CORS`接口，就可以跨源通信。**

#### 4.3.1 `CORS` 相关请求头和响应头字段

**请求头相关的：**

​	（1）**Access-Control-Request-Method**  （浏览器自动完成）

​	请求首部  **Access-Control-Request-Method** 出现于 [preflight request](https://developer.mozilla.org/en-US/docs/Glossary/preflight_request) （预检请求）中，用于通知服务器在真正的请求中会采用哪种  [HTTP 方法](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods)。因为预检请求所使用的方法总是 [`OPTIONS`](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Methods/OPTIONS) ，与实际请求所使用的方法不一样，所以这个首部是必要的。  

​	（2）**Access-Control-Request-Headers** （浏览器自动完成）

​	请求首部  **Access-Control-Request-Headers** 出现于 [preflight request](https://developer.mozilla.org/en-US/docs/Glossary/preflight_request) （预检请求）中，用于通知服务器在真正的请求中会采用哪些请求首部。

**响应头相关的：**

​	(3) **Access-Control-Allow-Origin** 

​	表示服务端接受哪些域名的请求.  如果允许任何域访问可以指定 ``  *``

​	(4) **Access-Control-Allow-Methods**  

​	该字段必需，它的值是逗号分隔的一个字符串，表明服务器支持的所有跨域请求的方法。注意，返回的是所有支持的方法，而不单是浏览器请求的那个方法。这是为了避免多次"预检"请求。上例中 ``PUT`` . 

​	(5) **Access-Control-Allow-Headers**  

​	浏览器请求包括`Access-Control-Request-Headers`字段，则`Access-Control-Allow-Headers`字段是必需的。它也是一个逗号分隔的字符串，表明服务器支持的所有头信息字段，不限于浏览器在"预检"中请求的字段。

​	(6) **Access-Control-Expose-Headers**

​	该字段可选。**`CORS`** 请求时，`XMLHttpRequest`对象的`getResponseHeader()`方法只能拿到6个基本字段：`Cache-Control`、`Content-Language`、`Content-Type`、`Expires`、`Last-Modified`、`Pragma`。如果想拿到其他字段，就必须在`Access-Control-Expose-Headers`里面指定。

​       (7) **Access-Control-Max-Age**

​	该字段可选，用来指定本次预检请求的有效期，单位为秒。

​	**注意**： 在浏览器禁用缓存时是不生效的.

​      (8) **Access-Control-Allow-Credentials**  

​	该字段可选。它的值是一个布尔值，表示是否允许发送Cookie。默认情况下，Cookie不包括在`CORS`请求之中。设为`true`，即表示服务器明确许可，Cookie可以包含在请求中，一起发给服务器。这个值也只能设为`true`，如果服务器不要浏览器发送Cookie，删除该字段即可。

#### 4.3.2 `CORS`中的两种请求

浏览器将`CORS`请求分成两类：简单请求（simple request）和非简单请求（not-so-simple request）。

只要同时满足以下两大条件，就属于简单请求。

（1) 请求方法是以下三种方法之一：

- HEAD
- GET
- POST

（2）HTTP的头信息不超出以下几种字段：

- Accept
- Accept-Language
- Content-Language
- Last-Event-ID
- Content-Type：只限于三个值`application/x-www-form-urlencoded`、`multipart/form-data`、`text/plain`

**凡是不同时满足上面两个条件，就属于非简单请求。** 浏览器对这两种请求的处理，是不一样的。

##### 4.3.2.1 简单请求

对于**简单请求**，浏览器直接发出`CORS`请求。具体来说，就是在头信息之中，**增加一个`Origin`字段。**

```javascript
// 客户端
 var xhr = new XMLHttpRequest();
 xhr.onreadystatechange = function (){
      if(xhr.readyState == 4  && xhr.status == 200) {
      	console.log(xhr.responseText);
      }
  };
 xhr.onerror = function(error){
 	console.log(error);   
 }
 xhr.open('get', 'http://127.0.0.1:3000/user', true);
 xhr.send();

// 服务端 node.js
const http = require('http');
const fs = require('fs');
const path = require('path');
http.createServer(function(req,res){
 	if(req.method === 'GET' && req.url === "/user"){
        let data = {};
        res.writeHead(200, {"Content-Type":"application/json;charset=utf-8"});
        res.end(JSON.stringify(data));
    }
}).listen(3000);
```

**可以对比`非跨域`和`跨域`针对同一个请求情况下，头信息是否添加一个 `Origin**`

在同源(非跨域)发送的请求头信息：

```http
GET /user HTTP/1.1
Host: 127.0.0.1:3000
Connection: keep-alive
Pragma: no-cache
Cache-Control: no-cache
...
```

在非同源（跨域）情况下请求头：

```http
GET /user HTTP/1.1
Host: 127.0.0.1:3000
Connection: keep-alive
Pragma: no-cache
Cache-Control: no-cache
Origin: http://127.0.0.1:5500
...
```

上面的头信息中，**`Origin `** 字段用来说明，本次请求来自哪个源（协议 + 域名 + 端口）。服务器根据这个值，决定是否同意这次请求。

如果 **`Origin`** 指定的源，不在许可范围内，服务器也会返回一个正常的HTTP回应。

```http
HTTP/1.1 200 OK
Content-Type: application/json;charset=utf-8
Date: Sun, 25 Nov 2018 00:35:01 GMT
Connection: keep-alive
Transfer-Encoding: chunked
```

浏览器发现，这个回应的头信息没有包含 **`Access-Control-Allow-Origin`** 字段，就知道出错了，从而抛出一个错误，被**`XMLHttpRequest`** 的 **`onerror`** 回调函数捕获。注意，这种错误无法通过状态码识别，因为HTTP回应的状态码有可能是200。

如果 **`Origin`** 指定的域名在服务端许可范围内，服务器正常返回以及浏览器正常接收返回内容。

```javascript
// 服务端添加允许访问 origin
res.setHeader('Access-Control-Allow-Origin', 'http://127.0.0.1:5500');
```

响应头信息：

```http
HTTP/1.1 200 OK
Access-Control-Allow-Origin: http://127.0.0.1:5500
Content-Type: application/json;charset=utf-8
```



##### 4.3.2.2 非简单请求

------

​	非简单请求是那种对服务器有特殊要求的请求， 比如请求方式是  **`PUT`**  、**``DELETE``**, 或 **``Content-Type``** 字段的类型是 ``application/json``.

**预检请求**

------

​	非简单请求的`CORS`请求， 会正式通信之前，增加一次HTTP查询请求，称为``预检`` 请求（preflight）.

​	浏览器先询问服务器，当前网页所在的域名是否在服务器的许可名单之中，以及可以使用哪些HTTP动词和头信息字段。只有得到肯定答复，浏览器才会发出正式的`XMLHttpRequest`请求，否则就报错

构建非简单请求 ，JavaScript 代码：

```javascript
// http://127.0.0.1:5500/index.html

var xhr = new XMLHttpRequest();
var data = {id: 1001, name: "张三", age: 20};
xhr.onreadystatechange = function (){
    if(xhr.readyState == 4  && xhr.status == 200) {
        console.log(xhr.responseText);
    }
};
xhr.open('put', 'http://127.0.0.1:3000/user/update', true);
xhr.setRequestHeader('X-Custom-Header', 'value');
xhr.send(JSON.stringify(data));

```

服务端代码：

```javascript
const http = require('http');

http.createServer(function (req, res) {
 	if (req.method === 'OPTIONS' && req.url === "/user/update") {
        
        res.end();
        
    } else if (req.method === 'PUT' && req.url === "/user/update") {
        res.setHeader('Access-Control-Allow-Origin', 'http://127.0.0.1:5500');
        res.writeHead(200, {
            "Content-Type": "application/json;charset=utf-8"
        });
        res.end(JSON.stringify({
            success: '更新成功'
        }));
    }
}).listen(3000);
```

**执行步骤：**

​	1 ： 发送预检请求

​	2 ： 浏览器根据后端返回的头信息，判断当前网页所在的域名是否在服务器的许可名单之中，以及是否允许使用PUT方法和添加自定义头 "X-Custom-Header"

​	3：  检测通过则发送正式请求，否则失败 

假如执行上述代码，会出现如下错误：

```javascript
Failed to load http://127.0.0.1:3000/user/update: Response to preflight request doesn't pass access control check: No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'http://127.0.0.1:5500' is therefore not allowed access
```

大概意思，预检请求没有通过检测，请求资源响应头中不存在 Access-Control-Allow-Origin 字段.  因此 http://127.0.0.1:5500 是不允许被访问的。

如果希望不出现上述的错误，服务端必须添加头信息。

```javascript
if (req.method === 'OPTIONS' && req.url === "/user/update") {
    res.setHeader('Access-Control-Allow-Origin', 'http://127.0.0.1:5500')； 
    res.end();
} 
```

刷新一看，咦咦咦 又发现一个错误：

```javascript
Failed to load http://127.0.0.1:3000/user/update: Method PUT is not allowed by Access-Control-Allow-Methods in preflight response.
```

简单说，发送 ``PUT`` 请求没有被服务端许可，需要添加下面头信息

```javascript
if (req.method === 'OPTIONS' && req.url === "/user/update") {
	..
    res.setHeader('Access-Control-Allow-Methods', 'PUT')； 
    res.end();
}  
```

再刷新一看，咦咦咦 又发现一个错误：

```javascript
Failed to load http://127.0.0.1:3000/user/update: Request header field X-Custom-Header is not allowed by Access-Control-Allow-Headers in preflight response.
```

简单说，请求头字段 ``X-Custom-Header`` 不被允许.  则需要在服务端添加下面头信息

```javascript
if (req.method === 'OPTIONS' && req.url === "/user/update") {
    ..
    res.setHeader('Access-Control-Allow-Headers', 
                  'X-Custom-Header')； 
    res.end();
}  
```

设置上述头信息后，预检请求是发送完，并接收到响应。 如下

```http
HTTP/1.1 200 OK
Access-Control-Allow-Origin: *
Access-Control-Allow-Methods: PUT
Access-Control-Allow-Headers: X-Custom-Header
```

**浏览器的正常请求和回应**

------

一旦服务器通过了"预检"请求，以后每次浏览器正常的`CORS`请求，就都跟简单请求一样，会有一个`Origin`头信息字段。服务器的回应，也都会有一个`Access-Control-Allow-Origin`头信息字段。

下面是"预检"请求之后，浏览器的正常`CORS`请求。

```http
PUT /user/update HTTP/1.1
Host: 127.0.0.1:3000
Connection: keep-alive
Content-Length: 36
Pragma: no-cache
Cache-Control: no-cache
Origin: http://127.0.0.1:5500
X-Custom-Header: value
Content-Type: text/plain;charset=UTF-8
```

上面头信息的`Origin`字段是浏览器自动添加的。

下面是服务器正常的回应。

```http
Access-Control-Allow-Origin: http://127.0.0.1:5500
Content-Type: application/json;charset=utf-8
```

上面头信息中，`Access-Control-Allow-Origin`字段是每次回应都必定包含的。

#### 4.3.3  与`JSONP`的比较

`CORS`与`JSONP`的使用目的相同，但是比`JSONP`更强大。

`JSONP`只支持`GET`请求，`CORS`支持所有类型的HTTP请求。`JSONP`的优势在于支持老式浏览器，以及可以向不支持`CORS`的网站请求数据。

#### 5 经常使用的设置

##### 如果希望预检请求在一段时间内容不在重复发送？

------

用 `Access-Control-Max-Age` 响应头字段；

```javascript
 res.setHeader('Access-Control-Max-Age', 600);
```

表示预检请求在10分钟内不会再重复发送! 

##### 如何想获取服务端返回的自定义头信息?  

------

使用 `Access-Control-Expose-Headers` 响应头字段；

```javascript
 res.setHeader('authorization', '0123456789');
 res.setHeader('Access-Control-Expose-Headers', 'authorization');
```

上述在头信息添加自定字段 ``authorization`` ，如果希望在 ``getResponseHeader`` 获取必须在设置 ``Access-Control-Expose-Headers`` 进行暴露才能访问，否则会报错（Refused to get unsafe header "authorization"）。

##### 如何发送 cookie  ?

------

`CORS` 请求默认不发送Cookie和HTTP认证信息。先来了解下跟`CORS`有关一个响应头字段；

如果要把Cookie 发送到服务器满足如下两点：

(1) 服务端设置 ``Access-Control-Allow-Credentials:true``：

```javascript
res.setHeader('Access-Control-Allow-Credentials', true);
```

(2) 开发者必须在AJAX请求中打开  ``withCredentials``

```javascript
xhr.withCredentials = true;
```

否则，即使服务器同意发送Cookie，浏览器也不会发送。或者，服务器要求设置Cookie，浏览器也不会处理。

但是，如果省略`withCredentials`设置，有的浏览器还是会一起发送Cookie。这时，可以显式关闭`withCredentials`。

```javascript
xhr.withCredentials = false;
```

**注意**：

​	（1） 如果要发送Cookie，`Access-Control-Allow-Origin`就不能设为``*``，必须指定明确的、与请求网页一致的域名。否则如下错误并请求失败

```javascript
The value of the 'Access-Control-Allow-Origin' header in the response must not be the wildcard '*' when the request's credentials mode is include.
```

​	（2）Cookie依然遵循同源政策，只有用服务器域名设置的Cookie才会上传，其他域名的Cookie并不会上传，且（跨源）原网页代码中的`document.cookie`也无法读取服务器域名下的Cookie。

