# **ORM**

**什么是 ORM ？**

​	对象关系映射（英语：**(Object Relational Mapping**，简称**ORM**，或**O/RM**，或**O/R mapping**），是一种程序技术，用于实现面向对象编程语言里对象与关系数据间的转换 

**为什么需要引入ORM ?**

​	   让我们从O/R开始。字母O起源于"对象"(Object),而R则来自于"关系"(Relational)。几乎所有的程序里面，都存在对象和关系数据库。在业务逻辑层和用户界面层中，我们是面向对象的。当对象信息发生变化的时候，我们需要把对象的信息保存在关系数据库中。 

​        当你开发一个应用程序的时候(不使用O/R Mapping),你可能会写不少数据访问层的代码，用来从数据库保存，删除，读取对象信息，等等。你在DAL（Data Access  Layer）中写了很多的方法来读取对象数据，改变状态对象等等任务。而这些代码写起来总是重复的。 

​        ORM解决的主要问题是对象关系的映射。域模型和关系模型分别是建立在概念模型的基础上的。域模型是面向对象的，而关系模型是面向关系的。一般情况下，一个持久化类和一个表对应，类的每个实例对应表中的一条记录，类的每个属性对应表的每个字段。   

**ORM技术特点：** 
​        1.提高了开发效率。由于ORM可以自动对Entity对象与数据库中的Table进行字段与属性的映射，所以我们实际可能已经不需要一个专用的、庞大的数据访问层。 
​        2.ORM提供了对数据库的映射，不用sql直接编码，能够像操作对象一样从数据库获取数据。 

![](orm.jpg)





# Sequelize
Sequelize 是一个基于 promise 的 Node.js ORM, 目前支持 Postgres, MySQL, SQLite 和 Microsoft SQL Server. 它具有强大的事务支持, 关联关系, 读取和复制等功能.
### **安装**
```javascript
npm install --save sequelize
```
**注意:** 在项目除安装  ``Sequelize`` `` 之外也需要安装其关系数据库对应的方言。

```javascript
$ npm install --save pg pg-hstore   // Postgres
$ npm install --save mysql2			// MySQL
$ npm install --save sqlite3		// SQLite 
$ npm install --save tedious 		// Microsoft SQL Server
```

## 建立连接

​	``Sequelize`` 将在初始化时设置连接池，所以如果从单个进程连接到数据库，你最好每个数据库只创建一个实例。 如果要从多个进程连接到数据库，则必须为每个进程创建一个实例，但每个实例应具有“最大连接池大小除以实例数”的最大连接池大小。 因此，如果您希望最大连接池大小为90，并且有3个工作进程，则每个进程的实例应具有30的最大连接池大小。

```javascript
const Sequelize = require('sequelize');
const sequelize = new Sequelize('database', 'username', 'password', {
  host: 'localhost',
  dialect: 'mysql'|'sqlite'|'postgres'|'mssql',

  pool: {
    max: 5,
    min: 0,
    acquire: 30000,
    idle: 10000
  },

  // 仅限 SQLite
  storage: 'path/to/database.sqlite'
});

// 或者你可以简单地使用 uri 连接
const sequelize = new Sequelize('postgres://user:pass@example.com:5432/dbname');
```

## 测试连接

您可以使用 `.authenticate()` 函数来测试连接。

```javascript
sequelize
  .authenticate()
  .then(() => {
    console.log('Connection has been established successfully.');
  })
  .catch(err => {
    console.error('Unable to connect to the database:', err);
  });
```

## 定义模型

使用 ``sequelize.define('name', {attributes}, {options})`` 来定义.

```javascript
const User = sequelize.define('user', {
  firstName: {
    type: Sequelize.STRING
  },
  lastName: {
    type: Sequelize.STRING
  }
});

// force: true 如果表已经存在，将会丢弃表
User.sync({force: true}).then(() => {
  // 表已创建
  return User.create({
    firstName: 'John',
    lastName: 'Hancock'
  });
});
```

您可以在 [Model API Reference](http://docs.sequelizejs.com/class/lib/model.js~Model.html)  查看更多创建模型信息

## 使用查询

```javascript
User.findAll().then(users => {
  console.log(users)
})
```

您可以在 [Data retrieval](https://github.com/demopark/sequelize-docs-Zh-CN/blob/master/models-usage.md#%E6%95%B0%E6%8D%AE%E6%A3%80%E7%B4%A2/%E6%9F%A5%E6%89%BE%E5%99%A8) 上查看更多关于模型的查找器功能,如 `.findAll()` 。或者在 [Querying](https://github.com/demopark/sequelize-docs-Zh-CN/blob/master/querying.md) 上查看如何执行特定查询，如 `WHERE` 和 `JSONB` 。

## 应用全局的模型参数

Sequelize 构造函数使用 `define` 参数，该参数将用作所有定义模型的默认参数。

```javascript
const sequelize = new Sequelize('connectionUri', {
  define: {
    timestamps: false // 默认为 true
  }
});

const User = sequelize.define('user', {}); // 时间戳默认为 false
const Post = sequelize.define('post', {}, {
  timestamps: true // 时间戳此时为 true
});
```

## Promise

Sequelize 使用 [Bluebird](http://bluebirdjs.com/) promise 来控制异步控制流程。

**注意:** *Sequelize 使用 Bluebird 实例的独立副本。如果你想设置任何 Bluebird 特定的参数可以通过使用 Sequelize.Promise 来访问它。*





