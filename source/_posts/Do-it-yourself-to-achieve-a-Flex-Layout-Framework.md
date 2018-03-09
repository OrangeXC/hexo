---
title: 自己动手实现一个 Flex 布局框架
date: 2016-10-14 09:59:37
tags: ['Flex', 'CSS']
---

<!--
![](/uploads/自己动手实现一个 Flex 布局框架1.png)
-->

![](https://ws1.sinaimg.cn/large/005Yd2Thly1fl9k479ue7j30xc0dw44t.jpg)

本文作为 Flex 布局进阶，不对基础做详细介绍，关于 Flex 基础知识请到阮一峰老师的[Flex 布局教程：语法篇](http://www.ruanyifeng.com/blog/2015/07/flex-grammar.html)

看过基础，或者已经使用 Flex 布局的朋友可以根据本文试着动手写一写，先不急着开工，看看其它大型框架怎么实现的。

<!--more-->

## Bootstrap 框架

相信大家都用过 Bootstrap 框架，目前最受欢迎的响应式布局框架，在 Github 上 10w＋ 的 star

而其中的栅格系统深入人心，针对不同尺寸的屏幕提供一套完整布局方案，不了解栅格系统的可以看[中文官方文档栅格系统](http://v3.bootcss.com/css/#grid)

对于新人概念有点多，跳跃性挺强，不过跟着跳转链接一步一步摸索很快就能入门，这里给的都是中文链接。

给出一段栅格系统的代码片段

```html
<div class="row">
  <div class="col-md-1">.col-md-1</div>
  <div class="col-md-1">.col-md-1</div>
  <div class="col-md-1">.col-md-1</div>
  <div class="col-md-1">.col-md-1</div>
  <div class="col-md-1">.col-md-1</div>
  <div class="col-md-1">.col-md-1</div>
  <div class="col-md-1">.col-md-1</div>
  <div class="col-md-1">.col-md-1</div>
  <div class="col-md-1">.col-md-1</div>
  <div class="col-md-1">.col-md-1</div>
  <div class="col-md-1">.col-md-1</div>
  <div class="col-md-1">.col-md-1</div>
</div>
<div class="row">
  <div class="col-md-8">.col-md-8</div>
  <div class="col-md-4">.col-md-4</div>
</div>
<div class="row">
  <div class="col-md-4">.col-md-4</div>
  <div class="col-md-4">.col-md-4</div>
  <div class="col-md-4">.col-md-4</div>
</div>
<div class="row">
  <div class="col-md-6">.col-md-6</div>
  <div class="col-md-6">.col-md-6</div>
</div>
```

效果如下

<!--
![](/uploads/自己动手实现一个 Flex 布局框架2.png)
-->

![](https://ws1.sinaimg.cn/large/005Yd2Thly1fl9k47av1bj32tk0s0q8b.jpg)

这里栅格系统将屏幕水平均分成 12 份。通过加对应的 class 调整布局。语法也通俗易懂不过多解释。

再来看另一个列偏移的例子

```html
<div class="row">
  <div class="col-md-4">.col-md-4</div>
  <div class="col-md-4 col-md-offset-4">.col-md-4 .col-md-offset-4</div>
</div>
<div class="row">
  <div class="col-md-3 col-md-offset-3">.col-md-3 .col-md-offset-3</div>
  <div class="col-md-3 col-md-offset-3">.col-md-3 .col-md-offset-3</div>
</div>
<div class="row">
  <div class="col-md-6 col-md-offset-3">.col-md-6 .col-md-offset-3</div>
</div>
```

效果如下

<!--
![](/uploads/自己动手实现一个 Flex 布局框架3.png)
-->

![](https://ws1.sinaimg.cn/large/005Yd2Thly1fl9k47apa9j32rs0jsq5y.jpg)

使用 `.col-md-offset-*` 类可以将列向右侧偏移。这些类实际是通过使用 `*` 选择器为当前元素增加了左侧的边距（margin）。例如，`.col-md-offset-4` 类将 `.col-md-4` 元素向右侧偏移了4个列（column）的宽度。

看到这里大家感觉这个方案很完美，既有相应布局又有布局的偏移，但我的项目需求是这样的

<!--
![](/uploads/自己动手实现一个 Flex 布局框架4.png)
-->

![](https://ws1.sinaimg.cn/large/005Yd2Thly1fl9k47bdb0j30my112q8t.jpg)

这里单选按钮和票的名称居左，而票价居右，左右给相同的 `padding` 后，单选按钮和票价分别在左右处于临界状态，我并不知道右侧的票价占几个栅格，也不知道左侧的偏移到底给多少合适（因为票价是变量，可能 10 位数，当然可能性为 0）

了解 flex 基础的一眼识破，不是有 `space-between` 嘛，对就是它，不了解的朋友继续转到文章开头的链接温习一下。

下文我们去找设计灵感

## Angular Material 框架

What is Angular Material?

> For developers using AngularJS, Angular Material is both a UI Component framework and a reference implementation of Google's Material Design Specification. This project provides a set of reusable, well-tested, and accessible UI components based on Material Design.

用过 AngularJS 的人应该多少有所耳闻，没听说的也没关系。我们学习的是设计思想而不是研讨一门框架。

这里的案例来源于：https://material.angularjs.org/1.0.8/layout/alignment

上面链接是 Angular Material 框架布局部分的 API 文档，文档下方有单选按钮组合来呈现不同的布局实现。

先给出基本代码

```html
<div layout="row" layout-align="center center">
  <div>one</div>
  <div>two</div>
  <div>three</div>
</div>
```

效果如下

<!--
![](/uploads/自己动手实现一个 Flex 布局框架5.png)
-->

![](https://ws1.sinaimg.cn/large/005Yd2Thly1fl9k473mw2j32io0jojtv.jpg)

其它属性如下，

<!--
![](/uploads/自己动手实现一个 Flex 布局框架6.png)
-->

![](https://ws1.sinaimg.cn/large/005Yd2Thly1fl9k479zaaj32ik0u80x5.jpg)

进入上方链接可以在线感受一下，所有布局效果，这里不一一截图

同样也支持栅格系统不过这里更精密一些，是 100 份的均分，官网例子给的特别全面，链接： https://material.angularjs.org/1.0.8/layout/children

这里给大家选出一个比较通用的例子，代码如下

```html
<div layout="row" layout-wrap>
  <div flex="30">
    [flex="30"]
  </div>
  <div flex="45">
    [flex="45"]
  </div>
  <div flex="25">
    [flex="25"]
  </div>
  <div flex="33">
    [flex="33"]
  </div>
  <div flex="66">
    [flex="66"]
  </div>
  <div flex="50">
    [flex="50"]
  </div>
  <div flex>
    [flex]
  </div>
</div>
```

效果如下

<!--
![](/uploads/自己动手实现一个 Flex 布局框架7.png)
-->

![](https://ws1.sinaimg.cn/large/005Yd2Thly1fl9k473m0uj32io0e4jtl.jpg)

代码简洁易懂，`layout="row"`表示在水平方向分布，最后的 `flex` 不带参数表明自动填充，将不带 `flex` 属性的元素之前的空间填满。

下面我们回到需求，针对需求给出 html 结构的设想

```html
<div layout="row">
  <div flex>单选按钮和票的名称</div>
  <div>票价</div>
</div>
```

或者干脆

```html
<div layout="row" layout-align="space-between center">
  <div>单选按钮和票的名称</div>
  <div>票价</div>
</div>
```

好，有的朋友说使用 `float` 或者 `text-align` 也可以满足需求的啊，干嘛写这么长篇幅的文章解释这个案例？

问的好，首先 flex 布局优势特别明显，弹性布局，不存在兼容问题，也不用清除浮动。

设想一下项目复杂度再大一点呢，守旧的方案还能不能保持清晰的 html 文档结构？css 又该从哪里下手？

既然我们出发点是对的，接下来选择一下设计模式。

简单说两种模式

* class 属性为代表的 Bootstrap 框架
* 自定义属性为代表的 Angular Material 框架

我个人认为 class 过多导致布局和样式混在一起不好分辨，后期维护较困难，决定采用 Angular Material 框架的设计模式。

首先大家要了解 [css 属性选择器](http://www.w3school.com.cn/css/css_selector_attribute.asp)，常用的有 class选择器，id选择器，tag选择器，属性选择器还是比较少用的。

下面给 w3school 的截图，子串匹配属性选择器的语法

<!--
![](/uploads/自己动手实现一个 Flex 布局框架8.png)
-->

![](https://ws1.sinaimg.cn/large/005Yd2Thly1fl9k475k8sj31vs0bojuj.jpg)

简单易懂，下面直接上写好的代码 `layout.scss`

```scss
[layout] {
  display: flex;
}

[flex] {
  flex: 1;
}

[layout-wrap] {
  flex-wrap: wrap;
}

[layout="row"] {
  flex-direction: row;
}

[layout-wrap] {
  flex-wrap: wrap;
}

[layout="column"] {
  flex-direction: column;
}

[layout-align="start start"],
[layout-align="start center"],
[layout-align="start end"] {
  justify-content: flex-start;
}

[layout-align="center start"],
[layout-align="center center"],
[layout-align="center end"] {
  justify-content: center;
}

[layout-align="end start"],
[layout-align="end center"],
[layout-align="end end"] {
  justify-content: flex-end;
}

[layout-align="space-between start"],
[layout-align="space-between center"],
[layout-align="space-between end"] {
  justify-content: space-between;
}

[layout-align="space-arround start"],
[layout-align="space-arround center"],
[layout-align="space-arround end"] {
  justify-content: space-arround;
}

[layout-align="start start"],
[layout-align="center start"],
[layout-align="end start"],
[layout-align="space-between start"],
[layout-align="space-arround start"] {
  align-items: flex-start;
}

[layout-align="start center"],
[layout-align="center center"],
[layout-align="end center"],
[layout-align="space-between center"],
[layout-align="space-arround center"] {
  align-items: center;
}

[layout-align="start end"],
[layout-align="center end"],
[layout-align="end end"],
[layout-align="space-between end"],
[layout-align="space-arround end"] {
  align-items: flex-end;
}
```

好，到这为止我们的 flex 框架已经实现了，效果语法和 Angular Material 框架是一样的。大家自行尝试。

细心的朋友发现这里 orange 并没有实现栅格系统，因为现实需求中栅格系统布局的实用价值不是很大（各元素宽度根据内容变化，手机端在元素宽度不变的情况可以通过相同的 rem 值针对不同屏幕适配，而 n 等分可以通过 `space-arround` 属性实现），而且本文把开发的重点放在了 flex 的封装上。

## 总结

在现代复杂 css 样式的开发中，尽量避免重复书写相同的布局代码，除非特殊需求（真对相应的 class 给样式），这样既满足模块化思想又保证了代码复用，项目中只需引入 `layout.scss` 即可。如果你针对 css 代码模块化有不同的想法欢迎留言交流。
