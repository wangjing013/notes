# 主/明细组件

目前，`HeroesComponent`  展示了英雄列表和所选的英雄明细；

随着应用程序的增长，将所有功能保持在一个组件中是不可维护的。您需要将大型组件拆分为较小的子组件，每个都专注于特定的任务或工作流程。

在当前页面中，通过将英雄详细信息移动到单独，可重用的 `HeroDetailComponent` 中。

`HeroesComponent`  只会显示英雄列表.  `HeroDetailComponent` 将显示所选中的英雄详情。

## 构造 HeroDetailComponent 

------

使用  Angular CLI 生成一个新的组件，命名为 `hero-detail`

```
ng generate component hero-detail
// 或简写
ng g c hero-detail
```

上述命令构建如下：

​	创建一个文件 `src/app/hero-detail`

在这文件夹里面生成4个文件：

- 组件样式的`CSS`文件
- 组件模板的`HTML`文件
- 具有名为 `HeroDetailComponent` 的组件类的`TypeScript`文件。
- `HeroDetailComponent` 	组件的测试类文件

该命令还在 `src/app/app.module.ts` 文件中的`@NgModule` 声明了 `HeroDetailComponent` 组件。

## 编写模板

------

剪切 `HeroesComponent`  模板底部的英雄详情HTML代码，并粘贴到 `HeroDetailComponent` 模板中。

粘贴的HTML指的是`selectedHero`。新的`HeroDetailComponent`可以呈现任何英雄，而不仅仅是选定的英雄。因此，在模板中的任何地方用`hero`替换`selectedHero`。

当你完成时，`HeroDetailComponent`  模板应该看起来像下面这样：

```html
// src/app/hero-detail/hero-detail.component.html

<div *ngIf="hero">
  <h2>{{hero.name | uppercase}} Details</h2>
  <div><span>id: </span>{{hero.id}}</div>
  <div>
    <label>name:
      <input [(ngModel)]="hero.name" placeholder="name"/>
    </label>
  </div>
</div>
```

## 添加 [@Input()](https://angular.io/api/core/Input) hero 属性 
------

`HeroDetailComponent`  模板绑定组件中 **hero**  属性，其类型为 **Hero**.

打开 `HeroDetailComponent` 类文件添加导入语句.

```typescript
// src/app/hero-detail/hero-detail.component.ts (import Hero)

import { Hero } from '../hero';
```

Hero属性必须是 Input 属性 ， 使用 @Input() 装饰器注释。因为外部 `HeroesComponent` 将像下面这样绑定它：

```html
<app-hero-detail [hero]="selectedHero"></app-hero-detail>
```

修改 `@angular/core`导入声明应该包含 `Input` 。

```typescript

```

并添加一个 hero 属性，前面加上 @input() 装饰器。

```typescript
@Input() hero: Hero;
```

## 显示 HeroDetailComponent 组件

------

这 `HeroesComponent`  组件任然显示英雄主/明细信息。

`HeroDetailComponent`自身展示的是明细信息。 这两个组件将具有 父/子的关系。当用户从列表中选择英雄时，父组件 `HeroesComponent` 通过发送一个新的 hero 对象来控制 `HeroDetailComponent`展示。

更改 `HeroesComponent`  组件模板。

### 更新  HeroesComponent 组件的模板

------

这 `HeroDetailComponent`  选择器是 'app-hero-detail'，在 `HeroesComponent` 组件模板的底部添加一个 `<app-hero-detail>` 元素。当显示英雄详情时使用它。

绑定 `HeroesComponent.selectedHero` 到该元素的 hero 属性上，像这样：

```html
<app-hero-detail [hero]="selectedHero"></app-hero-detail>
```

[hero] = "selectedHero" 是Angular 的 [property binding](https://angular.io/guide/template-syntax#property-binding) 指令. 

上面语句，表示它是从 `HeroesComponent` 的 `selectedHero` 属性到目标元素（app-hero-detail）中的`hero` 属性的单向数据绑定。它映射到 `HeroDetailComponent` 组件的 `hero` 属性。

当用用户从列表中点一个英雄时，`selectedHero`  属性改变 . 当 `selectedHero` 值改变，这属性绑定将更新 `hero` ，而 `HeroDetailComponent` 组件显示新的英雄信息。

修改后的 `HeroesComponent` 模板应该像这样：

```html
<h2>显示我拥有的英雄</h2>
<ul class="heroes">
  <li *ngFor="let hero of heroes" (click)="onSelect(hero)" 	       [class.selected]="hero === selectedHero">
    <span class="badge">{{hero.id}}</span> {{hero.name}}
  </li>
</ul>
<app-hero-detail [hero]="selectedHero"></app-hero-detail>
```

刷新浏览器，应用程序正常工作。

## 总结

------



- 创建一个独立，可重用的 HeroDetailComponent 组件
- 使用属性绑定来让父组件控制子组件
- 使用@Input装饰器使 hero 属性可以由外部 HeroesComponent 组件绑定。