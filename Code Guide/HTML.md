# HTML

## 语法

* 使用两个软空格作为缩进
* 嵌套元素应该缩进一次(一般两个空格 切记不是 tab)
* 在属性上,使用双引号而不是单引号
* 不要在自闭元素中包含尾部斜杆(HTML5中称为可选 常见的 img input)

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Page title</title>
  </head>

  <body>
    <img src="images/company-logo.png" alt="Company">
    <h1 class="hello-world">Hello, world!</h1>
  </body>
</html>
```

## 使用标准的文档类型 -  HTML5 DOCTYPE

> 在每个HTML页面的开头使用这个简单的doctype，可以在每个浏览器中执行标准模式和更一致的呈现。

```html
<!DOCTYPE html>
<html>
  <head>
  </head>
</html>
```

## 指定语言属性 - lang

```html
<html lang="en-us">
  <!-- ... -->
</html>
```



## IE 兼容模式

> Internet Explorer支持使用文档兼容性<meta>标记来指定页面应该呈现为哪个版本的IE。
>
> 除非情况需要，它是最有用的指示IE使用最新的支持的 edge模式。

```html
<meta http-equiv="X-UA-Compatible" content="IE=Edge">
```



## 指定字符编码

> 通过声明显式字符编码，可以快速轻松地确保正确呈现内容。

```html
<!DOCTYPE html>
<html lang="en-us">
  <head>
    <meta charset="UTF-8">
  </head>
</html>
```

## 引入 CSS 和 JavaScript 文件时 

> 根据HTML5规范,通常在包含CSS和JavaScript文件时不需要指定类型,因为text/css 和 text/javascript是它们各自的默认值.

```html
<!-- External CSS -->
<link rel="stylesheet" href="code-guide.css">

<!-- In-document CSS -->
<style>

  /* ... */

</style>

<!-- JavaScript -->
<script src="code-guide.js"></script>

```

 ## Attribute order 

> HTML属性应该按照一定的顺序, 并于阅读代码.

- class
- id, name
- data-*
- src, for, type, href,value
- title, alt
- role, aria-*

类可以构成可重用的组件,所以排放第一位.  ID更具体，应谨慎使用，因此排到第二位。

## Boolean attributes

> 一个boolean属性使用时不需要声明值.
> **XHMLT**  需要你声明一个值, 但是 **HTML5** 中没有这样的要求.

这是 WhatWG 关于布尔属性的部分

> The presence of a boolean attribute on an element represents the true value, and the absence of the attribute represents the false value.

在一个元素上存在布尔值属性就表示为 true  , 不存在表示 false

```html
<input type="text" disabled>
<input type="checkbox" value="1" checked>
<select>
  <option value="1" selected>1</option>
</select>
```

## Reducing markup

减少使用标签

```	html
<!-- Not so great -->
<span class="avatar">
  <img src="...">
</span>

<!-- Better -->
<img class="avatar" src="...">
```

## JavaScript generated markup 

不要在 JavaScript 文件中创建 HTML标记，这样很难查找和维护 并且低性能的. 尽可能避免使用.



