---
title: 解决 font-weight 无效的问题
date: 2016-12-13 12:14:18
tags: CSS
---

<!--
![](/uploads/font-weight 无效问题1.png)
-->

![](https://ws1.sinaimg.cn/large/005Yd2Thly1fl9kknc3udj312o070gm9.jpg)

近期调页面时有几个 font-weight 需要修改，无论怎么调整字体粗细都没有变化，深入研究后总结下文

<!--more-->

## 初探

本地写个例子，代码如下

```html
<p class="thin">This is a paragraph</p>
<p class="normal">This is a paragraph</p>
<p class="thick">This is a paragraph</p>
```

```css
p {
  font-size: 50px;
}
p.thin {
  font-weight: 100;
}
p.normal {
  font-weight: normal;
}
p.thick {
  font-weight: bold;
}
```

在 Mac OS 下 Chrome、Firefox、Safari 结果分别如下(从左到右)

<!--
![](/uploads/font-weight 无效问题2.png)
-->

![](https://ws1.sinaimg.cn/large/005Yd2Thly1fl9kknfvlcj312i07ytav.jpg)

我的浏览器均为最新版本，发现一个简单的 `font-weight` 属性，在三个浏览器有三个表现。

* Chrome 下所有字重均一样
* Firefox 下表现正常，是我们期待的结果
* Safari 下 100 无效，被解析为 normal

## 解决表现不一致的问题

这种不同浏览器表现不同是我们不能接受的，对于后期排错造成困难，于是我首先想到是字体的惹得货，修改我的样式文件

```css
p {
  font-size: 50px;
  font-family: Arial;
}
p.thin {
  font-weight: 100;
}
p.normal {
  font-weight: normal;
}
p.thick {
  font-weight: bold;
}
```

效果如下

<!--
![](/uploads/font-weight 无效问题3.png)
-->

![](https://ws1.sinaimg.cn/large/005Yd2Thly1fl9kkni4rej312i07yjtd.jpg)

这里的表现倒是一样的，我们可以忽略图中字体大小（截屏的误差导致），只看字体粗细就好，`font-weight: 100;` 都失效了。

MDN 文档的解释

> This means that for fonts that provide only normal and bold, 100-500 are normal, and 600-900 are bold.

文章开始没有介绍基本语法，相信前端们都知道，normal 等同于 400， bold 等同于 700。

这也很好的解释了这个例子的表象，但我瞬间推翻了这句话，因为在例子1中 Firefox 在没有设置字体的情况下可以正常显示。

## 问题根源

到这里相信你已经知道答案了，我们要针对不同浏览器和运行环境进行全面配置 `font-family` 属性，**全局的字体建议放在 body 选择器下**。

```css
p {
  font-size: 50px;
  font-family: -apple-system, BlinkMacSystemFont,
    "Segoe UI", "Roboto", "Oxygen", "Ubuntu", "Cantarell",
    "Fira Sans", "Droid Sans", "Helvetica Neue",
    sans-serif;
}
p.thin {
  font-weight: 100;
}
p.normal {
  font-weight: normal;
}
p.thick {
  font-weight: bold;
}
```

看下三个浏览器的表现

<!--
![](/uploads/font-weight 无效问题4.png)
-->

![](https://ws1.sinaimg.cn/large/005Yd2Thly1fl9kknc3g9j312i08gdhx.jpg)

在字体和字重上达到了完全一致，仔细的观察会发现，Chrome 与 Safari 渲染不同字重的字体总宽度变化明显，而 Firefox 下则不是十分明显

> 温馨提示：尽量不要用字体去撑容器的宽度，尽量避免 hover 改变字重。因为不同环境下渲染的差异会导致表现不一致。

上面给的一大串字体又代表兼容那些环境和设备哪？

首先我们分成三组来解释

```
font-family:
/* 1 */ -apple-system, BlinkMacSystemFont,
/* 2 */ "Segoe UI", "Roboto", "Oxygen", "Ubuntu", "Cantarell", "Fira Sans", "Droid Sans",
/* 3 */ "Helvetica Neue", sans-serif;
```

1.第一个分组是映射到系统 UI 字体的 CSS 属性。这涵盖了很多环境，并且不会将这些字体误认为别的字体

* `-apple-system` 在 Mac OS X 和 iOS 上的 Safari 中设置 San Francisco，并在旧版本的 Mac OS X 上设置成 Neue Helvetica 和 Lucida Grande。它根据字体大小正确选择 San Francisco Text 和 San Francisco Display。
* `BlinkMacSystemFont` 只针对于 Mac OS X 上的 Chrome。

2.第二个分组用于已知的系统 UI 字体

* `Segoe UI` 针对 Windows 和 Windows Phone。
* `Roboto` 针对 Android 和更高版本的 Chrome 操作系统。故意列出在 Segoe UI 后，因为如果你是 Windows 上的 Android 开发人员，并安装 Roboto，将使用 Segoe UI。
* `Oxygen` 针对 KDE，Ubuntu，你可以猜到，Cantarell 针对 GNOME。这一开始感到徒劳，因为一些 Linux 发行版有许多这样的字体。
* `Fira Sans` 针对 Firefox OS 系统。
* `Droid Sans` 针对旧版本系统的安卓

> 请注意，我们不需要添加 San Francisco。在 iOS 和 Mac OS X 上，San Francisco 并不是显而易见的，而是作为“隐藏”字体存在。我们也不使用 .SFNSText-Regular，在 Mac OS X 上的 San Francisco 的内部 PostScript 名称来指定 San Francisco。它只适用于 Chrome，并且不如 BlinkMacSystemFont 通用。

3.第三个分组是我们的后备字体

* `Helvetica Neue` 针对旧 El Capitan 版本的 Mac OS X。它被列在接近结尾，因为它是一个流行的字体在其他非 El Capitan 计算机上。
* `sans-serif` 默认的是 sans-serif 后备字体。

以下是目前已知的的问题：

1. 在 Mac OS X 的 Firefox 中，San Francisco 的字母间距比 Safari 和 Chrome 更紧。
2. 它不会使 Lucida Grande 在 Mac OS X 的 pre-Yosemite 版本上降级到 Neue Helvetica。并且它可能不匹配不太受欢迎的操作系统上的正确字体或更复杂的配置。

说到这里上面都是英文的字体，我们需要针对中文设置字体，可以针对不同操作系统给中文字体。

## 总结

对于字体的统一展示，目前为止还没有完善的解决办法，只能针对不同设备给出对应的解决方案，至于为什么不引入外部的三方字库来统一字体呢？因为会增加网页的请求时长，渲染也会耗时，尽量避免三方字库。下次再有类似字重渲染误差问题可以先从字体下手，整个例子没有跑过 Windows 系统，可能在 Windows 下还会有问题。至少切入点有了改变，并不是 Chrome 下 font-weight 无效。
