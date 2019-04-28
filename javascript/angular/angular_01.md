

[TOC]

# Angular

## 项目基本结构

------

利用 @angular/cli 生成项目大概的结构：

```typescript
**root**
    |--e2e	 	端到端测试配置目录
    |--projects	测试目录
    |--src		
    	|---app	   组件目录
    		|----`app.component.css`	style	- 组件私有样式
    		|----`app.component.html`	view	- 视图
    		|----`app.component.ts`		model	- 视图Model
    		|----`app.module.ts` 		注册App模块的一些依赖项，路由、表单、请求Client 、管道、指令 等
    	|---assets 组件共用的内容 比如 `css`, `js`, image 等
    	|---environments  环境变量配置
		|---browserslist  浏览器版本，主要用在编译时
		|---index.html	  母版页
		|---main.ts		  程序的启动入口
		|---styles.css	  母版页样式

    |--.editorconfig		主要编译器代码格式
    |--.angular.json		主要配置打包的配置信息
    |--package.json	    主要项目的依赖信息
    |--tsconfig.json		TypeScript配置文件
    |--tslint.json    	检测语法

```
## 创建第一个组件：

------
使用 Angular CLI, 生成一个新的组件命名为 heroes， 一般组件定义在 src/app目录下
```javascript
ng generate component heroes
//或
ng g c heroes
```
执行上述命令会创建一个新文件夹，默认生成是在 **src/app** 目录下。大概的组件文件结构：

```typescript
|-app
	|--heroes
    	|---heroes.component.ts		
        |---heroes.component.html	
        |---heroes.component.css	 
        |---heroes.component.spec.ts
```

这 HeroesComponent 类文件如下：

```typescript
// heroes.component.ts	

import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-heroes',
  templateUrl: './heroes.component.html',
  styleUrls: ['./heroes.component.css']
})
// 始终使用 export 导出供其它模块使用 比如 App.module.ts
export class HeroesComponent implements OnInit {
  constructor() { }
    
  // 声明周期钩子函数  
  ngOnInit() {
  }
}

```
始终从 Angular **core** library 导入 **Component** 并使用 **@Component** 注释组件类。

@Component  是一个装饰器函数，它指定组件的元数据。主要如下三个元数据属性：

1. selector - 组件的元素选择器
2. templateUrl - 模板文件
3. styleUrls - 组件私有层叠样式

`ngOnInit`  声明周期钩子函数，在组件创建后自动调用，这里一般放置初始化逻辑代码。


### 给英雄添加一个 hero 属性 

------

为`HeroesComponent` 添加一个英雄属性, 为一个名为“风暴”的英雄

```typescript

hero = 'Windstorm';
```

### 显示 hero

打开 `heroes.component.html` 模板文件，修改如下

```typescript
{{hero}}
```

## 显示 *HeroesComponent*   视图
------

首先需要把当前组件添加到  AppComponent 中。
```html
<h1>{{title}}</h1>
<app-heroes></app-heroes>
```
上述代码中 app-heroes  表示HeroesComponent 组件的元素选择器。

## 创建一个 Hero class

------

真实的英雄不只有名字. 

在 `src/app` 文件夹创建一个 **Hero** 类

```typescript
export class Hero {
  id: number;
  name: string;
}
```

返回 **HeroesComponent** 类并且导入 **Hero lei**.

将组件的**hero**属性重构为 **Hero** 类型， 并初始化 id 和 name，修改

```typescript
import { Component, OnInit } from '@angular/core';
import {Hero} from '../hero';

@Component({
  selector: 'app-heroes',
  templateUrl: './heroes.component.html',
  styleUrls: ['./heroes.component.css']
})
export class HeroesComponent implements OnInit {
  hero: Hero = {
    id: 1,
    name: 'Windstorm'
  }
  constructor() { }
  ngOnInit() {
  }
}
```

因为把 **hero** 类型从字符串修改为对象，通用模板中也需要同步修改否则不会正常显示。


## 显示 hero 对象

------
模板更新，显示英雄名称并显示英雄详情。 如下：

```html
<h2>{{hero.name}} Details</h2>
<div><span>id: </span>{{hero.id}}</div>
<div><span>name: </span>{{hero.name}}</div>
```

## 使用 *UppercasePipe*   格式化

------

修改 **hero.name** 绑定方式，如下：

```
<h2>{{hero.name | uppercase}} Details</h2>
```

刷新浏览器，现在英雄名字是大写字母方式显示。

`|`  管道操作符 ， `uppercase` 为内置管道

