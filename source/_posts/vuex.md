---
title: Vuex 实战
date: 2016-10-28 10:12:43
tags: ['Vue', 'Vuex']
---

![](/uploads/vuex1.png)

本文就着之前几天的文章 [Vue2 移动端开发环境搭建](http://orangexc.xyz/2016/10/18/Vue2-mobile-terminal-development-environment-to-build/) 继续扩展，上一篇文章有人反馈说讲到最后只有 `rem` 是移动端相关的知识，没错我个人认为除了 `rem` 和 `touch` 事件特殊外其它与 pc 端无异（手机系统版本和浏览器的 bug 放在这里讨论无意义），下面请出今天的大咖 vuex

<!--more-->

## 状态管理工具

说到这里有两个疑：

1.什么是状态管理工具？

状态管理工具起源于早期的 Flux，意在管理项目中各个状态，状态保存在 store 中，状态是只读的，只能通过 action 触发 store 的更新。

2.问我们为什么使用状态管理工具？

当一个项目逐渐壮大我们需要管理的状态就会变得繁杂，例如：父级组件传递状态到子组件，子组件再次改变同一个状态时就要想办法告诉父级组件状态改变了，情况再复杂点呢！子组件之间又相互影响，又影响到其他子组件的父组件呢？情况变得越来越糟糕，我们需要一个全局的公共状态管理容器，把我们的状态都放进去统一管理。

大型项目对状态的管理需求越来约大，从而发展到今天的 react + redux 组合，angular + ngRedux 组合， vue + vuex 组合。

这里 redux 作为后起之秀有很多真对 Flux 的改进。感兴趣的可以深入了解，这里只关心 vuex 而且是 2.x，用过 1.x 版本的可以顺利过渡到 2.x。

## Vuex2 概览

> Vuex2 官方文档：http://vuex.vuejs.org/en/index.html

能看懂文档的可以跳过实战了。。。

实战开始之前先放出文档，归根结底还是因为文档描述过于简单（简陋），看的我云里雾里，这个对于搞过 redux 的我而言不是原理不理解，而是用法上没有一个能让我一眼看懂的简单粗暴的例子。

方便大家理解工作流程，先给出官方的配图

![](/uploads/vuex2.png)

从左边看 vue components(组件) ->  action（只能通过 dispatch 调用，与此同时可以异步与后端的 API 做交互） -> mutations(只能通过 action 发起 commit 调用，此时开发工具可以监测到数据的流动) -> state （mutations 传递修改过的状态到 state）-> state 自动同步到视图

完成整个循环数据的流动是单向的，从而避免了双向数据流动的复杂性，不同的组件可以修改同一个状态，并将修改后的状态同步到所有关联此状态的组件。

## 实战

原理就解释到这，直接上一个实际例子。

这个例子的背景是我需要一个 webview，说到这不得不提 android 和 ios 的区别，目的就是全局判断设备类型存入 state 在局部直接取到类型做判断。

这里其实可以通过 props 传递 data 实现，说到这里考虑到可能产生多级子组件，我每一层都需要 props 传递，又顾及到是全局属性（因为嵌入到哪个平台的页面就走那个平台的接口）

安装 vuex 的步骤就省了，在之前的文章介绍过了

在 src 下新建文件夹 vuex，进入 vuex 新建 store.js

然后去 main.js 加入

```js
import store from './vuex/store'
```

再修改 Vue 实例如下

```js
new Vue({
  el: '#app',
  router,
  store,
  render: h => h(App)
})
```

我们去新建的 store.js

```js
import Vue  from 'vue';
import Vuex from 'vuex';

Vue.use(Vuex)

export default new Vuex.Store({
  state: {
    platform: ''
  },
  mutations: {
    SET_APP(state, platform) {
      state.platform = platform;
    }
  },
  actions: {
    setApp({commit}, platform) {
      commit('SET_APP', platform);
    }
  },
  getters: {
    getApp: (state) => state.platform
  }
})
```

> 看到这里不急着往下进行，官方给的建议是 state、mutations、actions、getters 都分离成单独文件再引入到 store.js 中，这里只是为了提供一个简单粗暴的例子，把大部分流程都封装在一个文件里了，也方便修改测试。跟着上面工作流程图一步一步的走一遍，不难发现我们整个流程走下来只差组件分别调用 setApp 和 getApp 了。

说到组件先来看看完整的 App.vue

```html
<template>
  <div id="app">
    <div class="logo">
      <img src="./assets/logo.png">
    </div>
    <article-content></article-content>
    <share></share>
    <comment></comment>
  </div>
</template>

<script>
import comment from './component/Comment.vue';
import articleContent from './component/ArticleContent.vue';
import share from './component/Share.vue';

export default {
  data () {
    return {
    }
  },
  components: {
    articleContent,
    comment,
    share
  },
  mounted () {
    let u = navigator.userAgent;

    if ( u.indexOf('Android') > -1 || u.indexOf('Adr') > -1 ) {
      this.$store.dispatch('setApp', 'android');
    } else if ( !!u.match(/\(i[^;]+;( U;)? CPU.+Mac OS X/) ) {
      this.$store.dispatch('setApp', 'ios');
    }
  }
}
</script>

<style lang="sass">
  @import "/style/base.scss";
</style>
```

这里引入了三个组件，在 `mounted()` 里我们判断设备类型并发起 dispatch ，关键方法是 `this.$store.dispatch('setApp', ...args);`

简单易懂我们 dispatch 了一个 action（setApp），然后 commit 到 mutations（SET_APP），在 SET_APP 中修改了 state.platform

下面看看子组件是怎么获取,因为都是重复的用法所以只给一个 share 组件

```html
<template>
  <div class="article-share-block">
    <div class="divider-line" layout="row" layout-align="center center">
      <div class="left-line" flex></div>
      <p class="label">分享到</p>
      <div class="right-line" flex></div>
    </div>
    <div layout="row" layout-align="space-between center">
      <img class="share-icon" v-tap="{ methods: share, target: 'wechatTimeline' }" src="../img/wechat_timeline.png" alt="">
      <img class="share-icon" v-tap="{ methods: share, target: 'wechatFriend' }" src="../img/wechat_friend.png" alt="">
      <img class="share-icon" v-tap="{ methods: share, target: 'weibo' }" src="../img/weibo.png" alt="">
      <img class="share-icon" v-tap="{ methods: share, target: 'qq' }" src="../img/qq.png" alt="">
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      platform: ''
    }
  },
  mounted () {
    this.platform = this.$store.getters.getApp;
  },
  methods: {
    initIOS() {
      window.connectWebViewJavascriptBridge((bridge) => {
        this.webviewBridge = bridge;
      });
    },
    share(target) {
      if (this.platform === 'ios') {
        this.initIOS();
        this.webviewBridge.callHandler('invokeArticleShare', {
          shareTarget: target
        });
      } else if (this.platform === 'android') {
        window.idarex.invokeArticleShare(target);
      }
    }
  }
}
</script>

<style lang="sass">
  @import "../style/component/share.scss";
</style>
```

还是看关键语句 `this.platform = this.$store.getters.getApp;` ，这样我们可以取到 state 中的 platform 完成一个完整存取数据的循环。

> 说到这，不要急着把完整的组件粘贴运行，一定会抱错的，因为引入了第三方指令库（v-tap 实现移动端 tap 事件），这些都是次要的也没必要还原我的项目，整个原理和流程说的已经很清楚了，直接创建自己的组件跑一下就没什么问题了，大同小异。

## 总结

vuex 的原理其实简单易懂，本文通过一个小 demo 完成了一个简单流程，但真实项目里我不推荐这么做，最好把 store.js 模块化，方便后期维护，官方还提供了中间件等概念，准备在项目应用的可以研读源代码，从工程化的角度规划一下项目，当前 vue2 比较盛行，相信不久在 github 上会有大量的优秀项目供大家参考。
