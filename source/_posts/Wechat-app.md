---
title: 微信小程序实战
date: 2017-01-12 10:14:18
tags: App
---

<!--
![](/uploads/微信小程序实战1.jpeg)
-->

![](https://ws1.sinaimg.cn/large/005Yd2Thly1fl9jxqj96sj30uy0asjrt.jpg)

微信小程序经过几个月的内侧，在今年的 1 月 9 日正式上线，在微信通讯录页面就可以搜索你想找的小程序，然后在发现页最底部就会有你曾经浏览过的小程序的入口。

一番体验后比橙子想象的效果好的多，所以自己起手也写了一个。下面具体介绍细节。

想写小程序的大家都知道只有企业账户可以发布，但是开发却不需要企业账号，但是我们需要实名认证我们的小程序账号。

<!--more-->

## 注册账号

直接给官网链接 https://mp.weixin.qq.com/debug/wxadoc/introduction/?t=201718#注册小程序帐号

> 注意：没有企业认证记得在 **选择主体类型** 时选择->其他组织，组织名称自己起，机构代码只要符合它的规则就行（输入框下有提示直接复制粘贴），公章的扫描件上传随意图片即可，管理员信息务必填写真实信息（切记）

## 开发工具

在官网给出了下载入口 https://mp.weixin.qq.com/debug/wxadoc/dev/devtools/download.html?t=201715

安装后会有扫码登录，然后按照这个链接执行 https://mp.weixin.qq.com/debug/wxadoc/introduction/?t=201718#登录

记得你在官网注册成功后小程序的 AppID，这里不用选择本地开发目录，为了起手方便当你添加项目时会问你是不是使用它的初始化模板（点击使用）。

现在不出差错的话你的页面会出现 你的 头像、用户名、Hello world

如果显示正常，恭喜你环境配置成功

有时却不是那么顺利，橙子在打开初始化项目报错 `Failed to load resource: net::ERR_NAME_NOT_RESOLVED`

如果你有相同的错误解决办法很简单，关掉你的 vpn 即可（并不是所有 vpn 均存在问题，因为橙子咨询其他人的 vpn 没关也不会产生这个问题），记得重启你的开发者工具。

## 起步

如果你了解几种文件的作用可以跳过此步，不了解的可以看下官网起步教程，https://mp.weixin.qq.com/debug/wxadoc/dev/?t=201715

然后在上面链接仔细过一遍 框架、组件、API（切记认真阅读）

也许你会问官网给的那么详细我写这个文章的意义在哪？

问的好，如果官网能给我一个完整的小程序我也不会费周折去自己写，官网的 API 固然重要，都是人家开发的我们无权篡改，本文的目的是尽量让大家少踩坑。

## 实战

自己定义下产品详情，一个展示视频的 app，列表与播放页，非常简单的例子，上手简单

这里涉及到网络请求，首先去管理平台，最下面的设置里配置你的服务器域名，你自己有接口最好，不必走本例的接口，根据官方文档你可以写出自己的程序，另外值得一提的就是 V2EX，知乎日报，豆瓣等都提供外部接口供大家使用

> 说明：没有接口的可以用本例的 `https://api.idarex.com`（去自己的小程序网页控制台->设置->填写到**request合法域名**即可），此步不设置后面请求会报错，每月只能修改三次谨慎使用

两个页面构成？倒也没那个必要，增加整个小程序的体积，而且跨页面传输数据多一层逻辑，这里采用 `wx:if` 实现条件渲染来实现。

我们只需要保留 `index` 文件夹下的文件

从全局文件改起 log 部分可以选择保留，获取用户信息部份暂时用不到

app.js

```js
App({
  onLaunch: function () {
    //调用API从本地缓存中获取数据
    var logs = wx.getStorageSync('logs') || []
    logs.unshift(Date.now())
    wx.setStorageSync('logs', logs)
  }
})
```

全局的配置文件只需要 window 的基础配置，这里修改 navigationBar 的内容样式

app.json

```json
{
  "pages":[
    "pages/index/index",
    "pages/logs/logs"
  ],
  "window":{
    "backgroundTextStyle":"light",
    "navigationBarBackgroundColor": "#f8ac09",
    "navigationBarTitleText": "敢玩原创视频",
    "navigationBarTextStyle":"black"
  }
}
```

全局样式没有太多规则，这里避免后期的计算误差问题注意两点

app.wxss

```css
/* 让页面高占满全屏，类似给 html 设置 100% 让子集元素的百分比为全屏的百分比 */
page {
  height: 100%;
}

/* border-box 非常方便我们计算宽高 */
view, text {
  box-sizing: border-box;
}

.container {
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: space-between;
  box-sizing: border-box;
}
```

> 注：`*` 选择器无效，强烈推荐使用 flex 进行布局，许多场景下普通的布局规则无效

主页上面一个轮播，轮播图改变时剧集也发生变化，默认的 player 隐藏，页面结构如下

pages/index/index.wxml

```html
<view class="container">
  <swiper class="swiper" bindchange="changeAlbum" indicator-dots="{{indicatorDots}}" autoplay="{{autoplay}}" duration="{{duration}}">
    <block wx:for="{{albums}}" wx:key="title">
      <swiper-item class="swiper-item">
        <image src="{{item.cover}}" class="slide-image" />
      </swiper-item>
    </block>
  </swiper>
  <scroll-view class="scroll-view" scroll-y="true" scroll-top="{{scrollTop}}">
    <block wx:for="{{videoList}}" wx:key="id">
      <view class="video-item" data-id="{{index}}" catchtap="playVideo">
        <image src="{{item.cover}}" class="video-cover"></image>
        <view class="video-info">
          <text class="video-title">{{item.title}}</text>
          <text class="video-play-count">{{item.play_count}}次播放</text>
        </view>
      </view>
    </block>
  </scroll-view>
  <view class="player" wx:if="{{playerShow}}">
    <icon class="close" type="clear" size="45" color="rgba(255, 255, 255, 0.5)" catchtap="closeVideo" />
    <video class="video" src="{{videoUrl}}" autoplay="true" />
    <text class="player-title">第{{videoIndex + 1}}集:{{videoTitle}}</text>
    <view class="handle-bar">
      <button class="pre-btn" catchtap="preVideo">上一集</button>
      <button class="next-btn" catchtap="nextVideo">下一集</button>
    </view>
    <text class="msg">没看过瘾？返回主页侧滑看更多炸裂专辑</text>
  </view>
</view>
```

> 注：`wx:for` 建议写到外层的 block 标签中，而且必须加上 `wx:key` 定义你的标志字段，否则报错。绑定事件如果为 tap，强烈建议使用 `catchtap` 而不采用 `bindtap`，除非有特殊的冒泡触发事件需求，否则使用 `catchtap`避免冒泡。这里为什么使用 `wx:if`（首屏加载块但切换成本大） 而不用官方建议的 hidden（首屏加载慢但切换成本小），在这个场景 player 层会频繁切换适合使用 hidden，但 hidden 存在 bug，它隐藏不掉 view 层里面包含的元素，video 标签依然暴露在且占据空间，破坏了我们的整体布局，实际测试 `wx:if` 在微应用场景无任何性能问题。

看下整体的样式文件

pages/index/index.wxss

```css
.swiper {
  width: 100%;
  height: 40%;
}

.swiper-item {
  display: flex;
  justify-content: center;

  background: #dedede;
}

.slide-image {
  width: 50%;
  height: 100%;
}

.scroll-view {
  height: 60%;
}

.video-item {
  display: flex;
  flex-direction: row;
  justify-content: flex-start;
  height: 115px;
  padding: 10px;

  background: rgba(255, 255, 255, 0.9);
  border-bottom: 1px solid #dedede;
}

.video-cover {
  display: block;
  width: 150px;
  height: 95px;

  border-radius: 5px;
}

.video-info {
  display: flex;
  flex: 1;
  flex-direction: column;
  justify-content: space-between;
  padding: 5px 0 5px 20px;
}

.video-title {
  width: 100%;

  line-height: 1.5;
  font-size: 14px;
}

.video-play-count {
  width: 100%;
  display: flex;
  justify-content: flex-end;
  font-size: 12px;
}

.player {
  width: 100%;
  height: 100%;
  position: absolute;
  top: 0;
  left: 0;
  z-index: 100;

  background: #000;
}

.player-title {
  width: 100%;
  padding-left: 10px;

  border-left: 3px solid #ccc;

  overflow:hidden;
  white-space:nowrap;
  text-overflow:ellipsis;

  color: #fff;
  font-size: 14px;
}

.close {
  position: fixed;
  top: 20px;
  right: 20px;
}

.video {
  width: 100%;
  margin-top: 80px;
}

.handle-bar {
  display: flex;
  flex-direction: row;
  justify-content: space-around;

  margin-top: 20px;
}

.pre-btn, .next-btn {
  width: 40%;

  background: #f49d0d;

  color: #673806;
}

.button-hover {
  background-color: #f9bd3a;
}

.msg {
  width: 216px;
  margin-left: -108px;
  position: fixed;
  bottom: 10px;
  left: 50%;

  font-size: 12px;
  color: #999;
}
```

> 注：样式以 flex + 百分比做整体布局，具体块级元素和字体给 px 值

pages/index/index.js

```js
//获取应用实例
var app = getApp()

Page({
  data: {
    albums: [],
    videoList: [],
    scrollTop: 0,
    indicatorDots: true,
    autoplay: false,
    duration: 1000,
    playerShow: false,
    videoUrl: '',
    videoTitle: '',
    videoIndex: 0
  },
  changeAlbum: function (e) {
    this.setData({
      videoList: this.data.albums[e.detail.current].videos,
      scrollTop: 0
    })
  },
  playVideo: function (e) {
    let index = e.currentTarget.dataset.id

    this.setData({
      videoTitle: this.data.videoList[index].title,
      videoUrl: this.data.videoList[index].play_url,
      playerShow: true,
      videoIndex: index
    })
  },
  closeVideo: function () {
    this.setData({
      videoUrl: '',
      playerShow: false
    })
  },
  preVideo: function () {
    let index = this.data.videoIndex

    if (index === 0) {
      wx.showToast({
        title: '前面没有啦！',
        icon: 'loading',
        duration: 10000
      })

      setTimeout(function(){
        wx.hideToast()
      },1000)
    } else {
      this.setData({
        videoTitle: this.data.videoList[index - 1].title,
        videoUrl: this.data.videoList[index - 1].play_url,
        videoIndex: index - 1
      })
    }
  },
  nextVideo: function () {
    let index = this.data.videoIndex

    if (index === this.data.videoList.length - 1) {
      wx.showToast({
        title: '后面没有啦！',
        icon: 'loading',
        duration: 10000
      })

      setTimeout(function(){
        wx.hideToast()
      },1000)
    } else {
      this.setData({
        videoTitle: this.data.videoList[index + 1].title,
        videoUrl: this.data.videoList[index + 1].play_url,
        videoIndex: index + 1
      })
    }
  },
  onReady: function () {
    var that = this
    wx.request({
      url: 'https://api.idarex.com/www/index',
      success (res) {
        that.setData({
          albums: res.data.columns,
          videoList: res.data.columns[0].videos
        })
      }
    })
  }
})

```

> 注：`onReady` 取代 `onLoad` 安卓 6.4.3 存在 bug，事件的传参要通过 `event.currentTarget.dataset` 传递在 wxml 里通过定义 data-xxx 属性绑定 key，其它代码简单易懂没有高级语法，可以直接复制粘贴然后自定义修改

最后的效果如下

<!--
![](/uploads/微信小程序实战2.jpeg)
-->

![](https://ws1.sinaimg.cn/large/005Yd2Thly1fl9jxqp1mmj30hs0vktbi.jpg)

<!--
![](/uploads/微信小程序实战3.jpeg)
-->

![](https://ws1.sinaimg.cn/large/005Yd2Thly1fl9jxqlw7nj30hs0vkgn5.jpg)

## 进阶

如果你的小程序拥有一定的规模你一定会尝试，模块化开发、ES6，7 的高级语法、第三方库等等。

实现原理就是按模块化的开发去写，npm 去安装依赖库，然后编译成微信开发者工具可以识别的项目结构

映着需求簇生的 Github 项目列举几个，[wepy](https://github.com/wepyjs/wepy)、[labrador](https://github.com/maichong/labrador)

感兴趣的可以点进去看看，使用与否取决于你项目的复杂程度，你的异步操作过多 ES6 的 promise 无法满足你的需求，ES7 的 async/await 可以帮到你，或是你的状态过多想使用 redux，你就可以尝试微信小程序组件化开发框架。

本例中再封装一层框架纯属没事找事，没有任何意义。

## 总结

小程序的评价褒贬不一，与其评价它存在的意义不如看下它的适用场景，对于功能性强的 App 很合适，本例不太合适但是人家腾讯视频可以做小程序我们就可以做，说到以后的发展如何谁都说不准，最起码稳定性可以保证，借着写小程序可以加深对类 vue 框架的了解，内联的事件写法也是今后的一个趋势，数据的绑定类 react 的 setData，如果你之前尝试过这两个框架开发小程序会非常得心应手。
