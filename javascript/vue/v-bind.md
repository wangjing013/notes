# v-bind

> v-bind 指令用于响应式更新 HTML特性 ,将一个或多个attibute, 或者一个组件prop动态绑定到表达式.

简单语法:

```html
<image v-bind:src='imageSrc'></image>
// or
<image :src='imageSrc'></image>
```

​	修饰符：

- .sync  (2.3.0+) 语法糖，会扩展成一个更新父组件绑定值的 `v-on` 侦听器
- .prop  被用于绑定 DOM 属性 (property)。([差别在哪里？](https://stackoverflow.com/questions/6003819/properties-and-attributes-in-html#answer-6004028))

