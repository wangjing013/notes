# 知识点

1 路由配置 	

​	引入 RouterModule  ； 

​	routes 路由， 定义路由

​	RouterModule.forRoot(routes ) 安装路由



2 模板驱动表单			FormsModule

------
- 数据模型		LoginUser - class
- 双向数据绑定  	[()]
- 校验	 		基于H5验证规则
- 校验状态		通过"ngModel" 定义模板变量，获取校验状态
- 校验样式添加        添加表单验证后， 如果存在错误状态，默认angular 会在其表单元素加上错误相关的类  例如  ng-invalid \  ng-dirty

3 Http请求

------
- httpClient模块：导入到 BrowseModule后面

- 依赖注入    

- 发送请求    get\post\put\delete

- 响应处理   返回- Observable 对象

4 Session 处理

------
1. 配置

- 安装 express-session

- express.use() 放置在 cookie模块后


2. 使用

- 赋值：req.session.xx = xx

- 取值：req.session.xx
- 删除： delete req.session.xx	

3. session持久化- 存入数据库   （express-mysql-session）


## 路由容错处理几种方：
------
1. 定义404页面 

2. 重定向

   { path:"**", redirectTo:'logom'}

## 项目代码组织分层

------

定义分层模块 - User Module

1 创建模块

2 在AppModule中的@NgModule 中的imports 中导入UserModule 

3 创建模块路由 ，管理模块内部的路由管理  这样便于维护以及简化全局的路由配置

![模块划分](F:\kkb-notes\course\模块划分.png)

1 重构App模块

​	添加 app-routing-module.ts 管理App模块中的路由

2 重构User 模块

- 添加User模块
- 把定义 login、register 组件移植到 User模块中
- user-routing-module 中管理 login、register组件
- 再把 Usert Module 添加到 AppModule 中 @ngModule装饰器中 imports数组中

## 定义注册模块

------

注册模块： 手机号码、图形验证码、短信验证码、密码 四个字段

### 验证手机号码

------

- 自定义指令       phone-validator.directive
- 添加验证服务   verify-phone
- 发送请求验证手机号码  

### 图形验证码

------

生成并返回图形验证码步骤：

1 安装模块	trek-captcha

2 生成逻辑

```javascript
// 图形验证码生成
const captcha = require('trek-captcha');
router.get('/code-img', async (req, res) => {
    try {
        // toke: 数字和字母表示形式
        // buffer: 图片的二进制形式
        const {token, buffer} =  await captcha({size: 4});
        req.session.token = token;
        res.json({
            success: true,
            data: buffer.toString('base64')
        })
    } catch (error) {
        
    }
});
```

 3  验证图形验证码，填写有效性

​	大概步骤: 

- 自定义指令：captcha-validator.directive  并 在模块中注册 

- 添加验证服务： verify-captcha

- 发送请求验证验证码

​	

### 发送短信验证码

------

​	大概步骤：

1. UserService中定义服务方法， sendCodeSms

2. 发送请求，触发发送短信功能

3. 接收短信验证码




​		



​	