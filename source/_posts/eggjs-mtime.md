---
title: Egg 实现一个 mTime 时光网
date: 2017-09-20 17:06:50
tags: ['egg']
---

![](/uploads/eggjs-mtime1.png)

<!-- more -->

先放出项目地址：https://github.com/OrangeXC/mtime

有一段时间没更新博客了，今天的文章主要围绕 egg 进行，长时间沉浸在前端框架中，游离到传统 MVC 的开发模式还真不太适应，好久不写 MVC 项目了。

说下今天的主角 egg，在前几天的腾讯 IMweb Conf 2017 大会第一个演讲就是 egg，egg 是一个 node 框架基于 koa，寓意孵化新生，本项目的 logo 就有点怪咖，是个煎蛋，这个嘛，没有新生了，因为这个项目没有发挥 egg 太多优势。

之所有这么说是因为我调用的三方 api，说到这里为什么不用 vue，react，angular 等直接请求接口呢，因为这里涉及到一点点数据库操作，算是没白折腾 egg。

不写科普文，简单的文档层面可以直接到 [egg官网](https://eggjs.org/)

说说项目的搭建，egg 提供了 cli，项目的目录也遵循约定规范，不可随意篡改。

```
egg-project
├── package.json
├── app.js (可选)
├── agent.js (可选)
├── app
|   ├── router.js
│   ├── controller
│   |   └── home.js
│   ├── service (可选)
│   |   └── user.js
│   ├── middleware (可选)
│   |   └── response_time.js
│   ├── schedule (可选)
│   |   └── my_task.js
│   ├── public (可选)
│   |   └── reset.css
│   ├── view (可选)
│   |   └── home.tpl
│   └── extend (可选)
│       ├── helper.js (可选)
│       ├── request.js (可选)
│       ├── response.js (可选)
│       ├── context.js (可选)
│       ├── application.js (可选)
│       └── agent.js (可选)
├── config
|   ├── plugin.js
|   ├── config.default.js
│   ├── config.prod.js
|   ├── config.test.js (可选)
|   ├── config.local.js (可选)
|   └── config.unittest.js (可选)
└── test
    ├── middleware
    |   └── response_time.test.js
    └── controller
        └── home.test.js
```

## 初始配置

这里主要关注配置文件 `config/config.default.js` 和 `app` 目录

先看下 `config/config.default.js` 文件

```js
module.exports = appInfo => {
  const config = {};

  // should change to your own
  config.keys = appInfo.name + '_1504252356337_1029';

  // view
  config.view = {
    defaultViewEngine: 'nunjucks',
    mapping: {
      '.tpl': 'nunjucks',
    },
  };

  config.sequelize = {
    dialect: 'mysql', // support: mysql, mariadb, postgres, mssql
    database: 'mtime',
    host: '127.0.0.1',
    port: '3306',
    username: 'root',
    password: '',
  };

  config.mysql = {
    client: {
      host: '127.0.0.1',
      port: '3306',
      user: 'root',
      password: '',
      database: 'mtime',
    },
    app: true,
    agent: false,
  };

  return config;
};
```

这里面指定了模板文件，数据库的连接参数。注意这样并不能生效，因为我们没有指定 plugin 对应的 npm 包，前提是要安装这些依赖。

在 `config/plugin.js` 里

```js
exports.nunjucks = {
  enable: true,
  package: 'egg-view-nunjucks',
};

exports.sequelize = {
  enable: true,
  package: 'egg-sequelize',
};

exports.mysql = {
  enable: true,
  package: 'egg-mysql',
};
```

在网页的公共头部有城市选择，如下

![](/uploads/eggjs-mtime2.png)

这里比较坑的是 api 是非官方的 api，只能自己整理城市列表，存到了本地的 init 目录下的 location.json

那么问题来了，每次调用本地文件明显是不合情理的，以后还会涉及到城市信息变动，这里作为第一次导入的 init 数据写入 mysql，之后统一从 myspl 获取 location。

初始化代码按约定放在 `app.js` 中，允许我们进行初始化操作。

```js
const fs = require('fs');

module.exports = app => {
  app.beforeStart(async () => {
    // 应用会等待这个函数执行完成才启动
    await app.model.sync({ force: true });

    app.database = await app.mysql.createInstance(app.config.mysql.client);

    const locations = JSON.parse(fs.readFileSync('./init/location.json'));

    await app.mysql.insert('locations', locations.data);
  });
};
```

虽然 egg-mysql 和 egg-sequelize 文档都有介绍，这里简单说下

* `await app.model.sync({ force: true });` 是同步 model 到数据库，主要是同步数据库表和字段

* `app.database = await app.mysql.createInstance(app.config.mysql.client);` 是在应用运行时动态的从配置中心获取实际的参数，再来初始化一个实例。

> 注：这里官网的代码有点小坑，亲测下面官网代码的 `configCenter` 并没有 `fetch` 方法，遇到相同坑的该用上面的代码即可

```js
const mysqlConfig = yield app.configCenter.fetch('mysql');
app.database = app.mysql.createInstance(mysqlConfig);
```

* `const locations = JSON.parse(fs.readFileSync('./init/location.json'));` 这句不解释了，看不懂先学学 node 基础

* `await app.mysql.insert('locations', locations.data);` 这句是将 Array 直接存到数据库 `locations` 表，这里见官网 [如何编写 CRUD 语句](https://eggjs.org/zh-cn/tutorials/mysql.html#如何编写-crud-语句)部分，官网的例子是插入单条数据（以 Object 的格式），当然这里 Array 创建多条也是可以的

> 注：第一句我们创建了 `locations` 表，表里多了两个默认字段分别是 `created_at` 和 `updated_at`，批量导入数据里没有这两个字段，故报错，解决办法是在 model 的 location.js 里面给这两个字段 default 值，如下

```js
created_at: {
  type: DATE,
  default: new Date(),
},
updated_at: {
  type: DATE,
  default: new Date(),
}
```

上面的代码分别依赖两个库 `egg-sequelize` 和 `egg-mysql`，两者均与操作 mysql 有关，当然根据业务需要选择其一也可，至于两个库分别有哪些功能可以直接转到 Github 看 document

到这里我们去 mysql 看一眼

![](/uploads/eggjs-mtime3.png)

一切准备工作就绪

## 路由搭建

页面整体分成三部分

```
header
router
footer
```

router 根据路由动态渲染，也是主要业务逻辑的区块，主页电影列表分为三类 `正在售票` `正在热映` `即将上映` 分别对应三个路由 `/` `/hot` `/new`，当然考虑到城市因素（不同城市上映电影有细微差别），在本项目里写到了 `query` 里面，所有路由后面都带着一个 `query` 看着着实不爽，下一步我会把它移到 `cookie` 或者 `localStorage`，代码约定写在 `app/router.js` 里如下

```js
app.get('/', 'home.index');
app.get('/hot', 'hot.index.index');
app.get('/new', 'new.index.index');
```

解释下路由对应 controller 的语法，规则是 `文件名/函数名` 或 `文件夹/文件名/函数名`，当然这是我个人猜测，发现奏效，官网的写法法可以到 [如何定义router](https://eggjs.org/zh-cn/basics/router.html#如何定义-router)

下面还有`详情页` `短评页` `热评页` `剧照和海报页` `预告花絮页`，分别对应下面的路由

```js
app.get('/movie/:id', 'movie.index.index');
app.get('/comment/:movieId', 'comment.normal.index');
app.get('/hot_comment/:movieId', 'comment.hot.index');
app.get('/stills/:movieId', 'stills.index.index');
app.get('/video/:movieId', 'video.index.index');
```

路由到这里介绍完了，没什么好讲的，不看网站这个项目的概况也是一目了然，接下来的事情就是 controller 来调取 model 数据渲染到页面了，下面不一一陈述 controller，抽离一点可讲的。

## controller 搭建

按照约定我们直接找到 controller 目录，看到全部的 controller

`const locations = await ctx.model.Location.findAll();` 这一句找到所有的 location 数组。

细心看的人会发现每个 controller 都有下面的代码，我们要获取全部 location 列表，并找到 query 里面的那条数据，默认是北京，这里页面公共部分 header 一直存在一个 location 下拉列表，所以每次都要将数据抛给页面，更好的解法是点开下拉列表异步拉取所有 location 再渲染进去。

```js
ctx.query.location
  ? location = locations.find(({ id }) => id === Number(ctx.query.location))
  : location = {
    id: 290,
    name: '北京',
  };
```

这里完全的 get 数据，一个前端层面的 ajax 都没有，原谅我的偷懒，这样导致了所有 controller 的代码冗余。

剩下的就是去调用 mtime 的 api 了，感觉很好的是 egg 为我们封装了全局 http 方法，[HttpClient](https://eggjs.org/zh-cn/core/httpclient.html)

使用简单，参数简单，堪比 axios 的便捷。大家自己体会吧。

## view 搭建

说到 view 层就到了大前端的天下，玩的 6 的话，这一层可以无限延展，从最简单的模板（pug，ejs，swig，nunjucks 等），到 (vue，react，angular 等），再到（Andriod, IOS），再再到（RN，weex），甚至是小程序接口。

服务层让我喜欢的就是可渲染模板，可吐数据，本来是想搞个前后端分离，后来被自己气到，调用人家的 api，竟然不直接写 view 层，搞个 egg 进来没起到——卵用。

不自觉讲起了段子，索性就回归 10 年前的前端，抛开 MVVM，甚至撸起了 jquery。

牢骚一堆，这里用的是 nunjucks，官网的例子用的就是这个模板。

到了这里我们的页面完成了。。。虽然什么也没讲，我默认大家都看得懂模板的哈，至于 Bulma 的初衷是不想用 Jquery（bootStrap 大家懂得），奈何找不到喜欢的轮播，找了许久的轮播竟然还依赖 jquery。。。

## 不足

这里声明人家 mTime 的接口并不是官方公开的，来自 https://github.com/jokermonn/-Api/blob/master/Time.md，mTime 的服务器禁止跨域请求 MP4 资源，尝试以下几个方法解决这个问题

* iframe，没能成功，访问失败
* `<a target="_blank">`，也没解决问题，不过奇怪的是复制 mp4 的 url 到新 tab 回车可以访问，a 标签跳转新 tab 则失败，js 的 `window.open()` 没有尝试
* node 请求 mp4 的 buffer 转成 stream 后再抛给前端，能力有限没能解决问题。

一方面作者能力原因，一方面违背 mTime 节省视频服务器流量的想法，在视频页给了链接可以跳到真正的 mTime 官网。

算是遗憾吧，如果有 node 端转发视频请求经验的大神欢迎赐教。

## 总结

项目是在短期内速成的，好多细节没考虑到位，望大家多吐槽，写这篇文章的目的是给想了解 egg 的开发者一个小 demo，真正的生产模式比这个复杂的多，本文也自然就不值一提。
