---
index_img: https://static001.geekbang.org/infoq/0c/0c422e63f61599cc7be9da745d211dfa.jpeg
title: SpringBoot2.X系列（2）：快速构建项目
date: 2022-06-14 18:51:45
updated: 2022-06-14 18:51:45
tags:
  - Java
  - SpringBoot
categories:
  - 技术
comments: true
---
## 前言

- 随着动态语言的流行（Ruby、Scala、Node.js）, Java的开发显得格外的笨重；繁多的配置、低下的开发效
  率、复杂的部署流程以及第三方技术整合难度大。

- 在上述环境下，Spring Boot由此诞生，Spring Boot是由Pivotal团队提供的全新框架，其设计目的是用来简化新Spring应用的初始搭建以及开发过程。该框架使用了特定的方式来进行配置，从而使开发人员不再需要定义样板化的配置。用我的话来理解，就是spring boot其实不是什么新的框架，它默认配置了很多框架的使用方式，就像maven整合了所有的jar包，spring boot整合了所有的框架。

## SpringBoot优点

- 快速构建独立运行的Spring项目。
- 无须依赖外部Servlet容器，应用无需打成WAR包；项目可以打成jar包独自运行。
- 提供 一系列 starter pom 来简化 Maven 的依赖加载。
- 大量的自动配置，对主流开发框架的无配置集成。
- 无须配置XML，开箱即用，简化开发，同时也可以修改默认值来满足特定的需求。
- Spring Boot 并不是对 Spring 功能上的增强，而是提供了一种快速使用 Spring 的方式。
- 极大提高了开发、部署效率。

## 快速入门

### 一、环境

| jdk1.8        | Spring Boot 推荐jdk1.8及以上                   |
| ------------- | ---------------------------------------------- |
| Maven 3.x     | maven 3.0 以上版本（本篇 apache-maven-3.8.1）  |
| IntelliJ IDEA | IntelliJ IDEA 2018版左右 （本篇 2020.1.1 x64） |
| SpringBoot    | 最新版本 （本篇 2.5.2）                        |

### 二、快速创建SpringBoot项目

在 IDEA上新建一个项目 `File`  => `New`  => `Project` 选择 `Spring Initializr` 

![image-20210721105420407](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210721105420.png)



点击 **Next** 进入下一步，设置项目工程



![image-20210721105726899](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210721105727.png)



选择项目所需要的依赖项，选择合适的，这里也可以不用选，我们后面在 `Pom`文件手动添加依赖也可以



![image-20210721110117394](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210721110117.png)



然后选择项目名称、项目存储位置等，点击 `Finish` 创建即可



![image-20210721110605531](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210721110605.png)



遇到设置的目录不存在时，创建会弹出提示框表示是否创建同目录，点击 **OK** 即可



![image-20210721110720133](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210721110720.png)



### 三、了解项目目录结构

项目创建完成后，我们来解读一下目录结构



![image-20210721111806809](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210721111807.png)

- `src/main/java` 入口(启动) 类及程序的开发目录。在这个目录下进行业务开发、创建实体层、控制器层、数据连接层等；
- `resources` 文件夹中目录结构：
  - `static` ：保存所有的静态资文件， **js css images**
  - `templates` ：保存所有的模板页面（Spring Boot默认jar包使用嵌入式的Tomcat，默认不支持JSP页
    面），可以使用模板引擎（freemarker、thymeleaf）；
  - `application.properties` ：Spring Boot应用的配置文件；可以修改一些默认设置；
    如修改默认端口 `server.prot=8081`



创建控制器 HelloController



![image-20210721144316793](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210721144316.png)



要确保有 启动入口类 `Application` 一般IDEA创建springBoot项目时会自动创建，如果是创建maven项目要自己创建



![image-20210721144457168](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210721144457.png)



**运行效果**

- 在IDEA界面上方点击 `右三角` 按钮即可启动项目

![image-20210721144844693](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210721144844.png)



可以看到启动日志有 SpringBoot自带 logo，因为 SrpingBoot 内置了 tomcat，所以我们直接启动就行了，默认是 **8080** 端口， 出现了 `Started **Application` 字样时说明已经成功启动了，接下来我们打开浏览器访问一下



![image-20210721145031035](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210721145031.png)



在浏览器地址栏输入 `localhost:8080/hello` 即可看到运行结果



![image-20210721145400879](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210721145400.png)

这样一个简单的 `SpringBoot` 已经创建完成了，接下来我们再来一个简单的部署，需要打包成jar包



### 四、简单部署

首先要确认 `POM` 文件里面是否有 maven 编译插件

![image-20210721151615032](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210721151615.png)



1. 然后我们可以通过 命令行 `mvn package` 进行打包

![image-20210721151829276](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210721151829.png)

2. 也可以通过 `idea` 右侧 有个 `maven` 组件 打开， 点击 `package` 也可以进行打包

![image-20210721154437634](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210721154437.png)

可以看到已经成功打包，并且多了一个 `target` 文件夹，里面就有我们打包好的`jar包`

![image-20210721151954056](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210721151954.png)



然后就可以进入目录，通过 `java -jar` 命令启动jar包了

![image-20210721152225690](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210721152225.png)

> 在 Linux 服务器中我们简单的可以通过本地打包成 jar包 然后上传到linux服务器的某一个目录位置，确定安装了 jdk环境，然后直接通过 `java -jar xxx.jar > /dev/null 2>&1 &` 命令就可以启动了

## 源码

[https://github.com/wilbur147/xiangStudy/tree/main/lab-springBoot/SpringBoot-2](https://github.com/wilbur147/xiangStudy/tree/main/lab-springBoot/SpringBoot-2)

