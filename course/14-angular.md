# 勘误	

cookie 设置问题

表单提交问题

## 项目功能开发：

------

**自动登录步骤**

- 新增Home组件： 作为首页（判断当前用户是否登录）

- 根据返回结果进行路由跳转

**判断逻辑**

​	后端根据当前cookie是否有效，判断用户是否存在，存在则认为已经登录 

**路由跳转** 

- 链接方式 <a routerLink=""></a> 

  - 传参方式
    - [routerLink] = "['path', {可选参数}]"
    - 查询参 [queryParams] = "{bar: 'bar'}"

- 编程式 router.navigate()   

  - 导入服务 Router
  - 依赖注入

  传参数方式：

  ​	1 必选参（URL传参）

  ​		获取参数方式：

  ​			引入 [ActivatedRoute](https://www.angular.cn/api/router/ActivatedRoute) 

  ​	2 可选参数	

  ​	3 查询参  queryParams

**记住：** 使用编程序式路由，必须先导入 Router 服务

主页架构	 

​	定义Main Module

​	新增子模块	

1. ucenter
   1. course       课程
   2. message    消息
   3. comment  评论
   4. collection  收藏
   5. account      用户
2. course