---
title: 返回顶部的几种实现
date: 2016-12-07 19:59:29
tags: JavaScript
---

<!--
![](/uploads/返回顶部的几种实现.jpg)
-->

![](https://ws1.sinaimg.cn/large/005Yd2Thly1fl9jxqo6j7j312m0f00vi.jpg)

返回顶部的按钮大家并不陌生，针对长滚动条的信息流页面添加返回顶部的按钮可以给用户良好的体验，而返回顶部的实现也是多种多样，本文分享几个案例并给出评价

作为引子讲一个常用的案例，对与不知道的作为一个小彩蛋吧！对于微博与微信朋友圈的返回顶部可以双击最顶部的区域（包含时间，网络，电量）来返回顶部

大家见过哪些创意的返回顶部呢？欢迎留言交流，下面进入正文

<!--more-->

## 直接跳转的方式

对于追求极简的方式达到预期效果的人可能会喜欢这种方式，不托泥带水也不用考虑兼容问题

国内的大站如[新浪主页](http://www.sina.com.cn/),[淘宝网](https://www.taobao.com/),[京东](https://www.jd.com/),[百度新闻](http://news.baidu.com/)

不难发现此类的大站为了兼容性与浏览器渲染速度选择了**直接跳转的方式**，实现基本含两种方式

### 命名锚点跳转的方式

> 用锚点 #top（可变）返回到顶部预设的 id 为 top（可变）的元素，或者在页面顶部的 a 标签 name 属性值为 top（可变），新版本的浏览器（chrome等）默认支持 #top 锚点返回顶部且不需要前面所提条件，但是前面条件的优先级大于默认跳转。

上面说的有些绕口，下面逐句解释，读明白且理解的可以直接看下节

```html
<a href="#top">返回顶部</a>  <!-- fiexd 定位在页面右下角 -->

<p id="top">顶部</p>  <!-- id 为 top -->

<a name="top"></a>  <!-- a 标签且 name 为 top -->
```

上面代码包含了所有的描述，简单易懂，下面说下优先级，这个是关键，避免无意中犯了错误却无从下手

> 注意: id > name > 默认

不建议大家用默认的方式，具体有兼容问题和优先级低的问题，id 与 name 均可，但要注意命名冲突

### scroll 函数跳转

> window.scroll(x, y) 方法，x 是水平滚动位置，y 是垂直滚动位置，必须两个参数都给进去不然会报错，大家可能发现有 window.scrollY，这里只改变垂直方向的滚动为什么不能用，一个方面它返回的不是函数无法传参，另一方面无法被赋值

```html
<a href="javascript: scroll(0, 0)">返回顶部</a>
```

这里方便展示，实际不建议在标签内使用 js，可以调用一个方法，简单易懂。

## setTimeout 模拟滚动动画

> setTimeout() 方法递归调用，改变滚动条距顶部位置

```js
function backTop () {
  window.scrollBy(0, -100)

  scrolldelay = setTimeout('backTop()', 10)

  var sTop = document.documentElement.scrollTop + document.body.scrollTop

  if (sTop === 0) {
    clearTimeout(scrolldelay)
  }
}
```

注意优化网页资源，使用不当会非常卡顿，可以通过调节 setTimeout 的第二个参数来控制滚动速度(回调时间单位 ms)，下面解释声明sTop 变量的写法

获取scrollTop值，声明了DTD的标准网页取 document.documentElement.scrollTop，否则取 document.body.scrollTop 因为二者只有一个会生效，另一个就恒为 0，所以取和值可以得到网页的真正的 scrollTop 值

> 注：非标准版（webkit 内核）采用的是 `document.body.scrollTop`

## window.requestAnimationFrame

> window.requestAnimationFrame() 这个方法是用来在页面重绘之前，通知浏览器调用一个指定的函数，以满足开发者操作动画的需求。这个方法接受一个函数为参，该函数会在重绘前调用。

完整的文档看 [MDN](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestAnimationFrame) 和 [JavaScript 标准参考教程（alpha）](http://javascript.ruanyifeng.com/htmlapi/requestanimationframe.html)，里面有详细描述和兼容性详情

看过文档发现 requestAnimationFrame 适合用于连续的动画，而我们的需求与连续动画关系大吗？我们不妨一试。

```js
const isWebkit = navigator.userAgent.toLowerCase().match(/webkit\/([\d.]+)/)
const requestAnimationFrame = window.requestAnimationFrame || window.mozRequestAnimationFrame || window.webkitRequestAnimationFrame || window.msRequestAnimationFrame

function backTop () {
  let move = window.scrollY
  let span = move / 15

  function step () {
    if (document.documentElement.scrollTop + document.body.scrollTop > 0) {
      if (isWebkit) {
        document.body.scrollTop -= 300
      } else {
        document.documentElement.scrollTop -= 300
      }

      requestAnimationFrame(step)
    } else {
      return
    }
  }

  requestAnimationFrame(step)
}
```

上面代码为什么定义 move 与 span 这里是方便计算速度，粘贴代码运行你会发现滚动的距离越大速度就会越快，反而会越慢

这种用法本质上可以使动画更加流畅，但是重点在于页面优化，requestAnimationFrame是在主线程上完成。这意味着，如果主线程非常繁忙，requestAnimationFrame的动画效果会大打折扣。

## 借助 jQuery

目前的前端把焦点从 jQuery 移到了 MVVM 框架，不论如何当初的 jQuery 对于 DOM 操作与低版本浏览器兼容贡献巨大。如果你即考虑兼容性又想要流畅的动画可以继续使用 jQuery。

```js
function backTop (minHeight) {
  var backTopHtml = '<div id="backTopBtn">返回顶部</div>'

  $('#page').append(backTopHtml)

  $('#backTopBtn').click(function () {
    $('html, body').animate({scrollTop: 0}, 700)
  }).hover(
    function () {
      $(this).addClass('hover')
    },
    function () {
      $(this).removeClass('hover')
    }
  )

  minHeight ? minHeight = minHeight : minHeight = 600

  $(window).scroll(function () {
    var s = $(window).scrollTop()

    if (s > minHeight) {
      $('#backTopBtn').fadeIn(100)
    } else {
      $('#backTopBtn').fadeOut(100)
    }
  })
}

backTop()
```

这里初始状态的返回顶部为不可见，通过判断页面滚动高度切换显示隐藏，hover 的样式可以自己设计。

## 总结

总结了几种返回顶部的方式，orange 认为滚动或跳转对用户体验并没有影响（影响用户体验的是有没有返回顶部的按钮），既然选择了返回顶部的按钮那么性能一定是第一位的，一味追求滚动动画导致页面滚动过程中卡顿进而影响了整个网页的用户体验，在这个细节上负面的例子太多不一一列举。

总之本文的用意除了阐述各个方法的用法外还想提醒大家在前端开发中细节才是致命的，orange 也一直在踩坑，说到聪明的大厂为什么选用跳转，哈哈，先回去看看负面教材给我们的教训就知道了，技术点不在于滚动，而在于整个网页的优化，如果你的网站优化的足够好，那么拿滚动来炫技没人会反对。
