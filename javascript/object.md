# 对象	

## 创建对象的方式

- 字面量形式
- 构造形式

字面量方式（推荐）：

```javascript
var o = {
    name: '王景',
    age: 20
}
```

构造形式：

```javascript
var o = new Object();
o.name = '王景';
o.age = 20;
```

> 字面量形式和构造形式创建对象是一样，唯一的区别在于字面量的方式创建对象时可以同时添加多个 key/value 对， 而构造形式必须逐个去进行添加。

