

[TOC]



# Type Script

## 基本语法

### 接口

​	对值所具有的*结构*进行类型检查.  在`TypeScript`里，接口的作用就是为这些类型命名和为你的代码或第三方代码定义契约。

#### 语法：

```typescript
interface InterfaceName {
	property: type;
}
```

#### 简单实例：

```typescript
interface LabelledValue {
  label: string;
}

function printLabel(labelledObj: LabelledValue) {
  console.log(labelledObj.label);
}

let myObj = {size: 10, label: "Size 10 Object"};
printLabel(myObj);
```

上面代码主要做了这些，类型检查器会查看 `printLabel`的调用. `printLabel`有一个参数，并要求这个对象参数有一个名为`label`类型为`string`的属性.  

**注意：**

1.  类型检测器不会去检查属性顺序，只要相应的属性存在并且类型也是对的就可以.    
2. 类型检测器也不会去关注传入参数对象中属性个数，编译器只会检查那些必须的属性是否存在，并且其类型是否匹配。

#### 可选属性 - ?

------

​	接口里的属性不全都是必需的. 有些是只在某些条件下存在，或者根本不存在. 可选属性在应用“option bags”模式时很常用，即给函数传入的参数对象中只有部分属性赋值了。

**定义语法**

```typescript
interface InterfaceName {
	property?: type;
}
```

定义方式跟普通接口定义差不多，只是在可选属性名字定义的后面加一个 **?** 符号.

**用途:**   

1. 可能存在的属性进行预定义.
2. 可以捕获引用了不存在的属性时的错误.

#### 只读属性 - readonly

------

`readonly`来指定只读属性,

 语法：

```typescript
interface Point {
	readonly x: number;
	readonly y: number;
}
```

例如：你可以通过赋值一个对象字面量来构造一个`Point`。 赋值后， `x`和`y`再也不能被改变了。

```typescript
let p1: Point = { x: 10, y: 20 };
p1.x = 5; // error!
```

**注意** `TypeScript` 具有 `ReadonlyArray<T>`类型，只是把所有可变方法去掉了，因此可以确保数组创建后再也不能被修改：

```typescript
let a: number[] = [1, 2, 3, 4];
let ro: ReadonlyArray<number> = a;
ro[0] = 12; // error!
ro.push(5); // error!

a = ro; // error!  
```

把整个 `ReadonlyArray` 赋值到一个普通数组也是不可以的. 但是你可以用类型断言重写：

```typescript
a = ro as number[];
```

##### `readonly` vs `const`

标识符做为变量使用的话用`const`，若做为属性则使用`readonly`。



#### 用接口描述函数类型 

------

接口也可以描述函数类型。

**语法**

```typescript
interface SearchFunc {
  (source: string, subString: string): boolean;
}

```

​	上述定义，可以把它当做只有参数列表和返回值类型的函数定义。参数列表里的每个参数都需要名字和类型。

使用方式同其它接口用法一样：

```typescript
let mySearch: SearchFunc;
mySearch = function(source: string, subString: string):boolean {
  let result = source.search(subString);
  return result > -1;
}
```

对于函数类型的类型检查来说，函数的参数名不需要与接口里定义的名字相匹配，如下

```typescript
let mySearch: SearchFunc;
mySearch = function(src: string, sub: string):boolean {
  let result = src.search(sub);
  return result > -1;
}
```

​	主要是检查器不会去找对应名称而是会对形参参数逐个进行检查，要求对应位置上的参数类型是兼容的。

​	可以定义时不指定类型，`TypeScript`的类型系统会自动推断出参数类型，函数的返回值类型是通过其返回值推断出来的.

```typescript
interface SearchFunc {
  (source: string, subString: string): boolean;
}
let mySearch: SearchFunc;
mySearch = function(src, sub) {
  let result = src.search(sub);
  return result > -1;
}
```

**注意：**

1. 函数的参数名不需要与接口里定义的名字相匹配
2. 参数类型也可以省略，`TypeScript`的类型系统会推断出参数类型
3. 函数返回值也可以省略，根据返回值进行推断

### 定义可索引的类型

​	描述那些能够“通过索引得到”的类型， 比如 a[10] 或 ageMap['daniel']. 可索引类型具有一个 **索引签名**，它描述了对象索引的类型，还有相应的索引返回值类型。 

**实例：**

```typescript
interface StringArray {
  [index: number]: string;
}

let myArray: StringArray;
myArray = ["Bob", "Fred"];

let myStr: string = myArray[0];
```

上述定义了 `StringArray` 接口.

