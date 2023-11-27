---
index_img: https://static001.geekbang.org/infoq/0c/0c422e63f61599cc7be9da745d211dfa.jpeg
title: SpringBoot2.X系列（6）：数据访问-jdbc整合
date: 2022-06-18 21:22:16
updated: 2022-06-18 21:22:16
tags:
  - Java
  - SpringBoot
categories:
  - 技术
comments: true
---

[TOC]



## 前言

在之前文章中我们介绍了Spring Boot 项目的创建及启动原理等一系列知识，接下来我们将继续学习在使用Spring Boot开发服务项目时，如何对现在流行的存储产品（数据库）的操作。

## SpringData介绍

对于数据层访问分 SQL(关系型数据库) 、 NOSQL(非关系型数据库)，我们在Spring Boot 时，底层都是采用 Spring Data 的方式进行统一处理的。

使用 Spring Data 的方式统一处理各种数据库，Spring Data 也是 Spring 中与 Spring Boot、Spring Cloud 等齐名的知名项目。

> 官网： [https://spring.io/projects/spring-data](https://spring.io/projects/spring-data)
>
> 数据库相关启动器文档介绍：[https://docs.spring.io/spring-boot/docs/2.2.5.RELEASE/reference/htmlsingle/#using-boot-starter](https://docs.spring.io/spring-boot/docs/2.2.5.RELEASE/reference/htmlsingle/#using-boot-starter)

## JDBC项目整合

### 一、创建项目

![image-20210809172606389](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210809172606.png)

### 二、配置数据源

在我们访问数据库的时候，需要先配置一个数据源，因为要连接数据库，我们需要引入jdbc支持，在`pom.xml`中引入如下配置：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-jdbc</artifactId>
</dependency>
```

- 本篇以MySQL数据库为例，所以要引入MySQL连接的依赖包，在`pom.xml`中加入：

```xml
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
```



### 三、编辑yaml配置文件连接数据库

原目录在 `src\main\resources\application.properties` ，我们需要把 `application.properties` 改成  `application.yml` 



```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/jdbc-learn?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8
    username: root
    password: 123456
```

> **注意：因为Spring Boot 2.1.x默认使用了MySQL 8.0的驱动，所以这里采用`com.mysql.cj.jdbc.Driver`，而不是老的`com.mysql.jdbc.Driver`**



### 四、测试连接

通过上面的配置我们就可以直接使用了，因为SpringBoot已经默认帮我们进行了自动配置；去测试类测试一下：



```java
import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.SQLException;

@SpringBootTest
class JdbcApplicationTests {

    //注入数据源
    @Autowired
    DataSource dataSource;

    @Test
    void contextLoads()  throws SQLException {
        //看一下默认数据源
        System.out.println(dataSource.getClass());
        //获得连接
        Connection connection = dataSource.getConnection();
        System.out.println(connection);
        //关闭连接
        connection.close();
    }
}
```

**输出结果：**

```bash
# 默认数据源
class com.zaxxer.hikari.HikariDataSource
# 连接
HikariProxyConnection@708004780 wrapping com.mysql.cj.jdbc.ConnectionImpl@470d183
```



可以看到默认给我们配置的数据源为 : `class com.zaxxer.hikari.HikariDataSource` ， 我们并没有手动配置

<span style="color:red">**HikariDataSource 号称 Java WEB 当前速度最快的数据源，相比于传统的 C3P0 、DBCP、Tomcat jdbc 等连接池更加优秀；**</span>

关于数据源我们并不做介绍，有了数据库连接，显然就可以 CRUD 操作数据库了。但是我们需要先了解一个对象 **`JdbcTemplate`**

## JDBCTemplate

1、有了数据源(com.zaxxer.hikari.HikariDataSource)，然后可以拿到数据库连接(java.sql.Connection)，有了连接，就可以使用原生的 JDBC 语句来操作数据库；

2、即使不使用第三方第数据库操作框架，如 MyBatis等，Spring 本身也对原生的JDBC 做了轻量级的封装，即JdbcTemplate。

3、数据库操作的所有 CRUD 方法都在 JdbcTemplate 中。

4、Spring Boot 不仅提供了默认的数据源，同时默认已经配置好了 JdbcTemplate 放在了容器中，程序员只需自己注入即可使用

5、JdbcTemplate 的自动配置是依赖 org.springframework.boot.autoconfigure.jdbc 包下的 JdbcTemplateConfiguration 类

**JdbcTemplate主要提供以下几类方法：**

- execute方法：可以用于执行任何SQL语句，一般用于执行DDL语句；
- update方法及batchUpdate方法：update方法用于执行新增、修改、删除等语句；batchUpdate方法用于执行批处理相关语句；
- query方法及queryForXXX方法：用于执行查询相关语句；
- call方法：用于执行存储过程、函数相关语句。



### 使用JdbcTemplate操作数据库

##### 1、 准备数据库表

先创建`User`表，包含属性`name`、`age`。可以通过执行下面的建表语句：

```sql
CREATE TABLE `springboot_learn`.`User`  (
  `name` varchar(100) NOT NULL COMMENT '姓名',
  `age` int NULL COMMENT '年龄'
);
```



##### 2、 创建领域对象

根据数据库中创建的`User`表，创建领域对象：

```java
@Data
@NoArgsConstructor
public class User {

    private String name;
    private Integer age;

}
```

使用了Lombok的`@Data`和`@NoArgsConstructor`注解来自动生成各参数的Set、Get函数以及不带参数的构造函数



##### 3、 创建数据服务访问对象

- 定义有插入、删除、查询的抽象接口UserService

```java
public interface UserService {

    /**
     * 新增一个用户
     *
     * @param name
     * @param age
     */
    int create(String name, Integer age);

    /**
     * 根据name查询用户
     *
     * @param name
     * @return
     */
    List<User> getByName(String name);

    /**
     * 根据name删除用户
     *
     * @param name
     */
    int deleteByName(String name);

    /**
     * 获取用户总量
     */
    int getAllUsers();

    /**
     * 删除所有用户
     */
    int deleteAllUsers();

}
```



- 通过`JdbcTemplate`实现`UserService`中定义的数据访问操作

```java
@Service
public class UserServiceImpl implements UserService {

    private JdbcTemplate jdbcTemplate;

    UserServiceImpl(JdbcTemplate jdbcTemplate) {
        this.jdbcTemplate = jdbcTemplate;
    }

    @Override
    public int create(String name, Integer age) {
        return jdbcTemplate.update("insert into USER(NAME, AGE) values(?, ?)", name, age);
    }

    @Override
    public List<User> getByName(String name) {
        List<User> users = jdbcTemplate.query("select NAME, AGE from USER where NAME = ?", (resultSet, i) -> {
            User user = new User();
            user.setName(resultSet.getString("NAME"));
            user.setAge(resultSet.getInt("AGE"));
            return user;
        }, name);
        return users;
    }

    @Override
    public int deleteByName(String name) {
        return jdbcTemplate.update("delete from USER where NAME = ?", name);
    }

    @Override
    public int getAllUsers() {
        return jdbcTemplate.queryForObject("select count(1) from USER", Integer.class);
    }

    @Override
    public int deleteAllUsers() {
        return jdbcTemplate.update("delete from USER");
    }

}
```



##### 4、 编写单元测试用例

- 创建对`UserService`的单元测试用例，通过创建、删除和查询来验证数据库操作的正确性。

```java
@SpringBootTest
class JdbcApplicationTests {

    @Autowired
    private UserService userSerivce;

    @Test
    public void test() throws Exception {
        // 准备，清空user表
        userSerivce.deleteAllUsers();

        // 插入5个用户
        userSerivce.create("User1", 11);
        userSerivce.create("User2", 13);
        userSerivce.create("User3", 36);
        userSerivce.create("User4", 19);
        userSerivce.create("User5", 18);

        // 查询名为User1的用户，判断年龄是否匹配
        List<User> userList = userSerivce.getByName("User1");
        Assert.assertEquals(11, userList.get(0).getAge().intValue());

        // 查数据库，应该有5个用户
        Assert.assertEquals(5, userSerivce.getAllUsers());

        // 删除两个用户
        userSerivce.deleteByName("User4");
        userSerivce.deleteByName("User5");

        // 查数据库，应该有3个用户
        Assert.assertEquals(3, userSerivce.getAllUsers());

    }
```

测试请求结果正常，*上面介绍的`JdbcTemplate`只是最基本的几个操作；*

到此，CURD的基本操作，使用 JDBC 就搞定了。



## 参考源码

**jdbc整合源码demo：**[https://github.com/wilbur147/xiangStudy/tree/main/lab-springBoot/SpringBoot-4](https://github.com/wilbur147/xiangStudy/tree/main/lab-springBoot/SpringBoot-4)

****