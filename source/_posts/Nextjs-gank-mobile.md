---
title: React 服务端渲染实现 Gank 移动端
date: 2017-11-14 17:35:45
tags: ['Next', 'React', 'SSR']
---

<!--
![](/uploads/React 服务端渲染实现 Gank 移动端.jpg)
-->

![](https://ws1.sinaimg.cn/large/005Yd2Thly1flipe2lhwkj31cw0oragw.jpg)

Github: https://github.com/OrangeXC/gank

请使用手机或开发者工具手机模拟器打开

<!--more-->

接上一篇内容：[React 服务端渲染框架 Next.js 基于 Gank api 实战](https://orangexc.xyz/2017/10/19/Nextjs-gank/)

在上一篇结尾说到要实现移动端，不单单是响应式布局，而是采用移动端组件库进行开发。

本文重点介绍如何在一个项目里面实现两类端的服务端渲染。

## 前提

1. 明确的 router 分割规格
2. 判断设备跳转对应端的 router
3. 两套 UI 组件库

根据三个前提条件逐一给出解决方案。下面首先说下路由分割。

### 路由分割

路由分割规则大致上分为两种：

* 子域名形式`（m.xxx.xxx）`
* 相同域名形式`（xxx.xxx/m）`

这里强调是一个项目没必要部署到两个域名下，故排除子域名的形式。

作为区分移动端在所有的域名前加了 `/m`，进而实现 page 级别的组件区分

映射到 next.js 里面就是在 `pages` 目录下新增一个名为 `m` 的文件夹，里面的每个文件都对应着移动端的路由

例如：`xxx.com/fe` 移动端对应着 `xxx.com/m/fe`

### 判断设备跳转路由

这里直接上代码比口述来的痛快

```js
if (/Mobile/i.test(ua) && pathname.indexOf('/m') === -1) {
  app.render(req, res, `/m${pathname}`, query)
} else if (!/Mobile/i.test(ua) && pathname.indexOf('/m') > -1) {
  app.render(req, res, pathname.slice(2), query)
} else {
  handle(req, res, parsedUrl)
}
```

逻辑十分简单，疑问点是此段代码应该放在什么地方，next.js 既然是服务端渲染，判断理应在服务端进行。

next.js 允许我们自定义入口 server.js 文件，启动时直接运行 `node server.js` 命令。

在这个 server 里面进行中间件的挂载，以及服务端层面的路由控制，具体的实现官网和本项目都可查看。

### 两套 UI 组件库

对于个人或者小项目没那么大精力开发组件库，也没有精力设计样式。

前面的 pc 端用的是 antd，这里为了保持风格一致使用了 antd-mobile

当然引入 antd-mobile 时 iocn 是个问题，想使用自定义的 icon 需要自己配置 webpack

新建 `next.config.js`，重要代码如下

```js
config.module.rules.push(
  {
    test: /\.(svg)$/i,
    loader: 'emit-file-loader',
    options: {
      name: 'dist/[path][name].[ext]'
    },
    include: [
      moduleDir('antd-mobile'),
      __dirname
    ]
  },
  {
    test: /\.(svg)$/i,
    loader: 'svg-sprite-loader',
    include: [
      moduleDir('antd-mobile'),
      __dirname
    ]
  }
)
```

这里重点说下 `svg-sprite-loader` 这个库的坑，版本最好控制在 `0.3.x`，如果升级到最新版会有意外的 bug 惊喜等着你

## 实现

前提环境搞定了剩下的就是动手开干了。

这里不逐一展开解释，可以看前面 pc 的文章，解释的够详细，这里单说下实现时可能遇到的问题

### 问题 1 - 自定义图标

上面介绍了自定义图标的配置，在组件里面具体怎么实现呢，首先要写一个渲染函数

```js
const CustomIcon = ({ type, className = '', size = 'md', ...restProps }) => (
  <svg
    className={`am-icon am-icon-${type.substr(1)} am-icon-${size} ${className}`}
    {...restProps}
  >
    <use xlinkHref={type} /> {/* svg-sprite-loader@0.3.x */}
    {/* <use xlinkHref={#${type.default.id}} /> */} {/* svg-sprite-loader@lastest */}
  </svg>
)
```

代码里面注释掉的有 `svg-sprite-loader@lastest` 版本的写法，亲测无效，也不建议尝试。

在 render 里面就可以这样调用

```html
<CustomIcon type={require('../../static/icon/github.svg')} />
```

到这里可以展示任意自定义 icon 了。

### 问题 2 - 长列表

众所周知移动端的长列表性能堪忧，如果采用前文每次 load more 时，直接把请求回来的数据 `concat` 或 `push` 到列表尾部，后果就是页面逐渐变卡，知道你滑不动列表，甚至网页卡死。

庆幸 antd-mobile 为我们提供了 `ListView` 组件，让我们轻松实现长列表渲染

那么问题来了，antd-mobile 官网为我们提供的例子都是完全基于客户端的实现，在预渲染阶段，我们需要渲染首屏数据，而不是在页面加载完成后在 `componentDidMount` 钩子里初始化首屏数据。

为了使页面更快速的渲染首屏列表内容，首次请求需要在服务端获取数据后立即初始化 `ListView` 组件。

本项目的做法是，在 page 组件中

```js
static async getInitialProps ({ req }) {
  const language = req ? req.headers['accept-language'] : navigator.language

  const res = await fetch('https://gank.io/api/data/all/20/1')
  const json = await res.json()

  return { list: json.results, language }
}
```

然后进一步封装 `ListView` 组件成一个公用组件，每个页面都可调用

关键代码是在构造器里面初始化 `ListView` 数据源实例

```js
constructor (props) {
  super(props)

  const dataSource = new ListView.DataSource({
    rowHasChanged: (row1, row2) => row1 !== row2,
  }).cloneWithRows(props.initList)

  this.state = {
    rData: [],
    dataSource,
    isLoading: false
  }
}
```

在加载更多的时候进行数据的拼接。

注意的是判断下当前页数把 props 里面传进来的初始化数据拼接进去

```js
this.setState({ isLoading: true })

this.setState((prevState) => ({
  rData: pIndex === 2
    ? this.props.initList.concat(prevState.rData).concat(json.results)
    : prevState.rData.concat(json.results)
}))
```

在请求完成后不要忘记刷新 `dataSource`，使得 `ListView` 可以相应数据变化

```js
this.setState({
  dataSource: this.state.dataSource.cloneWithRows(this.state.rData),
  isLoading: false
})
```

到这为止，整个列表请求就实现了

至于展示上的配置项还是蛮多的，官网写的十分详细，配置的优劣也会影响性能。

### 问题 3 - MenuBar 高度问题

由于我们需要全屏高度的展示效果，NavBar 与 Menubar 分别吸附在上下，不随内容滚动。

尴尬的点是 NavBar 被包在 Menubar 中，而 Menubar 使用了 transform，如果内容区长度超过屏幕高度，会导致 NavBar 的 `position: fixed` 失效，NavBar 会随着内容区域一同滚动上去。

尝试了几个解决办法，就算解决了这个问题，还存在 iphone safari 上的滑动导致的视窗高度拉长，进而影响定位不准确的问题。

这里直接摒弃 body 层面的滚动，所有的滚动区域通过 `屏幕高度 - NavBar - Menubar底部 - 其它垂直占位空间` 计算得出。

既保证了滚动区域的高度恰好填充剩余垂直空间，又保证了 Safari 不触发视窗的高度拉长

因为高度需要计算获得，本项目里面初始化给的是 `height: 100vh`（iphone safari 会把下面的菜单栏算到 `100vh` 里面，导致 MenuBar 定位不准确）

页面加载后计算一次屏高 `document.documentElement.clientHeight` 改变屏幕整体展示高度，滚动区域高度也可计算获得。

## 总结

由于本文是基于前一篇写的，踩坑的点数明显减少，行文的目的也是希望看到本文的人遇到相同问题时可以少踩坑，多一个解决问题的思路。