### Class 类

------

**定义基本结构：**

```
class Greeter {
    greeting: string;
    constructor(message: string) {
        this.greeting = message;
    }
    greet() {
        return "Hello, " + this.greeting;
    }
}

```
类的基本组成：

1. 构造方法
2. 属性
3. 方法

属性：

根据修饰符不同分为：

1. 公有（public ） **默认**
2. 受保护的（protected ）
3. 私有（private ）不能在外部调用
4. 只读 （readOnly） 

按照访问方式：类属性、实例属性

#### 继承

------

基于类的程序设计中一种最基本的模式是允许使用继承来扩展现有的类。这也是类经常使用的一种方式。继承使用 `extends` 关键字。

基本结构：

```typescript
class Animal {
    move(distanceInMeters: number = 0) {
        console.log(`Animal moved ${distanceInMeters}m.`);
    }
}

class Dog extends Animal {
    bark() {
        console.log('Woof! Woof!');
    }
}

```

​	这个例子展示了最基本的继承：类从基类中继承了属性和方法。 这里， `Dog`是一个 *派生类*，它派生自 `Animal` *基类*，通过 `extends`关键字。 派生类通常被称作 *子类*，基类通常被称作 *超类*。

​	与继承有关另外一个关键字： **super**  

例如想让动物都拥有一个名字，那么**Animal**	做如下修改：

```typescript
class Animal {
  name: string;
  constructor(theName: string) {
    console.log('super');
    this.name = theName;
  }

  move (distance: number = 0) {
    console.log(`${this.name}移动了多少${distance}米`);
  }
}

class Dog extends  Animal {
  move (distance  = 5) {   // 方法重写
    super.move(distance);
  }
}
class Horse  extends  Animal {
  move (distance  = 55) {    // 方法重写
    super.move(distance);
  }
}

const dog = new Dog('狗');
dog.move(); // => super
            // => 狗移动了多少5米
const horse = new Horse('黑马');
horse.move(); // => super
            // => 黑马移动了多少55米
```

​	注意点：

- 如果需要访问父类的方法可以通过`super` 关键字访问
- 子类没有定义构造函数情况下，子类实例化时默认会调用父类的构造方法
- 如果子类显示定义构造方法，其必需显示调用父类构造函数，通过`super（）` 方式。



#### 访问修饰符

**public**  默认值

------

​	可以在任意地方进行访问。

```typescript
class Animal {
    public name: string;
    public constructor(theName: string) { this.name = theName; }
    public move(distanceInMeters: number) {
        console.log(`${this.name} moved ${distanceInMeters}m.`);
    }
}
```

**private**

------

​	只能类的内部访问（这里可以理解函数中定义变量）， 同样私有属性也不能在派生类中访问。

```typescript
class Animal {
    private name: string;
    constructor(theName: string) { this.name = theName; }
}

new Animal("Cat").name; // 错误: 'name' 是私有的.
```

**protected**

------

​	`protected`修饰符与 `private`修饰符的行为很相似，但有一点不同， `protected`成员在派生类中仍然可以访问。 属性、方法、构造方法都遵循这一点。

```typescript
class Animal {
  protected name: string;
  constructor(theName: string) { this.name = theName; }
}
class Dog extends  Animal {
  printName() {
    console.log(this.name);
  }
}
let dog = new Dog('Cat');
dog.printName();  // Cat
console.log(dog.name); // 错误
```

构造函数也可以被标记为 `protected` 。 这意味着这个类不能在包含它的类外被实例化，但是能被继承。

```typescript
class Animal {
  protected name: string;
  protected constructor(theName: string) { this.name = theName; }
}
class Dog extends  Animal {
  constructor(name: string) {
     super(name);
  }  
  printName() {
    console.log(this.name);
  }
}
let dog = new Dog('Cat');
dog.printName();  // Cat
new Animal() // 错误
```

**readOnly**

------

​	可以使用 `readonly`关键字将属性设置为只读的。 **只读属性**必须**在声明时**或**构造函数**里被**初始化**。

```javascript
class Octopus {
    readonly name: string;
    readonly numberOfLegs: number = 8;
    constructor (theName: string) {
        this.name = theName;
    }
}
let dad = new Octopus("Man with the 8 strong legs");
dad.name = "Man with the 3-piece suit"; // 错误! name 是只读的.
```



#### 参数属性

------

参数属性通过给构造函数参数前面添加一个访问限定符来声明。

参数属性可以方便地让我们在一个地方定义并初始化一个成员。也可以认为是一种简化参数定义、赋值的方式。

