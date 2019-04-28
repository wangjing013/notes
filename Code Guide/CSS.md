# CSS

## 语法

- 使用带有两个空格的软选项卡——这是保证代码在任何环境中呈现相同效果的唯一方法
- 当对选择器进行分组时，请将单个选择器保持为一行。
- 在声明块的左大括号前加上一个空格，以确保可读性。
- 将声明块的右括号放在新行上。
- 在 **:** 后面为每个声明添加一个空格。
- 为了更准确地报告错误，每个声明应该出现在它自己的行上。
- 用分号结束所有声明。最后一个声明是可选的，但是如果没有它，代码更容易出错。
- 逗号分隔的属性值应该在每个逗号后面包含一个空格(例如，box-shadow)。
- 在rgb()、rgba()、hsl()、hsla()或rect()值中，逗号后面不要包含空格。这有助于区分多个颜色值(逗号，无空格)和多个属性值(逗号，空格)。
- 不要在属性值或颜色参数前加上前导零(例如，.5而不是0.5和-)。5px而不是-0.5px)
- 小写所有十六进制值，例如#fff。扫描文档时，小写字母更容易识别，因为它们往往具有更独特的形状。
- 在可用的地方使用简写十六进制值，例如#fff而不是#ffffff。
- 避免为零值指定单位，例如:margin: 0;而不是margin: 0px;。

```css
/* Bad CSS */
.selector, .selector-secondary, .selector[type=text] {
  padding:15px;
  margin:0px 0px 15px;
  background-color:rgba(0, 0, 0, 0.5);
  box-shadow:0px 1px 2px #CCC,inset 0 1px 0 #FFFFFF
}

/* Good CSS */
.selector,
.selector-secondary,
.selector[type="text"] {
  padding: 15px;
  margin-bottom: 15px;
  background-color: rgba(0,0,0,.5);
  box-shadow: 0 1px 2px #ccc, inset 0 1px 0 #fff;
}
```

## 声明的顺序

相关属性声明应按顺序组合在一起：

1. Positioning
2. Box model
3. Typographic 
4. Visual

```css
.declaration-order {
  /* Positioning */
  position: absolute;
  top: 0;
  right: 0;
  bottom: 0;
  left: 0;
  z-index: 100;

  /* Box-model */
  display: block;
  float: right;
  width: 100px;
  height: 100px;

  /* Typography */
  font: normal 13px "Helvetica Neue", sans-serif;
  line-height: 1.5;
  color: #333;
  text-align: center;

  /* Visual */
  background-color: #f5f5f5;
  border: 1px solid #e5e5e5;
  border-radius: 3px;

  /* Misc */
  opacity: 1;
}
```

## Don't use @import 

与<link>相比，@ import更慢，添加额外的页面请求，并可能导致其他无法预料的问题。避免使用它们，而是选择其他方法：

- 使用多个 link 元素
- 使用预处理器(如Sass或less)将CSS编译为单个文件

## Media query placement 媒体查询展示位置

尽可能将媒体查询放在尽可能靠近相关规则集的位置。不要将它们全部捆绑在单独的样式表中或文档的末尾。这样做只会让人们在将来更容易错过它们。这是一个典型的设置。

```css
.element { ... }
.element-avatar { ... }
.element-selected { ... }

@media (min-width: 480px) {
  .element { ...}
  .element-avatar { ... }
  .element-selected { ... }
}
```

##  Prefixed properties 前缀属性

使用供应商（浏览器）前缀属性时，缩进每个属性，使声明的值垂直排列，以便于多行编辑。

```css
/* Prefixed properties */
.selector {
  -webkit-box-shadow: 0 1px 2px rgba(0,0,0,.15);
          box-shadow: 0 1px 2px rgba(0,0,0,.15);
}
```

## Single declarations  单一声明

在规则集只包含一个声明的情况下，考虑删除换行符以提高可读性和更快的编辑速度。任何具有多个声明的规则集都应该被分割成单独的行。

