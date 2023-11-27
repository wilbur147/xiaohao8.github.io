---
index_img: https://static001.geekbang.org/infoq/0c/0c422e63f61599cc7be9da745d211dfa.jpeg
title: SpringBoot2.X系列（3）：你要明白SpringBoot是怎样自动装置的
date: 2022-06-15 19:13:22
updated: 2022-06-15 19:13:22
tags:
  - Java
  - SpringBoot
categories:
  - 技术
comments: true
---

## 前言

我知道很多小伙伴可能都是从Spring转到SpringBoot的，明白之前整合`Spring + MyBatis + SpringMVC`我们需要写一大堆的配置文件，堪称配置文件地狱，而且随着项目的越来越庞大，配置文件也会越来越繁琐，这在一定程度上也给开发者带来了困扰，于是 `SpringBoot` 就应运而生了。

自从使用SpringBoot后，新建一个项目几乎不需要做任何改动，我们就可以运行起来。pom文件里，我们只需要引入一个`spring-boot-starter-web`就可以，之前我们所做的一切，SpringBoot都在底层帮我们做了。

本篇就来学习一下SpringBoot是如何做依赖管理以及自动配置的。

## 依赖管理

大家都知道SpringBoot项目，pom.xml文件都会给我们定义一个parent节点

![image-20210722144010502](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210722144010.png)

该节点指定了`version`版本号，所以在pom.xml文件里我们很多引入的jar都没有定义版本号，但这样也不会出错，因为SpringBoot帮我们为一些常用的jar包指定了版本号。

`ctrl + 鼠标左键`点击进入`spring-boot-starter-parent`这个jar包，会发现它的父项目是`spring-boot-dependencies`

![image-20210722144254221](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210722144254.png)



而在这个jar包里，就会发现声明了很多开发中常用jar的版本号



![image-20210722144332672](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210722144332.png)



