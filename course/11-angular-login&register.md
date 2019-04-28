

## 功能模块

------

-  用户登录
- 注册

## 为什么需要表单验证？

​	通过验证用户输入的准确性和完整性，来增强整体数据质量.

## 验证的两种方式:

- [模板驱动表单](https://www.angular.cn/guide/forms-overview)
- [响应式表单](https://www.angular.cn/guide/reactive-forms)

## 数据的校验基于H5表单校验

------

常用的包括	:  [pattern](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/input#attr-pattern) ，[min](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/input#attr-min) ，[max](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/input#attr-max) ，[required](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/input#attr-required) ，[step](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/input#attr-step) ，[maxlength](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/input#attr-maxlength) .

校验状态	：valid，invalid，untouched，dirty（肮脏的-值被修改）， pristine(纯净的) 

## 使用模块：

------

模板驱动验证：[FormsModule](https://www.angular.cn/api/forms/FormsModule) 

响应式表单验证：[ReactiveFormsModule](https://www.angular.cn/api/forms/ReactiveFormsModule#reactiveformsmodule)

请求：[HttpClient](https://www.angular.cn/guide/http)	 







## 后端准备工作

------

1	创建表模型：user table

```mysql

DROP TABLE IF EXISTS `user`;
 SET character_set_client = utf8mb4 ;
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `username` varchar(45) DEFAULT NULL,
  `password` varchar(32) NOT NULL,
  `phone` varchar(11) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=2 DEFAULT CHARSET=utf8;


LOCK TABLES `user` WRITE;
INSERT INTO `user` VALUES (1,'jamesharden','123456','13055199735');
UNLOCK TABLES;
```

2 	定义接口:   `api / users.js`

```javascript
var express = require('express');
var router = express.Router();
var {query} = require('../../models/db');

router.post('/login', async function (req, res, next) {
  let sql = `select phone, password from kkb.user where phone = ? && password = ?`;
  try {
    let results = await query(sql, [req.body.phone, req.body.password]);
    if (results.length) {
        let user = results[0];
        req.session.user = user;
        res.json({
            result: user,
            success: true,
            message: '登录成功！'
        });
    } else {
        res.json({
            success: false,
            message: '手机号码或密码错误！'
        })
    }
  } catch(error) {
    res.json({
        success: false,
        message: '服务端错误！'
    })
  }
});

module.exports = router;
```

3	管理会话 （安装 [express-session](https://www.npmjs.com/package/express-session)） 

​	



