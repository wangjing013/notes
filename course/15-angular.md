# 15课时

1. Http
2. Cookie
3. Session

## [Cookie](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Cookies)

------

HTTP Cookie（也叫Web Cookie或浏览器Cookie）是服务器发送到用户浏览器并保存在本地的一小块数据，它会在浏览器下次向同一服务器再发起请求时被携带并发送到服务器上。通常，它用于告知服务端两个请求是否来自同一浏览器，如保持用户的登录状态。Cookie使基于[无状态](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview#HTTP_is_stateless_but_not_sessionless)的HTTP协议记录稳定的状态信息成为了可能。

Cookie主要应用：

- 会话状态管理（如用户登录状态、购物车、游戏分数或其它需要记录的信息）
- 个性化设置（如用户自定义设置、主题等）
- 浏览器行为跟踪（如跟踪分析用户行为等）

## 创建 Cookie

服务端接收到请求，可以在响应头中添加一个 Set-Cookie 选项。浏览器收到响应后通常会保持下 Cookie，之后对该服务的每一次请求中都通过 Cookie 请求头部将Cookie 信息发送给服务器。

### Set-Cookie

------
响应首部 **Set-Cookie** 被用来由服务器端向客户端发送 cookie

#### 基本语法

```http
Set-Cookie: <cookie-name>=<cookie-value> 
Set-Cookie: <cookie-name>=<cookie-value>; Expires=<date>； Max-Age=<non-zero-digit>； Domain=<domain-value>； Path=<path-value>；Secure； HttpOnly
```

#### 指令大概分类

##### 设置Cookie有效时间

​	**Expires** = <date> | 可选

​	cookie 的最长有效时间， 形式符合HTTP-date规范的时间戳. 参考 [Date](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Date) （格式为 new Date().toUTCString() ）。如果没有设置该属性和(Max-Age)，那么表示这是一个 **会话期 cookie** .  

​	**Max-Age**=<non-zero-digit> | 可选 

​	在 cookie 失效之前需要经过的**秒**数。 老式浏览器（ie6,ie7,ie8）中不支持这个属性.  则可以用 Expires 来替代。假如同时存在 `Expires` 和`Max-Age` 那么后者优先级更高；



##### 设置 Cookie 访问以及安全相关的指令 

​	**HttpOnly** | 可选

​	设置了 HttpOnly 属性的 cookie 不能使用 JavaScript 经由  [`Document.cookie`](https://developer.mozilla.org/zh-CN/docs/Web/API/Document/cookie) 属性、[`XMLHttpRequest`](https://developer.mozilla.org/zh-CN/docs/Web/API/XMLHttpRequest) 和  [`Request`](https://developer.mozilla.org/zh-CN/docs/Web/API/Request) APIs 进行访问，以防范跨站脚本攻击（[XSS](https://developer.mozilla.org/en-US/docs/Glossary/XSS)）。

​	**Secure** | 可选

​	一个带有安全属性的 cookie 只有在请求使用SSL和HTTPS协议的时候才会被发送到服务器。然而，保密或敏感信息永远不要在 HTTP cookie 中存储或传输，因为整个机制从本质上来说都是不安全的，比如前述协议并不意味着所有的信息都是经过加密的。

> **注意：**非安全站点（http:）已经不能再在 cookie 中设置 secure 指令了（在Chrome 52+ and Firefox 52+ 中新引入的限制）。



##### Cookie 的作用域相关的指令

------

**Path** | 可选

指定一个 URL 路径，这个路径必须出现在要请求的资源的路径中才可以发送 Cookie 首部。 

​	示例：假如现在有两个请求地址： /user，/user/update， 后端响应中设置Set-Cookie 为如下：

​	服务端设置：以NodeJS为例：

```javascript
res.setHeader('Set-Cookie', `gab=123456;path=/user/update`);
```

​	响应头:

```http
Set-Cookie: gab=123456; path=/user/update;
```

​	通过上述设置后， **/user** 对应请求时浏览器不会添加 cookie请求头字段；

**Domain** | 可选

指定 cookie 可以送达的主机名	。假如没有指定，那么默认值为当前文档访问地址中的主机部分（但是不包含子域名）。假如指定了域名，那么相当于各个子域名也包含在内了。例如：指定 domain = study.com 那么 my.study.com子域名对应也会发送 cookie

#### 会话期 cookie

------

​	会话期 cookies 将会在客户端关闭时被移除。 会话期 cookie 不设置 Expires 或 Max-Age 指令。注意浏览器通常支持会话恢复功能。

```http
Set-Cookie: gab=123456; HttpOnly; Path=/
```

#### 持久化 cookie

------

​	持久化 Cookie 不会在客户端关闭时失效，而是在特定的日期（Expires）或者经过一段特定的时间之后（Max-Age）才会失效。

```http
Set-Cookie: gab=123456;Expires=Sun Dec 16 2018 16:36:34 GMT+0800;
```

> **提示：**当Cookie的过期时间被设定时，设定的日期和时间只与客户端相关，而不是服务端。





