



# Class 与 Style

> 主要用途操作 class  和 style 内联样式。

Useage

​	因为class 和 style 两者都属于Element  attribute ，所以可以通过v-bind 来处理它们

用法：

> v-bind:class ="expression"
> v-bind:style ="expression"

`expression`  可以是字符串、对象、数组 

### 绑定 Class 属性

------

#### 对象语法

##### 方式一：

```html
// template
<div id="app">
       <div class="default" :class="{'didi-orange': isRipe, 'didi-green': isNotRipe}">学习Vue</div>
</div>
```

```javascript
// javascript
new Vue({
    el: '#app',
    data: function () {
        return {
            isRipe: true,
            isNotRipe: false
        }
    }
});
```

最终渲染为：

```html
<div class="default didi-orange">学习Vue</div>
```

当被观察属性 isRipe 和  isNotRipe 改变时， class 属性对应的内容相应变化； 假如 isNotRipe 改为 true，calss 渲染值为 "default didi-orange didi-green" ;  

##### 方式二

直接绑定数据中一个对象

```html
// template
<div id="app">
       <div class="default" :class="ddfe">学习Vue</div>
</div>
```

```javascript
new Vue({
	el: '#app',
 	data: function () {
     	return {
        	ddfe: {
                'didi-orange': true,
                'didi-green': false
        	}
		}
	}
});
```

##### 方式三

绑定一个返回对象的计算属性， 由于计算属性能根据被观察的属性值变化而动态更新，这也是最常用的一种模式；

```javascript
new Vue({
    el: '#app',
    data: function () {
        return {
            didiAge: 4,
            ddiMember: 6000
        }
    },
    computed: {
        ddfe: function(){
            return {
                'didi-orange': this.didiAge > 3 ? true:false,
                'didi-large': this.ddiMember > 1000 ? true: false 
            }
        }
    },
});
```

### 数组语法

------

如需要添加 class 列表， 可以用数组的语法；

```html
// template
<div id="app">
       <div class="default" :class="[class1, class2]">学习Vue</div>
</div>
```

```javascript
// javascript
new Vue({
    el: '#app',
    data: function () {
        return {
            class1: 'class1',
            class2: 'class2',
            class3: 'class2',
            bool1: true,
            bool2: true,
            bool3: false
        }
    }
});
```

可以使用三目运算符，根据条件来切换列表中的选项，示例代码如下：

```html
<div class="default" :class="[class1, bool ? class2 :'']">学习Vue</div>
```

当有多个条件的 class时，可以使用对象写法

```html
<div class="default" :class="[class1, {class2: bool2, class3:bool3 }]">学习Vue</div>
```



### Style 绑定样式

------

使用对象的方式添加样式