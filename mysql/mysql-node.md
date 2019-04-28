# Node.js 使用MySQL

## 安装

```javascript
npm install mysql
or
npm install mysqljs/mysql
```

## 基本介绍
​	连接mysql的驱动程序，通过JS编写的不需要进行编译即可使用。
## 简单使用方法

```javascript
var mysql      = require('mysql');  // 引入驱动
var connection = mysql.createConnection({   //创建链接
  host     : 'localhost',   // 主机
  user     : 'root',		// 登录名
  password : '123456',		// 密码
  database : 'my_db'		// 数据库名
});

connection.connect();   	// 连接  这部是可以省略的 因为query隐性创建一个连接

connection.query('SELECT 1 + 1 AS solution', function (error, results, fields) {
  if (error) throw error;
  console.log('The solution is: ', results[0].solution);
});	// 查询

connection.end();			// 关闭连接
```

上面这个实例，可以学习大概流程：

​	1	连接调用的方法会依次按照队列顺序执行

​	2	使用 ``end()`` 方法关闭连接，这样可以确保退出前执行完所有请求

## 建立连接

​	推荐的连接方式

```javascript
var mysql      = require('mysql');
var connection = mysql.createConnection({
  host     : 'localhost',
  user     : 'root',
  password : '123456',
  database : 'my_db'		// 数据库名
});

connection.connect(function(err) {
  if (err) {
    console.error('error connecting: ' + err.stack);
    return;
  }
  console.log('connected as id ' + connection.threadId);
});
```

​	当然也可以通过调用 ``query`` 隐式建立连接

```javascript
var mysql      = require('mysql');
var connection = mysql.createConnection(...);

connection.query('SELECT 1', function (error, results, fields) {
  if (error) throw error;
  // connected!
});
```

## Connection options

