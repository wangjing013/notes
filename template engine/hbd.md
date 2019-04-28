# Handlebars

​	Handlebars 一种模板语法，弱逻辑语法 。让您可以有效地构建语义话模板.

## 快速起步

### Install

------

常用的三种方式

#### NPM

------
```javascript
npm install --save handlebars
```

如果在node或webpack中使用可以通过如下方式

```javascript
require('handlebars');
```

或

```javascript
require('handlebars/runtime');
```

> 如果是在浏览器中使用的，可以直接去 node_modules/handlebars/dist/ 目录下进行复制JS文件 

#### Bower

------

```javascript
bower install --save handlebars
```

​	默认的bower库是umd风格的，因此这应该可以在global、AMD模块系统中正常工作。handlebars.js和handlebar .runtime.js是主要的源文件.

#### CDNs

------

[cdnjs](https://cdnjs.com/libraries/handlebars.js)

## 快速使用

------

### 表达式

------

handlebars 模板看起来像常规的 HTML代码，在其中嵌入表达式。

```javascript
const handlebars = require('handlebars');

let source ="<div class='entry'>" +
            "    <h1>{{title}}</h1>" +
            "    <div class='body'>" +
            "        {{body}}" +
            "    </div>" +
            "</div>";
let context  = 
    {	
        title: "My New Post", 
     	body: "This is my first post!"
    };
let template = handlebars.compile(source);
let html     = template(context);
console.log(html);
```

​	**输出**

```html
<div class="entry">
    <h1>My New Post</h1>
    <div class="body">
        This is my first post!
    </div>
</div>	
```

`{{ }}` 表示 handlebars 表达式，如果输入值包含html标签 ``<p></p>`` 会被转义后再输出

### HTML 转义

------

默认 ``{{}} ``会对值进行转义， 如果不想对HTML进行转义（输入什么就原样输出）可以使用 `` {{{ }}}``

​	**模板**

```handlebars
<div class="entry">
  <h1>{{title}}</h1>
  <div class="body">
    {{{body}}}
  </div>
</div>
```

​	**渲染的内容  - Context 模板上下文对象**

```json
{
  title: "All about <p> Tags",
  body: "<p>This is a post about &lt;p&gt; tags</p>"
}
```

​	**模板渲染后的输出的HTML代码**

```html
<div class="entry">
  <h1>All About &lt;p&gt; Tags</h1>
  <div class="body">
    <p>This is a post about &lt;p&gt; tags</p>
  </div>
</div>
```

​	**完整代码**

```javascript
const handlebars = require('handlebars');

let source = "<div class=\"entry\">\n" +
    "    <h1>{{title}}</h1>\n" +
    "    <div class=\"body\">\n" +
    "        {{{body}}}\n" +
    "    </div>\n" +
    "</div>";

let context  = {
    title: "All about <p> Tags",
    body: "<p>This is a post about &lt;p&gt; tags</p>"
};
let template = handlebars.compile(source);
let html     = template(context);
console.log(html);
```

​	为什么默认会转义？主要防止常见的XSS攻击，不想转可以使用 Handlebars 提供内置函数  Handlebars.SafeString . 

​	Handlebars 不会再对 `Handlebars.SafeString` 安全字符串进行编码。如果你写的 helper 用来生成 HTML，就需要返回一个 `new Handlebars.SafeString(result)`。在这种情况下，你就需要手动的来编码参数了。

​	比如想动态生成链接可以用下面的方式，

```javascript
Handlebars.registerHelper('link', function(text, url) {
  // 防止 text 或 url 包含不安全的字符 或 存在需要转义的
  text = Handlebars.Utils.escapeExpression(text);
  url  = Handlebars.Utils.escapeExpression(url);

  var result = '<a href="' + url + '">' + text + '</a>';

  return new Handlebars.SafeString(result);
});
```

​	**完整案例**

```javascript
const Handlebars = require('handlebars');

Handlebars.registerHelper('link', function(text, url) {
    // 防止 text 或 url 包含不安全的字符 或 存在需要转义的
    text = Handlebars.Utils.escapeExpression(text);
    url  = Handlebars.Utils.escapeExpression(url);

    var result = '<a href="' + url + '">' + text + '</a>';

    return new Handlebars.SafeString(result);
});

let source = "<div class='links'> {{link text url}}</div>";
let context  = {
    url: 'www.jdong.com',
    text: '手机'
};
let template = Handlebars.compile(source);
let html     = template(context);

console.log(html);
```

​	**输出**

```html
<div class='links'>
	<a href="www.jdong.com">手机</a>
</div>
```

这样来编码传递进来的参数，并把返回的值标记为 安全，这样的话，即便不使用用“{{{”，Handlebars 也不会再次编码它了。

### 块级表达式

------

块级表达式允许你定义一个helpers，并使用一个不同于当前的上下文（context）来调用你模板的一部分。

**语法**

```handlebars
{{#**}} {{/**}}
```

现在考虑下这种情况，你需要一个helper来生成一段 HTML 列表：

```html
{{#list people}}{{firstName}} {{lastName}}{{/list}}
```

并使用下面的上下文（数据）：

```javascript
{
  people: [
    {firstName: "Yehuda", lastName: "Katz"},
    {firstName: "Carl", lastName: "Lerche"},
    {firstName: "Alan", lastName: "Johnson"}
  ]
}
```

此时需要创建一个 名为 `list` 的 helper 来生成这段 HTML 列表。这个 helper 使用 `people` 作为第一个参数，还有一个 `options` 对象（hash哈希）作为第二个参数。这个 options 对象有一个叫 `fn` 的属性，你可以传递一个上下文给它（fn），就跟执行一个普通的 Handlebars 模板一样：	

```javascript
Handlebars.registerHelper('list', function(items, options) {
  var out = "<ul>";

  for(var i=0, l=items.length; i<l; i++) {
    out = out + "<li>" + options.fn(items[i]) + "</li>";
  }

  return out + "</ul>";
});
```

执行之后，这个模板就会渲染出：

```
<ul>
  <li>Yehuda Katz</li>
  <li>Carl Lerche</li>
  <li>Alan Johnson</li>
</ul>
```

**注意**，因为在你执行 `options.fn(context)` 的时候，这个 helper 已经把内容编码一次了，所以 Handlebars 不会再对这个 helper 输出的值进行编码了。如果编码了，这些内容就会被编码两次！

比如输入内容为 ``<p>123456</p>`` 经过options.fn 后编码成 ``&lt;p&gt;123456 &lt;/p&gt;``  如果对与内容是已编码的由于 Handlebars 不知道，会对输入内容再进行编码   ``&amp;lt;p&amp;gt;123456 &amp;lt;/p&amp;gt;``





## Partials 部分视图

Handlebars允许通过partials重复使用模板。 Partials是普通的Handlebars模板，可以由其他模板直接调用.

### **Basic Partials**

------

使用partial，必须使用 Handlebars.registerParital 注册

```handlebars
Handlebars.registerPartial('myPartial', '{{name}}')
```

上述注册了一个名为``myPartial``的 parital。 

调用方式：

```handlebars
{{> myPartial }}
```

当parital 执行时，它对应的上下文对象就是当前的执行环境。

### **Dynamic Partials**

------

为什么需要引入动态 partials ?  顾名思义动态言外之意就是可变，那么如果某个页面中需要根据不同的状态来渲染不同的内容。例如常见的根据登录状态选择显示不同的内容。

可以使用子表达式的方式来实现动态选择并执行的 partial ， 使用方式

```handlebars
{{> (whichPartial) }}
```
当前渲染 partial ，根据 whichPartial 方法返回的名字决定。

**注意:** 子表达式不支持变量的方式，所以``whilPartial`` 必须是一个函数。 如果一个简单的变量表示 partial 名字，需要借助 ``lookup`` helper 来解决

```handlebars
{{> (lookup . 'myVariable') }}
```

### **Partial Contexts**

------

默认情况下partial上下文就是当前执行partial上下文对象，如果想自定义上下文对象可以通过如果下方式进行指定。
```handlebars
{{> myPartial myOtherContext }}
```

## Partial Parameters

------

如果想给 partials 传递参数，可以通过如下方式
```handlebars
{{> myPartial parameter=value }}
```
比如 把父上下文对象中的数据暴露给 partial  非常有用。
```handlebars
{{> myPartial name=../name }}
```

如果希望传递多个参数可以使用如下格式：

```handlebars
{{> myPartial parameter1=value   parameter2=value  parameter3=value}}
```

### **Partial Blocks**

------

基本语法

```handlebars
{{#> myPartial }}
	...
{{/myPartial}}
```

实例：

```
{{#> myPartial }}
  Failover content
{{/myPartial}}
```

​	运行时会尝试查找名为 ``myPartial`` 的partial， 如果没有找到不会报错而是会将``Failover content``输出.  

​	块语法也可用于将模板传递给 partial，这里需要执行名为 ``@partial-block`` 的partial

实例： 定义一个partial 名字为 layout.hbs，内容包含如下

```handlebars
Site Content
{{> @partial-block }}
```

​	执行 partial

```handlebars
{{#> layout }}
  My Content
{{/layout}}
```

​	会渲染：

```
Site Content
My Content
```

​	@partial-block 表示块中的模板内容。



## Inline Partials

​	通过 ``inline``修饰符 ，定义块级区域的模板 

```handlebars
{{#*inline "myPartial"}}
  My Content
{{/inline}}

{{#each children}}
  {{> myPartial}}
{{/each}}

```