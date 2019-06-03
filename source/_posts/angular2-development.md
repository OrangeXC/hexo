---
title: Angular2 从搭建环境到开发
date: 2016-10-15 10:38:55
tags: Angular
---

![](/uploads/angular2-development1.png)

Angular2 的发布带来了一阵热议，很久之前就在筹备了，当时的官方答复就是彻底推翻重写，问世之后大家的呼声就是学习成本太高，虽然去掉了 1.x 里的一部分概念，但是加进了 typescript，虽然不强制使用，但是我推荐大家都试一试，毕竟此次改版是谷歌和微软两大家的产物。

对于会部署环境的可以尝试本文最后一节加入 Angular material2 ，个人认为对高度个性化的项目不推荐使用，对企业级的 CMS 省去了写样式的时间，直接开始正文。

<!--more-->

## Angular-CLI

说到 cli 大家不陌生，没出一个框架都会有对应的 cli ，俗称脚手架。angular2 本身提供了起步项目 [angular2-quickstart](https://github.com/valor-software/angular2-quickstart)，我尝试了一下，发现不是很好用，其它的大部分扩展需要自行安装，之后看了一下 angular-cli 部署简单易用，还提供了快捷搭建项目的目录。

Github地址： https://github.com/angular/angular-cli

我就简单说下 Github 里的文档吧，细部的大家扩展阅读。

### 安装

首先，最好先升级 node 到 6.x 可以避免 node 版本过低带来的不必要的麻烦。

```bash
npm install -g angular-cli
```

### 用法

```bash
ng --help
```

查看所有用法

### 创建本地开发环境生成和运行angular2项目

```bash
ng new PROJECT_NAME
cd PROJECT_NAME
ng serve
```

PROJECT_NAME 是你自己的项目名

部署成功后不报错的情况下到浏览器 http://localhost:4200/，修改项目中文件后会自动部署

您可以配置默认的 HTTP 端口和一个 LiveReload server 用 `--`， 形如：

```bash
ng serve --host 0.0.0.0 --port 4201 --live-reload-port 49153
```

### 生成组件、指令、管道和服务

命令以 `ng generate` 开头，可以缩写为 `ng g`，下面给出创建 component 的几种方式。

```bash
ng generate component my-new-component
ng g component my-new-component # using the alias

# components support relative path generation
# if in the directory src/app/feature/ and you run
ng g component new-cmp
# your component will be generated in src/app/feature/new-cmp
# but if you were to run
ng g component ../newer-cmp
# your component will be generated in src/app/newer-cmp
```

下表里是所有的命令：

Scaffold	  | Usage
------------|------------------------------------------------------------------
Component   | ng g component my-new-component
Directive   | ng g directive my-new-directive
Pipe        | ng g pipe my-new-pipe
Service     | ng g service my-new-service
Class       | ng g class my-new-class
Interface   | ng g interface my-new-interface
Enum        | ng g enum my-new-enum
Module      | ng g module my-module

### 创建路由

这里 cli 暂时禁用了创建路由，新的路由生成器即将到来，您可以在这里阅读新路由器的官方文档：https://angular.io/docs/ts/latest/guide/router.html

### 建立一个 build

```
ng build
```

会生成到 `dist/` 目录下，其它关于测试，配置文件请大家去 Github 仔细阅读，这里只给最基本的搭建流程。

## 组件实战

看到这你可能已经开始尝试了，创建项目的步骤相信大家参照上文可以轻松解决，这里我先尝试创建一个 component，命令如下。

```
ng g component nav
```

这里我创建了一个 nav 组件。执行成功后，后台会自动部署。我们看一下文件目录有什么改变

![](/uploads/angular2-development2.png)

多了一个叫做 nav 的文件夹，看一看文件目录

![](/uploads/angular2-development3.png)

我们发现与项目创建时自带的 app component 目录结构相同，内容也大同小异，大家可以尝试去创建一个自己的组件，组件的样式可以去对应的 css 文件中修改。

这时我的 `app.module.ts` 变成了如下

```js
import { BrowserModule } from '@angular/platform-browser';
import { NgModule } from '@angular/core';
import { FormsModule } from '@angular/forms';
import { HttpModule } from '@angular/http';

import { AppComponent } from './app.component';
import { NavComponent } from './nav/nav.component';

@NgModule({
  declarations: [
    AppComponent,
    NavComponent
  ],
  imports: [
    BrowserModule,
    FormsModule,
    HttpModule,
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

这里不难看出全局自动引入了 nav.component 组件。我们现在关心的问题是组件之间的引用和数据传输，这里为了简单起见，只给引入的方法，而数据传输、路由机制这里不做解释大家自行官网。

下面说一下 app 内引入 nav 组件，只需要改变 `app.component.html` 如下。

```html
<h1 class="title">
  {{title}}
</h1>
<app-nav></app-nav>
```

这里的 class 在对应的 `app.component.css` 如下

```css
.title {
  font-size: 100px;
}
```

这时页面自动刷新字号变大 ，那么这里的 `app-nav` 标签从哪里得到的呢？

我们去 `nav.component.ts` 里看一眼

```js
import { Component, OnInit } from '@angular/core';

@Component({
  selector: 'app-nav',
  templateUrl: './nav.component.html',
  styleUrls: ['./nav.component.css']
})
export class NavComponent implements OnInit {

  constructor() { }

  ngOnInit() {
  }

}
```

这里的 `selector: 'app-nav'` 说明我们的选择器选择的是 `app-nav` 标签，同样的可以通过 `[app-nav]` 选择属性。

> 注：这里 selector 类似 css 中的选择器，大家也可以根据 1.x 中的 directive 来理解这里的组件

此时页面会呈现成这样

![](/uploads/angular2-development4.png)

好，到这里简单的组件引用已经实现。

## 引入 Angular material2

文章开头已经阐述了引入 Angular material2 的优点，用过其它组件样式框架的都明白。

安装命令

```bash
npm install --save @angular/material
```

在 `src/app/app.module.ts` 中引入框架

```js
import { MaterialModule } from '@angular/material';
// other imports
@NgModule({
  imports: [MaterialModule.forRoot()],
  ...
})
export class PizzaPartyAppModule { }
```

引入核心和主体风格，较 Angular material 1.x 的改进在于可以选择不同的色系。具体看文档链接：https://github.com/angular/material2/blob/master/docs/theming.md

我们这里用的是 Angular CLI 这里又可以钻空子啦，添加下面一行到 `style.css`，注意是 `src` 目录下的文件

```css
@import '~@angular/material/core/theming/prebuilt/deeppurple-amber.css';
```

`deeppurple-amber` 主题颜色是可变的，具体看上文的文档链接。

到这里一直打开控制台（是个好习惯）的朋友会发现类似下面的报错。

```js
client:49 [default] J:\workspace\angular2\ts\epimss\node_modules\@angular2-material\slide-toggle\slide-toggle.d.ts:67:19
Cannot find name 'HammerInput'.

client:49 [default] J:\workspace\angular2\ts\epimss\node_modules\@angular2-material\core\gestures\MdGestureConfig.d.ts:4:39
Cannot find name 'HammerManager'.
```

文档也给出了解释，因为框架中 `md-slide-toggle` 和 `md-slider` 两个组件依赖外部第三方组件 [HammerJS](http://hammerjs.github.io/) 需要额外的配置。

我们不急着用文档给的 npm 或引入 cdn 路径，因为亲测还是会报错，可能我引入方式有误，为了大家少走弯路直接给亲测有效的方法

我们先去命令行工具运行 `npm i --save-dev @types/hammerjs`

然后编辑 `tsconfig.json` 文件将 hammerjs 添加到 types 下

```js
"types": [
  "jasmine", "hammerjs"
]
```

到这里发现页面自动刷新后报错消失了，如果需要字体图标可以在 `src/index.html` 中引入

```html
<link href="https://fonts.googleapis.com/icon?family=Material+Icons" rel="stylesheet">
```

目前为止，整个 Angular material2 已经整装待发。全部语法看这里：https://github.com/angular/material2#feature-status

我们尝试着添加多个按钮组件测试一下，修改 `app.component.html` 文件，完整代码如下

```html
<h1 class="title">
  {{title}}
</h1>
<app-nav></app-nav>

<button md-button>FLAT</button>
<button md-raised-button>RAISED</button>
<button md-icon-button>
  <md-icon class="md-24">favorite</md-icon>
</button>
<button md-fab>
  <md-icon class="md-24">add</md-icon>
</button>
<button md-mini-fab>
  <md-icon class="md-24">add</md-icon>
</button>
<br/>
<br/>

<button md-raised-button color="primary">PRIMARY</button>
<button md-raised-button color="accent">ACCENT</button>
<button md-raised-button color="warn">WARN</button>
<br/>
<br/>

<button md-button disabled>OFF</button>
<button md-raised-button [disabled]="isDisabled">OFF</button>
<button md-mini-fab [disabled]="isDisabled"><md-icon>check</md-icon></button>
```

没问题这里手懒不写布局样式了，直接给 br 换行大家方便看些，待页面部署完成后我们会看到以下效果

![](/uploads/angular2-development5.png)

炫酷的组件，更多组件语法参考上面给的链接，到这里相信大家学习 angular2 的信心倍增，真对已有组件可以完成快速开发，下一步就是大家去 Angular2 官网看其它概念的时候啦，处理数据实现与后端对接。项目上线，大功告成。

## 总结

orange 最近也是在学习新技术，更底层框架方面的知识还再学习，当今前端框架层出不穷，不要盲从，要根据公司需求和员工的工作经验选择框架，真说到性能方面哪个框架快的话，我虽然没测试过，但我确定 React、Vue、Angular2 几个之间相差无几，除非在实现的时候代码存在问题，因为这几个框架都经过了大型项目的考验。
