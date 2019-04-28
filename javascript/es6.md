# ES6

[TOC]

------



## let	

​	let 与 var 用法类似``var``，但两者间使用还是有比较大的区别；

**1 定义的变量只有``let``代码块中有效。**

```javascript
{
  let a = 10;
  var b = 1;
}

a // ReferenceError: a is not defined.
b // 1
```

**2 定义参数名不能重复声明。**

```javascript
let a = 0;
let a = 1; 

//Uncaught SyntaxError: Identifier 'a' has already been declared
```

**3 不存在变量提升**

```javascript
console.log(b);
var b = 1;
// undefined

console.log(b);
let b = 1;	
// Uncaught ReferenceError: b is not defined
```

**4 暂时性死区**

​	ES6 明确规定，如果区块中存在`let`和`const`命令，这个区块对这些命令声明的变量，从一开始就形成了封闭作用域。凡是在声明之前就使用这些变量，就会报错。

​	在代码块内，使用`let`命令声明变量之前，该变量都是不可用的。这在语法上，称为“暂时性死区”（temporal dead zone，简称 TDZ）。

```javascript
if (true) {
  // TDZ开始
  tmp = 'abc'; // ReferenceError
  console.log(tmp); // ReferenceError

  let tmp; // TDZ结束
  console.log(tmp); // undefined

  tmp = 123;
  console.log(tmp); // 123
}
```



## const

## 解构赋值

------

结构分类：

- 数组结构赋值
- 对象解构赋值
- 字符串解构赋值
- 布尔值解构赋值
- 函数参数结构赋值
-  数值结构赋值



## 数组扩展

## 函数扩展

​	默认参数

## 对象扩展

------

​	简洁语法

```
属性和方法
let a = 3,b = 4;

let es6 = {
    
}
```

## Symbol

------

概述:

> 通过Symbol声明的值， 其值独一无二的；

基本使用：

```javascript
// 验证生成的值 - 是否独立无二
let a1 = Symbol();
let a2 = Symbol();
a1 === a2; // false

```

通过指定一个key的方式声明，这样也方便在其它地方获取到该生成的值

```
let a1 = Symbol.for('a3');
let a2 = Symbol.for('a3');
a1===a2;  // true
```

最常用的用途， 声明对象属性名

```
let a1 = Symbol.for('a3');
let o = {
    [a1]: "我是一个Symbol声明的值"
}
```

这样有什么好处尼？比如防止定义对象在继承中属性被覆盖



> **注意： Symbol 声明的属性，不能通过 in ，keys，of， entries 等方式访问到。**



获取 **Symbol** 属性方式： 

​	第一种方式： 通过Object.getOwnPropertySymbols 返回一个数组

```javascript
let a3 = Symbol.for('a3');
let o = {
    [a3]: "我是一个Symbol声明的值"
}
//
Object.getOwnPropertySymbols(p)  // 返回的是一个数组
[Symbol(a3)]

```

​	上面这种方式只能获取 Symbol 声明的属性，那么是不是获取非 Symbol还要去用其它方式结合尼？ 比如  Object.getOwnPropertyNames()、Object.keys 等. 那么看第二种方式；

第二种方式： 使用  [Reflect](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Reflect)  对象， Reflect 也是ES6新增一个内置对象；  

```javascript
let a3 = Symbol.for('a3');
let o = {
    a3: '1',
    [a3]: "我是一个Symbol声明的值"
}
Reflect.ownKeys(o)
// ["a3", Symbol(a3)]
```

## 数据结构

------

### Set 	