看到这里也解惑了上一期 [SpringBoot系列（二）入门，快速构建项目](https://www.weiye.link/229.html/)中我没有指定版本也不会报错的问题。



所以在pom.xml文件中引入jar的时候，如果该jar在`spring-boot-dependencies`中定义了版本号，那么你可以不写。如果你想使用其他的版本号，那么也可以在pom.xml中定义version，遵循就近原则。比如我现在想自己从新定义 `kafka` 的版本号，我就可以添加依赖时，也添加上版本号。

```xml
<dependency>
    <groupId>org.apache.kafka</groupId>
    <artifactId>kafka-clients</artifactId>
    <version>2.7.1</version>
</dependency>
```

### Web项目启动详解

我们日常使用SpringBoot开发web项目时，简单的配置就是只导入了 `spring-boot-starter-web` 这 一个依赖就可以写接口并且进行访问，因为在这个starter中整合了我们之前写Spring项目时引入的`spring-aop`、`spring-context`、`spring-webmvc`等jar包，包括tomcat，所以SpringBoot项目不需要外部的tomcat，只需要启动application类使用内置的tomcat服务器即可。

![image-20210722151305413](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210722151305.png)



在SpringBoot项目中，根据官方文档，有各种场景的`spring-boot-starter-*`可以使用，只要引入了starter，这个场景所有常规需要的依赖就会自动引入。

所以我们在启动项目时启动器最底层的依赖就是`spring-boot-starter`，该jar包是核心启动包，包含了自动配置的支持，日志以及YAML。Core starter, including auto-configuration support, logging and YAML，来自官方描述。

在上图中我们看到有 `spring-boot-autoconfigure` 依赖，这个就关系到SpringBoot自动配置功能。

## 自动装配

### 一、SpringApplication.run()

首先我们要确认项目的启动入口就是通过 `SpringApplication.run()`来启动的，所以我们就先从这里入手一步步来解析是如何自动装载的。

![image-20210722154640893](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210722154640.png)



我们进入到`SpringApplication`这个类中看一下`run()`方法的核心实现，都注释了看解释，要重点关注红框两行代码

![image-20210722165939368](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210722165939.png)

`SpringApplication.run()`方法中，我把关键点用序号标识出来了。

1. **第一个就是创建ApplicationContext容器。**
2. **第二个是刷新ApplicationContext容器。**

在创建ApplicationContext时，会根据用户是否明确设置了`ApplicationContextClass`类型以及初始化阶段的推断结果，决定为当前SpringBoot应用创建什么类型的ApplicationContext。

![image-20210722170414024](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210722170414.png)

下面开始初始化各种插件在异常失败后给出的提示。
然后执行准备刷新上下文的一些操作。其实`prepareContext()`方法也是非常关键的，它起到了一个承上启下的作用。下面我们来看一下`prepareContext()`方法里面具体执行了什么。

关键的地方我也标注出来了，主要就是`getAllSoures()`方法，这个方法中，获取到的一个source就是启动类`LearnApplication`。

![image-20210722171225508](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210722171225.png)

这样就通过获取这个启动类就可以在后load()方法中去加载这个启动类到容器中。

然后，后面再通过`listeners.contextLoaded(context)`;
将所有监听器加载到ApplicationContext容器中。

最后就是我们上面说的核心的第二部刷新ApplicationContext容器操作，如果没有这一步操作上面的内容也都白做的，通过`SpringApplication的refreshContext(context)`方法完成最后一道工序将启动类上的注解配置，刷新到当前运行的容器环境中。

### 二、启动类的注解

在启动类上有一个 `@SpringBootApplication` 注解，这一个注解帮我们启动了SpringBoot项目，其背后默认帮我们配置了很多自动配置类。我们点进去看一下 **@SpringBootApplication** 这个注解。

#### @SpringBootApplication

![image-20210722172007417](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210722172007.png)

解说一下三个注解：

- @SpringBootConfiguration : Spring Boot的配置类，标注在某个类上，表示这是一个Spring Boot的配置类（等于xml方式下.xml文件）。
- @EnableAutoConfiguration: 开启自动配置类，SpringBoot的精华所在。
- @ComponentScan：包扫描，等同于xml下开启并设置包扫描路径。

#### @EnableAutoConfiguration

> @EnableAutoConfiguration：告诉SpringBoot开启自动配置功能，这样自动配置才能生效。

![image-20210722172117609](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210722172117.png)

- @AutoConfigurationPackage：自动配置包

- @Import: 导入自动配置的组件

- - @Import：通常用于有时没有把某个类注入到IOC容器中，但在运用的时候需要获取该类对应的bean，此时就需要用到`@Import`注解。加入IOC容器的方式有很多种，当然@Bean注解也可以，但是@Import注解快速导入的方式更加便捷。

#### @AutoConfigurationPackage

![image-20210722172216698](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210722172216.png)

它其实是注册了一个Bean的定义，返回了当前主程序类的同级以及子级的包组件。如果你把类写在了APP类的上级时SpringBoot便不会帮你加载，报出如404的问题，这也就是为什么，我们要把App放在项目的最高级中。

- SpringApplication.run(App.class, args);为什么要传入主启动类？

- - 其一方面就是要根据该类去解析他所在包，并实现对同级和下级类的扫描。

#### @Import

![image-20210722172731421](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210722172731.png)

到这就到自动装配的核心了，离成功不远了，咱们直接看重点，通过继承 `**ImportSelector**`重写`selectImports`方法。

该方法奠定了SpringBoot自动批量装配的核心功能逻辑。

![image-20210722172907576](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210722172907.png)

可以看到`getCandidateConfigurations()`方法，它其实是去加载

**FACTORIES_RESOURCE_LOCATION="META-INF/spring.factories"** 外部文件。这个外部文件里面，默认有很多自动配置的类，这些类的定义信息将会被SpringBoot批量的加载到Bean定义Map中等待被创建实例。如下：

![image-20210722174257169](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210722174257.png)

其实通过查阅资料发现，这就是一种自定义SPI的实现方式的功能。
那么我们以第一个配置类：
`org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration`来看一下，这些类都是如果实现的。
打开`org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration`的源码：

![image-20210722174350549](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210722174350.png)

我们看到这个类有三个注解`@Configuration`、`@AutoConfigureAfter`、`@ConditionalOnProperty`、因为有`@Configuration`注解所以它也是一个配置类，然后第二注解中的参数类`JmxAutoConfiguration.class`进入之后是这样的：

![image-20210722174447334](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210722174447.png)

也是存在`@ConditionalOnProperty`注解的。那看来关键点就是`@ConditionalOnProperty`这个注解了。
这个注解其实是一个条件判断注解，这个条件注解后面的参数的意思是当存在系统属性前缀为`spring.application.admin`，并且属性名称为`enabled`，并且值为`true`时，才加载当前这个Bean并进行实例化。

这种spring4.0后面出现的的条件注解，可以极大的增加了框架的灵活性和扩展性，可以保证很多组件可以通过后期配置，而且阅读源码的人，通过这些注解就能明白在什么情况下才会实例化当前Bean。

后面还有不少这种条件注解呢：

- @ConditionalOnBean：当容器里有指定Bean的条件下
- @ConditionalOnClass：当类路径下有指定的类的条件下
- @ConditionalOnExpression：基于SpEL表达式为true的时候作为判断条件才去实例化
- @ConditionalOnJava：基于JVM版本作为判断条件
- @ConditionalOnJndi：在JNDI存在的条件下查找指定的位置
- @ConditionalOnMissingBean：当容器里没有指定Bean的情况下
- @ConditionalOnMissingClass：当容器里没有指定类的情况下
- @ConditionalOnWebApplication：当前项目时Web项目的条件下
- @ConditionalOnNotWebApplication：当前项目不是Web项目的条件下
- @ConditionalOnProperty：指定的属性是否有指定的值
- @ConditionalOnResource：类路径是否有指定的值
- @ConditionalOnOnSingleCandidate：当指定Bean在容器中只有一个，或者有多个但是指定首选的Bean

这些注解其实都是通过`@Conditional`注解扩展而来的，只是使用了不同的组合条件来判断是否需要加载和初始化当前Bean。

## 总结

**springboot启动时，是依靠启动类的main方法来进行启动的，而main方法中执行的是`SpringApplication.run()`方法，而`SpringApplication.run()`方法中会创建spring的容器，并且刷新容器。而在刷新容器的时候就会去解析启动类，然后就会去解析启动类上的`@SpringBootApplication`注解**，**而这个注解是个复合注解，这个注解中有一个`@EnableAutoConfiguration`注解，这个注解就是开启自动配置，当我们使用`@EnableAutoConfiguration`注解激活自动装配时，实质对应着很多`XXXAutoConfiguration`类在执行装配工作，这些`XXXAutoConfiguration`类是在spring-boot-autoconfigure jar中的META-INF/spring.factories文件中配置好的，`@EnableAutoConfiguration`通过SpringFactoriesLoader机制创建`XXXAutoConfiguration`这些bean。`XXXAutoConfiguration`的bean是配置了符合这些条件注解的配置来进行装载的**。

以上就是今天分享的，关于**SpringBoot自动装配原理**。

## 参考资料

**SpringBoot配置项**:https://docs.spring.io/spring-boot/docs/2.1.0.RELEASE/reference/htmlsingle/#common-application-properties

















