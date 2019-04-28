# JavaScript 三种方法，可以确定一个值到底是什么类型。

> typeof
> instanceof
> Object.prototype.toString

## 为什么需要确定类型 ？

​	只有确定类型的情况，才知道当前对象拥有哪些功能；  比如使用 push，unshfit，shfit 等方法时，那么其必须为数组类型时才能正确使用；

​	当某些情况添加类型检查时，这样代码更加健壮，安全；

# typeof 运算符

> 返回一个值的数据类型。

基本语法：

```javascript
typeof operand
or
typeof (operand)
```

`operand` 是一个表达式，表示对象或[原始值](https://developer.mozilla.org/en-US/docs/Glossary/Primitive)，其类型将被返回。括号是可选的.

### 示例：基本数据类型

```javascript
{

    typeof 1            // "number"
    typeof Number(1)	// "number"
    typeof ''           // "string"
    typeof true         // "boolean"
    typeof null         // "object"
    typeof undefined    // "undefined"
    typeof Symbol       // "function"

}
```

当操作数（operand）为基本数据类型，其返回字符串与期望一样，也能够辨别当前操作数得类型；

这里需要提及下基本数据类型  null  ，为什么 typeof null 返回的字符串为 "object";

在 JavaScript 最初的实现中，JavaScript 中的值是由一个表示类型的标签和实际数据值表示的。对象的类型标签是 0。由于 `null` 代表的是空指针（大多数平台下值为 0x00），因此，null的类型标签也成为了 0，`typeof null`就错误的返回了"`object"`。

### 示例：引用数据类型

```javascript
	typeof new Number(1)  		//  'object'
    typeof new String()         //  'object'
    typeof new Array()          //  'object' 
    typeof new Date()         	//  'object'
    
 	typeof new Function()       //  'function'
```

从上面看出，所有通过 new 关键实例化的构造函数返回都 object 类型.  当然函数是一个例外；

从上述两个示例可以看出，当typeof 的操作数为基本数据类型、函数返回字符串能够区分其数据类型； 

那么如果需要区分引用类型时需要借助 JavaScript中另外一个运算符；



## instanceof  运算符

------

> 判断实例对象是不是类（构造函数）的实例

`instanceof`  工作原理基于原型链，也就是说如果在对象的原型链上能够找到构造函数的 prototype 对象，那么该操作就返回 true；

基本语法：**

```javascript
	object instanceof constructor
```
**参数**

- object
  要检测的对象.
- constructor
  某个构造函数

### 示例一：

------

```JavaScript
let o = new Object();
let bool = new Boolean();
let num = new Number(1);
let str = new String();
let arr = new Array();
// 自定义构造函数
function Person(){} 
function Animal(){} 

let person = new Person();
let animal = new Animal();
person instanceof Person   // true
animal instanceof Animal   // true

o instanceof Object;       // true
bool instanceof Boolean;   // true
num instanceof Number;     // true
str instanceof String;     // true
arr instanceof Array;      // true
```

这样弥补 `typeof` 在检测引用类型的时的问题； 



通过下面的示例，验证下 `instanceof` 工作原理：

```javascript
{
    function Person(){}
    function Animal(){}

    // 例如自定义定义两个类
    let person = new Person();
    let animal = new Animal();


    console.log(animal instanceof Object);  // =>  true
    console.log(animal instanceof Animal);   // true

    console.log(person instanceof Object); // =>  true
    console.log(person instanceof Person);  // true

    console.log(person instanceof Animal);  // =>  false
    console.log(animal instanceof Person);  // =>  false
}
```

上面应该跟我们预期的一样,  那么有没有通过一种办法让 

```javascript
person instanceof Animal  // => true
```

 那么来针对上面作如下调整：

```javascript
person.__proto__ = Animal.prototype;
console.log(person instanceof Animal) // => true
console.log(person instanceof Person) // => false

```

严格意义上来说, 上述这么改是没有意思；这里只是为了验证 `instanceof` 如何工作的；其次说明 `person instanceof Person` 返回 true, 则并不意味着给表达式永远返回 true， `Person.prototype` 和 `person.__proto__` 都是可变的；

### `instanceof` 和多个全局对象（多个frame或多个window之间的交互）

------

可以定义两个页面测试:   parent.html、 child.html

```html
//	parent.html

    <iframe src="child.html" onload="test()">
    </iframe>
    <script>
        function test(){
            var value = window.frames[0].v;
            console.log(value instanceof Array); // false
        }
    </script>
```

```html
//  child.html
    <script>
        window.name = 'child';
        var v = [];
    </script>
```

严格上来说`value` 就是数组，但parent页面中打印输出： **false** ；也就是 parent 页面中 Array.prototype != window.frames[0].Array.prototype ；

主要引起问题是，因为多个窗口意味着多个全局环境，不同的全局环境拥有不同的全局对象，从而拥有不同的内置类型构造函数.  

### instanceof 应用  -  检测作用域的安全

------
```javascript
function Person(name, age){
    this.name = name;
    this.age = age
}

// 正常情况下
{
	let person = new Person('托尼', 20);
	console.log(person.name,person.age);
}

// 假如在实例化时候忘记添加 new 关键字尼? 会出现什么情况尼？
{
 	let person = Person('托尼', 20);
	console.log(person.name, person.age);  // Uncaught TypeError: Cannot read property 'name' of undefined
    
    // Person 被当作普通的方法执行，其次 name ，age 被添加window上
    console.log(name, age); //=> 托尼 20
}

// 如何避免这样情况，在加 new 关键字或省略时候都能正常返回实例对象 
{
    function Person(name, age){
        if(this instanceof Person) {
            this.name = name;
    		this.age = age
            return this;
        }else {
            return new Person(name, age);
        }
	}
    
    let person = Person('托尼', 20);
	console.log(person.name, person.age);  // 托尼 20
}
```



## Object.prototype.toString 方法

------

默认情况下(不覆盖 `toString` 方法前提下)，任何一个对象调用 Object 原生的 `toString` 方法都会返回 "[object type]"，其中 type 是对象的类型；
每个类的内部都有一个 [[Class]] 属性，这个属性中就指定了上述字符串中的  type(构造函数名) ；

举个例子吧： 

```javascript
console.log(Object.prototype.toString.call([])); // [object Array]
```

上述中： `Array`  对应也就当前对象的 type，同样就是当前对象的构造函数名；

前面在说 `instanceof`  在多个作用域的情况下，尝试用这种方式解决：

```javascript
  function isArray(arr){
      return Object.prototype.toString.call(arr) === "[object Array]"
  }
  
  console.log(isArray(value)); // true
```

从输出来看时完美解决了；

为什么这种方式是可以咧？这里辨别类型根据 **构造函数名称**，由于原生数组的构造函数名称与全局作用域无关，因此使用 toString() 就能保证返回一致的值。

同理其它原生对象检测也能按照这种方式来，如  Date，RegExp，Function

```javascript
function isFunction(value) {
        return Object.prototype.toString.call(value) === "[object Function]"
    }
function isDate(value) {
        return Object.prototype.toString.call(value) === "[object Date]"
    }
function isRegExp(value) {
        return Object.prototype.toString.call(value) === "[object RegExp]"
    }

isDate(new Date()); // true
isRegExp(/\w/);		// true
isFunction(function(){}); //true

```

上述代码，可以作进一步改进，由于 isFunction、isDate、isRegExp 方法中，其实只有 [object ConstructorName]  ConstructorName不一样，可以利用工厂函数再修饰一下：

```javascript
 function generator(type){
        return function(value){
            return Object.prototype.toString.call(value) === "[object "+ type +"]"
        }
    }

 let isFunction = generator('Function')
 let isArray = generator('Array');
 let isDate = generator('Date');
 let isRegExp = generator('RegExp');

isArray([]));	// true
isDate(new Date()); // true
isRegExp(/\w/);		// true
isFunction(function(){}); //true
```

这样即使要生成其它原生对象验证函数前面时就简单多了；












