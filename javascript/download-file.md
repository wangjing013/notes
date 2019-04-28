# 常用下载附件的方式

​	下载附件（image，doc，docx， excel，zip，pdf），应该是实际工作中经常遇到一个问题；这里使用过几种方式分享出来，有问题希望纠正；

​	主要了解的几个知识点：

- http 响应头设置
  - [Content-Disposition](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Disposition) 
  - [Access-Control-Expose-Headers](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Access-Control-Expose-Headers)  这里只需要涉及跨域的时才使用，用于暴露JavaScript中能够获取到响应头字段
- [Blob](https://developer.mozilla.org/zh-CN/docs/Web/API/Blob)  、 [FileReader](https://developer.mozilla.org/zh-CN/docs/Web/API/FileReader)
- [URL](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/URL)

先来介绍常用方式：

## 利用  iframe 或  a  连接

### 服务端代码

```javascript
// nodejs
const http = require('http');
const fs = require('fs');
const path = require('path');
http.createServer(function (req, res) { 
    let filename = encodeURIComponent('微信多开的步骤.doc');
    // 下面两个主要在跨域情况下，需要设置的
	res.setHeader('Access-Control-Allow-Origin', '*');
 	res.setHeader('Access-Control-Expose-Headers', 'Content-Disposition');
    
    // 设置响应头
	res.setHeader('Content-Type', 'application/zip;charset=UTF-8');
	res.setHeader('Content-Disposition', `attachment; filename=${filename}`);
	let fs.readFile(path.resolve(__dirname, `./微信多开的步骤.doc`), function (err, data) {
      if (err) throw err;
      res.end(data);});
}).listen(3000);
```

​	`Content-Disposition` 消息头指示回复的内容该以何种形式展示，是以**内联**的形式（即网页或者页面的一部分），还是以**附件**的形式下载并保存到本地。

​	大概流程：
​		1  下载时浏览器会尝试去找下响应头中 `Content-Disposition` ；
​		2   如果不存在，首先尝试去预览方式打开该文件 ，如果能就直接显示否则以附件的形式下载并保存；

​	**注意**：指定在下载文件名中文情况下，必须先进行编码；

### JS

```javascript
// iframe 
var downloadFileUrl = "http://localhost:3000"
var elemIF = document.createElement("iframe");
elemIF.src = downloadFileUrl;
elemIF.style.display = "none";
document.body.appendChild(elemIF);

// a 
var a = document.createElement('a');
a.href = downloadFileUrl;
a.click();
```

上述两种方式仅仅就是发送一个请求，主要依赖后端的支持；对不需要精确知道文件下载的状态，上面方式就能满足下载；

大家可能有疑问，iframe 不是可以通过 onload 来捕获加载的完成状态 ？
先来看看 load 适用哪些对象?

### [load](https://www.w3.org/TR/DOM-Level-3-Events/#event-type-load) 

​	W3C 对 load 定义

> | Type            | **load**                                                     |
> | --------------- | ------------------------------------------------------------ |
> | Sync / Async    | Async                                                        |
> | Bubbles         | No                                                           |
> | Trusted Targets | [`Window`](https://www.w3.org/TR/DOM-Level-3-Events/#window), `Document`, `Element` |

适用对象：window，Document，Element  那么对于我们下载的文件并在其范围；

如果需要捕获文件下载的进度以及文件下载完成的状态，需要使用下面的方式；

## XMLHttpRequest 

js代码如下：

```javascript
var xhr = new XMLHttpRequest();
xhr.open('GET', url);
xhr.onprogress = function (event) {
    console.log(Math.round(event.loaded / event.total * 100) + "%");
};
xhr.responseType = 'arraybuffer';
xhr.addEventListener('readystatechange', function (event) {
    if (xhr.status === 200 && xhr.readyState === 4) {
        // 获取响应头主要获取附件名称
        var contentDisposition = xhr.getResponseHeader('content-disposition');
        // 获取类型类型和编码  
        var contentType = xhr.getResponseHeader('content-type');
        var blob = new Blob([xhr.response], {
            type: contentType
        });
        var url = window.URL.createObjectURL(blob);
        // 获取文件夹名
        var regex = /filename=[^;]*/;
        var matchs = contentDisposition.match(regex);
        if (matchs) {
            filename = decodeURIComponent(matchs[0].split("=")[1]);
        } else {
            filename = +Date.now() + ".doc";
        }
        var a = document.createElement('a');
        a.href = url;
        a.download = filename;
        a.click();
        window.URL.revokeObjectURL(url);
        // dosomething
    }
})
xhr.send();
```

上述对比第一种方式，通过 `onprogress` 捕获下载进度（界面通过显示进度条来提升体验）；通过 `readystatechange` 监听下载完后并可以做其它的事情；

**注意：** 必须指定 responseType 类型，允许类型 arraybuffer 或 blob， 否则可能出现错误问题  比如 zip，pdf等 文件下载打不开， .doc文件内容乱码等；



