---
title: 用 Nuxt 开发部署一个 v2ex
date: 2017-06-19 10:57:48
tags: ['Vue', 'SSR', 'Nuxt']
---

![](/uploads/n2ex1.png)

<!-- more -->

先放出Github地址：https://github.com/OrangeXC/n2ex

里面有线上网站的链接，因为链接随时可能变，在这里不直接给网站链接。

之前写过一篇 nuxt 入门级的文章 [Vue 基于 NUXT 的 SSR](http://orangexc.xyz/2016/12/27/Vue-nuxt-based-ssr/)，主要说一下 nuxt 是什么，以及为什么使用。

这里声明一下，不建议去阅读上一篇文章，因为当时写博文的时候是 0.8.0 版本，目前是 1.0.0alpha4，已经有一部分改动，建议去看最新的[nuxt文档](https://nuxtjs.org/)

了解 nuxt 后，就可以轻松的看下文了，简单易懂，也没写什么复杂的项目。

本着自己学习的目的分享给大家，因为上篇文章之后有好多读者问 orange，怎么开发，怎么部署到服务器。

下面进入正题

## 环境搭建

nuxt 相关的脚手架已经集成到了 vue-cli，同时提供 starter、express、koa、adonuxt

这里我们用的是 koa2（脚手架会询问使用 koa1 或 koa2）

```bash
vue init nuxt/koa <project-name>
cd <project-name> # move to your project
npm install # or yarn install*[see note below]
npm run dev
```

> 此时监听 3000 端口，如果有 bug，别犹豫，先升级 node 版本到最新。

项目跑起来之后，有一个简单的轮廓，两个页面，index 和 about。

## v2ex API

写一个三方 API 项目时，首先要看看人家都支持什么 API，才能决定我们如何展示页面。

来看看[官方 API 文档](https://www.v2ex.com/p/7v9TEc53)

这个文档说来仔细，但是仅仅提供了 4 个 API，对于我们来说远远不够，那本站的 API 从哪里来的呢

Github 的确是个好网站，我找到了这个项目下的一个文件：https://github.com/ochapman/v2ex/blob/master/v2ex.go

不会 go 语言的没关系，我也不熟悉 go 语言，读一读会发现给出了比官方文档更多的 API，当然还有更详细的 API 暂且不谈。

本项目取的就是这个文件里（隐藏）的 API

* 热门话题
* 最新话题
* 节点列表
* 节点信息
* 话题详情
* 话题评论
* 用户详情
* 用户话题

我们也就实现了上面列表这么多接口的前端展示

## 路由结构

nuxt 的特点之一就是以目录结构划分路由。

router 由 pages 目录决定，那么分析接口可以得到以下目录结构

```
pages
  |
  |-- member
  |    |
  |    |-- _name.vue
  |
  |-- node
  |    |
  |    |-- _name.vue
  |
  |-- topic
  |   |
  |   |-- _id.vue
  |
  |-- index.vue
  |
  |-- new.vue
```

很清晰的可以看出我们的路由结构，细心的会发现 params 有的是 name 有的是 id，为什么？

> 这里详细解释下，v2ex 接口提供了 id 和 name 两种 url 传参形式，任何一种查询都可以匹配结果，唯独 topic 只能 id 查询，因为 name 不唯一，那用户和节点也提供了 id 查询啊，这里的坑就在评论的 `@` 部分，当 @ 一个人时，在评论可以直接链到个人详情页，v2ex 在评论里默认解析的就是 username 对应的链接，所以为了统一，其它地方也用的 name，另外无形当中提供了 search，在对应 url 后面替换成要查找的节点或用户名就可以直接跳转过去。

## 组件

这里只说两个最应该抽离的业务组件

* 话题 list
* 评论 list

话题 list 几乎每个列表页面里都有，而评论 list 在每个详情页里都有

基础组件用的是 [muse-ui](https://github.com/museui/muse-ui)，比较喜欢 Material 整体的设计风格，刚好在 muse-ui 的 2.0.3 版本支持了 SSR。

下面说下引入三方库相关的问题

## 引入三方库

muse-ui 建议使用 plugins 的方式引入，因为涉及到 Vue.use 挂载方法

在 plugins 下新建 muse-ui.js 如下

```js
import Vue from 'vue'
import MuseUI from 'muse-ui'

Vue.use(MuseUI)
```

然后在 nuxt.config.js 里面加上

```js
plugins: [
  { src: '~plugins/muse-ui.js', ssr: true }
]
```

另外值得注意的是，需要全局引入 google 字体库，这里我直接插入到了 head 的 link 标签里

```js
link: [
  { rel: 'stylesheet', href: 'https://fonts.googleapis.com/css?family=Roboto:300,400,500,700,400italic' },
  { rel: 'stylesheet', href: 'https://fonts.googleapis.com/icon?family=Material+Icons' }
]
```

http 请求用的是前后端同构的 axios 库

打包的时候注意要在配置文件加进去

```js
build: {
  vendor: ['axios']
}
```

## 异步请求

nuxt 提供了 asyncData，可以在页面加载之前请求数据。

在这里使用 es7 的 async/await 来实现数据请求

例如：pages/index.vue

```js
async asyncData () {
  try {
    const { data } = await axios.get(`https://proxy-uuptfgaypk.now.sh/topics/hot.json`)

    return {
      hotList: data
    }
  } catch (err) {
    console.error(err)
  }
}
```

可读性还是挺高的，请求回来的 object，取到里面的 data 赋值给 hotList，省去了 `.then` 的操作

在详情页需要同时得到话题详细内容和评论，走的是两个接口

那么问题来了，怎么才能同时请求多个资源，当多个资源全部请求完成时才返回。

await 只能顺次请求，promis + await ？？？

不不不，只要 promis 的 all 方法就可以了，axios 有相应的封装

```js
asyncData ({ params, error }) {
  return axios.all([
    axios.get(`https://proxy-uuptfgaypk.now.sh/topics/show.json?id=${params.id}`),
    axios.get(`https://proxy-uuptfgaypk.now.sh/replies/show.json?topic_id=${params.id}`)
  ])
  .then(axios.spread(function (detail, comments) {
    return {
      detail: detail.data[0],
      comments: comments.data
    }
  }))
  .catch(error => console.log(error))
}
```

这样一来解决了同时请求多个接口的问题。

## CORS

跨域 http 请求，在这里不详细解释，给大家 [MDN 链接](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Access_control_CORS)

细心的小伙伴发现上文代码的 url 是 `http://proxy...`，为什么不是官方给的 `https://www.v2ex.com/api`

那是因为跨域请求时浏览器限制请求跨域资源，正常走官方的请求会报错，信息如下

```
XMLHttpRequest cannot load https://www.v2ex.com/api/topics/latest.json. No 'Access-Control-Allow-Origin' header is present on the requested resource. Origin 'localhost:3000' is therefore not allowed access.
```

报错很明显没有 `Access-Control-Allow-Origin` 返回头，打开控制台发现有数据返回，但是被浏览器拦截了，并没有加载到页面中去

初次玩服务端渲染的还会遇到的问题就是我什么首屏刷新不会报错，而路由跳转的请求会报错呢？

> 这要从服务端渲染机制说起，首屏的请求是在服务端完成，服务端不存在跨域问题，而接下来的交互操作和页面跳转是在浏览器端进行，所以产生了类似的问题。够简单直接吧，不相信的可以自己打 console，看是在终端控制台输出还是浏览器控制台输出。

找到了问题接下来就需要解决问题，上面有说在服务端不存在跨域请求的问题。

那么我们就自己写一层 proxy 就好啦，写一个 node 服务，转发请求，然后在返回头里加上，`Access-Control-Allow-Origin: *`

这个服务实际上不到十行的代码，用到两个依赖，express 和 request

```js
const express = require('express')
const request = require('request')
const app = express()

app.use('/', function(req, res) {
  const url = 'https://www.v2ex.com/api' + req.url
  req.pipe(request(url)).pipe(res.set('Access-Control-Allow-Origin', '*'))
})

app.listen(process.env.PORT || 3001)
```

开发环境下先启动代理服务，然后将 url 指向本地服务就可以了

上面的方法呢，说实话有点蠢，实际项目当中呢可以直接让后端返回的接口支持跨域，当然了任何人都可以使用你们的 API，不是十分合理

再有就是 nuxt 官方有个 modules 组件库不知道大家有没有注意，地址：https://github.com/nuxt/modules

里面其中有 axios 和 proxy 的封装，意在解决 axios 的 baseUrl 和 proxy 跨域限制，安装配置都十分方便，本次为什么没用？

好问题，因为存在未知的坑，代码没有丝毫报错，就是不生效，只能静等 nuxt 官方修复主库与插件之间的 bug。

## 部署

怎么部署是大家最关心的问题，项目倒是好写，只要你会 vue 看看文档就可以写。

部署实际上官方提供了两个命令，打包和运行

```bash
npm run build
npm start
```

这里需要一个安装了 node 的服务器，可以安装一个 [pm2](https://github.com/Unitech/pm2) 来跑 node 服务

当然喜欢 docker，也可以用 docker 去部署

之前又有人问了，看了这两个东西，依旧不会部署，那我也无能为力了，只能说，科学上网，教程一大堆。

如果就是想跑一个自己的 DEMO 玩玩，不想单独买服务器，也不涉及到企业项目部署和安全问题

那么好！给两个可以免费跑 node 服务的供应商 heroku 和 now.sh

nuxt 项目怎么如何跑在这两个服务上官网有写 https://zh.nuxtjs.org/faq/heroku-deployment

本项目是跑在 now.sh 上的，这也就解释了为什么说这个在线链接打开速度超级慢，因为我们用的是三方的免费服务，为了提高服务器资源的利用率，减小服务器压力，当一段时间没人访问网站时，会自动把网站设置为 frozen

```
The deployment n2ex-yrgirchtae.now.sh was frozen
The deployment proxy-uuptfgaypk.now.sh was frozen
The deployment n2ex-nzkjwvytxe.now.sh was unfrozen
```

这是我控制台的最新报告，当有人访问时会切换到 unfrozen，算了下默认 frozen 时间是 15min 内无访问后。

不知道 heroku 是不是也有类似问题

## 未来

这个项目会持续更新，逐步加新的功能，大家感兴趣的可以提 issue，或者直接提 pr 给我。

## 总结

从项目分析到开发部署上线，一个 nuxt 项目就这样完成了，开发遇到的坑也随着项目递进渗透进去了，项目十分简单，没使用 vuex，写到这里，依旧不推荐大家深入使用，但是十分推荐玩一玩，抛开了 SSR 复杂的那一面，用着还是挺爽的。