```typescript
class Octopus {
    readonly numberOfLegs: number = 8;
    constructor(readonly name: string) {
    }
}
```

这是对上面`Octopus` 类定义简写 ， 参数 - 构造函数参数  ，属性 - 类的属性

#### 存取器

------

TS 通过getters/setters来截取对对象成员的访问，能够有效的控制对象成员的访问。

先看下没有使用存取器的：

```typescript
class Employee {
  fullName: string;
}

const employee = new Employee();
employee.fullName = 'Bob Smith';
if (employee.fullName) {
  console.log(employee.fullName);
}
```

我们可以随意的设置 `fullName`，这是非常方便的，但是这也可能会带来麻烦

如果希望我们先检查用户密码是否正确，然后再允许其修改员工信息。这时存取器就派上用场了。

```typescript
let passcode = "secret passcode";

class Employee {
    private _fullName: string;

    get fullName(): string {
        return this._fullName;
    }

    set fullName(newName: string) {
        if (passcode && passcode == "secret passcode") {
            this._fullName = newName;
        }
        else {
            console.log("Error: Unauthorized update of employee!");
        }
    }
}

let employee = new Employee();
employee.fullName = "Bob Smith";
if (employee.fullName) {
    alert(employee.fullName);
}
```

**注意：**

1. 存取器是`ES5`中才有特性，所以编译器必须支持`ES5`或更高版本

2. 如果只提供 `get`  存取器自动被推断为 ``readonly``  ，等价于如下

   ```javascript
    Object.defineProperty(Employee.prototype, 		      "fullName", {
           get: function () {
               return this.fullName;
           },
           enumerable: true,
           configurable: true
     });
   ```

#### 静态属性

------

通过static关键字声明的属性，称为静态属性 ； 这些存现存在类本身，而不是类的实例上。 其能被类直接访问。

一般常用比如实例对象通用的数据：

```typescript
class Grid {
    static origin = {x: 0, y: 0};
    calculateDistanceFromOrigin(point: {x: number; y: number;}) {
        let xDist = (point.x - Grid.origin.x);
        let yDist = (point.y - Grid.origin.y);
        return Math.sqrt(xDist * xDist + yDist * yDist) / this.scale;
    }
    constructor (public scale: number) { }
}

let grid1 = new Grid(1.0);  // 1x scale
let grid2 = new Grid(5.0);  // 5x scale

console.log(grid1.calculateDistanceFromOrigin({x: 10, y: 10}));
console.log(grid2.calculateDistanceFromOrigin({x: 10, y: 10}));
```

#### 抽象类

------

抽象类做为其它派上类的基类使用。

它不同于接口，抽象类可以包含成员的实现细节。使用 `abstract` 关键字来定义类并申明类中的抽象方法。

**语法结构：**

```typescript
abstract class Animal {
    abstract makeSound(): void;   // 每个动物叫的方式不一样，所有定义成抽象方法
    move(): void {
        console.log('roaming the earch...');
    }
}
```

注意：

1. 抽象类中抽象方法必需在派生类中实现
2. 存在抽象方法那么类必需定义为抽象类，反之不一定需要

**抽象类vs接口**

- 类的申明使用关键字不一样，前者使用 `abstract` 后者使用 ``interface`` 
- 前者允许包含方法的具体实现，后者不行
- 抽象方法更多用于定义模板，后者更倾向与定义规则和约束

#### 高级技巧

------





### 函数

------

#### 为函数定义类型

```typescript
function add(x, y) {
    return x + y;
}
let myAdd = function(x, y) { return x + y; };

// 添加定义类型
function add(x: number, y: number): number {
	return x + y;
}
let myAdd = function(x: number, y: number): number { return x + y; };

```

​	TS 能够根据返回语句自动推断除返回值类型，因此通常可以省略它。

#### 书写完整函数类型

------

```typescript
const myAdd: (x: number, y: number) => number 
	=
function(x: number, y: number): number { return x + y; };
```

函数类型包含两部分：**参数类型**和**返回值类型**。 当写出完整函数类型的时候，这两部分都是需要的。

**参数类型：** 我们以参数类表的形式写出参数类型，为每个参数指定一个名字和类型。这个名字只是为了增加可读性。我们也可以这么写：

```typescript
let myAdd: (baseValue: number, increment: number) => number =
    function(x: number, y: number): number { return x + y; };
```

​	只要参数类型是匹配的，那么就认为它是有效的函数类型，而不在乎参数名是否正确。

**返回值类型**：如果函数没有返回任何值，必须指定返回值类型为 `void`而不能省略。

