---
title: Node 版本管理
date: 2017-06-06 17:49:56
tags: Node
---

![](/uploads/node-version-management1.jpg)

<!--more-->

node 已经成为每个前端必备的技能，就算没研究过 node 的运行机制，也会用到依赖 node 运行的包管理器 npm。

近日 node 发布新版本 8.0，npm 也升级到了 5.0，加了 lock file，社区里关于有没有必要继续使用 yarn 管理工具争论不休，我认为静观其变，待 npm 5 逐渐稳定后再转过去也不迟，目前 yarn 还是比较靠谱的替代方案。

当然本文要讲的不是 node 也不是 npm，但又离不开这两者。

当 node 发布新版本时，每个关注 node 的开发都会安装下新版本尝尝鲜，升级新版本会替换旧版本，典型例子使用 Homebrew 管理软件，当 `upgrade node` 时，node 的确更新了，但是旧的不见了。

因为 node 升级版本也是遵循版本升级原则，版本号第一位升级代表可能会不兼容之前的版本（删除修改某些 api）。

之前的旧项目可能因为升级跑不起来了，这时候就有多个版本的 node 共存的需求。

Github 上开源的比较好用的有 nvm 和 n，下面分别介绍两者。

## nvm

Github 地址：https://github.com/creationix/nvm

nvm 并不支持 windows，不过已经有其它解决方案了，[nvm-windows](https://github.com/coreybutler/nvm-windows) 和 [nodist](https://github.com/marcelklehr/nodist)

基本安装：

使用 cURL:

```bash
curl -o- https://raw.githubusercontent.com/creationix/nvm/v0.33.2/install.sh | bash
```

或者 Wget:

```bash
wget -qO- https://raw.githubusercontent.com/creationix/nvm/v0.33.2/install.sh | bash
```

安装的注意事项可以去 github 上查看。

下面说下简单的用法：

* `nvm install node` 安装 node latest
* `nvm install --version` 安装指定版本
* `nvm use node` 在任何新的 shell 只是使用已安装的版本
* `nvm use --version` 在任何新的 shell 只是使用指定版本
* `nvm run node --version` 运行指定版本
* `nvm ls` 查看已安装的版本
* `nvm ls-remote` 查看可安装版本

以上几条是常用的命令，可以解决 node 版本管理的需求。

## n

Github 地址：https://github.com/tj/n

基本安装：

```bash
npm install -g n
```

基本用法：

* `n <version>` 安装指定版本，如果指定版本已经安装那么会启动此版本
* `n` 获取版本列表，上下可以移动选择版本，enter选择版本，^C 退出
* `n latest` 安装 lts 版本
* `n stable` 安装或运行稳定版本
* `n lts` 安装或运行 lts 版本
* `n rm 0.9.4 v0.10.0` 移除某些版本，或者简写为 `n - 0.9.4`
* `n prune` 删除非当前版本的其它所有版本

以上是 n 的简单使用

## 总结

本篇主要是工具篇，简单介绍下两种工具，具体大家可以去 GitHub 查看，在简洁程度上我比较喜欢 n 这个工具，大家可以都尝试尝试，重点在于解决版本切换问题。
