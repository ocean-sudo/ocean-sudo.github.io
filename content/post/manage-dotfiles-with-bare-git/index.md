---
title: 用 Bare Git 管理 Dotfiles（精简实战版）
description: 用一个裸仓库按文件粒度管理 $HOME 下的 dotfiles，配置简单、同步清晰、提交可控。
date: 2026-02-25 00:00:00+0000
slug: manage-dotfiles-with-bare-git
image: cover.jpg
tags:
    - git
    - dotfiles
    - linux
categories:
    - DevOps
---

这篇是基于 Atlassian 教程 **How to Store Dotfiles - A Bare Git Repository** 的精简实践版。想深入理解细节，建议阅读原文。  
原文链接：https://www.atlassian.com/git/tutorials/dotfiles

<!--more-->

## Bare Git 是什么

`bare git` 可以简单理解为：

- 只有 Git 元数据（类似 `.git` 目录内容）
- 没有实际工作区文件

用它管理 dotfiles 时，本质上是把：

- Git 数据库存放在一个独立目录（例如 `~/.cfg`）
- 实际配置文件继续放在 `$HOME`

这样就实现了「Git 仓库」与「配置文件本体」分离。

## Bash 下四步完成初始化

```bash
# 1) 在 $HOME 下创建名为 .cfg 的裸仓库
git init --bare $HOME/.cfg

# 2) 创建 config 别名：
#    --git-dir 指向 Git 数据库
#    --work-tree 指向真实工作区（$HOME）
alias config='/usr/bin/git --git-dir=$HOME/.cfg/ --work-tree=$HOME'

# 3) 只对该仓库生效：不显示未跟踪文件
#    否则整个 $HOME 都会显示为未跟踪
config config --local status.showUntrackedFiles no

# 4) 写入 .bashrc，确保每次 shell 启动都可用
echo "alias config='/usr/bin/git --git-dir=$HOME/.cfg/ --work-tree=$HOME'" >> $HOME/.bashrc
```

> 如果你使用 fish，请将 alias 改为对应的 function 写法。

## 之后怎么用

从这一步开始，把 `config` 当作 `git` 来使用：

```bash
# 示例：跟踪一个配置文件并提交
config add ~/.bashrc
config commit -m "chore(dotfiles): add config alias for bare repo"
```

你可以继续像普通 Git 仓库一样使用：`config status`、`config log`、`config push`。

## 这样做的价值

- 不再需要「一个软件一个仓库」分散维护（kitty、fish、niri...）
- 多机同步更统一，可按分支区分设备或场景
- 主动 `add` 文件，避免 `add .` 带来的误跟踪
- 缓存、自动生成文件、插件目录等内容不易误提交
- 降低对复杂 `.gitignore` 的依赖

## 一句话总结

核心思想只有一句：**用一个裸 Git 仓库，以文件粒度管理你手动选择的 `$HOME` 配置文件。**