#### 可选参数

------

默认情况下，传递给一个函数的参数个数必须与函数期望的参数一致。

```typescript
function buildName(firstName: string, lastName: string) {
	return firstName + " " + lastName;
}

let result1 = buildName("Bob");  // 错误
let result2 = buildName("Bob", "Adams", "Sr.");  // 错误
let result3 = buildName("Bob", "Adams");  //  正常

```

JavaScript里，每个参数都是可选的，可传可不传。没传参的时候，它的值就是 `undefined` 。 

在 TypeScript 中可以通过在参数名旁添加 **？**实现可选参数的功能。比如,我们想让last name 是可选的：

```typescript
function buildName(firstName: string, lastName?: string) {
    if (lastName)
        return firstName + " " + lastName;
    else
        return firstName;
}

let result1 = buildName("Bob");  // 正常
let result2 = buildName("Bob", "Adams", "Sr.");  // 错误
let result3 = buildName("Bob", "Adams");  // 正常
```

**注意：** 可选参数必须跟在必须参数后面。

#### 默认参数

------

**TypeScript** 里，我们可以为参数提供一个默认值当用户没有传递这个参数或传递的值是 **undefined** 时生效。

```typescript
function buildName(firstName: string, lastName = "Smith") {
    return firstName + " " + lastName;
}

let r1 = buildName("Bob"); // "Bob Smith" 
let r2 = buildName2('Bob', undefined);  // "Bob Smith"
let r3 = buildName2("Bob", "Adams", "Sr.");  // error
let r4 = buildName('Bob', 'Adams'); // "Bob Adams" 


// 默认参数出现可选参数之前
function buildName(firstName = 'will' , lastName = "Smith") {
    return firstName + " " + lastName;
}
buildName("Bob");// error, too few parameters
buildName(undefined, 'Adams') // "will Adams"
```

**注意：** 

- 默认值参数不必放必须参数后，如果带默认值参数出现在必须参数前面，用户必须明确的传入 **undefined** 值来获得默认值。
- 默认参数都是可选的，与可选参数一样，在调用函数的时候可以省略

> *函数参数一个参数不能即可选又存在默认值 *

#### 剩余参数

------

必要参数、默认参数、可选参数有个共同点： 它们表示某一个参数。 

有时，你想同时操作多个参数，或者对传入参数个数不确定情况下。那么就可以应用 **剩余参数**  

在 **TypeScript** 里，可以把参数收集到一个变量里：

```typescript
function buildName(firstName: string, ...restOfName: string[]) {
  return firstName + " " + restOfName.join(" ");
}

let employeeName = buildName("Joseph", "Samuel", "Lucas", "MacKinzie");
```

`...`  表示这里应用剩余参数， `restOfName` 表示剩余参数的名称，方便在函数内容访问。

**注意：**

- 可选参数必须放在参数列表最后。
- 可选参数接收个数不限，可以一个都没有，也可以传入多个



### 泛型

------

如果期望返回值类型与输入类型相同，并且保持足够的复用性等那么需要使用**泛型** 

那是怎么做到返回值类型与输入类型相同，这里需要引入 **类型变量** （T 表示），它是一种特殊的变量，只用于表示类型而不是值。

例如：创建一个 **identity** 函数。这个函数会返回任何传入它的值。

不使用泛型时候：

```typescript

```

或者，使用 any 类型来定义函数：

```typescript
function identity(arg: any): any {
    return arg;
}
```

​	前者只能输入Number类型， 后续需要改成字符串只能该方法 。 **any** 类型会导致这个函数可以接收任何类型，但是返回值就不能控制。我们只知道任何类型的值都有可能被返回。

通过类型变量来改造下 **identity** ,如下：

```typescript
function identity<T>(arg: T): T {
    return arg;
}
```

泛型使用方式：

**第一种是，传入所有的参数，包含类型参数：**

```typescript
let output  = identity<String>("myString");
```

注意： 这里明确指定了T是String类型，并作为一个参数传给函数，使用了`<>`括起来而不是`()`.

**第二种方式更普遍。利用了类型推论-- 即编译器会根据传入的参数自动帮助我们确定T的类型：**

```typescript
let output  = identity("myString");
```

注意我们没必要使用尖括号（`<>`）来明确地传入类型；编译器可以查看`myString`的值，然后把`T`设置为它的类型。

如果编译器不能够自动地推断出类型的话，只能像上面那样明确的传入T的类型，在一些复杂的情况下，这是可能出现的。



### 泛型变量

------