- `host`: 连接数据库主机名字(Default: `localhost`)
- `port`: 连接端口 (Default: `3306`)
- `localAddress`: TCP连接
- `socketPath`: 要连接到的unix域套接字的路径。当使用“host”和“port”时将被忽略
- `user`:   用户名.
- `password`:  密码.
- `database`:  数据库名.
- `charset`: The charset for the connection. This is called "collation" in the SQL-level of MySQL (like `utf8_general_ci`). If a SQL-level charset is specified (like `utf8mb4`) then the default collation for that charset is used. (Default: `'UTF8_GENERAL_CI'`)
- `timezone`: The timezone configured on the MySQL server. This is used to type cast server date/time values to JavaScript `Date` object and vice versa. This can be `'local'`, `'Z'`, or an offset in the form `+HH:MM` or `-HH:MM`. (Default: `'local'`)
- `connectTimeout`: 设置连接超时时间，默认是毫秒 (Default: `10000`)
- `stringifyObjects`: Stringify objects instead of converting to values. See issue [#501](https://github.com/mysqljs/mysql/issues/501). (Default: `false`)
- `insecureAuth`: Allow connecting to MySQL instances that ask for the old (insecure) authentication method. (Default: `false`)
- `typeCast`: Determines if column values should be converted to native JavaScript types. (Default: `true`)
- `queryFormat`: A custom query format function. See [Custom format](https://github.com/mysqljs/mysql#custom-format).
- `supportBigNumbers`: When dealing with big numbers (BIGINT and DECIMAL columns) in the database, you should enable this option (Default: `false`).
- `bigNumberStrings`: Enabling both `supportBigNumbers` and `bigNumberStrings` forces big numbers (BIGINT and DECIMAL columns) to be always returned as JavaScript String objects (Default: `false`). Enabling `supportBigNumbers` but leaving `bigNumberStrings` disabled will return big numbers as String objects only when they cannot be accurately represented with [JavaScript Number objects] (<http://ecma262-5.com/ELS5_HTML.htm#Section_8.5>) (which happens when they exceed the [-2^53, +2^53] range), otherwise they will be returned as Number objects. This option is ignored if `supportBigNumbers` is disabled.
- `dateStrings`: Force date types (TIMESTAMP, DATETIME, DATE) to be returned as strings rather then inflated into JavaScript Date objects. Can be `true`/`false` or an array of type names to keep as strings. (Default: `false`)
- `debug`: Prints protocol details to stdout. Can be `true`/`false` or an array of packet type names that should be printed. (Default: `false`)
- `trace`: Generates stack traces on `Error` to include call site of library entrance ("long stack traces"). Slight performance penalty for most calls. (Default: `true`)
- `multipleStatements`: Allow multiple mysql statements per query. Be careful with this, it could increase the scope of SQL injection attacks. (Default: `false`)
- `flags`: List of connection flags to use other than the default ones. It is also possible to blacklist default ones. For more information, check [Connection Flags](https://github.com/mysqljs/mysql#connection-flags).
- `ssl`: object with ssl parameters or a string containing name of ssl profile. See [SSL options](https://github.com/mysqljs/mysql#ssl-options).



## Terminating connections - 终止连接

这里有两种终止连接的方式

第一种通过调用`end()`

```javascript
connection.end(function(err) {
  // The connection is terminated now
});
```

在发送 ``COM_QUIT`` 数据包到MySQL服务之前，会确保已添加到排队的查询任然正常执行。 简单理解就是 ``end()``在不抛出异常错情况，会等待所有查询任务完成后再发送 ``COM_QUIT``.

第二种方式：

```javascript
	connection.destroy();
```

两者区别在于： 前者可以接收回调函数而后者不行

## Pooling connections - 连接池

> 连接池是维护的数据库连接的缓存，以便在将来对数据库的请求需要时可以重用连接。

> 连接池用于提高数据库上执行命令的性能。如果为每个用户打开和维护一个数据库连接，特别是对动态数据库驱动的网站应用程序的请求，既昂贵又浪费资源。在连接池中，在创建连接之后，它被放在池中并再次使用，这样就不必建立新的连接。如果连接池中连接都被使用，就会创建一个新的连接并将其添加到池中或等待其它空闲连接。连接池还减少了用户必须等待建立一个连接的时间。

创建连接池方式：

```javascript
var mysql = require('mysql');
var pool  = mysql.createPool({
  connectionLimit : 10,
  host            : '127.0.0.1',
  user            : 'root',
  password        : '123456',
  database        : 'kkb'
});
 
pool.query('SELECT 1 + 1 AS solution', function (error, results, fields) {
  if (error) throw error;
  console.log('The solution is: ', results[0].solution);
});
```

上面是一种简便的写法，代码实际执行如下操作步骤：

1. pool.getConnection()    获取连接
2. connection.query()         执行查询
3. connection.release()      释放连接

使用``pool.getConnection() ``这种方式有利于后续查询中分享连接状态。因为调用两次 ``pool.query()`` 会使用两个不同的连接而且并行执行。

```javascript
var mysql = require('mysql');
var pool  = mysql.createPool(...);
 
pool.getConnection(function(err, connection) {
  if (err) throw err; // not connected!
  // Use the connection
  connection.query('SELECT something FROM sometable', function (error, results, fields) {
    // When done with the connection, release it.
    connection.release();
 
    // Handle error after the release.
    if (error) throw error;
 
    // Don't use the connection here, it has been returned to the pool.
  });
});
```

如果想从连接池中关闭和移除连接。 可以执行 ``connection.destroy()`` . 连接池将会在下一次需要(用户请求等等)时创建新的连接。

特点： 

​	1 连接由池惰性创建的。

​	2 连接池队列方式进行存取和管理。 

## Pool options

除了可以指定 ``Connection 对象 options ``中属性外， 还可以使用如下的

- acquireTimeout ：设置获取连接超时时间 （毫秒）默认(10000) 
- waitForConnections :  在没有连接可用且达到限制时池的操作。如果为``true``，则池将对连接请求排队，并在一个连接请求可用时调用它。如果为false，则池将立即回调一个错误。(默认值：true)
- connectionLimit ： 连接限制 （默认值：10)
- queueLimit	:  排队的最大连接请求数。如果设置为0，则排队连接请求的数量没有限制。 （默认值：0）

### Pool events

------
#### acquire - 获取请求时触发

​	从连接池中获取请求的时触发，其触发时机请求发送之后，并在连接对象返回调用处之前执行。

```javascript
pool.on('acquire', function (connection) {
  console.log('Connection %d acquired', connection.threadId);
});
```

#### connection -连接

​	在池中建立新连接时，池将发出```connection` ``事件。如果需要在连接使用之前在连接上设置会话变量，则可以侦听``connection``事件.

```javascript
pool.on('connection', function (connection) {
  connection.query('SET SESSION auto_increment_increment=1')
});
```

#### enqueue - 队列

​	当回调排队等待可用连接时，池将发出``enqueue ``事件。

```javascript
pool.on('enqueue', function () {
  console.log('Waiting for available connection slot');
});
```

#### release - 释放

​	当连接释放回池时，池会发出 ``release`` 事件.   在对连接执行 ``release()``后触发。

```javascript
pool.on('release', function (connection) {
  console.log('Connection %d released', 		connection.threadId);
});
```

### 关闭连接池中连接

​	

```javascript
pool.end(function (err) {
  // all connections in the pool have ended
});
```