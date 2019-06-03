---
title: React 服务端渲染框架 Next.js 基于 Gank api 实战
date: 2017-10-19 17:04:13
tags: ['Next', 'React', 'SSR']
---

![](/uploads/nextjs-gank1.png)

<!--more-->

最开始先摆出地址，有在线 demo：https://github.com/OrangeXC/gank

鉴于最近 vue 相关的文章写的比较多，抽出时间写点 react 的项目，当时用 react 还是 v15 现在都 v16 了，感慨跟不上所有框架的节奏（玩笑话），框架的本质都是大同小异的，每次高 star 框架更新看一下 change 是个好习惯。

之前用 Nuxt 写了个简单的 v2ex，今天的主角依然是 SSR 服务端渲染

Nuxt 文档里有写到灵感源于 Next.js，那么就是说 Next.js 算是 SSR 框架中的元老级别的了。

## 为什么选择 SSR 框架

前面的文章总是在官方文档上小费功夫说明下，这里对于不熟悉 Next.js 的读者建议直接转到 Github，别犹豫，当然熟悉 Nuxt 也可以无障碍阅读本文

不论是 Next.js 或 Nuxt，服务端渲染框架主要两个重要功能

* 首屏 node.js 服务端渲染
* 生成纯静态的 web 站

至于它们是基于哪个前端库封装的，还要看库本身是否支持 SSR，然后就是对外提供 render 函数。

用此类库的原因也不必多说，节省开发成本，不再纠结于环境搭建以及渲染细节。

## 直接开工

