# v-model

## 概述： 
- 适用对象：

  - `<input>`
  - `<select>`
  - `<textarea>`
  - `components`
- 返回结果：

  - 随表单控件类型不同而不同
    例如 type ="text" 时则返回 字符串， type="checkbox" 时则返回值为数组等
  
- 修饰符：

  - .lazy - 取代 input 监听 change 事件
  - .number - 输入字符串转为有效的数字
  - .trim - 输入首尾空格过滤

- 用法：

  在表单控件上或者组件上创建双向绑定。

### v-model 在表单中的应用：

***示例一：*** 

```html
// template  
<div id="app">
        <form action="" method="get">
            <div>
				<label>用户名:</label><input type="text" v-model="name">
            </div>
            <div>
                <label>性别:</label>
                <label>男:</label><input type="radio" name="sex" v-model="sex" value="男">
                 <label>女:</label><input type="radio" name="sex" v-model="sex" value="女">
                 <label>保密:</label><input type="radio" name="sex" v-model="sex" value="保密">
            </div>
            <div>
                 <label>篮球：</label><input type="checkbox" value="basketball" v-model="likes">
                 <label>唱歌：</label><input type="checkbox" value="sing" v-model="likes">
                 <label>看书：</label><input type="checkbox" value="book" v-model="likes">
            </div>
            <div>
                 <label>特长：<textarea cols="30" rows="10" v-model="desc"></textarea>
            </div>
        </form>
    </div>
```

```javascript
// js
var vm = new Vue({
    el: '#app',
    data() {
        return {
            name: '默认值',
            sex: '',
            likes: [],
            desc: ''
        }
    }
})
```

上述完成基本`v-model`在表单上应用，当用户在表单上输入或选择值其绑定的属性相应的更改. 相比传统的使用原生、JQ来说减少繁琐操作 如 取值、赋值；

###  v-model 在这过程中做了什么  ？ 它是如何实现双向绑定咧？

------

先抛开 v-model , 我们用原生来实现这样的类似效果 ？ 先来下面的代码 

***示例二：***  

```html
<!--模板-->
<input value="默认值" oninput="handleInput(event)">
<!--输出文本框输入的值-->
<div id="output" style="color: red;">
```

```javascript
// 事件处理
var output = document.querySelector('#output')
function handleInput(event){
 	output.innerHTML = event.target.value;
}
```

​	或直接把事件处理绑定行内：

```html
<input value="默认值" 					              oninput="document.querySelector('#output').innerHTML = event.target.value">
```

**效果：**

![v-model_01](F:\kkb-notes\javascript\vue\v-model_01.gif)

其实我们可以把 input 看成一个组件：`value` 属性代表组件输入 ,`oninput`属性代表组件输出；这样大概完成简单的双向绑定；

再回头来看看  `v-model` 指令，用一个示例来看看，并使用Chrome开发者工具辅助我们理解：

```html
<!--添加 v-model-->
<input type="text" name = "name">
<!--添加 v-model-->
<input type="text" name = "name" v-model="name">
```

用 `v-model` 之前：
![v-model_02](F:\kkb-notes\javascript\vue\v-model_02.png)

用 `v-model` 之后：
![v-model_01](F:\kkb-notes\javascript\vue\v-model_01.png)

仔细观察 Event Listeners 是添加几个事件的；对比**示例二**   v-model 自动对原生表单组件的添加事件，数据同步等；

v-model 等价于这种写法：

```html
<input
  v-bind:value="name"    // 输入
  v-on:input=  "name = $event.target.value" // 输出
>
```

上面写法是不是容易理解些了，与**示例二** 代码等同.

v-model 实际是一个语法糖主要作用：

- 把输入框的数据同步到 Vue实例 data 属性中
- 同时对input，radio，select等原生的表单组件提供一些语法糖使表单操作更加方便.

### 修饰符

------
### lazy

> 把输入值同步到Vue实例Data属性中，从 input 转移到  change .

### number
> 默认情况，v-model默认返回值为字符串类型，添加number修饰符把输入值格式化并同步到 Data属性中 .



