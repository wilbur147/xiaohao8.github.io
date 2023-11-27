---
title: 低配服务器福音，标星37K+Star开源项目Gogs秒搭Git服务
date: 2021-08-11 20:10:37
updated: 2021-08-11 20:10:37
categories:
  - 技术
comments: true
---

## 下载安装

**需要卸载干净本地MYSQL环境的请参考这一篇文章：[Win10卸载本地离线版mysql](https://www.weiye.link/297.html)**



废话不多说直接上链接

 [MySQL5.7.29官方下载地址 ](https://dev.mysql.com/downloads/mysql/5.7.html) 官方下载很慢。

这里分享一下下载地址加速下载：

链接：https://pan.baidu.com/s/1velvEZBf2tV6KAzC-k41gg 
提取码：fqeg

### 1、 配置 `my.ini`

首先解压mysql压缩包，在mysql-5.7.29-winx64目录下创建文件夹`data` （data创建一个空文件夹就行）以及 `my.ini`文件



![image-20210810100654859](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210810100654.png)



**my.ini文件内容如下：**

```ini
[mysqld]
#设置3306端
port = 3306
# MySQL程序安装目录
basedir=E:\mysql-5.7.29-winx64
# 数据库文件存放地址
datadir=E:\mysql-5.7.29-winx64\data 
#设置最大连接数
max_connections=512
#允许临时存放在查询缓存区大小
query_cache_size=0
#服务端使用的字符集默认为8比特编码的latin1字符集
character-set-server=utf8
# 数据库默认使用引擎
default-storage-engine=INNODB
[mysql]
# mysql客户端默认的字符集，5.7才有的，5.6以及之前的版本没有default-character-set属性
default-character-set=utf8
```

**basedir**:指的是MySQL的根目录；

**datadir**：值得是随后数据库的数据存放目录；

`注意，选项组[mysqld]、[mysql]不能漏掉了。`



### 2、配置path环境

打开系统变量的path 新建一个MySQL — bin的安装路径

![image-20210810101525225](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210810101525.png)



### 3. 安装mysql

以管理员身份运行CMD命令行，进入MySQL的安装的bin目录，比如`E:\mysql-5.7.29-winx64\bin`。注意一定要以管理员的身份运行，否则在安装过程中会出现因为管理权限不够而导致的Install/Remove of the Service Denied!（安装/卸载服务被拒绝）。

**1、输入 `.\mysqld -install` 命令安装**

若出现Service successfully installed，证明安装成功；如出现Install of the Service Denied，则说明没有以管理员权限来运行cmd.

**2、初始化数据库，获取管理员密码输入命令 `.\mysqld --initialize --user=mysql --console`**

注意下图红色框的就是初始化的管理员密码，这时候，mysql的安装目录下data文件夹已经有数据了。

​    这时候，mysql的安装目录下已经生产Data文件夹了。

**3、启动mysql，输入命令 `net start mysql`**



![image-20210810102858086](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210810102858.png)



### 4. 登录并修改密码

**1、使用初始密码登录，输入命令 `.\mysql -u root -p`**

密码请输入上图中红色方框部分，注意是你自己的。

![image-20210810103217475](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210810103217.png)



**2、 重置密码，修改成自己的密码**

`set password for root@localhost=password("你的密码"); ` **(别漏了分号)**

![image-20210810103405049](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210810103405.png)



### 5. 可视化工具 Navicat 连接

直接输入 ：

主机：`localhost` 

端口：`3306`

密码：刚才修改的密码，比如我的是 `123456`

![image-20210810103648525](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210810103648.png)



测试连接，显示连接成功就大功告成拉！！！！！！！！！！！



