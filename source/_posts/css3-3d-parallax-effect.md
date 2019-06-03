---
title: 3D 视差效果
date: 2016-12-20 12:15:00
tags: ['CSS3', 'JavaScript']
---

![](/uploads/css3-3d-parallax-effect1.gif)

前一周敢玩新版PC端上线，其中原创视频封面用的就是上图的效果，下面详细说一下怎么实现

<!--more-->

## 起因

这个效果有着相对较好的用户体验，在 hover 的基础上又有了与用户交互的体验，仿佛用户一直在不同角度按压这张图片。

当然这个效果早就有人在写并用于官网了，感兴趣可以去[锤子官网](https://www.smartisan.com/)，看下轮播图的鼠标交互效果。

## 基本思路

单借助 CSS3 的 hover 不足以支配这个效果，JS 方案考虑以下步骤

1. 绑定鼠标事件（mouseover），绑定离开事件（mouseleave）
2. 判断鼠标相对于图片的位置
3. 计算出应该翻转（rotate）的角度，同时改变阴影的方向
4. 将图片复位

这里涉及 CSS3 的一个比较少用的属性 `perspective`

> MDN: perspective 属性指定了观察者与 z = 0 平面的距离，使具有三维位置变换的元素产生透视效果。z > 0 的三维元素比正常大，而 z < 0 时则比正常小，大小程度由该属性的值决定。

深入了解去看这个文章[CSS3 Transform 的 perspective 属性](http://www.alloyteam.com/2012/10/the-css3-transform-perspective-property/)，时间比较久但是很经典，除了兼容性描述有变其它描述很准确。

## 开始构建

html：

```html
<div class="avatar"></div>
```

css:

```css
.avatar {
  width: 300px;
  height: 300px;
  margin: 50px auto;

  background: url('https://ww1.sinaimg.cn/large/005Yd2Thly1fl8hsldx4tj30hs0hsgnq.jpg');
  background-size: contain;

  transition: all .3s linear;
  transform-origin: 50%;
}
```

js:

```js
let el = document.querySelector('.avatar')

el.addEventListener('mousemove', (e) => {
  let thisPX = el.getBoundingClientRect().left
  let thisPY = el.getBoundingClientRect().top
  let boxWidth = el.getBoundingClientRect().width
  let boxHeight = el.getBoundingClientRect().height

  let mouseX = e.pageX - thisPX
  let mouseY = e.pageY - thisPY
  let X
  let Y

  X = mouseX - boxWidth / 2
  Y = boxHeight / 2 - mouseY

  el.style.transform = `perspective(300px) rotateY(${X / 10}deg) rotateX(${Y / 10}deg)`
  el.style.boxShadow = `${-X / 20}px ${Y / 20}px 50px rgba(0, 0, 0, 0.3)`
})


el.addEventListener('mouseleave', () => {
  el.style.transform = `perspective(300px) rotateY(0deg) rotateX(0deg)`
  el.style.boxShadow = ''
})
```

以上代码看似没什么问题，也许你在新版本浏览器（无需babel）已经顺利执行了，但是这里有一个坑。

除非你能确定你的图片在一屏内，就是说你的 body 最大高度就是窗口高度，不然 `let mouseY = e.pageY - thisPY` 这句计算出来的不一定是真实的鼠标偏移量，而是加上滚动调偏移后的值。

解决办法就是

```js
let scrollTop = document.documentElement.scrollTop + document.body.scrollTop  //计算滚动高度

let mouseY = e.pageY - scrollTop - thisPY  //减去滚动高度
```

一般的项目考虑到这就可以了，如果你的项目存在 X 轴上的偏移计算原理相同，减去偏移量。

## 实例

我自己的代码放在了 codepen，如下

<p data-height="500" data-theme-id="dark" data-slug-hash="VmgoVX" data-default-tab="js,result" data-user="orangexc" data-embed-version="2" data-pen-title="3D parallax effect" class="codepen">See the Pen <a href="https://codepen.io/orangexc/pen/VmgoVX/">3D parallax effect</a> by orangexc (<a href="https://codepen.io/orangexc">@orangexc</a>) on <a href="https://codepen.io">CodePen</a>.</p>
<script async src="https://production-assets.codepen.io/assets/embed/ei.js"></script>

另外最近发现在 codepen 上的一个项目，在热门榜单上，基本思路是一样的只不过换了种方式去写，很不错的例子，对于需要多个元素循环绑定的情况很好用。

<p data-height="265" data-theme-id="dark" data-slug-hash="aBPRaX" data-default-tab="js,result" data-user="PavelDoGreat" data-embed-version="2" data-pen-title="Interactive Floating Panels" class="codepen">See the Pen <a href="https://codepen.io/PavelDoGreat/pen/aBPRaX/">Interactive Floating Panels</a> by Pavel Dobryakov (<a href="https://codepen.io/PavelDoGreat">@PavelDoGreat</a>) on <a href="https://codepen.io">CodePen</a>.</p>

> 注：此种方法规避了高度差计算的问题，因为是基于 offsetX（作用元素的偏移量），**推荐使用**

## 总结

JS 动画考虑的会相对多一些，可以获取宽高及鼠标位置（方法多样），根据鼠标位置可以计算出动画的对应效果，选择合适的且兼容性好的代码很关键
