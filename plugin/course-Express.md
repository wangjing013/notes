# Express

> 基于 [Node.js](https://nodejs.org/en/) 平台，快速、开放、极简的 Web 开发框架

## 安装

​	Express 本身是一个node模块，那么可以通过NPM进行下载，安装之前先提前下载 [download and install Node.js](https://nodejs.org/en/download/)  Node.js 要求版本 0.10 或 更高

​	安装命令如下：

```javascript
npm install express
```

## 功能

1. 强壮的路由
2. 高性能
3. 超高的测试覆盖率
4. HTTP helpers (重定向、缓存等等 )
5. 视图系统支持14+模板引擎
6. 用于快速生成应用程序的可执行文件

## 快速启动

​	通过 express-generator 快速生成 express 应用程序，如下

​	**安装**

```
 npm install -g express-generator@4
```

​	**创建应用**

```javascript
express  demo && cd demo
```

​	**安装依赖**

```javascript
npm install
```

​	**启动服务**

```javascript
npm start
```

查看官网提供的完整实例，可以 [github](https://github.com/expressjs/express/)直接下载或通过 git下载并安装依赖。

```
 git clone git://github.com/expressjs/express.git --depth 1
 cd express
 npm install
```

运行实例：

```
node examples/content-negotiation
```

------



# Routing 路由

简单的说路由决定应用程序如何响应客户端请求。每个路由允许一个或多个处理函数，这些函数在路由匹配成功时执行。

### 路由定义的基本结构：

```javascript
app.METHOD(PATH, HANDLER)
```

说明：

- app： 是一个express实例
- METHOD:	是HTTP请求的[方法](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol#Request_methods)，必须小写
- PATH:	服务器上的路径地址
- HANDLER: 	当路由匹配成功时执行的方法

下面定义些简单路由：

```javascript
app.all('/', function(req, res, next){
    next();
})
app.get('/', function (req, res) {
  res.send('Hello World!')
})
app.delete('/user', function (req, res) {
  res.send('Got a DELETE request at /user')
})
app.delete('/user', function (req, res) {
  res.send('Got a DELETE request at /user')
})
app.delete('/user', function (req, res) {
  res.send('Got a DELETE request at /user')
})
```

### Route Methods

​	Route 方法是由HTTP 方法衍生出来，也就说支持HTTP所有的请求的 [方法](https://en.wikipedia.org/wiki/Hypertext_Transfer_Protocol#Request_methods)，

并且所有方法都添加到了``express class``的实例上。

​	如下实例

```javascript
var express = require('express');
var app = express();

// GET method route
app.get('/', function (req, res) {
  res.send('GET request to the homepage')
})

// POST method route
app.post('/', function (req, res) {
  res.send('POST request to the homepage')
})
```

查看完整列表 [app.METHOD](http://www.expressjs.com.cn/en/4x/api.html#app.METHOD)

express 提供一种特殊的方法， app.all()  一般用来用在处理所有请求路径上加载的中间件。例如 希望处理请求参数、cookie等

```javascript
app.all("*", function(req,res,next){
    // 权限判断
    // 加载用户等等
});
```

------
### Route paths

​	route paths 和 请求方法结合，处理指定客户端请求。

​	paths 可以是字符串、字符串模式、正则表达式

​	字符串模式中可以包含常见正则字符集： ? ，+，*，（）,  其中连字符 (-) 或 (.) 没有特殊含义.

​	