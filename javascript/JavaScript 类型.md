JavaScript 类型

原始类型 （primitive type）

​	string，number，boolean，null，undefined，Symbol

对象类型 （object type）

​	对象包括（函数，数组，日期，正则，错误对象）

原始类型中 null，undefined 是代表两种特殊类型的唯一的成员；



可变(mutable)类型： 对象属于可变类型；

```javascript
var o = {
   age: 20
}
o.age // => 20
o.age = 30;
o.age // => 30
```

不可(immutable)变类型：数字，布尔值，null，undefined属于不可变类型 比如修改一个数值的内容本身就说不通。字符串可以看成由字符组成的数组，你可能认为它是可变的。然而在JavaScript中，字符串是不可变的：因为JavaScript中并没有提供修改已知字符串的文本内容的方法。

可拥有方法的类型

不可拥有方法的类型



