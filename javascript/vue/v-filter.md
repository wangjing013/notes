# 过滤器

### 什么是过滤器？

> 过滤器的本质就是是一个函数

### 过滤器的作用

> 对用户的输入内容进行加工处理，并返回处理后的结果

### 定义过滤器的方式

​	根据作用范围来分，分为两种

- 全局定义

- 局部定义

  全局定义基本语法：

```javascript
 Vue.filter('filterName', func(value, [params1,params2,param3...]));
```

 		 局部定义方式：

```javascript
new Vue({
    el: '#app',
    filters:{
        filterName: function(value,[params1,params2,params3...]){}
    }
})
```

**注意：** 全局定义必须在生成实例化之前进行定义，否则实例对象内部无法使用；

### 使用

​	**方式一**：

```html
{{ content | filterName}}
<!-- 需给过滤器传参时 -->
{{content | filterName([params1,params2,params3...])}}
```
* `content` 
		需要处理的内容

* `|` 管道符
		用来连接输入内容和过滤器

* `filterName`
		定义的过滤器名称

* `params`  |  可选
		过滤器处理函数第一个参数永远是当前需要处理的内容（content），参数类型没有限制可以支持基本类型，引用类型；

熟悉Linux shell 的同学可以对管道符号（|）比较熟悉和了解，命令的上一个输出可以作为下一个命令的输入。Vuejs 过滤器中的管道符号同样支持这种方式，意味过滤器是可以进行链式调用；

​	**方式二**：链式调用

```html
{{ content | filterName1 | filterName2 | filterName3}}
```

![filter](F:\kkb-notes\javascript\vue\filter.svg)

基本语法和概念差不多，那么接下来我们来实战一下.



### 常见应用

#### 字段翻译

------

​	一般在实际应用中包括类型，状态等字段在后端存储大部分为数字，如果在前端显示成数字显然对用户来说是不友好的；以用户性别为例，来看看下面的实例：

```html
<div id="app">
    <p>{{'0' | translate(arr)}}</p>  
    <p>{{'1' | translate(arr)}}</p>
</div>
```

```javascript
Vue.filter('translate', function(value, arr){
    if(!value) return;
    for(let {key, name} of arr) {
        if(key === value) {
            return name
        }
    }
    return value;
});
```

​	![filter-02](F:\kkb-notes\javascript\vue\filter-02.png) 

#### 单词首字母大写处理

------



```html
<p>{{'james' | capitalize}}</p>
```
```javascript
Vue.filter('capitalize', function (value) {
    if (!value) return;
    return value.match(/\b\w+\b/g).map(function (item) {
        return item.replace(/^\w/, item.charAt(0).toLocaleUpperCase());
    }).join(" ")
});
```
​	![filter-01](F:\kkb-notes\javascript\vue\filter-01.png)	

#### currency 过滤器

------

```javascript
Vue.filter('currency', function (value, symbol, digit) {
    if (typeof value != 'number') return;
    symbol = symbol !== undefined ? symbol : '$';
    value = value.toString();
    let index = value.indexOf('.');
    let prefix = value; // 存储整数
    let postfix = ''; // 小数部分
    if (index !== -1) {
        prefix = value.substring(0, index);
        postfix = value.substring(index);
    }
    // 对整数部分进行分组
    let base = 3;
    if (prefix.length > base) { // 只有大于基准才需要
        let i = 0,
            start = base,
            arr = prefix.split(''),
            len = arr.length;
        // 计算偏移量，这里要注意一个点，每次插入逗号那么数组长度也变话的 例如
        /*
        *    var arr = [1,2,3,4,5,6,7,8,9,10];
        *    arr.splice(-(3 + 3 * 0  + 0),0,',');
        *    arr.splice(-(3 + 3 * 1  + 1),0,',');
        *    arr.splice(-(3 + 3 * 2  + 2),0,',');
        */
        while (len > (start = base + base * i + i)) {
            arr.splice(-start, 0, ',');
            i++;
            len = arr.length;
        }
        prefix = arr.join('');
    }

    // 小数部分处理
    if (typeof digit === 'number' && digit) {
        if (!postfix.length) {
            for (let i = 0; i < digit; i++) {
                postfix += "0";
            }
        } else {
            if (postfix.length > digit) {
                postfix = postfix.substring(0, digit);
            } else {
                let len = postfix.length;
                for (let i = 0; i < len - digit; i++) {
                    postfix += "0";
                }
            }
        }
        postfix += ".";
    }
    return `${symbol}${prefix}${postfix}`
})
```

上述的指令都是自己实现的，大家可以各自实现；