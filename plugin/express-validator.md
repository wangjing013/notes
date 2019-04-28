# express-validator

[TOC]



## 简介

> express-validator 是一套 express.js 中间件，包括 [validator.js ](https://github.com/chriso/validator.js) 验证器 和 sanitizer 函数

## 安装

```
	npm install --save express-validator
```

## 基本指南

> 在继续阅读本指南之前，建议您具备express.js模块的基本知识。

让我们开始编写一个基本路由来在数据库中创建用户：

```javascript
const express = require('express');
const app = express();
app.use(express.json());
app.post('/user', (req, res) => {
	User.create({
		username: req.body.username,
		password: req.body.password
	}).then(user => res.json(user));
})
```

如果想在创建User对象之前先验证输入内容并报告任何错误。如下

```javascript
const {
    check,
    validationResult
} = require('express-validator/check');

app.post('/user', [
    // username must be an email
    check('username').isEmail(),
    // password must be at least 5 chars long
    check('password').isLength({
        min: 5
    })
], (req, res) => {
    // 在此请求中查找验证错误，并将它们包含在具有便捷功能的对象中
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
        // 422 表示存在语法错误，无法响应
        return res.status(422).json({
            errors: errors.array()
        });
    }
    User.create({
        username: req.body.username,
        password: req.body.password
    }).then(user => res.json(user));
});

```

验证代码添加完成，当提交的请求对象中包含无效的 ``username`` 或 ``password``字段，你服务将响应像下面这样：

```json
{
	"errors": [
		{
            "location": "body",
            "param": "username",
            "value": "wangjing",
            "msg": "Invalid value"
		}
	]
}
```

查看 express-validator 所有可用的验证器，可以查看 [validator.js docs](https://github.com/chriso/validator.js#validators) 文档



## 高级功能

### Sanitization

有时，在HTTP请求中接收输入不仅要确保数据格式正确，还要确保它没有安全问题。

[validator.js 提供了一些过滤方法](https://github.com/chriso/validator.js#sanitizers)，可用于处理进来的数据。

```javascript
const express = require('express');
const { body } = require('express-validator/check');
const { sanitizeBody } = require('express-validator/filter');

const app = express();
app.use(express.json());

app.post('/comment', [
  body('email')
    .isEmail()
    .normalizeEmail(),
  body('text')
    .not().isEmpty()
    .trim()
    .escape(),
  sanitizeBody('notifyOnReply').toBoolean()
], (req, res) => {
  // Handle the request somehow
});
```

上面的实例，我们验证了 ``email`` 和 ``text`` 字段，过滤方法可以通过链接的方式调用。 像电子邮件规范化和修改、HTML转义。

由于 ``notifyOnReply`` 字段没有经过验证， 我们可以使用 [filter Api](https://express-validator.github.io/docs/filter-api.html) 中去 ``sanitizeBody`` 函数把 ``notifyOnReply``转换为JavaScript 布尔值。

>**重点**： 请注意，sanitization 会改变请求对象， 如果 ``req.body.text`` 发送的值是 ``Hello world:>`` , 清理之后其值将为 ``Hello world :&gt;)``

上述输入正确值情况下，服务响应

```json
// 输入正常值情况下
{
	"email": "974742841@qq.com",
	"text": "1234",
	"notifyOnReply": true
}
// 当输入错误邮箱地址时候
{
    "errors": [
        {
            "location": "body",
            "param": "email",
            "value": "974742841@qq.com1234345",
            "msg": "Invalid value"
        }
    ]
}

// 当``text``字段中输入包含HTML代码内容的时候
// text 输入内容： <a>a</a>输入一个超级链接
{
    "email": "974742841@qq.com",
    "text": "&lt;a&gt;a&lt;&#x2F;a&gt;输入一个超级链接",
    "notifyOnReply": true
}
```



### 自定义验证器和清理方法

虽然 express-validator 通过底层依赖 [validator.js](https://github.com/chriso/validator.js) 提供了丰富的``validators``和 ``sanitizers`` 。在构建应用程序时，它并不总是满足。

因为这些情况，你可以考虑编写自定义``validator`` 或 自定义 ``sanitizer``.

### 自定义 validator

------

可以通过使用链式方法 ``.custom()`` 实现自定义验证器。 需要传入验证函数。

自定义验证器可以返回 Promises 以支持异步验证， 错误信息可以通过``throw any value`` 或 ``Promise reject``抛出

[使用自定义错误信息](https://express-validator.github.io/docs/custom-error-messages.html#custom-validator-level)

**实例**：验证邮箱是否使用

```javascript
// user.js 
module.exports = (sequelize, Types) => {
    // 定义模型
    const User = sequelize.define('User', {
        username: Types.STRING(20),
        password: Types.STRING(20),
        email: Types.STRING(20)
    }, {
        tableName: 'user',
        timestamps: false
    });
    User.findUserByEmail = email => {
        return User.findOne({
            where: {email}
        });
    }
    User.sync({force:true});
    return User;
}



const { body } = require('express-validator/check');
app.post('/user', body('email').custom(value => {
  // findUserByEmail 在模型中封装的方法
  return User.findUserByEmail(value).then(user => {
    if (user) {
      return Promise.reject('E-mail already in use');
    }
  });
}), (req, res) => {
   	const errors = validationResult(req);
    if (!errors.isEmpty()) {
        res.status(422).json({
            errors: errors.array()
        });
        return;
    }
    res.send('邮箱未使用，可以正常使用');
});
```



**实例**： 检查确认密码和密码是否匹配

```javascript
const { body } = require('express-validator/check');

app.post('/user', body('passwordConfirmation').custom((value, { req }) => {
  if (value !== req.body.password) {
    throw new Error('Password confirmation does not match password');
  }
  // 注意：验证函数一定要手动为true的返回值， 否则即使上面验证通过也会出现错误验证信息
  return true; // 或 promise  
}), (req, res) => {
  // Handle the request
});
```



### 自定义 sanitizers 

可使用``.customSanitizer()`` 方法实现自定义 sanitizers； 就像验证器一样，你指定一个 sanitizer 函数，目前只支持同步方式。

**实例** 转换为 MongoDB's ObjectID

```javascript
const { sanitizeParam } = require('express-validator/filter');

app.post('/object/:id', sanitizeParam('id').customSanitizer(value => {
  return ObjectId(value);
}), (req, res) => {
  // Handle the request
  // 因为 sanitizers 会改变req对象，所以在当前方法访问是处理过后的值
});
```

### 自定义错误信息

**express-validator's**  默认的错误消息是简单的 "Invalid value" 字符串；如果想显示更详细

```json
{
    "errors": [
            {
                "location": "body",
                "param": "passwordConfirmation",
                "value": "123456",
                "msg": "Invalid value"
            }
    ]
}
```

例如 （密码长度不够，密码格式错误 ，密码必须包含特殊字符）这显然是做不到的。

#### 错误消息等级

##### Validator  Level （验证器级别）

当你想对每个验证器的错误信息进行精确控制时， 你可以使用 [.withMessage()](https://express-validator.github.io/docs/validation-chain-api.html#withmessagemessage) 函数进行指定

```javascript
const { check } = require('express-validator/check');

app.post('/user', [
  // ...some other validations...
  check('password')
    .isLength({ min: 5 }).withMessage('must be at least 5 chars long')
    .matches(/\d/).withMessage('must contain a number')
], (req, res) => {
  // Handle the request somehow
});
```

上面实例，如果密码字符串长度少于5 ，将输出一个错误消息 "must be at least 5 chars long"; 如果它也不包含数字，则将输出一个错误消息  "must contain a number" 

##### Custom Validator Level  自定义验证器

如果想使用自定义验证器，那么可通过使用 throw 和 promises reject 来表示无效值。在这些情况下，报告的错误消息等于验证器抛出的消息.

```javascript
const { check } = require('express-validator/check');
app.post('/user', [
  check('email').custom(value => {
    return User.findByEmail(value).then(user => {
      if (user) {
        return Promise.reject('E-mail already in use');
      }
    });
  }),
  check('password').custom((value, { req }) => {
    if (value !== req.body.passwordConfirmation) {
      throw new Error('Password confirmation is incorrect');
    }
  })
], (req, res) => {
  // Handle the request somehow
});
```



##### Field  Level 字段级别

可使用 [验证器](https://express-validator.github.io/docs/check-api.html#check-field-message) 的第二个参数来指定字段级别的消息。当验证器没有指定自己消息时，默认则使用字段级消息。

```javascript
const { check } = require('express-validator/check');

app.post('/user', [
  // ...some other validations...
  check('password', 'The password must be 5+ chars long and contain a number')
    .not().in(['123', 'password', 'god']).withMessage('Do not use a common word as the password')
    .isLength({ min: 5 })
    .matches(/\d/)
], (req, res) => {
  // Handle the request somehow
});
```

上面实例，当 ``password`` 字段长度短于5字符 或 不包含数字，就会报告错误消息 ``The password must be 5+ chars long and contain a number``  ，因为这些验证器是没有指定自己的错误信息 。



##### Dynamic messages 



### Schema Validation （结构验证）





## API

### check API

这些方法可以通过  `require('express-validator/check')` 获得。

#### check([field, message])

- field(可选的) ： 验证的字段名，可以提供字符串或字符串数组
- message(可选的)：当验证器失败也没指定的消息，则应用该错误消息  默认是 "Invalid value" 

> 返回值： 返回一个验证链

为一个或多个字段创建验证链。 它们可能位于一下任何请求对象中:

- `req.body`
- `req.cookies`
- `req.headers`
- `req.params`
- `req.query`

如果多个对象存在同一个字段，则所有对象中该字段值必须通过验证。 

例如： 我在请求参数中附带了一个查询参数 ``username ``,  同时表单中也存在同名的 ``username``  前者存在 ``req.query``中，后者存在 ``req.body`` 中.

```html
<form action="/user/addUser?username=12345@111111" method="POST" >
    <div>
        用户名：<input type="text" name="username"/>
    </div>
    <div>
        密码：<input type="password" name="password"/>
    </div>
    <div><button type="submit">提交</button></div>
</form>
```

后端处理代码如下：

```javascript

router.post('/addUser', [
    check('username').isEmail(),
    check('password').isLength({
        min: 5
    })
], (req, res, next) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
        res.status(422).json({
            errors: errors.array()
        });
        return;
    }
    User.create({
        username: req.body.username,
        email: req.body.username,
        password: req.body.password
    }).then(user => res.json(user));
});

```

执行后程序得到一个结果：

```json
{
    "errors": [
        {
            "location": "query",
            "param": "username",
            "value": "12345@111111",
            "msg": "Invalid value"
        }
    ]
}
```

**注意：** 如果省略 ``fields``，那么是验证整个请求对象。这只适用于 ``req.body`` 

- 验证器针对同一个字段始终已串行方式执行。 
- 如果链指向多个字段，那么它们将并行运行，但是它们的每个验证器都是串行的。 

通过下面实例理解上述两句话。 

```html
<form action="/user/addUser" method="POST" >
    <div>
        用户名：<input type="text" name="username"/>
    </div>
    <div>
        密码：<input type="password" name="password"/>
    </div>
    <div><button type="submit">提交</button></div>
</form>
```

```javascript
router.post('/addUser', [
    check('username').not().isEmpty().withMessage('邮箱地址不能为空').isEmail().withMessage('邮箱无效'),
    check('password').isLength({
        min: 5
    }).withMessage('密码长度不能少于5个字符')
], (req, res, next) => {
    const errors = validationResult(req);
    if (!errors.isEmpty()) {
        res.status(422).json({
            errors: errors.array()
        });
        return;
    }
    User.create({
        username: req.body.username,
        email: req.body.username,
        password: req.body.password
    }).then(user => res.json(user));
});
```

``not()``、``isEmpty()``、``isEmail()`` 表示就是验证器。 为了显示执行先后顺序，这里利用 ``withMessage``来指定每个验证器的验证消息。

**第一种情况：** 当不填写用户名、且填写正确密码时提交时，返回报告的错误

```json
{
	"errors": [
        {
            "location": "body",
            "param": "username",
            "value": "",
            "msg": "邮箱地址不能为空"
        },
		{
            "location": "body",
            "param": "username",
            "value": "",
            "msg": "邮箱无效"
		}
	]
}
```

**第二种情况：**当用户和密码都填写错误时候提交；

```json
{
    "errors": [
        {
            "location": "body",
            "param": "username",
            "value": "13055199735",
            "msg": "邮箱无效"
        },
        {
            "location": "body",
            "param": "password",
            "value": "1234",
            "msg": "密码长度不能少于5个字符"
        }
    ]
}
```



#### body([fields, message])

与 `check([fields, message])` 一样, 但只检查`req.body`.

#### cookie([fields, message])

与 `check([fields, message])` 一样, 但只检查`req.cookies`.

#### header([fields, message])

与 `check([fields, message])` 一样, 但只检查`req.headers`.

#### param([fields, message])

与 `check([fields, message])` 一样, 但只检查`req.params`.

#### query([fields, message])

与 `check([fields, message])` 一样, 但只检查`req.query`.

#### checkSchema(schema)

schema:  要验证的格式。 必须遵守所描述的格式  [Schema Validation](https://express-validator.github.io/docs/schema-validation.html).

> 返回值：验证链的数组

