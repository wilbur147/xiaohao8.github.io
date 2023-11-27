---
index_img: https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210716200319.gif
title: 低配服务器福音，标星37K+Star开源项目Gogs秒搭Git服务
date: 2021-08-09 23:30:37
updated: 2021-08-09 23:30:37
categories:
  - 运维部署
comments: true
---
## 前言

这两天在研究自动化部署，虽然一直在用公司的自动化 `Jenkins + Gitlab`，不过用到现在不得不说还是很耗内存的，对于我自己学习买的1核2G轻量服务器来说负担真不是一般的大，所以了解到轻量级，功能也实用的搭配 `Gogs + Drone` 这俩家伙占的内存不是一般的小，用起来也是真的爽，就我这小小的服务器也是轻松带起，**安装方便**，**特别轻量级**，所以就推荐给大家！

**推荐点**

1. 响应时间短，平均100ms左右
2. 安装简单，功能简要够用（对于小团队，功能太多未必是好事，git版本库，问题管理，wiki，真的够了）
3. 稳定性没出过什么问题（使用了大半年）

> **[Gitea](https://gitea.io/zh-cn/)** 也不错，有兴趣的小伙伴也可以去了解一下，后期有时间也会出一篇它的功能介绍使用



## Gogs简介

Gogs 的目标是打造一个最简单、最快速和最轻松的方式搭建自助 Git 服务。使用 Go 语言开发使得 Gogs 能够通过独立的二进制分发，并且支持 Go 语言支持的 **所有平台**，包括 Linux、Mac OS X、Windows 以及 ARM 平台。Gogs对系统硬件要求极低，你直接可以在树莓派上搭建它。

**项目地址**：**[https://github.com/gogs/gogs](https://github.com/gogs/gogs)**

## Docker部署安装Gogs

Gogs我推荐直接用Docker环境下安装，因为很简单，只需要两个命令就行。

- 首先我们需要先下载Gogs的Docker镜像；

```bash
docker pull gogs/gogs
```

- 下载完成后使用`docker run`命令即可运行服务；

```bash
docker run -p 30022:22 -p 30080:3000 --name=gogs \
-v /www/wwwroot/data/docker/gogs:/data  \
-d gogs/gogs
```

- 这里我们说下命令中值得注意的地方，`30022`对应的是Gogs的SSH服务端口，`30080`对应的使用Gogs的HTTP服务端口，我们还将容器的数据目录挂载到了宿主机的`/mydata/gogs`目录下，这样就算我们重新创建容器数据也不会丢失。

> **温馨提醒：**
>
> 购买的腾讯云/阿里云等服务器的需要服务器网站上放开对应的 `30022、30080` 端口，否则是访问不了的
>
> 安装了宝塔面板的小伙伴记得也要放开对应的端口

## 配置

**Gogs数据存储在数据库中**，因为我们平时都有自己的mysql服务，如果小伙伴们没安装mysql也别慌，Gogs自带了 `SQLite3` 数据库，所以都是灵活选择的

1. 安装完成后，我们第一次访问Gogs服务会显示一个设置页面，访问地址：http://IP:30080/
2. 数据库设置，我这里设置 `Mysql` 数据库

![image-20210716154522164](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210716154522.png)

![image-20210716154712977](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210716154713.png)

![image-20210716160056935](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210716160057.png)

## 使用

前面安装的时候直接配置了一个账户，所以直接登录即可，如果没有配置账户会先注册，默认注册的第一个账户是管理员

- 安装完成后就会自动进入我们的控制面板

![image-20210716160506035](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210716160506.png)

### 一、创建仓库

- 直接点击右上角 `+` 号 即可创建新的仓库，简单设置下仓库名称和可见性来完成创建

![image-20210716161012751](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210716161012.png)

- 创建成功后我们就可以像Github和Gitlab一样上传我们的代码了

![image-20210716161243957](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210716161244.png).然后我们本地通过Git命令加入我们的代码直接提交、推送，在Gogs里面就可以看到我们提交的代码了。

![image-20210716163401034](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210716163401.png)



### 二、迁移外部仓库

- Gogs还提供了从外部仓库迁移代码的能力，通过右上角的`+`号，然后选择`迁移外部仓库`

![image-20210716163832471](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210716163832.png)

- 我以迁移 [Gitee](https://gitee.com/) 上 的 `jeecg-boot` 项目为例，地址 [https://gitee.com/jeecg/jeecg-boot](https://gitee.com/jeecg/jeecg-boot)

![image-20210716164904239](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210716164904.png)

- 可以看到已经成功迁移了外部仓库代码 `jeecg-boot` 

![image-20210716164945732](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210716164945.png)

### 三、 工单管理

- Gogs的工单管理类似 `issues` ,进入 `工单管理` 然后点击 `创建工单` 按钮来创建一个bug试试

- **首先要进入`标签管理`** 进行标签组初始化

![image-20210716165720700](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210716165720.png)

- 创建工单完成后显示效果如下。

![image-20210716165839492](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210716165839.png)



## 关于Gogs访问速度很慢的问题

- 这里我推荐用 `nginx` 配置代理，缓存静态文件来进行加速，有域名的小伙伴跟着做吧，确实IP访问这个速度慢是个痛点。

- 因为我的服务器是阿里云的，所以我们要先在域名控制台新增一个 gogs的子域名，其它服务器同样的道理，阿里域名解析地址： [https://dns.console.aliyun.com/](https://dns.console.aliyun.com/)

![image-20210809111507012](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210809111507.png)

- 点击确定就新增好了一个子域名
- 然后安装nginx并配置域名反向代理我们服务器本地的ip加端口
- 找到我们的nginx目录并进入nginx的conf文件夹，vi编辑nginx.conf，新增server代码

HTTP配置： [Nginx-Http配置点我](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/gogs-http-nginx.conf)

有证书HTTPS配置： [Nginx-SSL配置HTTPS点我](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/gogs-https-nginx .conf)

配置好了重启一下 nginx 我们直接域名访问gogs就行了，可以看到速度明显快多了

> **温馨提醒：**
>
> nginx配置好了记得清理一下浏览器缓存，不然可能无效

![GogsOper](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210716200319.gif)

## 使用内存

接下来我们看一下gogs占用的内存大小

- **docker镜像大小** 不超过 `100M`

![image-20210716170304585](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210716170304.png)

- **内存占用大小** 可以看到内存大约在 `72.5MB`这个是浮动的，占用了总内存（2G内存的服务器）的 `4%` 左右

![image-20210716172212092](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210716172212.png)





























