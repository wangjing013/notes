# Transition

​	通常，当CSS属性的值发生变化时，呈现的结果会立即更新，受影响的元素会立即从旧属性值更改为新属性值.  值的更新可以理解从一种状态更新到另一种状态.这样对于用来说是没有感知，就好比某个东西突然出现在面前，会感觉很突兀和生硬；

​	css3 中引入 transition 属性，下面W3C中对 transition 介绍

> CSS Transitions allows property changes in CSS values to occur smoothly over a specified duration.

​	CSS Transitions 允许在指定的持续时间内平稳地进行CSS属性值的更改。

### 1 语法

------

```css
transition: [ none | <single-transition-property> ] || <time> || <timing-function> || <time>

transition:
	transition-property 
	transition-duration
	transition-timing-function
	transition-delay 
```

#### 1.1  transition-property          

​	指定元素哪些属性需要应用过渡效果；

​	**可选值**：

- none    
- all
- custom-ident (一个或多个属性名，如果指定多个属性名，属性名之间用逗号隔开)

```css
div {
    transition-property: none;
    transition-property: all;
    transition-property: width,padding;
}
```

#### 1.2  transition-duration        

​	指定转换完成的时间

​	默认值： '0 s' 

​	`s`  表示时间单位秒， 时间单位可选 ： 's' ，'ms'，一般推荐使用 's';

```css
div {
    transition-duration: .3s;        
}
```

​	上述需要应用过渡效果的属性，完成时间都是 0.3s;  如果给每个属性指定一个时间也可以分别指定，如下：

```css
div {
	transition-property: width,padding;
    transition-duration: .3s, .5s;        
}	
```

​	如果不想一一指定也可以这样

```css
div {
	transition-property: width,padding,left, top;
    transition-duration: .3s, .5s;        
}
```

​	`width` ， `left`  对应 .3s ,  `padding`，`top` .5s;  计算方式有点和 margin，padding应用方式一样；

#### 1.3 transition-timing-function   

​	表示如何计算转换期间使用的中间值。它允许转换以在其持续时间（transition-duration 指定值）内改变速度。这些效果通常称为缓动函数。

​	这里可能比较抽象，我们举一个贴近生活例子， 比如张三从出去办事， 从A地点 - D地点，其中途经过的点 B ，C (中间值) ；   张三 从 A->B 选择加速方式，B-C 选择减速，C-D 匀速； 不管使用那种方式，最终从A-D时间刚好为  transition-duration 指定即可。

​	可选值：

- [ease](https://www.w3.org/TR/css-easing-1/#valdef-cubic-bezier-timing-function-ease)
- [ease-in](https://www.w3.org/TR/css-easing-1/#valdef-cubic-bezier-timing-function-ease-in)
- [ease-out](https://www.w3.org/TR/css-easing-1/#valdef-cubic-bezier-timing-function-ease-out)
- [ease-in-out](https://www.w3.org/TR/css-easing-1/#valdef-cubic-bezier-timing-function-ease-in-out)
- [cubic-bezier()](https://www.w3.org/TR/css-easing-1/#funcdef-cubic-bezier-timing-function-cubic-bezier)

#### 1.4 transition-delay     指定过渡延迟开始时间

​	指定过渡从应用到开始执行的时间； 默认值 '0s' 也就是说当指定的属性值发生变化的时候，过渡效果立即执行； 否则指定的值为从更改属性值到应用过渡效果的一个偏移量（延迟开始时间） ；

### 2 事件

------

**transitionend**  
	过渡应用完成时调用；

​	