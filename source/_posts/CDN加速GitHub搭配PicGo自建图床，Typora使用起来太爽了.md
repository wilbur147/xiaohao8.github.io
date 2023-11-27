---
index_img: https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210714114128.png
title: CDN加速GitHub搭配PicGo自建图床，Typora使用起来太爽了
date: 2021-07-14 23:50:37
updated: 2021-07-14 23:50:37
categories:
  - 技术
comments: true
---



## 前言

之前我一直使用的是 `七牛云 + PicGo` 搭配使用图床，平时用Typora写文章也习惯了，上传图片这些也是经常的事，后面七牛云域名回收，我之前的图床也就没有什么用了，好多图片都无法展示，还是挺气的，不过也不气馁，还好每次写的文章都有备份，思考过自己搭建服务，不过成本还是挺高的， `域名 + 服务器` 一年也是好几百，对于我这种不靠写字谋生的人而言也是没有必要的，今天也用上了 `GitHub + JsDelivr` 免费的CDN加速图床，使用起来也是很爽的，就目前来看图床是不用担心用不上了。所以跟大家分享一下。

> - 适用环境：Windows + Mac + Linux
> - 需要工具：GitHub 账号 + PicGo 客户端
> - 稳定性：背靠 GitHub 和微软，比自建服务器都稳
> - 隐私性：这算是唯一缺点，你的图片别人可以访问

## 搭建教程

### 1、 GitHub仓库设置

> 流程：新建 public 仓库 -> 创建 token -> 复制 token 备用

#### 1.1 仓库新建

点击 github 主页右上角的 `+` 创建 `New repository`

![image-20210714111759759](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210714111759.png)

填写仓库信息，仓库得设置为 `Public` 因为后面通过客户端访问算是外部访问，因此无法访问 `Private` ，这样的话图片传上来之后只能存储不能显示。所以要设置为 `Public`，填写好后直接点击 `Create repository` 就行了

![image-20210714111925490](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210714111925.png)

#### 1.2 创建并复制Token

此时仓库已经建立，点击右上角头像，然后进入`Settings`

![image-20210714112149784](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210714112149.png)

找到 `Developer settings` ，并进入 `Personal access tokens`  创建 token；

![image-20210714112508450](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210714112508.png)

填 Note，勾选复选框 repo ，接着到页面底部 `Generate token` 就完成了；

![image-20210714112715472](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210714112715.png)



然后复制生成一串字符 token，这个 token 只出现一次，所以要保存一下。

![image-20210714113036301](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210714113036.png)

### 2、PicGo 客户端安装配置

#### 2.1 下载&安装

PicGo （目前 2.2.2）是一个开源的图床工具，非常优秀。可以到 GitHub 上下载，但下载速度太慢，所以我放了一个百度云的链接，速度快很多。

GitHub地址：[PicGo](https://github.com/Molunerfinn/PicGo)

Windows版下载链接：[百度云](https://pan.baidu.com/s/1R3dypZFJUmoNzBYtKowWUw) 密码：okzv

#### 2.2 配置

直接按照图中示例填写即可

> **需要注意的点：**
>
> 1. 仓库名称是你的用户名/仓库名组合
> 2. 现在GitHub默认主分支是 `main` 
> 3. 文件夹可以看你心情写，一般都是 `img` 
> 4. **CDN加速使用的是JsDeliver** 直接填写 `https://cdn.jsdelivr.net/gh/git用户名/仓库名` 即可

![image-20210714114128873](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210714114128.png)



然后点击确认就可以了

#### 2.3 PicGo测试上传

![GicGoOper1](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210714124741.gif)

可以看到PicGo已经可以成功上传并且可以在浏览器展示了

### 3、Typora搭配PicGo实现上传图片

#### 3.1 下载&安装

Typora（目前 0.10.11）是一款由Abner Lee开发的轻量级Markdown编辑器，适用于OS X、Windows和Linux三种操作系统，是一款免费软件。与其他Markdown编辑器不同的是，Typora没有采用源代码和预览双栏显示的方式，而是采用所见即所得的编辑方式，实现了即时预览的功能，但也可切换至源代码编辑模式。

下载地址：[Typora](https://typora.io/windows/typora-setup-x64.exe)

#### 3.2 配置

启动程序，`文件`-`偏好设置`，插入图片时选择`上传图片`，上传服务选择`PicGo(app)`，并输入PicGo的安装路径

![image-20210714130959850](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210714131113.png)

![image-20210714131205567](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210714131205.png)

设置好后，可以点击验证图片上传选项

![image-20210714131734373](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210714131734.png)

#### 3.3 复制粘贴图片上传测试

将图片复制进文章后，会显示菜单，选择上传图片，显示`Uploading...`，稍等片刻上传完成后，图片路径就会自动转为`GitHub`图床的图片地址，无需其他操作。

![image-20210714131800641](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210714131800.png)



> **提示：**
>
> 对于安装在系统盘的Typora，在粘贴和上传图片时会没有反应或者报错，是因为系统盘的读写权限问题造成，建议以管理员身份运行程序，或将Typora安装至非系统目录中。

综上已经操作完成，操作还是很简单的就是唯一不足的地方就是不能私人，所有人都能在仓库里看到，不过应该还是没人会闲着没事翻图片来看。。。



