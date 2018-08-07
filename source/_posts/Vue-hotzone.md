---
title: Vue 实现热区图
date: 2018-08-07 10:48:58
tags: ['Vue']
---

<!--
![](/uploads/Vue 实现热区图1.png)
-->

![](https://ws1.sinaimg.cn/large/005Yd2Thly1fu13zfdsagj30rs0cn74l.jpg)

<!--more-->

## 前言

当今流行的前端框架把组件化开发的模式推进的十分出色，也激发了开发者针对各类框架写组件的热情，因为接口明确，上手也相对容易，一个组件只为了解决一个点或一类问题而生。

先看几个问题，由问题带入本文

* 热区图是什么？

直接看demo会更好: https://orangex_c.coding.me/vue-hotzone/

> 记得在 PC 端打开，移动端为了方便展示截图在下面，简单描述就是上图中选定热区，下图中点击热区位置会执行相应的操作（图中例子是点击热区直接跳转到我的 Github 主页），展示的虚线线框是为了方便查看区域边界，真实场景会是设计师放上去的按钮或点击区域

<!--
![](/uploads/Vue 实现热区图2.png)
-->

![](https://ws1.sinaimg.cn/large/005Yd2Thly1fu0zgijbj1j30rs105anv.jpg)

* 能解决什么问题？

看似画蛇添足的功能，有人会想直接在前端用定位+点击事件就能解决的问题，干嘛要一个管理热区的平台呢，这就是重复性工作和造轮子之间的取舍，如果重复性工作带来的开销远大于造轮子，为何不采取工程化和模块化的方案解决呢！

> 热区图解决了前端大量的重复写定位样式和事件绑定，我们更希望设计切一张完整的图片，让他们自己框定热区，不希望切多张图片前端定位拼凑，即节约了开发成本，又提升了网站性能。

* 使用场景有哪些？

1. 图片内存在多个点击区域
2. 活动弹出框（带有详情和取消按钮）

## vue-hotzone

之前看过几个版本的实现，恰好没有 vue 版的，当然本组件也是借鉴其它框架的组件写的，也是一个重复造轮子但不是第一个造轮子的组件

本组件参考：[regular-hotzone](https://github.com/Deol/regular-hotzone)
react版本：[react-multi-crops](https://github.com/beizhedenglong/react-multi-crops)

这里不聊具体实现细节，细节可以看源码

Github: https://github.com/OrangeXC/vue-hotzone

### 目录

```
vue-hotzone
├── __tests__ (单元测试)
├── lib (此组件)
├── public (DEMO网站入口)
├── src (DEMO网站)
├── .eslintrc.js (eslint配置)
├── jest.config.js (jest配置)
├── vue.config.js (vue配置)
├── package.json
├── ......
```

### 功能划分

在 `regular-hotzone` 中强调的是管理和展示共存，通过 props 判断是编辑状态还是展示状态

设计之初并没有实现这种模式，因为客户端vue代码短短几句代码就能实现定位，没必要把整体引入进来浪费首屏时间

本组件采用的设计只针对后台编辑端使用，将数据持久化存储后前端调取数据后可以参考 `src` 下的例子去做

### 数据结构

存下来的结构可以是带有自定义字段的本例子存的结构，本例如下

```js
zones: [{
  heightPer: 0.4374,
  leftPer: 0.1153,
  topPer: 0.238,
  widthPer: 0.2827,
  url: 'https://github.com/OrangeXC'
}]
```

这里的基础数据都是 `Per` 结尾的百分比数值，url 是我们自定义绑定到热区上的属性（根据需要可以绑定更多信息）

### 配置和事件

参考：https://github.com/OrangeXC/vue-hotzone#options

## 总结

这里强调一下行文目的，让这个组件被更多人发现，转变下开发模式或许效率真的可以翻倍，也希望更多的简化操作并可以提高开发效率的组件被开发出来。

至于态度还是开源的态度，也希望区别于直接贴 github 求 star 的文章，实感无聊，有意见欢迎提 pr 和 issue 交流。
