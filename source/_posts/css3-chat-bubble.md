---
title: CSS3 巧妙实现聊天气泡
date: 2016-10-13 14:08:32
tags: CSS3
---

![](/uploads/css3-chat-bubble1.jpg)

<!--more-->

前一阵子敢玩的 Mobile 页改版完成了，就之前的页面风格更加扁平化，从暗色系为主背景转到亮色背景，去掉更多的阴影，给用户简约的体验风格，哈哈我不是设计师不过多评价啦。感兴趣的朋友可以直接去 [idarex移动端主页](https://mobile.idarex.com/)。

这次改版的所有 style 都是 orange 写的，感触颇多，分期分享给大家

下面说正题，说好的聊天气泡呢？

## 传统的聊天气泡

什么又是传统的聊天气泡,直接上图

![](/uploads/css3-chat-bubble2.jpg)

代码如下

```html
<div class="comment"></div>

<style type="text/css">
  .comment {
    width: 150px;
    height: 35px;
    position: relative;
    margin: 30px auto 0;
    background: #f8ac09;
    border-radius: 5px;
  }

  .comment:after {
    content: '';
    width: 0;
    height: 0;
    position: absolute;
    top: 5px;
    right: -16px;
    border: solid 8px;
    border-color: transparent transparent transparent #f8ac09;
    font-size: 0;
  }
</style>
```

实现方式大家早有耳闻，圆角矩形和三角形嘛，三角形原理就是 border 可以设置为透明，可以复制上例中的代码修改 `border-color` 属性摸索三角形的实现。

> 注：IE8 更早版本对 border 的 transparent 支持不是很好。大家可以无视低版本缺陷，因为大部分浏览器都显示正常，非要兼容的话把 transparent 属性设置为主背景色而不是气泡背景色（前提是背景为纯色）。

想必大家都知道，这里不赘述，聊一聊其他实现方法。

这里的三角形部分可以使用正方形代替，实现同样效果，方法就是旋转小正方形使其一部分露在外面。代码如下

```css
.comment {
  position: relative;
  width: 150px;
  height: 35px;
  background: #f8ac09;
  border-radius: 5px;
  margin: 30px auto 0;
}

.comment:after {
  content: '';
  position:absolute;
  top: 10px;
  right: -4px;
  width: 8px;
  height: 8px;
  transform: rotate(45deg);
  background-color: #f8ac09;
}
```

缺点是小三角只能是直角三角形，当然也可以通过变换得到菱形再进行拼接，变换多了感觉没有第一种方式直接，浏览器兼容 transform(2D) 属性如下

![](/uploads/css3-chat-bubble3.jpg)

总体还不错，几种方法都能放心使用，不存在大的兼容问题。

## 现实案例

这里的设计稿多了一个边框，直接上设计稿

![](/uploads/css3-chat-bubble4.jpg)

🤔️ 想一想怎么处理，我们回顾上文

第一种方式本身就是 `border` 透明，怎么再给它设置 `border` 是个问题，暂且先不考虑。

第二种方式如果使用小正方形旋转，层级叠加是个问题，因为设计稿中的气泡背景为 `rgba(247, 188, 10, 0.03)` 先看下实现代码

```css
.comment {
  width: 150px;
  height: 35px;
  position:relative;
  margin: 30px auto 0;
  background-color: rgba(247, 188, 10, 0.03);
  border: 1px solid rgba(252, 185, 8, 0.35);
  border-radius: 5px;
}

.comment:after {
  content: '';
  width: 8px;
  height: 8px;
  position: absolute;
  top: 10px;
  right: -4px;
  transform: rotate(45deg);
  background-color: rgba(247, 188, 10, 0.03);
  border: 1px solid rgba(252, 185, 8, 0.35);
}
```

效果如下

![](/uploads/css3-chat-bubble5.jpg)

上面的思路有问题，因为小正方形与气泡的一部分会重合，半透明背景的部分总会出现问题，有人说了偷个懒总可以吧，把透明后的背景色吸取出来然后再进行叠加（因为大家注意到设计稿的整体背景是纯色）

按着这个思路去实现，那么问题又来了。具体两个问题如下。

1.如果小正方形叠加在上，那么小正方形左半部分的边框就会显示

```css
.comment {
  width: 150px;
  height: 35px;
  position: relative;
  margin: 30px auto 0;
  background-color: #faf8f3;
  border: 1px solid #fbe2a0;
  border-radius: 5px;
}

.comment:after {
  content: '';
  width: 8px;
  height: 8px;
  position:absolute;
  top: 10px;
  right: -4px;
  transform: rotate(45deg);
  background-color: #faf8f3;
  border: 1px solid #fbe2a0;
}
```

效果如下，比较之前的图片圆角矩形的右边确实遮住了，但小正方形左边的边框显示出来了

![](/uploads/css3-chat-bubble6.jpg)

处理方式呢，可以这样。

```css
.comment:after {
  content: '';
  width: 8px;
  height: 8px;
  position: absolute;
  top: 10px;
  right: -5px;
  transform: rotate(45deg);
  background-color: #faf8f3;
  border: 1px #fbe2a0;
  border-style: solid solid none none;
}
```

我们发现问题解决了。效果如下

![](/uploads/css3-chat-bubble7.jpg)

设计稿是有 `padding` 的，亲测本案例中可行，但是本着认真的原则 `padding-right` 如果过小，会出现什么问题呢？

我们向 div 中加文字。

```html
<div class="comment">Hello,orange.Welcome to FrontEnd World!</div>
```

效果如下

![](/uploads/css3-chat-bubble8.jpg)

我们发现字母 o 的右下角被小正方形左侧覆盖了，当然可以通过 `z-index` 属性 hack。

2.如果小正方形在圆角矩形下，那么圆角矩形的右边框就会完整显示，大家自行脑补，此方案不合理，不过多解释。

以上的方法缺点也都很明显，那怎么做才能更严谨，能根据需求的变化不大伤筋骨呢？

我们还用三角形的方案！ what? 不是说三角形的方案不可行了嘛 ？

一个三角形是不可行那两个呢，我们有请 `after` 的兄弟 `before` 出场。项目的真实代码如下

```scss
.reply {
  position: relative;
  margin: 0.672rem 0 0.096rem 0;
  padding: 0.408rem 0.816rem;

  border: 1px solid rgba(#fcb908, 0.35);
  border-radius: 0.2rem;
  background-color: rgba(#f7bc0a, 0.03);

  &:after {
    content: '';
    width: 0px;
    height: 0px;
    border-color:  transparent transparent #faf8f3 transparent ;
    border-style: solid;
    border-width: 6px;
    position: absolute;
    top: -11px;
    border-radius: 3px;
    left: 18px;
    right: auto;
  }

  &:before {
    content: '';
    width: 0px;
    height: 0px;
    border-color: transparent transparent rgba(#fcb908, 0.35) transparent;
    border-style: solid;
    border-width: 7px;
    position: absolute;
    top: -14px;
    border-radius: 3px;
    left: 17px;
    right: auto;
  }
}
```

> 注：这段代码用的是 SASS 进行预编译，如果从头仔细看到这里的话不难理解，两个三角形叠加，大三角形颜色是边框的颜色，小三角形是内部背景色，小三角形绝对定位时向下移 3px 把圆角矩形的一部分上边框遮挡，这样小三角下部也有溢出，具体在两像素之内，实际上不存在遮挡文本问题。

## 总结

实际问题解决的方法很多，就看大家怎么去思考，这个方案也不是最满意的方案，因为多了一个伪元素，主要还是设计思想的多样性，总之 css 很灵活。

有人不禁会问，这里设计稿给的是向上的箭头，为什么例子里却都是向右的，这里向右的都是我写的 demo ，理解原理的话，改变个位置方向都是大同小异。

最后，读本文有收获的或者有更好想法的朋友，欢迎下方留言交流。
