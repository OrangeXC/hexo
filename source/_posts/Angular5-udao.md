---
title: Angular 5 开发一个有道翻译
date: 2017-11-02 12:02:56
tags: Angular
---

<!--
![](/uploads/Angular5 开发一个有道翻译1.png)
-->

![](https://ws1.sinaimg.cn/large/005Yd2Thly1flblag45saj30jz07j0t8.jpg)

力争国内 Angular 5 第一篇轮子

> Github：https://github.com/OrangeXC/udao

<!--more-->

最近轮子造的比较多，意在给初学者一个参考例子，目前反馈来看，如果技术栈不符，很少有人会点进来读，以后可以考虑转换博文类型了。

之前写过一篇 [Angular2 从搭建环境到开发](https://orangexc.xyz/2016/10/15/Angular2-from-the-build-environment-to-the-development/)，在 segmentfault 上得到了 2016 年第四季度的 [top writer](https://segmentfault.com/a/1190000008093993) 文章表里第四名，如今已经 angular 5

给大家的常规印象就是，大版本跳跃会带来 breaking change，因为 angular 从 1.x 到 2.x 简直是两个框架，不对，就是两个框架。

angular 1.x 叫 angular.js 而 angular 2.x 以后就叫 angular，两个版本分别托管在两个 github repo。

更让我比较惊讶的是 angular.js 的 star 近乎 angular 的 double，而且社区更繁荣，本文苦在找能配合 angular 5 使用的组件，因为框架刚升级，相应的组件都还未更新，下文会告诉大家一个小技巧。

angular 2 到 4 到 5，的组件数成幂指数递减，但是好在可以轻松向后扩展，如果熟悉 angular 2 那么本项目完全可以看懂。

其实写轮子看文档谁都会写，我尽量多说些坑点，让开发者少踩坑。

撸起袖子开整。

## 搭建开发环境

```bash
npm install -g @angular/cli@1.5.0
```

这里直接把版本指向 1.5.0

```bash
ng new PROJECT-NAME
cd PROJECT-NAME
```

这时依赖已经安装完成，执行 `ng -v`，可以看到如下

<!--
![](/uploads/Angular5 开发一个有道翻译2.png)
-->

![](https://ws1.sinaimg.cn/large/005Yd2Thly1flblag4ibdj30da0dbq5g.jpg)

```bash
ng serve
```

默认 4200 端口，就可以看到初始化页面了。

安装过程可能较长，建议本地先安装 yarn，安装依赖的时候 cli 会自动使用 yarn 装依赖，会快不少。

到这就可以开发了。

## 开发

udao 词典的公开接口已经废弃，这里拿来的接口是非官方的，支持的功能有限

这里明确要用的 UI 库是 [ng-bootstrap](https://github.com/ng-bootstrap/ng-bootstrap)，loading 用的是 [ngx-loading](https://github.com/Zak-C/ngx-loading)

安装时会有依赖版本不符的警告，如下

<!--
![](/uploads/Angular5 开发一个有道翻译3.png)
-->

![](https://ws1.sinaimg.cn/large/005Yd2Thly1flblag5z2ej30jz05i77g.jpg)

但是勉强能用，前面说想找到合适的组件库比较困难，这里讲个小技巧，去 google 搜 `angular [some component]` 基本都是 angular 1.x 的组件，那么根据历史分析组件命名有 `ng-` `ng2-`，到了 4 大家感觉心累所以干脆叫 `ngx-`，搜索直接搜 `ngx-[some component]`。

这里为什么是 `ng-bootstrap` 而没选 `ngx-bootstrap` 呢，这里真的有 `ngx-bootstrap`，因为 `ng-bootstrap` 只支持 `bootstrap4`，后者支持 3 和 4，为了避免版本纠纷，直接用了 `ng-bootstrap`。

说到这远远不能证明 angular 5 可以用这个库，我的评判标准是 angular 4，如果支持 angular 4，那么 90% 支持 angular 5，因为改动确实不大。

## 路由

目前此项目只涉及到 4 个路由。

* `/` 主页
* `/translate` 翻译
* `/search` 模糊搜索
* `/detail/:word` 单词详情

在 `app.module.ts` 下面定义路由

```js
const routes: Routes = [
  {
    path: '',
    component: HomeComponent
  }, {
    path: 'translate',
    component: TranslateComponent
  }, {
    path: 'search',
    component: SearchComponent
  }, {
    path: 'detail/:word',
    component: DetailComponent
  }
]
```

这里说下路由跳转相关的问题，在 angular 5 里依然分为 a 标签的跳转和 js 跳转

* a 标签的写法

```js
@Directive({ selector: ':not(a)[routerLink]' })
class RouterLink {
  queryParams: {[k: string]: any}
  fragment: string
  queryParamsHandling: QueryParamsHandling
  preserveFragment: boolean
  skipLocationChange: boolean
  replaceUrl: boolean
  set routerLink: any[]|string
  set preserveQueryParams: boolean
  onClick(): boolean
  get urlTree: UrlTree
}
```

本项目例子：

```html
<li class="nav-item">
  <a class="nav-link" routerLink="/" routerLinkActive="active" [routerLinkActiveOptions]="{exact: true}">主页</a>
</li>
<li class="nav-item">
  <a class="nav-link" routerLink="/translate" routerLinkActive="active">翻译</a>
</li>
<li class="nav-item">
  <a class="nav-link" routerLink="/search" routerLinkActive="active">搜索</a>
</li>
```

> 这里注意一个坑，第一个 `li` 标签，多了 `[routerLinkActiveOptions]="{exact: true}"`，如果不加的话，会导致 `/` 路由下，active 不触发的情况。

* js 写法

```js
class Router {
  constructor(rootComponentType: Type<any>|null, urlSerializer: UrlSerializer, rootContexts: ChildrenOutletContexts, location: Location, injector: Injector, loader: NgModuleFactoryLoader, compiler: Compiler, config: Routes)
  get events: Observable<Event>
  get routerState: RouterState
  errorHandler: ErrorHandler
  navigated: boolean
  urlHandlingStrategy: UrlHandlingStrategy
  routeReuseStrategy: RouteReuseStrategy
  onSameUrlNavigation: 'reload'|'ignore'
  config: Routes
  initialNavigation(): void
  setUpLocationChangeListener(): void
  get url: string
  resetConfig(config: Routes): void
  ngOnDestroy(): void
  dispose(): void
  createUrlTree(commands: any[], navigationExtras: NavigationExtras = {}): UrlTree
  navigateByUrl(url: string|UrlTree, extras: NavigationExtras = {skipLocationChange: false}): Promise<boolean>
  navigate(commands: any[], extras: NavigationExtras = {skipLocationChange: false}): Promise<boolean>
  serializeUrl(url: UrlTree): string
  parseUrl(url: string): UrlTree
  isActive(url: string|UrlTree, exact: boolean): boolean
}
```

本项目例子：

```js
gotoDetail ({ entry }) {
  this.router.navigate([`/detail/${entry}`])
}
```

两个例子相比文档的概览都是最简单的用法，有需要的话可以看下其它方法，基本可以满足所有的路由需求。

## 请求

这里不同于 vue 和 react，angular 提供了前端全栈的解决方案，包含了 http 模块，只需要在 `app.module.ts` 里面引入

```
import { HttpClientModule } from '@angular/common/http'

// ...
imports: [
  HttpClientModule
]
// ...
```

请求的语法也很简单，具体可以到 github 看代码。

这里说一个小坑，在实现 `detail` 路由的时候在 `ngOnInit` 钩子里拿到当前路由参数进行请求，改变路由时没有触发请求更新，最后改版如下。

代码如下

```js
ngOnInit () {
  this.route.params.subscribe((params) => {
    this.loading = true

    const apiURL = `https://dict.youdao.com/jsonapi?q=${params['word']}`

    this.http.get(`/?url=${encodeURIComponent(apiURL)}`)
    .subscribe(res => {
      // set component data

      this.loading = false
    })
  })
}
```

之前无效是因为没写 `this.route.params.subscribe((params) => {})`，所以每次不会触发监听

这里的 `subscribe` 会一直监听 `this.route.params` 的变化。

## 请求路径

如同 axios 的 baseURL，在请求时我们不希望每个请求都写完整路径，需要配置全局的 baseURL 来使得请求路径简短。

angular 里面需要一个 `@Injectable`，熟悉的概念——依赖注入，关于细则有跟多文章介绍，这里说下针对此需求的解决方案

```js
@Injectable()
export class ExampleInterceptor implements HttpInterceptor {
  intercept(req: HttpRequest<any>, next: HttpHandler): Observable<HttpEvent<any>> {
    const url = 'https://proxy-oagpwnbkpe.now.sh'

    req = req.clone({
      url: url + req.url
    })

    return next.handle(req)
  }
}
// ...
providers: [
  AppComponent,
  { provide: HTTP_INTERCEPTORS, useClass: ExampleInterceptor, multi: true }
]
// ...
```

这段代码解决了 baseURL 的问题。

## 请求转发

注意到上一节的这里 `const url = 'https://proxy-oagpwnbkpe.now.sh'`，根路径不是有道的路径。

还是做了一层 node 的 proxy 处理。跨域问题还是要处理。

node 服务的代码也十分简单，这里使用了 [fly](https://github.com/wendux/fly) 进行 node 端请求

代码如下

```js
const express = require('express')
const fly = require('flyio')
const app = express()

app.use('/', async (req, res) => {
  const data = await fly.get(req.query.url).then(res => res.data)

  res.set('Access-Control-Allow-Origin', '*')

  res.send(data)
})

app.listen(process.env.PORT || 3001)
```

重点是在返回头 set 一个 `Access-Control-Allow-Origin: *`，这样浏览器就不会拦截请求了。

## 数据流动

在 `detail` 页面，拆分了 5 个子组件，当然父子组件是十分简单的单向数据流

例：父组件的 html 如下

```html
<app-detail-phrs-list-tab [simple]="simple" [ec]="ec"></app-detail-phrs-list-tab>
```

子组件的 `component.ts` 如下

```js
export class DetailPhrsListTabComponent {
  @Input() simple
  @Input() ec
}
```

就可以使用 `@Input` 取到父组件传进来的值了，说到这里全局的状态管理怎么做，要看下项目的复杂度

简单的全局状态管理可以创建一个 `global.ts`，再创建依赖注入，如下

```js
// globals.ts
import { Injectable } from '@angular/core';

@Injectable()
export class Globals {
  role: string = 'test';
}
```

在组件中可以这样调用

```js
// hello.component.ts
import { Component } from '@angular/core';
import { Globals } from './globals';

@Component({
  selector: 'hello',
  template: 'The global role is {{globals.role}}',
  providers: [Globals]
})

export class HelloComponent {
  constructor(private globals: Globals) {}
}
```

另一种方式是 SPA 开发者熟悉的全局状态管理库，如 flex, redux

angular 也提供了 [angular-redux](https://github.com/angular-redux/store)，复杂应用中建议使用。

## 打包上线

打包命令围绕 `ng build`，提供几种配置参数，这里不赘述，

部署这里使用的是 [surge](http://surge.sh/)

友情提示：不要将私有项目部署到此类公开服务，弊端很多。

## 总结

不论哪种前端框架，都有它的长处，由于此项目较小，到这里没机会释放 `rxjs` 的威力，angular-cli 默认装了这个库，处理复杂的异步数据流非常高效，写了好多轮子，毕竟还是样例，但是，折腾不能停。

## 相关文档

本次 angular 5 更新相关文档如下

官方文档：https://next.angular.io/docs
官方博客：https://blog.angular.io/version-5-0-0-of-angular-now-available-37e414935ced
官方cli：https://github.com/angular/angular-cli/releases/tag/v1.5.0

> 尽量翻墙查看，国内 https://angular.cn/ 文档还没更新。
