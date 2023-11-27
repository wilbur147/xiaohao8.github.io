---
index_img: https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210901163627.png
title: Windows电脑反编译微信小程序含分包详细操作
date: 2021-09-01 21:08:13
updated: 2021-09-01 21:08:13
categories:
  - 爱搞爱折腾
comments: true
---

现在网上也有很多关于小程序反编译的教程，随时间的流逝或许随着微信的更新，有出现编译不成功的现象。

本篇文章总结一下最新的编译过程，已成功获得小程序源码（有分包的小程序）



## 环境准备

### 1、 node 环境准备

下载链接：[https://nodejs.org/en/](https://nodejs.org/en/)

![image-20210901154759757](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210901154800.png)



安装后将nodejs设置为环境变量。
打开cmd，测试是否安装成功：在命令行输入node -v 出现版本号说明已经安装成功。



### 2、反编译工具

项目地址来自于： [https://github.com/xuedingmiaojun/wxappUnpacker](https://github.com/xuedingmiaojun/wxappUnpacker) 



**通过下面链接下载：**

链接：[https://pan.baidu.com/s/1p-wnX-mXr-Du0iJK_dT8RQ](https://pan.baidu.com/s/1p-wnX-mXr-Du0iJK_dT8RQ ) 
提取码：z06a

**下载下来解压到某个位置就可以了，一定要通过网盘下载，里面有解密包的工具和安装后的npm环境，直接使用即可**



## 具体操作

### 1. 微信PC获取小程序

**在通过微信PC打开小程序前，我们最好先找到缓存到本地的小程序包路径，一般都是在 `微信PC安装目录\WeChat Files\WeChat Files\Applet` **

比如我的就是安装到 `D盘根目录的`，所以路径为： `D:\WeChat\WeChat Files\WeChat Files\Applet`

![image-20210901160649022](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210901160649.png)



**上图中每个文件夹代表一个小程序，一般最新打开的小程序都是在第一个，如果不确定可以排序一下修改日期**



找到路径了我们就可以用微信PC打开小程序了，打开后就会发现当前目录新增了一个文件夹，里面存放的就是加密后的小程序包

![image-20210901161153138](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210901161153.png)



### 2. 解密包

刚获取到的包我们还不能进行反编译，必须要通过 `解密软件` 修改一下才能反编译

![image-20210901161556010](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210901161556.png) 





**本篇就演示一个主包和一个分包反编译的过程就可以了，先通过`解密软件`修改一下主包**

![image-20210901161936026](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210901161936.png)



**解密的主包自动到 `wxpack` 这个包里面来了，同样的步骤解密一个分包，下图是我解密好的两个，并且修改了一下名称，好区分**

![image-20210901162219406](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210901162219.png)





### 3. 反编译

进入 `wxpack` 的同级目录 `wxappUnpacker-master`，在路径栏输入 `cmd` 自动打开当前目录的 **命令窗口了**

![image-20210901162538413](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210901162538.png)



**先反编译一下主包，把反编译后的文件夹放到 `wxpack` 同级目录中 **

```bash
node wuWxapkg.js ..\wxpack\master-app.wxapkg
```



![image-20210901163011475](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210901163011.png)





**再反编译分包，把反编译后的文件夹放到 `wxpack` 同级目录中**

```bash
node wuWxapkg.js -s=..\ ..\wxpack\_pages_app.wxapkg
```

- `-s` 表示分包
- 第一个`..\` 表示输出位置
- `..\wxpack\_pages_app.wxapkg` 需要反编译的分包位置



![image-20210901163627239](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210901163627.png)





好了剩下的就是自己组合一下包的架构目录了~~~~



如果本篇文章给予了您一点帮助，还请点个赞收藏一下~~

谢谢您的支持！！！











