---
title: CSS 实现 1px 以内的移动
date: 2016-10-17 10:22:43
tags: CSS
---

<!--
![](/uploads/CSS 实现 1px 以内的移动1.png)
-->

![](https://ws1.sinaimg.cn/large/005Yd2Thly1fl9kc7v060j31z00j40tv.jpg)

之前的文章说过关于行内元素垂直方向对齐的方案。感兴趣的可以看我的往期文章。在上一篇文章里我们提到了 1px 内的移动问题。本文就一像素内的问题给出解决方案。

<!--more-->

可能大家看过关于 Retina 屏幕的一像素边框问题，注意这里是边框宽度而不是移动元素。

什么？border 小于 1px ？

对，因为前面有人给出相关方案而且好多种方案，这里不重复描述实现原理，给大家两个链接，感兴趣的自己跳转。

* [Retina 屏的移动设备如何实现真正 1px 的线？](http://jinlong.github.io/2015/05/24/css-retina-hairlines/#comments)
* [移动 web 点 5 像素的秘密](http://www.cnblogs.com/PeunZhang/p/4709822.html)

看完大彻大悟，佩服佩服，思路很多，回到本文重点

想一下能实现移动的方法 `position(top,right,bottom,left)`, `margin`, `padding`, `vertical-align`。

上面给的只是一部分可以通过具体单位(px, em, rem 等)进行移动的方法

本着实践的原则，上述方案都不可行，在最新的 chrome 中，当小于 0.5px 时是 0，当大于等于 0.5px 时就变成 1px。

因为案例过于简单，不做 demo ，感兴趣的自己实践，相信大家多数人试验过了。

那么还有什么以具体单位移动的属性呢？

## 解决方案

也许你早就知道有 `transform` 的  `translate` 属性了。没错它就能实现 1px 内的移动！

基本语法：

```scss
transform: translate(12px, 50%);
transform: translateX(2em);
transform: translateY(3in);
```

给出本文的 demo 代码，

```html
<div class="parent">
  <div class="child-first"></div>
  <div class="child-second"></div>
  <div class="child-third"></div>
</div>

<style>
  .parent {
    width: 310px;
    height: 150px;
    background-color: #666;
  }

  .parent div {
    display: inline-block;
  }

  .child-first {
    width: 100px;
    height: 100px;

    margin-top: .5px;

    transform: translateY(.3px);

    background-color: #f66;
  }

  .child-second {
    width: 100px;
    height: 100px;

    transform: translateY(.5px);

    background-color: #ff0;
  }

  .child-third {
    width: 100px;
    height: 100px;

    transform: translateY(1px);

    background-color: #06c;
  }
</style>
```

截图如下

<!--
![](/uploads/CSS 实现 1px 以内的移动2.png)
-->

![](https://ws1.sinaimg.cn/large/005Yd2Thly1fl9kc7zuq0j32wk1fo0xc.jpg)

这里为了更容易观察，我们把布局换成 `inline-block` ，我们发现元素与元素之间存在空隙回去再看一遍代码发现没什么问题，那这段距离是怎么引起的呢？

是空格? 没错! 在使用 `inline-block` 的时候一定注意代码缩进或换行带来的不必要的麻烦（无意中添加了空格）。

修改如下

```html
<div class="parent">
  <div class="child-first"></div><div class="child-second"></div><div class="child-third"></div>
</div>
```

得到最终结果，如下图

<!--
![](/uploads/CSS 实现 1px 以内的移动3.png)
-->

![](https://ws1.sinaimg.cn/large/005Yd2Thly1fl9kc80giaj32w81f0n1f.jpg)

这里特地将小块颜色做区分，浏览器视图放大到最大倍数，如果还是看不清的话，推荐大家亲手试一试，

## 总结

到这里我的方法讲完了，在最后欢迎大家讨论，方案不止一个， orange 目前只发现这一个方案，你也可以根据 js 判断屏幕然后给出 .5 像素的偏移也是可行的，我个人认为此方法简单一些。
