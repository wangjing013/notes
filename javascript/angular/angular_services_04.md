

[TOC]



# 服务

目前 `HeroesComponent` 组件是获取假数据来做展示的。

本文重构之后，`HeroesComponent` 将更加精简，并专注于支持视图。使用 mock service 进行单元测试更加容易。

## 什么是服务？

------

组件不应该直接获取或保存数据，当然也不应该故意提供虚假数据。它应该专注与数据展现，并将数据访问委托给服务。

在本前教程中，你将创建一个 `HeroService`  服务，所有的应用程序类使用它来获取英雄列表。你将依赖Angular的[依赖注入](https://angular.io/guide/dependency-injection)来将服务注入到  `HeroesComponent` 构造函数中，而不是通过new 去创建服务。

**服务用在两个不同类中进行共享信息的好方式**。您将创建一个 `MessageService` 服务并将其在两个地方注入：

1. 在HeroService中，它使用服务发送消息。
2. 在MessagesComponent中显示该消息。

## 创建 HeroService

------

使用 Angular CLI ， 创建一个叫 hero 的服务；

```javascript
ng g s hero
```

该命令在`src/app/hero.service.ts` 中生成了`HeroService` 类的基本骨架；
这 `HeroService` 类应该看起来像下面的例子：

```typescript
// src/app/hero.service.ts (new service)

import { Injectable } from '@angular/core';

@Injectable({
  providedIn: 'root'
})
export class HeroService {

  constructor() { }
}

```

### 使用 [@Injectable()](https://angular.io/api/core/Injectable) 注册服务

------

请注意：新创建的服务导入了 Angular **Injectable** 装饰器 ，并且使用 @Injectable() 装饰器注解该类；这将该类**标记**为参与依赖项注入系统的类。
`HeroService` 类将提供注册服务，它也可以有自己的注入依赖项。（这句话说明可以服务调用服务）.

@Injectable() 装饰器接受服务的元数据对象，与@Component()装饰器为组件类所做的相同。

### 获取英雄数据

------

`HeroService` 可以从任何地方获取英雄数据  -  web 服务，local storage 或 模拟数据源.

从组件中删除数据访问意味着您可以随时改变对实现想法，而无需接触任何组件。组件也不需要知道这项服务是如何运作的。

本教程中的实现将继续提供模拟的英雄数据。

在服务中导入 `Hero` 类 和 模拟数据 `HEROES`.

```typescript
// // src/app/hero.service.ts 

import { Hero } from './hero';
import { HEROES } from './mock-heroes';
```

添加一个 `getHeroes` 方法去返回模拟的英雄数据：

```typescript
getHeroes(): Hero[] {
  return HEROES;
}
```

## 提供（Provide ） HeroService

------

在 Angular  注入服务到 `HeroesComponent`之前，你必须把 `HeroService`  可用于依赖注入系统。你可以通过注册一个 *provider*.  provider 完成创建或传递服务；在这种情况下，它实例化`HeroService`类来提供服务。 

默认情况下，Angular CLI 命令 `ng generate service` 会通过给 `@Injectable` 装饰器添加元数据的形式，为该服务把提供商注册到*根注入器*上。

如果查看一下`HeroService`类定义之前的@Injectable()语句，就会发现`providedIn`元数据值是'root':

```typescript
@Injectable({
  providedIn: 'root',
})
```

当在 root 级别提供服务时，Angular 创建了`HeroService`的一个单独的共享实例，并将它注入到任何需要它的类中。 在 `@Injectable` 元数据中注册该提供商，还能让 Angular 可以通过移除那些完全没有用过的服务，来进行优化。

## 修改 HeroesComponent

------

打开 `HeroesComponent`  类文件。

删除 `HEROES` 导入语句， 因为你不再需要它. 相反导入 `HeroService` .

```typescript
// src/app/heroes/heroes.component.ts (import HeroService)

import { HeroService } from '../hero.service';
```

用一个简单的声明替换heroes属性的定义.

```typescript
heroes: Hero[];
```

### 注入 HeroService 服务

------

将类型为 `HeroService` 的私有的 `heroService` 参数添加到构造函数中。

```typescript
constructor(private heroService: HeroService) { }
```

这个参数同时做了两件事： 

1.  声明了一个私有 `heroService` 属性
2. 把它标记为一个 `HeroService` 的注入点。

当 Angular 创建组件时，依赖注入系统会将 `heroService`参数设置为`HeroService` 的单例对象.

### 添加 getHeroes方法

------

在服务中创建一个方法，返回英雄列表数据。

```typescript
getHeroes(): void {
	this.heroes = this.heroService.getHeroes();
}
```

### 在 ngOnInit 中调用

------

你也可以在构造函数中调用 getHeroes()， 不推荐这种方式.

构造函数尽量保持简单，只用来初始化，例如把构造函数参数赋值给属性； **构造函数**不应该做任何事情.  它肯定不能调用某个函数来向远端服务(获取真实数据)发起Http请求.

反而，应该在 `ngOnLinit` 生命周期钩子函数中调用 `getHeroes()`  ，之后交由 Angular 处理，它会在构造`HeroesComponent` 组件实例化之后的某个时机调用 `ngOnInit`;

```typescript
ngOnInit() {
  this.getHeroes();
}
```

### 查看运行效果

------

刷新浏览器后，应用程序应该像以前一样正常运行，显示英雄列表并且当用户点击英雄名字时显示其英雄明细信息；



## 可观察（Observable） 的数据

------

`HeroService.getHeroes()`  的函数签名是一个**同步的**；它所隐含的假设是 `HeroService` 总是能同步获取英雄列表数据。 而 `HeroesComponent` 也同样假设能同步取到 `getHeroes()` 的结果。

```typescript
this.heroes = this.heroService.getHeroes();
```

这在真实的应用是不存在的，现在之所以这样的方式因为当前服务返回的是模拟数据. 不过很快，该应用将从远程服务获取英雄数据，而那天生就是***异步***操作。

`HeroService` 必须等待服务的响应，`getHeroes()`不能立即返回英雄数据，浏览也不不会服务等待时堵塞。

`HeroService.getHeroes()` 必须是某种**异步签名函数** 

你可以采用回调函数，可以返回一个Promise函数，也可以返回一个 Observable 对象。

在这教程中，`HeroService.getHeroes()`  将返回一个 Observable  因为它（服务）最终会使用 `HttpClient.get` 方法去获取英雄数据，而`HttpClient.get`方法返回的是一个 `Observable` .



### 可观察对象版本的 HeroService 

------

`Observable` 是 [RxJS 库](http://reactivex.io/rxjs/)中的一个关键类。

在 HTTP 课程中，你会了解到  Angular's 中 `HttpClient`的方法返回是 RxJS 的 `Observable`，这节课，你将使用 RxJS 的 `of()` 函数来模拟从服务器返回数据。

打开 `HeroService` 文件并导从 RxJS 导入 `Observable` 和 `of` 符号.

```typescript
// src/app/hero.service.ts (Observable imports)
import { Observable, of } from 'rxjs';
```

重写 `getHeroes` 方法：

```typescript

getHeroes(): Observable<Hero[]> {
  return of(HEROES);
}
```

`of(HEROES)` 返回一个 Observable<Hero[]>，它会发出单个值，这个值就是这些模拟英雄的数组.

### 在 HeroesComponent 中订阅 （Subscribe ）

------

 `HeroService.getHeroes` 之前用于返回一个 Hero[]，现在它返回的是一个 Observable<Hero[]>。

在`HeroesComponent` 中也需要作相应的调整。

找到 getHeroes  方法，并修改为如下的代码：

```typescript
getHeroes(): void {
	this.heroService.getHeroes()
	.subscribe(heroes => this.heroes = heroes);
}
```

原版本：

```typescript
getHeroes(): void {
	this.heroes = this.heroService.getHeroes();
}	
```

Observable.subscribe() 是关键的区别。

在上一个版本中，把英雄数组赋值给组件 heroes 属性。这是属于**同步**方式赋值。 这两包含两种假设：

- 服务器可以立即返回英雄列表数据
- 浏览器可以在等待服务端响应时冻结页面

当 `HeroService` 真的向远端服务器发起请求时，这种方式就行不通了。

在新的版本中等待从 `Observable` 中发出的 heroes 数组--- 可能现在返回或几分钟之后。然后，subscribe 传递英雄数组给回调函数，该函数把值赋值给组件的 `heroes` 属性。

当 `HeroService`  从服务端请求英雄列表数据时，这种异步方就可以工作了。



## 显示消息列表

------

在本节中，您将：

- 新增一个 `MessagesComponent` 组件，在页面底部展示应用的消息；
- 创建一个可注入的、全应用级别的 `MessageService`，用于发送要显示的消息。
- 把 `MessageService`  服务注入到 `HeroService` 中。
- 当 `HeroService`  获取英雄数据成功时显示一条消息；

### 创建 MessagesComponent 组件

------

使用 CLI 去创建 MessagesComponent  组件

```
ng generate component messages 
或
ng g c messages
```

这命令在`src/app/messages` 文件夹中创建组件有关的文件，并在 AppMoudule中声明 `MessagesComponent` 组件。

修改`AppComponent` 模板，添加生成的  `MessagesComponent` 组件，用来显示消息

```html
// /src/app/app.component.html

<h1>{{title}}</h1>
<app-heroes></app-heroes>
<app-messages></app-messages>
```

你应该可以在页面底部看到 `MessagesComponent`  组件模板中默认的内容。

### 创建 MessagesService 服务

------

使用CLI 在 src/app 下创建 `MessageService` 服务；

```
ng generate service message
```

打开`MessageService` 并把内容修改成下面这样：

```typescript

```

这服务暴露缓存消息的 messages属性和两个方法： 一个`add()`方法用于添加消息到缓存中，另一个 `clear()`方法用于清除缓存。

### 把消息服务注入到 HeroService 服务

------

再次打开 `HeroService`  并导入 `MessageService` 服务

```typescript
// /src/app/hero.service.ts (import MessageService)

import { MessageService } from './message.service';

```

修改构造函数参数列表中声明一个私有  `messageService` 属性，Angular 将在组件创建的时把 `MessageService`  的单实例注入到该属性。

```typescript
constructor(private messageService: MessageService) { }
```

>这是典型的**服务中服务**的场景，你把 `MessageService` 注入到 `HeroService` 中，而 `HeroService` 又被注入到了 `HeroesComponent`中.

### 从 HeroService 中发送一条消息

------

修改 `getHeroes` 方法当获取英雄时发送一条消息；

```typescript
getHeroes(): Observable<Hero[]> {
  this.messageService.add('HeroService: fetched heroes');
  return of(HEROES);
}
```

### 显示从HeroService中添加的消息

------

这消息组件应该显示所有消息， 其中包括`HeroService`  在获取英雄时发送的消息.

打开`MessagesComponent` 并导入`MessageService`.

```typescript
// /src/app/messages/messages.component.ts (import MessageService)

import { MessageService } from '../message.service';
```

修改构造函数在参数列表中声明一个公有  `messageService` 属性，Angular 将在组件创建的时把 `MessageService`  的单实例注入到该属性。

```typescript
constructor(public messageService: MessageService) {}
```

这`messageService`必须是公有属性， 因为需要模板中使用它；

> Angular 只能绑定到组件中的**公共** 属性

### 绑定 MessageService

------

修改通过CLI 生成 MessagesComponent 模板内容如下：

```html
// src/app/messages/messages.component.html

<div *ngIf="messageService.messages.length">

  <h2>Messages</h2>
  <button class="clear"
          (click)="messageService.clear()">clear</button>
  <div *ngFor='let message of messageService.messages'> {{message}} </div>

</div>
```

模板中直接绑定了组件 `messageService` 属性

- *ngIf  只有messages存在数据时才显示消息区域
- *ngFor用来遍历消息列表
- Angular [event binding](https://angular.io/guide/template-syntax#event-binding)   把按钮点击事件绑定到 `MessageService.clear()`上。



