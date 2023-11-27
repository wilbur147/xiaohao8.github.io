---
index_img: https://gitee.com/wilburplus/image-cdn/raw/master/blog/202201211209641.webp
title: 一个基于SpringBoot2+Vue+Redis的企业级聚合支付系统
date: 2022-01-21 22:30:37
updated: 2022-01-21 22:30:37
categories:
  - 开源精选
comments: true
---
**项目介绍**



Jeepay是一套适合互联网企业使用的开源支付系统，支持多渠道服务商和普通商户模式。已对接微信支付，支付宝，云闪付官方接口，支持聚合码支付。



Jeepay使用Spring Boot和Ant Design Vue开发，集成Spring Security实现权限管理功能，是一套非常实用的web开发框架。



### **项目特点**



- 支持多渠道对接，支付网关自动路由
- 已对接微信服务商和普通商户接口，支持V2和V3接口
- 已对接支付宝服务商和普通商户接口，支持RSA和RSA2签名
- 已对接云闪付服务商接口，可选择多家支付机构
- 提供http形式接口，提供各语言的sdk实现，方便对接
- 接口请求和响应数据采用签名机制，保证交易安全可靠
- 系统安全，支持分布式部署，高并发
- 管理端包括运营平台和商户系统
- 管理平台操作界面简洁、易用
- 支付平台到商户系统的订单通知使用MQ实现，保证了高可用，消息可达
- 支付渠道的接口参数配置界面自动化生成
- 使用spring security实现权限管理
- 前后端分离架构，方便二次开发
- 由原XxPay团队开发，有着多年支付系统开发经验

## 

## **系统架构**



> Jeepay计全支付系统架构图

![图片](https://gitee.com/wilburplus/image-cdn/raw/master/blog/202201211209641.webp)

> 核心技术栈

| 软件名称       | 描述                              | 版本                             |
| :------------- | :-------------------------------- | :------------------------------- |
| Jdk            | Java环境                          | 1.8                              |
| Spring Boot    | 开发框架                          | 2.4.5                            |
| Redis          | 分布式缓存                        | 3.2.8 或 高版本                  |
| MySQL          | 数据库                            | 5.7.X 或 8.0 高版本              |
| MQ             | 消息中间件                        | ActiveMQ 或 RabbitMQ 或 RocketMQ |
| Ant Design Vue | Ant Design的Vue实现，前端开发使用 | 2.1.2                            |
| MyBatis-Plus   | MyBatis增强工具                   | 3.4.2                            |
| WxJava         | 微信开发Java SDK                  | 4.1.0                            |
| Hutool         | Java工具类库                      | 5.6.6                            |

> 项目结构
>
> ```bash
> jeepay
> ├── conf -- 存放系统部署使用的.yml文件
> └── docs -- 存放项目相关文档说明
>      ├── script -- 项目启动shell脚本
>      └── sql -- 初始化sql文件
> ├── jeepay-core -- 核心依赖包
> ├── jeepay-manager -- 运营平台服务端[9217]
> ├── jeepay-merchant -- 商户系统服务端[9218]
> ├── jeepay-payment -- 支付网关[9216]
> ├── jeepay-service -- 业务层代码
> └── jeepay-z-codegen -- mybatis代码生成
> ```
>
> 

> 开发部署

- 系统开发：https://docs.jeequan.com/docs/jeepay/dev_serv
- 通道对接：https://docs.jeequan.com/docs/jeepay/dev_channel
- 线上部署：https://docs.jeequan.com/docs/jeepay/deploy
- 接口文档：https://docs.jeequan.com/docs/jeepay/payment_api



## **功能模块**



> Jeepay运营平台功能

![图片](https://gitee.com/wilburplus/image-cdn/raw/master/blog/202201211209076.webp)

> Jeepay商户系统功能

![图片](https://gitee.com/wilburplus/image-cdn/raw/master/blog/202201211209544.webp)

## 

## **系统截图**





![图片](https://gitee.com/wilburplus/image-cdn/raw/master/blog/202201211209114.webp)



![图片](https://gitee.com/wilburplus/image-cdn/raw/master/blog/202201211209941.webp)



![图片](https://gitee.com/wilburplus/image-cdn/raw/master/blog/202201211209408.webp)



![图片](https://gitee.com/wilburplus/image-cdn/raw/master/blog/202201211209708.webp)



![图片](https://gitee.com/wilburplus/image-cdn/raw/master/blog/202201211209880.webp)



![图片](https://gitee.com/wilburplus/image-cdn/raw/master/blog/202201211209368.webp)



![图片](https://gitee.com/wilburplus/image-cdn/raw/master/blog/202201211209405.webp)



![图片](https://gitee.com/wilburplus/image-cdn/raw/master/blog/202201211209159.webp)



![图片](https://gitee.com/wilburplus/image-cdn/raw/master/blog/202201211209234.webp)



![图片](https://gitee.com/wilburplus/image-cdn/raw/master/blog/202201211209585.webp)



![图片](https://gitee.com/wilburplus/image-cdn/raw/master/blog/202201211210233.webp)



![图片](https://gitee.com/wilburplus/image-cdn/raw/master/blog/202201211210889.webp)



![图片](https://gitee.com/wilburplus/image-cdn/raw/master/blog/202201211210497.webp)



**源码地址：** [https://gitee.com/jeequan/jeepay/stargazers](https://gitee.com/jeequan/jeepay/stargazers)