```css
/* Single declarations on one line */
.span1 { width: 60px; }
.span2 { width: 140px; }
.span3 { width: 220px; }

/* Multiple declarations, one per line */
.sprite {
  display: inline-block;
  width: 16px;
  height: 15px;
  background-image: url(../img/sprite.png);
}
.icon           { background-position: 0 0; }
.icon-home      { background-position: 0 -20px; }
.icon-account   { background-position: 0 -40px; }
```

## Shorthand notation  简写符号

[参考MDN上这篇文章](https://developer.mozilla.org/en-US/docs/Web/CSS/Shorthand_properties)

## Nesting in Less and Sass  在Less和Sass嵌套样式

避免不必要的嵌套，仅当必须将样式限定在父元素范围内，并且要嵌套多个元素时，才考虑嵌套。

```css
// Without nesting
.table > thead > tr > th { … }
.table > thead > tr > td { … }

// With nesting
.table > thead > tr {
  > th { … }
  > td { … }
}
```

## Operators in Less and Sass  Less 和 Sass 中的运算

为了提高可读性，请将所有数学运算包含在括号中，并在值，变量和运算符之间使用单个空格

```css
// Bad example
.element {
  margin: 10px 0 @variable*2 10px;
}

// Good example
.element {
  margin: 10px 0 (@variable * 2) 10px;
}
```

## Comments  注释

代码是由人编写和维护的。确保您的代码是描述性的、注释良好的，并且其他人可以访问。好的代码注释传达了上下文或目的。不要简单地重复组件或类名

 ```css
/* Bad example */
/* Modal header */
.modal-header {
  ...
}

/* Good example */
/* Wrapping element for .modal-title and .modal-close */
.modal-header {
  ...
}
 ```

## Class names 类名

- 保持类小写并使用破折号（不是下划线或camelCase）。破折号在相关类中起到自然断裂的作用（例如，.btn和.btn-danger）。
- 避免过度和任意的速记符号。 .btn对按钮很有用，但.s并不意味着什么。
- 尽可能简短，简洁。
- 用有意义的名字;使用结构或有目的的名称而不是表达。
- 基于最接近的父类或基类的前缀类。
- 使用.js-*类来表示行为(与样式相反)，但是不要让这些类出现在CSS中。

```css
/* Bad example */
.t { ... }
.red { ... }
.header { ... }

/* Good example */
.tweet { ... }
.important { ... }
.tweet-header { ... }
```

## Selectors 选择器 

- 保持选择器简短，并努力将每个选择器中的元素数量限制为三个。
- 避免在常见组件上使用多个属性选择器(例如，[class^="…"])。众所周知，浏览器性能会受到这些因素的影响。
- 仅在必要时(例如，当不使用前缀类时)将类限定到最近的父类。
- 使用类来替代标签属性，从而提供渲染性能

```css
/* Bad example */
span { ... }
.page-container #stream .stream-item .tweet .tweet-header .username { ... }
.avatar { ... }

/* Good example */
.avatar { ... }
.tweet-header .username { ... }
.tweet .avatar { ... }
```

## Organization 组织代码

- 按组件组织代码段。
- 开发一致的注释层次结构。
- 在为扫描较大的文档而分隔代码段时，请使用一致的空格。
- 当使用多个CSS文件时，请按组件而不是按页面来分解它们。可以重新排列页面和移动组件。

```css

/*
 * Component section heading
 */

.element { ... }


/*
 * Component section heading
 *
 * Sometimes you need to include optional context for the entire component. Do that up here if it's important enough.
 */

.element { ... }

/* Contextual sub-component or modifer */
.element-heading { ... }
```

## Editor preferences 编译器偏好设置

将编辑器设置为以下设置，以避免常见的代码不一致和脏的差异:

- 使用软制表符设置为两个空格。
- 在save上修剪尾随的空白。
- 将编码设置为UTF-8。
- 在文件末尾添加新行。

考虑记录并将这些首选项应用到项目的[.editorconfig](https://github.com/twbs/bootstrap/blob/master/.editorconfig)文件中。例如，请参见Bootstrap中的一个。了解更多关于[EditorConfig](https://editorconfig.org/)的信息。