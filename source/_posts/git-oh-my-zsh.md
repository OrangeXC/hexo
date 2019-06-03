---
title: Oh-My-Zsh 操作 Git 的快捷键
date: 2016-10-11 10:03:28
tags: Git
---

![](/uploads/git-oh-my-zsh1.png)

本篇文章介绍 Oh-My-Zsh 安装，以及 git 快捷键。也许你已经烦透了每天 `git add .` `git commit -m 'some fix'` `git push`...

如果操作变成这样 `ga .` `gcmsg 'some fix'` `gp`是不是很炫酷，好吧，对于已经使用的人没什么新鲜感。

接下来针对好奇的 Program development engineer（程序猿🐒），做详细介绍。

相信 git 大家并不陌生，不熟悉 git 操作的可以看我转载的一篇阮一峰大神的 [常用 Git 命令清单](http://orangexc.xyz/2016/05/25/List-of-commonly-used-Git-commands/)，重头戏 oh-my-zsh 大家可以看 [oh-my-zsh官网](http://ohmyz.sh/)。

<!--more-->

## Oh-My-Zsh

> Oh-My-Zsh is an open source, community-driven framework for managing your ZSH configuration. It comes bundled with a ton of helpful functions, helpers, plugins, themes, and a few things that make you shout...

简单说就是集成插件的命令行工具。针对 Mac 用户和 Linux 用户有很好的支持，windows 的朋友先不要急（有彩蛋）。

### 安装

Mac 用户和 Linux 用户通过在您的终端中运行以下命令之一来安装。可以安装通过 `curl` 或 `wget` 命令行。

via curl

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh)"
```

via wget

```bash
sh -c "$(wget https://raw.githubusercontent.com/robbyrussell/oh-my-zsh/master/tools/install.sh -O -)"
```

windows 我推荐一个集成 shell，git，oh-my-zsh 的命令行工具[Babun](http://babun.github.io/)。点击进入官网下载。当然官网还给出了其他炫酷功能大家可以详细阅读。

oh-my-zsh 有许多插件和主题大家可以去 `~/.zshrc` 配置

更多功能可以去官网阅读，这里强烈建议大家使用，因为会提高工作效率，对于追求完美开发体验的人帮助很大。

### 快捷键实现原理

这里只说 oh-my-zsh 针对 git 的改进。为什么支持缩写的形式。

大家可能知道 git 的快捷键 alias，它允许我们自己配制快捷键。例如

```bash
alias gst='git status'
alias gp='git push'
alias gp='git push'
```

需要配置的快捷键过多，如果对 git 操作了解不够深入的人无法自行配置。那么 oh-my-zsh 是如何实现的呢？

其实就是用的 `alias` ，这里是 Github 上的链接 https://github.com/robbyrussell/oh-my-zsh/blob/master/plugins/git/git.plugin.zsh ，通过 url 便知这里的`master`分支下，`plugins`插件目录，`git`插件下的配置文件。

喜欢阅读代码的直接点击链接查看所有快捷键实现。这里便于整理把平时工作常用的快捷键给大家，至于不常用的大家查看 Github 上的链接，可能记得多了这种对应关系容易弄混，建议大家只熟练记忆工作常有的快捷键，其它可以按着 git 官方文档去做。减少出错和不必要的麻烦。

## 常用快捷键

```
g - git
gst - git status
gl - git pull
gup - git pull --rebase
gp - git push
gd - git diff
gdc - git diff --cached
gdv - git diff -w "$@" | view
gc - git commit -v
gc! - git commit -v --amend
gca - git commit -v -a
gca! - git commit -v -a --amend
gcmsg - git commit -m
gco - git checkout
gcm - git checkout master
gr - git remote
grv - git remote -v
grmv - git remote rename
grrm - git remote remove
gsetr - git remote set-url
grup - git remote update
grbi - git rebase -i
grbc - git rebase --continue
grba - git rebase --abort
gb - git branch
gba - git branch -a
gcount - git shortlog -sn
gcl - git config --list
gcp - git cherry-pick
glg - git log --stat --max-count=10
glgg - git log --graph --max-count=10
glgga - git log --graph --decorate --all
glo - git log --oneline --decorate --color
glog - git log --oneline --decorate --color --graph
gss - git status -s
ga - git add
gm - git merge
grh - git reset HEAD
grhh - git reset HEAD --hard
gclean - git reset --hard && git clean -dfx
gwc - git whatchanged -p --abbrev-commit --pretty=medium
gsts - git stash show --text
gsta - git stash
gstp - git stash pop
gstd - git stash drop
ggpull - git pull origin $(current_branch)
ggpur - git pull --rebase origin $(current_branch)
ggpush - git push origin $(current_branch)
ggpnp - git pull origin $(current_branch) && git push origin $(current_branch)
glp - _git_log_prettily
```

就我而言已经足够了，当然也已经包括了 Github 上大部分快捷键。

## 总结

好的开发工具可以大大提升工作效率，应用适当即可，过多依赖快捷键自动补全之类的工具时间久了基础知识退化，导致没有好用的工具无法编程，建议新人熟记原始命令后再尝试快捷键，避免错误的发生（毕竟这是管理代码工具，操作不当很可能删除了最近 `add` 之前或者 `commit` 之前的代码快照，因为某些操作是不可逆的），亲身踩过坑删除了一天的工作，虽然重写两个小时就完成，但十分不爽。

哈哈，牢骚了一下，本人现在大四在校生，实习5个月，哪里写的不对还望大家指正。
