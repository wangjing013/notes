# computed

## 什么是计算属性？

> 计算属性当其依赖属性的值发生变化时，这个属性的值会自动更新，与之相关的DOM也会相应的更新.

## 为什么要使用计算属性？

> 通常会在模板中绑定表达式，模板主要是用来描述试图结构的。如果模板中的表达式存在过多的逻辑，模板会变得臃肿不堪（不符合表现和业务逻辑分离），维护也会变得非常困难。当某个属性的值依赖其它属性时，我们可以使用计算属性。

示例：不使用计算属性情况下

```html
// template
<div id="app">
        <h1>不使用计算属性之前</h1>
		<div>firstName:{{firstName}}</div>
		<div>lastName:{{lastName}}</div>
        <div>
            fullName: {{firstName + lastName}}
    	</div>
</div>
```

```javascript
// js
var vm = new Vue({
    el: '#app',
    data: {
        firstName: 'harden',
        lastName: 'james'
    }
});
```

​	上述显示个人的名字信息，分别显姓，名，完整名字； 在显示完整名称时用的 `{{firstName + lastName}}`  这看上去也没有什么问题；  

​	假如现在希望在显示完整的名字时把姓和名都转为大写? 那么该如何处理尼？

​	方式一：模板出需要作如下更改

```html
<div>
	fullName：{{firstName.toUpperCase() + lastName.toUpperCase()}}</div>
```

​	方式二：用计算属性方式来处理

```html
<div>fullName：{{fullname}}</div>
```

```javascript
// js
var vm = new Vue({
    el: '#app',
    data: {
        firstName: 'harden',
        lastName: 'james'
    },
    computed: {
        fullName: function(){
            return this.firstName + this.lastName
        }
    },
});
```

​	从结果来看，这两种方式都达到同样目的；从来模板（视图）角度来看，显然计算属性的方式更加整洁，可读性更高，把计算的逻辑放到`vm` 中，这样可实现解耦从而提高组件的可复用性；  一般推荐用第二种方式；

## 计算属性的 getter, setter


```javascript
computed: {
   fullName: function(){
      return this.firstName + this.lastName
   }
}
```

上述代码中，其实只提供计算属性 getter，实际上除了 getter，我们还可以设置计算属性的 setter.  示例：

```javascript
computed: {
   fullName:{
       get: function(){
      		return this.firstName + this.lastName
   	   },
   	   set: function(newVal){
           var names = newVal.split(' ');
           this.firstName = names[0];
           this.lastName = names[1];
   	   }
   }
}
```

当设置 fullName 的值时，firstName 和 lastName 同步修改；

## 计算属性缓存 - cache 

从计算属性的特性来说，只有关联的被观察的属性的值发生变化那么计算属性才会去调用getter也就是说被重新计算，否则直接取缓存中的计算结果；

### 这样设计的好处？

假如在计算属性内执行大量耗时操作，则可能会带来一些性能问题。例如，在计算属性中进行大量循环，递归等.

### 带来的问题 ？

上面提到计算属性只有当 vm 实例中被观察的属性的值被改变时才重新执行 getter.  但是有时计算属性依赖实时的非观察属性，示例代码如下：

```html
// template
<div id="app">
	{{generatorName}}
 	{{generatorName}}
 	{{generatorName}}
</div>
```

```javascript
// js
var vm = new Vue({
    el: '#app',
    data:function() {
        return {
            prefix: '游客'
        }
    },
   computed: {
       generatorName: function(){
		  console.log('只会输出一次');
          return this.prefix + (Date.now() + 1)
       }
	},
})
```

```javascript
// output 
只会输出一次
```

我们希望每次访问都能取到最新的时间而不是取的缓存内容； Vue 提供缓存的开关，通过 cache 字段让开发者手动控制是否开启缓存；

上述示例更改如下：

```javascript
// js
 computed: {
     generatorName: {
         cache: false,
         getter: function(){ 
			// 这里为了看到明显调用,添加一个日志;
		    console.log('每次调用，都会输出');
            return this.prefix + (+Date.now())
         }
     }
 }
```

这样每次访问都能获取不同名字；

```javascript
// output 
每次调用，都会输出
每次调用，都会输出
每次调用，都会输出
```

## 常见的问题

### 计算属性 getter 不执行的情况

​	前面说到计算属性只要依赖的观察属性值变化，值发生变化时都会执行 getter； 有些情况下，虽然依赖的观察的属性值发生变化，但是getter 也不会执行；

比如：当包含计算属性的节点被移除并且模板中其它地方没有再引用该属性时，那么对应的计算属性getter 方法不会执行。 代码实例如下：

```html
<div v-if="show">
  {{totalPrice}}
</div>
```

```javascript
var vm = new Vue({
    el: '#app',
    data: {
        basePrice: 100,
        show: true
    },
    computed: {
        totalPrice: function(){
            console.log('如果我被模板中引用并且basePrice发生变化，你就会看到这句输出');
            return this.basePrice + 1;
        }
    },
    mounted(){
        setInterval(()=>{
            this.basePrice+=1;
            if(this.basePrice > 110) {
                this.show = false;
            }
        }, 1000)
    }
});
```

可以观察控制台输出，当 basePrice 累加值为 111 时，totalPrice 计算属性getter 不会被执行；同时div元素将被移除；

从这里也可以看出 Vue 是基于模板进行依赖收集，只有模板中有依赖 vm 中属性，vm 才会对属性进行观察，当观察到值发生变化时来触发视图的更新；



 