> **Set** 对象允许你存储任何类型的唯一值，无论是[原始值](https://developer.mozilla.org/en-US/docs/Glossary/Primitive)或者是对象引用。

*简述：**

`Set`对象是值的集合，你可以按照插入的顺序迭代它的元素。 Set中的元素只会出现一次，即 Set 中的元素是唯一的。

#### 语法

```javascript
new Set([iterable]);
```

**参数** 

​	iterable : 如果传递一个[可迭代对象](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/for...of)，它的所有元素将被添加到新的 **Set**中。如果不指定此参数或其值为null，则新的 **Set**为空。

**返回值**

​	一个新的 **Set** 对象

------



##### 属性

`Set.length`

​	length属性的值为0.

`get Set[@@species]`

​	构造函数用来创建派生对象.

[`Set.prototype`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Set/prototype)

​	表示Set构造器的原型，允许向所有Set对象添加新的属性。

##### 实例属性

​	所有Set实例继承自 `Set.prototype`.

**属性：** 

`Set.prototype.constructor`

​	返回实例的构造函数。默认情况下是[`Set`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Set)。

[`Set.prototype.size`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Set/size) | **readonly**

​	返回`Set`对象的值的个数。

**方法：**

​	第一类：**集合的基本操作，增 、删除、查**

​	[Set.prototype.add(value)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Set/add)

​		在`Set对象尾部添加一个元素。` 返回值 `Set对象。`

 	[Set.prototype.delete(value)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Set/delete)

​		同集合中根据传入值进行删除，成功则返回 `true` 否则返回 `false` 

​	返回 `false` 情况，比如元素在集合中不存在

​	[Set.prototype.clear()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Set/clear)

​		移除`Set`对象内的所有元素。

​	[Set.prototype.has(value)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Set/has)

​		返回一个布尔值，表示该值在`Set中存在与否。`



​	 第二类：**迭代**

​	[Set.prototype.keys()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Set/keys)

​		与**values()**方法相同，返回一个新的迭代器对象，该对象包含Set对象中的按插入顺序排列的所有元素的值。

​	[Set.prototype.entries()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Set/entries)

​		返回一个新的迭代对象（☞ 同一个set对象调用两次entries 返回对象不同的）， 该对象包含Set对象中的按插入顺序排列的所有元素的值的[value, value]数组，每个值的键和值相等。

​	[Set.prototype.forEach(callbackFn, thisArg)](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Set/forEach)   

​	按照插入顺序，为Set对象中的每一个值调用一次callBackFn。如果提供了**thisArg** 参数，回调中的this会是这个参数。

​	[Set.prototype.values()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Set/values)

​	返回一个新的迭代器对象，该对象包含Set对象中的`按插入顺序排列的`所有元素的值。

​	[Set.prototype[@@iterator]()](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Set/@@iterator)   该迭代方法默认就是 values 方法

​	返回一个新的迭代器对象，该对象包含Set对象中的`按插入顺序排列的`所有元素的值。 使用方式  set[Symbol.iterator]

#### Set 最常用场景：

------

​	元素去重，在元素去重的时是不会进行元素类型的转换；也就是在元素间比较时用 `===` 方式

​	

### WeakSet 

------

​	WeakSet  对象是一组键/值对的集合，其中的键是弱引用的。其键必须是对象，而值可以是任意的。

​	WeakSet  可以理解为 Set 的阉割版本，

与Set区别：

- 它只支持存入对象；
- 没有size属性、 clear方法、以及不支持遍历；
- 有 get 方法

### Map

------

​	**Map** 对象保存键值对。任何值(对象或者[原始值](https://developer.mozilla.org/en-US/docs/Glossary/Primitive)) 都可以作为一个键或一个值。

**语法：**

```javascript
new Map([iterable])
```

**参数**:

​	iterable:  Iterable 可以是一个[`数组`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Array)或者其他 iterable 对象，其元素或为键值对，或为两个元素的数组。

​	 每个键值对都会添加到新的 Map。`null` 会被当做 `undefined。`

### WeakMap

------





### Array 与 Map 对比

------

数据结构的横向对比， 增、查、删、改

```javascript
// 增加
{
    
    
}
// 查

// 删

// 改

```

### Array 与 Set 对比

------

```javascript
// 增加

// 查

// 删

// 改

```

### Map 、Set、Object 对比 

------

```javascript
// 增加

// 查

// 删

// 改
```

 

## Proxy 代理、Reflect 映射

------



## Class

------



## Promise 

------

> 异步编程的一种解决方案；

什么是异步？

​	

Promise 基本用法

​		

​		then

​		catch

​		all

​		race



## Iterator 和 for...of

------

​	什么是 Iterator 接口？

​		约定各种数据集合的遍历方式。

​		

## Generator

------

​	高级异步编程方案；



## Decorator

------

- 是一个函数
- 修改类的行为（扩展类的行为）
- 修改类的行为（只能作用于类中 ）



第三方修饰器： core-de

​	

## Module 

​	import  export

​	export default

​	export 



## async 和 await



## ArrayBuffer



















































