---
title: Angular5 服务端渲染实战
date: 2018-01-03 15:28:04
tags: ['Angular', 'SSR']
---

<!--
![](/uploads/Angular5 服务端渲染实战1.png)
-->

![](https://ws1.sinaimg.cn/large/005Yd2Thly1fn3gisbt0hj31hc0f0wem.jpg)

<!--more-->

本文基于上一篇 Angular5 的文章继续进行开发，上文中讲了搭建 Angular5 有道翻译的过程，以及遇到问题的解决方案。

随后改了 UI，从 bootstrap4 改到 angular material，这里不详细讲，服务端渲染也与修改 UI 无关。

看过之前文章的人会发现，文章内容都偏向于服务端渲染，vue 的 nuxt，react 的 next。

在本次改版前也尝试去找类似 nuxt.js 与 next.js 的顶级封装库，可以大大节省时间，但是未果。

最后决定使用从 Angular2 开始就可用的前后端同构解决方案 [Angular Universal](https://github.com/angular/universal)（Universal (isomorphic) JavaScript support for Angular.）

在这里不详细介绍文档内容，本文也尽量使用通俗易懂的语言带入 Angular 的 SSR

## 前提

前面写的 udao 这个项目是完全遵从于 angular-cli 的，从搭建到打包，这也使得本文通用于所有 angular-cli 搭建的 angular5 项目。

## 搭建过程

首先安装服务端的依赖

```bash
yarn add @angular/platform-server express
yarn add -D ts-loader webpack-node-externals npm-run-all
```

> 这里需要注意的是 `@angular/platform-server` 的版本号最好根据当前 angular 版本进行安装，如: `@angular/platform-server@5.1.0`，避免与其它依赖有版本冲突。

创建文件: `src/app/app.server.module.ts`

```js
import { NgModule } from '@angular/core'
import { ServerModule } from '@angular/platform-server'

import { AppModule } from './app.module'
import { AppComponent } from './app.component'

@NgModule({
  imports: [
    AppModule,
    ServerModule
  ],
  bootstrap: [AppComponent],
})
export class AppServerModule { }
```

更新文件: `src/app/app.module.ts`

```js
import { BrowserModule } from '@angular/platform-browser'
import { NgModule } from '@angular/core'
// ...

import { AppComponent } from './app.component'
// ...

@NgModule({
  declarations: [
    AppComponent
    // ...
  ],
  imports: [
    BrowserModule.withServerTransition({ appId: 'udao' })
    // ...
  ],
  providers: [],
  bootstrap: [AppComponent]
})
export class AppModule { }
```

我们需要一个主文件来导出服务端模块

创建文件: `src/main.server.ts`

```js
export { AppServerModule } from './app/app.server.module'
```

现在来更新 `@angular/cli` 的配置文件 `.angular-cli.json`

```json
{
  "$schema": "./node_modules/@angular/cli/lib/config/schema.json",
  "project": {
    "name": "udao"
  },
  "apps": [
    {
      "root": "src",
      "outDir": "dist/browser",
      "assets": [
        "assets",
        "favicon.ico"
      ]
      // ...
    },
    {
      "platform": "server",
      "root": "src",
      "outDir": "dist/server",
      "assets": [],
      "index": "index.html",
      "main": "main.server.ts",
      "test": "test.ts",
      "tsconfig": "tsconfig.server.json",
      "testTsconfig": "tsconfig.spec.json",
      "prefix": "app",
      "scripts": [],
      "environmentSource": "environments/environment.ts",
      "environments": {
        "dev": "environments/environment.ts",
        "prod": "environments/environment.prod.ts"
      }
    }
  ]
  // ...
}
```

上面的 `// ...` 代表省略掉，但是 json 没有注释一说，看着怪怪的....

当然 `.angular-cli.json` 的配置不是固定的，根据需求自行修改

我们需要为服务端创建 `tsconfig` 配置文件: `src/tsconfig.server.json`

```json
{
  "extends": "../tsconfig.json",
  "compilerOptions": {
    "outDir": "../out-tsc/app",
    "baseUrl": "./",
    "module": "commonjs",
    "types": []
  },
  "exclude": [
    "test.ts",
    "**/*.spec.ts",
    "server.ts"
  ],
  "angularCompilerOptions": {
    "entryModule": "app/app.server.module#AppServerModule"
  }
}
```

然后更新: `src/tsconfig.app.json`

```json
{
  "extends": "../tsconfig.json",
  "compilerOptions": {
    "outDir": "../out-tsc/app",
    "baseUrl": "./",
    "module": "es2015",
    "types": []
  },
  "exclude": [
    "test.ts",
    "**/*.spec.ts",
    "server.ts"
  ]
}
```

现在可以执行以下命令，看配置是否有效

```bash
ng build -prod --build-optimizer --app 0
ng build --aot --app 1
```

运行结果应该如下图所示

<!--
![](/uploads/Angular5 服务端渲染实战2.png)
-->

![](https://ws1.sinaimg.cn/large/005Yd2Thly1fn3i6c6g2lj312k0hajv4.jpg)

然后就是创建 `Express.js` 服务, 创建文件: `src/server.ts`

```js
import 'reflect-metadata'
import 'zone.js/dist/zone-node'
import { renderModuleFactory } from '@angular/platform-server'
import { enableProdMode } from '@angular/core'
import * as express from 'express'
import { join } from 'path'
import { readFileSync } from 'fs'

enableProdMode();

const PORT = process.env.PORT || 4200
const DIST_FOLDER = join(process.cwd(), 'dist')

const app = express()

const template = readFileSync(join(DIST_FOLDER, 'browser', 'index.html')).toString()
const { AppServerModuleNgFactory } = require('main.server')

app.engine('html', (_, options, callback) => {
  const opts = { document: template, url: options.req.url }

  renderModuleFactory(AppServerModuleNgFactory, opts)
    .then(html => callback(null, html))
});

app.set('view engine', 'html')
app.set('views', 'src')

app.get('*.*', express.static(join(DIST_FOLDER, 'browser')))

app.get('*', (req, res) => {
  res.render('index', { req })
})

app.listen(PORT, () => {
  console.log(`listening on http://localhost:${PORT}!`)
})
```

理所当然需要一个 webpack 配置文件来打包 `server.ts` 文件: `webpack.config.js`

```js
const path = require('path');
var nodeExternals = require('webpack-node-externals');

module.exports = {
  entry: {
    server: './src/server.ts'
  },
  resolve: {
    extensions: ['.ts', '.js'],
    alias: {
      'main.server': path.join(__dirname, 'dist', 'server', 'main.bundle.js')
    }
  },
  target: 'node',
  externals: [nodeExternals()],
  output: {
    path: path.join(__dirname, 'dist'),
    filename: '[name].js'
  },
  module: {
    rules: [
      { test: /\.ts$/, loader: 'ts-loader' }
    ]
  }
}
```

为了打包方便最好在 `package.json` 里面加几行脚本，如下：

```json
"scripts": {
  "ng": "ng",
  "start": "ng serve",
  "build": "run-s build:client build:aot build:server",
  "build:client": "ng build -prod --build-optimizer --app 0",
  "build:aot": "ng build --aot --app 1",
  "build:server": "webpack -p",
  "test": "ng test",
  "lint": "ng lint",
  "e2e": "ng e2e"
}
```

现在尝试运行 `npm run build`，将会看到如下输出：

<!--
![](/uploads/Angular5 服务端渲染实战3.png)
-->

![](https://ws1.sinaimg.cn/large/005Yd2Thly1fn3ifqh45lj312g0x444m.jpg)

node 运行刚刚打包好的 `node dist/server.js` 文件

打开 `http://localhost:4200/` 会正常显示项目主页面

<!--
![](/uploads/Angular5 服务端渲染实战4.png)
-->

![](https://ws1.sinaimg.cn/large/005Yd2Thly1fn3ilp4i5yj313o0laain.jpg)

从上面的开发者工具可以看出 html 文档是服务端渲染直出的，接下来尝试请求数据试一下。

> 注意：本项目显式（菜单可点击）的几个路由初始化都没有请求数据，但是单词解释的详情页是会在 `ngOnInit()` 方法里获取数据，例如：`http://localhost:4200/detail/add` 直接打开时会发生奇怪的现象，请求在服务端和客户端分别发送一次，正常的服务端渲染项目首屏初始化数据的请求在服务端执行，在客户端不会二次请求!

发现问题后，就来踩平这个坑

试想如果采用一个标记来区分服务端是否已经拿到了数据，如果没拿到数据就在客户端请求，如果已经拿到数据就不发请求

当然 Angular 早有一手准备，那就是 `Angular Modules for Transfer State`

那么如何真实运用呢？见下文

## 请求填坑

在服务端入口和客户端入口分别引入 `TransferStateModule`

```js
import { ServerModule, ServerTransferStateModule } from '@angular/platform-server';
// ...

@NgModule({
  imports: [
    // ...
    ServerModule,
    ServerTransferStateModule
  ]
  // ...
})
export class AppServerModule { }
```

```js
import { BrowserModule, BrowserTransferStateModule } from '@angular/platform-browser';
// ...

@NgModule({
  declarations: [
    AppComponent
    // ...
  ],
  imports: [
    BrowserModule.withServerTransition({ appId: 'udao' }),
    BrowserTransferStateModule
    // ...
  ]
  // ...
})
export class AppModule { }
```

以本项目为例在 `detail.component.ts` 里面，修改如下

```js
import { Component, OnInit } from '@angular/core'
import { HttpClient } from '@angular/common/http'
import { Router,  ActivatedRoute, NavigationEnd } from '@angular/router'
import { TransferState, makeStateKey } from '@angular/platform-browser'

const DETAIL_KEY = makeStateKey('detail')

// ...

export class DetailComponent implements OnInit {
  details: any

  // some variable

  constructor(
    private http: HttpClient,
    private state: TransferState,
    private route: ActivatedRoute,
    private router: Router
  ) {}

  transData (res) {
    // translate res data
  }

  ngOnInit () {
    this.details = this.state.get(DETAIL_KEY, null as any)

    if (!this.details) {
      this.route.params.subscribe((params) => {
        this.loading = true

        const apiURL = `https://dict.youdao.com/jsonapi?q=${params['word']}`

        this.http.get(`/?url=${encodeURIComponent(apiURL)}`)
        .subscribe(res => {
          this.transData(res)
          this.state.set(DETAIL_KEY, res as any)
          this.loading = false
        })
      })
    } else {
      this.transData(this.details)
    }
  }
}
```

代码够简单清晰，和上面描述的原理一致

现在我们只需要对 `main.ts` 文件进行小小的调整，以便在 `DOMContentLoaded` 时运行我们的代码，以使 `TransferState` 正常工作：

```js
import { enableProdMode } from '@angular/core'
import { platformBrowserDynamic } from '@angular/platform-browser-dynamic'

import { AppModule } from './app/app.module'
import { environment } from './environments/environment'

if (environment.production) {
  enableProdMode()
}

document.addEventListener('DOMContentLoaded', () => {
  platformBrowserDynamic().bootstrapModule(AppModule)
    .catch(err => console.log(err))
})
```

到这里运行 `npm run build && node dist/server.js` 然后刷新 `http://localhost:4200/detail/add` 到控制台查看 network 如下：

<!--
![](/uploads/Angular5 服务端渲染实战5.png)
-->

![](https://ws1.sinaimg.cn/large/005Yd2Thly1fn3jhutkshj310r0kqjvm.jpg)

发现 XHR 分类里面没有发起任何请求，只有 service-worker 的 cache 命中。

到这里坑都踩完了，项目运行正常，没发现其它 bug。

## 总结

2018 第一篇，目的就是探索所有流行框架服务端渲染的实现，开辟了 angular 这个最后没尝试的框架。

当然 Orange 还是前端小学生一枚，只知道实现，原理说的不是很清楚，源码看的不是很明白，如有纰漏还望指教。

最后 Github 地址和之前文章一样：https://github.com/OrangeXC/udao

Github 附有在线链接，好的就说到这了