[Pipes](https://angular.io/guide/pipes) 是用来格式字符串，货币，日期以及其它的数据的好方法。Angular也允许自定义自己的管道。

## 编辑英雄信息

------

用户应该能够通过 <input> 框来编辑英雄的名字。

文本框应显示英雄的名称属性，并在用户键入时更新该属性。这意味着数据从组件类流出到界面，从界面返回到类。

要完成自动完成数据流，需要在 <input> 和 hero.name 属性两者间设置双向数据绑定。

### 双向绑定 

```html
<div>
  <label>name:
    <input [(ngModel)]="hero.name" placeholder="name">
  </label>
</div>
```

​	**[(ngModel)]**  为Angular 双向绑定的语法； 其实这里两个语法组合：

​	输入：`[]`   把 name 显示在文本框中，数据来源 hero 中。

​	输出：`()`   把文本框中修改更新到 hero 中。

上述就完成双向数据流。

### 缺少 FormsModule

------

请注意, 添加  **[(ngModel)]** 后应用停止工作。

查看错误，打开浏览器开发者工具并且看到如下的错误：

```javascript
Template parse errors:Can't bind to '[ngModel](https://angular.io/api/forms/NgModel)' since it isn't [a](https://angular.io/api/router/RouterLinkWithHref) known property of 'input'.
```

虽然 **ngModel** 是一个有效的 **Angular** 指定，但是默认是不可用的。

ngModel 属于 FormsModule 模块，必须使用引入它才行。 

## AppModule

------

Angular需要知道应用程序的各个部分是如何组合在一起的，以及应用程序还需要哪些文件和库。这些信息称为元数据

一些元数据位于您添加到组件类的@Component装饰器中。其他关键元数据在@NgModule装饰器中。

Angular CLI 在src/app/app.module中生成了一个AppModule类。这是可以加入**FormsModule** 模块的位置。

### 导入 FormsModule

```typescript
// app.module.ts (FormsModule symbol import)
import { FormsModule } from '@angular/forms';
```

然后添加 **FormsModules** 到   [@NgModule](https://angular.io/api/core/NgModule) 元数据  [imports](https://angular.io/api/core/NgModule#imports)  数组中，它包含当前应用需要外部的模块列表。

```typescript
// app.module.ts ( @NgModule imports)** 

imports: [
  BrowserModule,
  FormsModule
],
```

当刷新浏览器，应用程序应该运行正常。您可以编辑英雄的名称，并查看文本框上方<h2>中立即反映的更改。

### 声明 HeroesComponent

------

任何组件必须在一个模块（[NgModule](https://angular.io/guide/ngmodules)）中声明，否则组件是不能正常工作的。

上例中，并没有去声明 **HeroesComponent**  组件，为什么能够正常工作？

它工作的原因是   **Angular CLI**  在生成该组件的同时在 **AppModule**中声明了**HeroesComponent**   

打开 `src/app/app.module.ts` 并可以顶部看到 `HeroesComponent` 的导入。

```typescript
import { HeroesComponent } from './heroes/heroes.component';
```

HeroesComponent 在 @NgModule.declarations 数组中声明。

```typescript
declarations: [
  AppComponent,
  HeroesComponent
],
```

## 最终的代码

------

**heroes.component.ts**

```typescript
import { Component, OnInit } from '@angular/core';
import {Hero} from '../hero';

@Component({
  selector: 'app-heroes',
  templateUrl: './heroes.component.html',
  styleUrls: ['./heroes.component.css']
})
export class HeroesComponent implements OnInit {
  hero: Hero = {
    id: 1,
    name: 'Windstorm'
  }
  constructor() { }

  ngOnInit() {
  }

}
```

**heroes.component.html**

```
<h2>{{hero.name | uppercase}} Details</h2>
<div><span>id: </span>{{hero.id}}</div>
<div><span>name: </span>{{hero.name}}</div>

<h2>编辑</h2>
<div>
  <label>name:
    <input [(ngModel)]="hero.name" placeholder="name">
  </label>
</div>

```

**app.module.ts**

```typescript
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import {FormsModule} from '@angular/forms';
import { AppComponent } from './app.component';
import { HeroesComponent } from './heroes/heroes.component';


@NgModule({
  declarations: [   // 组件、指令、管道
    AppComponent,
    HeroesComponent
  ],
  imports: [  // 外部模块
    BrowserModule,
    FormsModule
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

**app.component.ts**

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-root',
  templateUrl: './app.component.html',
  styleUrls: ['./app.component.css']
})
export class AppComponent {
  title = '学习AngularJS';
}
```

**app.component.html**

```html
<h1>{{title}}</h1>
<app-heroes></app-heroes>
```

**hero.ts**

```typescript
export class Hero {
  id: number;
  name: string;
}
```

### 总结

------

1. 使用Angular CLI 创建第二组件 HeroesComponent .   
2. 把 HeroesComponent 添加到 AppComponent 中显示
3. 应用 UppercasePipe格式化 name 属性
4. 使用 ngModle指令完成双向数据绑定
5. 了解了 AppModule
6. 在AppModule中导入了FormsModule，以便Angular识别并应用ngModel指令。
7. 了解了在AppModule中声明组件的重要性，并了解CLI为您声明了它