本次要实现的是基于 [gank api](http://gank.io/api) 的项目，还是看人家支持什么 api，点前面链接查看详细 api

大体总结为 => 列表，搜索，提叫到审核

列表分为了许多类型，主要的 menu 也是针对不同类型的列表展开

## 路由

通过已知的 api 可以轻松的定义路由

* / （主页，最近的全部类型干货列表）
* /fe （前端干货列表）
* /android (安卓干货列表)
* /ios (iOS干货列表)
* /app (App干货列表)
* /expand (拓展资源干货列表)
* /videos (休息视频干货列表)
* /welfare (福利列表，前方高能，全是干货。。。)
* /timelien (时间轴，记录历史所有更新过干货的日期)
* /day (某天详情，分为以上几种类型的 tab 列表)
* /uplaod (发送干货到审核)
* /search (搜索页)

同 Nuxt 路由配置文件不需要手动创建，/pages 下默认会渲染为页面，文件名自然就是路由名

路由文件都创建完了，下一步思考如抽离出公共模板 Layout 代码，Next.js 提供了 [layout-component example](https://github.com/zeit/next.js/tree/master/examples/layout-component)

我们可以在里面定义 Head,Header,Footer，当然要留出一个内容区域的插槽 `{ children }`

引用于 example 的 layout.js 代码

```js
import Link from 'next/link'
import Head from 'next/head'

export default ({ children, title = 'This is the default title' }) => (
  <div>
    <Head>
      <title>{ title }</title>
      <meta charSet='utf-8' />
      <meta name='viewport' content='initial-scale=1.0, width=device-width' />
    </Head>
    <header>
      <nav>
        <Link href='/'><a>Home</a></Link> |
        <Link href='/about'><a>About</a></Link> |
        <Link href='/contact'><a>Contact</a></Link>
      </nav>
    </header>

    { children }

    <footer>
      {'I`m here to stay'}
    </footer>
  </div>
)
```

因为本次使用的是 antd 做 ui，固实现动态的导航展示上要注意些小问题，我们需要根据 path 动态的给 menu 激活状态。

两个解决方案：

1.在 pages 里面的每一个路由页面里获取 pathname，初始化方法 `getInitialProps` 里可以拿到 pathname，全部列表如下

* pathname - path section of URL
* query - query string section of URL parsed as an object
* asPath - String of the actual path (including the query) shows in the browser
* req - HTTP request object (server only)
* res - HTTP response object (server only)
* jsonPageRes - Fetch Response object (client only)
* err - Error object if any error is encountered during the rendering

调用方法也简单

```js
static async getInitialProps({ pathname }) {
  return { pathname }
}
```

这样一来可以通过传参到 layout 组件的方式 `<Layout pathname={this.props.pathname}></Layout>`

在 Layout 里面改变 Meun 的 active

2.写一个 ActiveLink 组件，再封装一层原有的 Menu

在选择方案前还是要看官方有没有 example，于是找到了 [using-with-router](https://github.com/zeit/next.js/tree/master/examples/using-with-router)

引用于 example 的 ActiveLink.js 代码

```js
import { withRouter } from 'next/router'

// typically you want to use `next/link` for this usecase
// but this example shows how you can also access the router
// using the withRouter utility.

const ActiveLink = ({ children, router, href }) => {
  const style = {
    marginRight: 10,
    color: router.pathname === href ? 'red' : 'black'
  }

  const handleClick = (e) => {
    e.preventDefault()
    router.push(href)
  }

  return (
    <a href={href} onClick={handleClick} style={style}>
      {children}
    </a>
  )
}

export default withRouter(ActiveLink)
```

简单易懂，在 withRouter 方法里可以取到 router 实例，这样可以取到 pathname，query 等等。

这里只需要稍稍修改下 style，变成 antd 的 className，如下

```js
const ActiveLink = ({ children, router, href }) => {
  const active = router.pathname === href
  const className = active ? 'ant-menu-item-selected ant-menu-item' : 'ant-menu-item'
  return (
    <li href='#' onClick={onClickHandler(href)} className={className} role="menuitem" aria-selected="false">
      {children}
    </li>
  )
}
```

在 Layout 组件的 Menu 里直接使用 ActiveLink 组件即可，到这为止解决了全部路由相关问题和 Layout 组件问题

## 数据流

解决了路由问题下一步就是每个页面的 content 的数据填充

我们依旧是在 `getInitialProps` 里面获取数据，相当于 prefatch 方法，服务端渲染会提前执行这个方法获取数据渲染到模板

这里涉及到一个 node 和 Browserify 同构的 fetch 库 [isomorphic-fetch](https://github.com/matthew-andrews/isomorphic-fetch)，cli 工具应该会自带这个库，没有的话提前安装下。

到这里就不用担心 fetch api 在服务端的问题了，这里获取的列表数据走的接口基本一致 `https://gank.io/api/data/{type}/{perPage}/{page}`

三个变量 type-类型、perPage-每页数量、page-页数

接下来可以把 List 和 ListItem 抽象出来，成为共用的组件，每个页面都可以调用，这里不详细展开说明，简单的使用 antd 的 Card 组件，没有特殊功能。

每个页面的请求数据部分也基本一致，将数据存到 props 里，传入 List 组件中去

形成了简单的单向数据流动

列表页面

page组件(fetch data) -> List组件(继承自 Layout) -> ListItem组件

时间轴页面

page组件(fetch data) -> Timeline组件(继承自 Layout)

提交干货页面

page组件 -> Form组件(继承自 Layout) -> post请求(发送formData)

搜索页面

page组件 -> Input组件+空ListItem组件(继承自 Layout) -> get请求(获取关键词对应query的列表数据) -> ListItem组件

## Mobx

既然前面说清楚了数据流都十分简单，那么为什么要引入全局状态管理徒增烦恼呢？

有一点无奈的地方是 `getInitialProps` 本身 return 的就是 props，在 react 里面 props 是单向的，只能向下传递，且不能修改

这里我们要分页功能，但是首屏数据是 props 的，我们换页之后没办法更新 props 的值，也就是没办法再次执行 `getInitialProps`

最简单粗暴的方式就是放弃 spa 的动态切换数据，我们每次 `Router.push({some page}/{per page}/{current page})`，一朝回到解放前的 MVC 版路由切换。

能不能解决问题，答案是能解决问题，那么既然是分页组件，人家 antd 也提供了 Pagination 组件，问题一个接着一个，人家返回的列表并没有告诉你 totalCount，没有 totalCount 就没办法知道有多少页。。。

好尴尬的问题，这个分页没法做，怒脸~~~

也不是没办法做，这个问题变向思考下可以做 loadMore，没错加载更多，当加载到最后一页（即的列表长度小于 perPage）或是此页恰巧等于 perPage 但下一页为空数组时，我们给一个提示，没有更多内容了。

涉及到向 props 的 list 里 `concat` 数组，我们不得不引入全局状态来解决这个问题，不论是 redux 还是 mobx 都可以解决问题，需要注意的是，next.js 中的用法和普通 spa 的 react 应用有所差别。

还是去找 example，[with mobx](https://github.com/zeit/next.js/tree/master/examples/with-mobx)

引用于 example 的 store.js 代码

```js
export function initStore (isServer, lastUpdate = Date.now()) {
  if (isServer) {
    return new Store(isServer, lastUpdate)
  } else {
    if (store === null) {
      store = new Store(isServer, lastUpdate)
    }
    return store
  }
}
```

这段代码太简单，没必要解释了，总之我们在初始化页面时调用 initStore 就好了，isServer 通过 `getInitialProps` 的 req 参数 `!!req` 判断

然后在 loadMore 时出发一个 action

```js
@action loadMoreList = (more) => {
  this.list = this.list.concat(more)
}
```

到这加载更多的功能也就实现了，不足的一点是 List 组件里的 `handleScroll` 方法写的有点简陋，虽说能用，但存在问题，如多次触发、未写兼容代码（后续会改进），放出代码供大家一笑

```js
handleScroll () {
  if (document.documentElement.offsetHeight + document.documentElement.scrollTop > document.documentElement.scrollHeight - 50) {
    this.handleLoadMore()
  }
}
```

其它代码感兴趣可以直接取仓库看，没有阅读难度。

## 表单提交

说到其它页的 fetch list 没什么可将全都是 get 请求，fetch 发一个 get 请求十分简单，不用声明请求类型。

fetch 操作 post 也仅仅在于设置 method 为 POST

之所以单独一章说表单提交，因为在提交表单时遇到了一些问题，由于要 fetch 模拟 form 的 post 请求

看了这个 issue：https://github.com/matthew-andrews/isomorphic-fetch/issues/30

开始怀疑人生，试了所有方法 POST，也走的通，但是接口返回的 msg 就是没接收到参数。

想了想还是回归到笨方法一个一个将参数拼接进去，没想到较优雅的方式，给出代码，同时欢迎讨论

```js
handleSubmit = (e) => {
  e.preventDefault()

  this.props.form.validateFieldsAndScroll(async (err, values) => {
    if (!err) {
      this.setState({ submitLoading: true })

      let strList = []

      Object.keys(values).forEach(item => {
        strList.push(`${item}=${values[item]}`)
      })

      const res = await fetch("https://gank.io/api/add2gank", {
        method: "POST",
        headers: {
          'Content-Type': 'application/x-www-form-urlencoded'
        },
        body: strList.join('&')
      })

      const json = await res.json()

      if (json.error) {
        message.error(json.msg)
      } else {
        message.success(json.msg)
      }

      this.setState({ submitLoading: false })
    }
  })
}
```

> 看过网站的读者也发现提交表单页面上方有提示语，让大家文明使用三方 api 提供者 gank 的发表干货接口，把真正的好内容提交上去，想测试接口的请走默认的 debug 模式，这里再次强调下，感谢配合

## 微交互

既然功能差不多了，再微交互上再加把劲，用过 NUXT 的知道 NUXT 内置了 Loading bar，切换路由时在页面顶端会有 loading 条，体验较好。

next.js 并没有内置这个功能，页面看起来会显得十分怪异，点击切换路由没有反应，顿一下再跳转，顿的时候在获取初始化数据。

官方推荐使用 nprogress

关键代码如下，写在了 Layout.js 组件里

```js
Router.onRouteChangeStart = (url) => {
  console.log(`Loading: ${url}`)
  NProgress.start()
}
Router.onRouteChangeComplete = () => NProgress.done()
Router.onRouteChangeError = () => NProgress.done()
```

这样整个网站看起来洋气多了，切换 router 页面顶端有 loading bar，右上角还有 loading icon

## 上线

开发 next.js 的组织叫 zeit，在官网他们的得意作品是 [now](https://zeit.co/now)，一个快速部署的工具，同时为免费用户提供三个免费的服务，支持 docker，node 等

看 5 分钟文档就能上手部署 node 项目，比 Heroku 简单的多

这里使用的就是 [now](https://zeit.co/now)，首先安装 [now-cli](https://github.com/zeit/now-cli)

在项目根路径下一句命令部署

```bash
now
```

线上的路径就不贴出来了，时刻关注 Github 上方的 website 地址，因为每次部署不绑定域名的情况下是 `项目名+随机哈希` 的域名，绑定域名需要 money。

至于上线就讲这么多，有疑问欢迎交流。

## 未来

下一步要解决几个问题

* 加载更多时的 bug
* 支持移动端
* 福利页面直接展示图片（点击可以全屏大轮播）
* 美化时间轴样式

说到福利页面本想着不加来着，因为个别写 demo 的人专门把福利列表拎出来做成妹子 App，既然是干货集中营，就应该多些技术元素，福利都是次要的。

## 总结

到这为止一个 next.js 版本的 gank（干货集中营）完成了，感慨现在开发工具越来越好用，还是之前的想法把好用的工具分享给大家，给一个完整的例子供学习者参考，不再每次都看各个版本的 Hacker News，而是给国内的学习者一个中文版的例子，同时文中也会将实现的时候遇到的问题。

本人 orange 也是再不断的学习当中，本文也是第一次接触学习 next.js 写的项目，文章或项目有不足之处欢迎指正，感谢阅读！
