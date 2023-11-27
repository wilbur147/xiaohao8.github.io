---
title: 低配服务器福音，标星37K+Star开源项目Gogs秒搭Git服务
date: 2021-08-10 22:30:37
updated: 2021-08-10 22:30:37
categories:
  - 技术
comments: true
---

## 卸载mysql

### 1. 管理员身份运行cmd窗口

- 在窗口中输入 `sc query mysql` 如果出现这个代表之前有

![image-20210810095633189](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210810095633.png)

- 输入`sc delete mysql`卸载MySQL服务

![image-20210810095701864](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210810095701.png)



### 2. 控制面板卸载mysql

打开控制面板卸载页查看是否有 `Mysql` 有的话删除

### 3. 注册表删除

`win+r`输入`regedit`打开注册表 分别在下面几个目录下找到删除mysql目录

`计算机\HKEY_LOCAL_MACHINE\SYSTEM\ControlSet001\Services\EventLog\Application\MySQL`

`计算机\HKEY_LOCAL_MACHINE\SYSTEM\ControlSet002\Services\EventLog\Application\MySQL`

`计算机\HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\Services\EventLog\Application\MySQL`



**然后删除自己安装MySQL的文件夹就可以了**

