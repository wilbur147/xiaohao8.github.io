---
index_img: https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20231201172543159.png
title: Jenkins + git submodule自动编译
date: 2023-07-18 18:56:12
updated: 2023-07-18 18:56:12
tags:
  - Jenkins
categories:
  - 运维部署
comments: true
---
## 前言

上一篇文章介绍了如何搭建jenkins以及如何配置gitlab实现代码自动编译，但是出现了一个问题就是，如果我们的项目是使用的 git submodule方式进行开发，即多个仓库（一个主仓库和多个子仓库）这种方式进行自动化编译的难点就是拉取代码及检测代码提交。

## **git submodule 基本使用**

### git submodule 是什么？

git submodule 是用于多模块管理的工具，它允许一个项目作为 repository，其他项目作为子模块存在于在父项目中。

父项目和子项目的提交是分开的，也就是说父项目提交的信息只包含子项目的信息，而不会包含子项目的代码；子项目有自己独立的 commit，push，pull操作。

git submodule 一般用在比较大的项目中，为了便于复用，或者为了代码的安全性，常常需要分成若干个子项目来进行代码管理。

常用的指令包括：

- 添加子模块： `git submodule add`

- 更新子模块： `git submodule update`

- 初始化子模块： `git submodule init`
- 递归的方式克隆整个项目： `git clone－－recursive`

- 拉取所有子模块： `git submodule foreach git pull`

## Jenkins任务配置

任务创建还是和上一篇一样，参照 [Jenkins任务创建](Jenkins.md#Jenkins任务创建)



**在源码管理这里新增以下配置**

> 英文：Advanced sub-modules behaviours

![image-20231201172543159](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20231201172543159.png)

> **Use credentials from default remote of parent repository：**是指更新的子模块，需要使用和主模块一样的认证。