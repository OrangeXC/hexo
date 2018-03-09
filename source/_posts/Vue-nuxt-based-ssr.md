---
title: Vue 基于 NUXT 的 SSR
date: 2016-12-27 10:09:36
tags: ['Vue', 'SSR', 'Nuxt']
---

<!--
![](/uploads/Vue 基于 NUXT 的 SSR1.png)
-->

![](https://ws1.sinaimg.cn/large/005Yd2Thly1fl9ko1me3tj30zz0h4aao.jpg)

<!--more-->

## SSR

首先说下 SSR，最近很热的词，意为 Server Side Rendering（服务端渲染），目的是为了解决单页面应用的 SEO 的问题，对于一般网站影响不大，但是对于论坛类，内容类网站来说是致命的，搜索引擎无法抓取页面相关内容，也就是用户搜不到此网站的相关信息。

抓取页面的前提是 html 含有被抓取内容，我们不妨看看基于 vue 的线上 SPA 页面请求时返回了什么

```html
<!DOCTYPE html>
<html>
  <head>
    <meta charset=utf-8>
    <title>iDareX敢玩</title>
    <meta name=keywords content="敢玩, iDareX, 敢玩TV, 敢玩活动, 敢玩自频道, 敢玩主题, 户外, 极限运动, 周边游, 探险, 时尚, 新潮, 运动视频, 体育, 新奇, 生活方式, 刺激, 惊险, 户外装备, 达人, 90后">
    <meta name=description content=自2014年10月创办以来，敢玩专注于极限户外和娱乐体育。从顽童、玩具、玩法三个方面，产出更专注于‘玩’的内容，已打造了一系列深受喜爱的娱乐体育真人秀和引爆网络的运动视频。!>
    <meta name=renderer content=webkit>
    <meta name=force-rendering content=webkit>
    <meta name=viewport content="width=1140">
    <meta http-equiv=X-UA-Compatible content="IE=edge,chrome=1">
    <link rel="shortcut icon" href=static/favicon.ico type=image/x-icon>
    <link href=/static/css/app.eef5b81a3d1bee5054a791f452a34147.css rel=stylesheet>
  </head>
  <body>
    <div id=app></div>
    <script type=text/javascript src=/static/js/manifest.6d0adb8f2d8884be1c03.js></script>
    <script type=text/javascript src=/static/js/vendor.ec1cc90c9847c434ba7d.js></script>
    <script type=text/javascript src=/static/js/app.d7fd10ae7e4a68598037.js></script>
  </body>
</html>
```

我们的组件都是这个 html 文件返回后再渲染到 `<div id=app></div>` 里的。这就合理的解释了 SEO 缺陷的原因。

既然说到 SSR 可以解决 SEO 的问题，不难想到原理就是将我们的 html 在服务端渲染，合成完整的 html 文件再输出到浏览器。

另外 SSR 还适用以下场景

* 客户端的网络比较慢
* 客户端运行在老的或者直接没有 JavaScript 引擎上

vue 官网给出了 SSR 原理图片

<!--
![](/uploads/Vue 基于 NUXT 的 SSR2.png)
-->

![](https://ws1.sinaimg.cn/large/005Yd2Thly1fl9ko1nf2yj30zz0il0xb.jpg)

对于这幅图的原理官网有详细解释，此类文章也很多，这里不赘述。

## NUXT

我们进入正题说下 NUXT

> Nuxt.js is a minimalistic framework for server-rendered Vue applications (inspired by Next.js)

作用就是在 node.js 上进一步封装，然后省去我们搭建服务端环境的步骤，只需要遵循这个库的一些规则就能轻松实现 SSR

### 安装流程

Nuxt.js 团队提供了 vue-cli 的初始化模板。前提安装 vue-cli，安装过的忽略此步

```bash
npm install -g vue-cli
```

完成后在需要创建的目录下执行以下

```bash
vue init nuxt/starter <project-name>
cd <project-name>
npm install
```

依赖安装完成后

```bash
npm run dev
```

打开浏览器 http://localhost:3000

> 说明：Nuxt.js 会监听 `pages` 目录下的改变，添加新 page 的时候不需要重启服务

### 目录结构

完成上面命令后你的目录结构会如下

<!--
![](/uploads/Vue 基于 NUXT 的 SSR3.png)
-->

![](https://ws1.sinaimg.cn/large/005Yd2Thly1fl9ko1mio3j30cq0ws0tp.jpg)

Nuxt.js 给出了最简单的目录结构

```
|-- pages
    |-- index.vue
|-- package.json
```

也就是说，至少需要一个 page 来作为展示页。

文件的路径建议都采用绝对路径，表格如下

<!--
![](/uploads/Vue 基于 NUXT 的 SSR4.png)
-->

![](https://ws1.sinaimg.cn/large/005Yd2Thly1fl9ko1n4zjj315q0kijt9.jpg)

例：怎么在 `/pages/user/me.vue` 引入一个 `static` 文件夹里的图片

```
<img src="~static/img/logo.png" alt="Logo"/>
```

### 路由

Nuxt.js 根据 pages 目录结构去生成 vue-router 配置，也就是说 pages 目录的结构直接影响路由结构

例1:

```
|-- pages
    |-- posts
        |-- index.vue
        |-- welcome.vue
    |-- about.vue
    |-- index.vue
```

会生成

```js
routes: [
  {
    path: '/posts',
    component: '~pages/posts/index.vue'
  }, {
    path: '/posts/welcome',
    component: '~pages/posts/welcome.vue'
  }, {
    path: '/about',
    component: '~pages/about.vue'
  }, {
    path: '/',
    component: '~pages/index.vue'
  }
]
```

例2:隐藏路由

在文件名前加 `_`

```
|-- pages
    |-- _about.vue
    |-- index.vue
```

会生成

```
routes: [
  {
    path: '/',
    component: '~pages/index.vue'
  }
]
```

### 配置文件

目录下的 `nuxt.config.js` 是我们唯一的配置入口，这里不建议修改 `.nuxt` 目录，除非特殊需求

默认的给力我们三个配置 ·head·css·loading· 分别是头部设置，全局css，loading进度条

nuxt.config.js 的全部的配置如下,点击查看具体例子

1. [cache](https://nuxtjs.org/examples/cached-components)
2. [loading](https://nuxtjs.org/examples/custom-loading)
3. [router](https://nuxtjs.org/examples/custom-routes)
4. [css](https://nuxtjs.org/examples/global-css)
5. [plugins](https://nuxtjs.org/examples/plugins)
6. [head](https://nuxtjs.org/examples/seo-html-head)

另外还提供了 vuex 等配置，感兴趣可以去 github 和官网。

## NUXT 能为我们做什么

对于使用就说上面这么多（官网上都有，这里给大家一个概览），说下为什么选择 NUXT 来做 SSR

问题1：就是我们无需为了路由划分而烦恼，你只需要按照对应的文件夹层级创建 .vue 文件就行
问题2：无需考虑数据传输问题，nuxt 会在模板输出之前异步请求数据（需要引入 axios 库），而且对 vuex 有进一步的封装
问题3：内置了 webpack，省去了配置 webpack 的步骤，nuxt 会根据配置打包对应的文件

还有很多便捷之处，可以尝试去写一写，读读源码

## 总结

本篇主要介绍 nuxt 的便捷之处，在使用上目前不推荐使用，几个原因：

* 文档不完善还有许多是空的，不是说我们什么信息都得不到，可以看文档的 examples，里面列举的比较全面。
* 目前是 0.8.0 版本，而且 README 里介绍 1.0 即将到来，可能会添加新功能，文档也会完善，待到版本稳定后再部署也不迟。
