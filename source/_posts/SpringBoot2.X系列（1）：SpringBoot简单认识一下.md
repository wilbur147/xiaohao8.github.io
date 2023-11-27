---
index_img: https://static001.geekbang.org/infoq/0c/0c422e63f61599cc7be9da745d211dfa.jpeg
title: SpringBoot2.X系列（1）：SpringBoot简单认识一下
date: 2022-06-13 19:23:33
updated: 2022-06-13 19:23:33
tags:
  - Java
  - SpringBoot
categories:
  - 技术
comments: true
---

## 1.1 什么是Spring Boot

我们知道，从 2002 年开始，Spring 一直在飞速的发展，如今已经成为了在Java EE（Java Enterprise
Edition）开发中真正意义上的标准，但是随着技术的发展，Java EE使用 Spring 逐渐变得笨重起来，大
量的 XML 文件存在于项目之中。繁琐的配置，整合第三方框架的配置问题，导致了开发和部署效率的
降低。

2012 年 10 月，Mike Youngstrom 在 Spring jira 中创建了一个功能请求，要求在 Spring 框架中支持
无容器 Web 应用程序体系结构。他谈到了在主容器引导 spring 容器内配置 Web 容器服务。这是 jira
请求的摘录：

> 我认为 Spring 的 Web 应用体系结构可以大大简化，如果它提供了从上到下利用 Spring 组件和配
> 置模型的工具和参考体系结构。在简单的 main() 方法引导的 Spring 容器内嵌入和统一这些常用
> Web 容器服务的配置。

这一要求促使了 2013 年初开始的 Spring Boot 项目的研发，到今天，Spring Boot 的版本已经到了
2.0.3 RELEASE。Spring Boot 并不是用来替代 Spring 的解决方案，而是和 Spring 框架紧密结合用于
提升 Spring 开发者体验的工具。

它集成了大量常用的第三方库配置，Spring Boot应用中这些第三方库几乎可以是零配置的开箱即用
（out-of-the-box），大部分的 Spring Boot 应用都只需要非常少量的配置代码（基于 Java 的配置），
开发者能够更加专注于业务逻辑。



## 1.2 Spring Boot 特色

1. **使用简单**

   Spring Boot支持用注解的方式轻松实现类的定义与功能开发、无代码生成和XML配置， 新手
   入门极易上手

2. **配置简单**

   Spring Boot根据在类路径中的JAR和类自动配置Bean ，能自动完成大量配置。同时， 还支持用自定
   义的方式来配置

3. **提供大量Starter简化配置**

   Spring Boot提供了大量的Starter来简化依赖配置，例如我们要使用redis缓存，直接 `pom`文件调用 `spring-boot-starter-data-redis`就可以了，会自动加载依赖，并提供redis的API操作

4. **部署简单**

   Spring Boot可以在具备JRE(Java运行环境) 的环境中独立运行， 它内置了嵌入式的Tomcat、Jetty、Netty等Servlet(Server Applet) 容器， 项目不用被打包成WAR格式， 可以直接以JAR包的方式运行。

5. **与云计算天然集成**

   非常流行的微服务开发框架Spring Cloud也是基于Sping Boot实现的。

6. **监控简单**
   它提供了一整套的监控、管理应用程序状态的功能模块，包括监控应用程序的线程信息、内存
   信息、应用程序健康状态等。



## 1.3 Spring Boot支持的开发语言

Spring Boot支持使用基于Java虚拟机(Java Virtual Machine， JVM) 的Java、Groovy和Kotlin进行应用程序的开发(Spring Boot 1.5.2及以上版本) 。

> 其实 ，Scala 也能开发 Spring Boot 应用程序，因为它也基于只叫语言，只是目前官方没有提供支持 ，需要自己进行相应的配置 。

近年来，Spring Boot是整个Java社区中最有影响力的项目之一，常常被人看作是Java EE( Java Platform Enterprise Edition )开发的颠覆者，它将逐渐替代传统SSM (Java EE 互联网轻量级框架整合开发——Spring MVC+Spring+MyBatis )架构。





















