---
index_img: https://mp.baomidou.com/img/mybatis-plus-framework.jpg
title: 一篇文章带你MybatisPlus从0到项目上手使用（详细配置）
date: 2021-07-14 20:26:46
updated: 2021-07-14 20:26:46
tags:
  - Java
categories:
  - 技术
comments: true
---

在没用MybatisPlus（MP）之前，如果你常常每天都要重复写CRUD的SQL，对这些大量重复性且单一的 **SQL** 已经不耐烦了，它是对 `MyBatis` 框架的进一步增强，能够极大地简化我们的持久层代码，那么你何不试试花几分钟来阅读这篇文章，学习了解一下。

博客网站 https://www.weiye.link

## MybatisPlus（MP） 是什么

MyBatis-Plus 官网地址 ：https://baomidou.com/ 



[MyBatis-Plus ](https://github.com/baomidou/mybatis-plus)（简称 MP）是一个 [MyBatis ](http://www.mybatis.org/mybatis-3/)的增强工具，在 MyBatis 的基础上只做增强不做改变，为简化开发、提高效率而生。

愿景

我们的愿景是成为 MyBatis 最好的搭档，就像 [魂斗罗](https://mp.baomidou.com/img/contra.jpg) 中的 1P、2P，基友搭配，效率翻倍。

![img](https://mp.baomidou.com/img/relationship-with-mybatis.png)

### `特性`

- **无侵入**：只做增强不做改变，引入它不会对现有工程产生影响，如丝般顺滑
- **损耗小**：启动即会自动注入基本 CURD，性能基本无损耗，直接面向对象操作
- **强大的 CRUD 操作**：内置通用 Mapper、通用 Service，仅仅通过少量配置即可实现单表大部分 CRUD 操作，更有强大的条件构造器，满足各类使用需求
- **支持 Lambda 形式调用**：通过 Lambda 表达式，方便的编写各类查询条件，无需再担心字段写错
- **支持主键自动生成**：支持多达 4 种主键策略（内含分布式唯一 ID 生成器 - Sequence），可自由配置，完美解决主键问题
- **支持 ActiveRecord 模式**：支持 ActiveRecord 形式调用，实体类只需继承 Model 类即可进行强大的 CRUD 操作
- **支持自定义全局通用操作**：支持全局通用方法注入（ Write once, use anywhere ）
- **内置代码生成器**：采用代码或者 Maven 插件可快速生成 Mapper 、 Model 、 Service 、 Controller 层代码，支持模板引擎，更有超多自定义配置等您来使用
- **内置分页插件**：基于 MyBatis 物理分页，开发者无需关心具体操作，配置好插件之后，写分页等同于普通 List 查询
- **分页插件支持多种数据库**：支持 MySQL、MariaDB、Oracle、DB2、H2、HSQL、SQLite、Postgre、SQLServer 等多种数据库
- **内置性能分析插件**：可输出 Sql 语句以及其执行时间，建议开发测试时启用该功能，能快速揪出慢查询
- **内置全局拦截插件**：提供全表 delete 、 update 操作智能分析阻断，也可自定义拦截规则，预防误操作

### `支持数据库`

- **mysql** 、**mariadb** 、**oracle** 、**db2** 、**h2** 、**hsql** 、**sqlite** 、**postgresql** 、**sqlserver** 、**presto** 、**Gauss** 、**Firebird**
- **Phoenix** 、**clickhouse** 、**Sybase ASE** 、 **OceanBase** 、达梦数据库 、虚谷数据库 、人大金仓数据库 、南大通用数据库

### `框架结构`

![framework](https://mp.baomidou.com/img/mybatis-plus-framework.jpg)



## 快速入门



#### 1、 创建数据库并建立一张`User`表

表结构：

| id   | name   | age  | email              |
| ---- | ------ | ---- | ------------------ |
| 1    | Jone   | 18   | test1@baomidou.com |
| 2    | Jack   | 20   | test2@baomidou.com |
| 3    | Tom    | 28   | test3@baomidou.com |
| 4    | Sandy  | 21   | test4@baomidou.com |
| 5    | Billie | 24   | test5@baomidou.com |

其对应的数据库 Schema 脚本如下：

```sql
DROP TABLE IF EXISTS tbl_user;

CREATE TABLE tbl_user
(
	id BIGINT(20) NOT NULL COMMENT '主键ID',
	name VARCHAR(30) NULL DEFAULT NULL COMMENT '姓名',
	age INT(11) NULL DEFAULT NULL COMMENT '年龄',
	email VARCHAR(50) NULL DEFAULT NULL COMMENT '邮箱',
	PRIMARY KEY (id)
);
```

其对应的数据库 Data 脚本如下：

```sql
DELETE FROM tbl_user;

INSERT INTO tbl_user (id, name, age, email) VALUES
(1, 'Jone', 18, 'test1@baomidou.com'),
(2, 'Jack', 20, 'test2@baomidou.com'),
(3, 'Tom', 28, 'test3@baomidou.com'),
(4, 'Sandy', 21, 'test4@baomidou.com'),
(5, 'Billie', 24, 'test5@baomidou.com');
```

#### 2、创建SpringBoot项目

![20210702170553](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210714141008.png)

#### 3、添加依赖

```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.4.3</version>
</dependency>
<dependency>
    <groupId>mysql</groupId>
    <artifactId>mysql-connector-java</artifactId>
    <scope>runtime</scope>
</dependency>
<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
</dependency>

<dependency>
    <groupId>cn.hutool</groupId>
    <artifactId>hutool-all</artifactId>
    <version>5.7.2</version>
</dependency>
```

#### 4、 配置

##### 1. yml 配置

```yaml
spring:
  datasource:
    driver-class-name: com.mysql.cj.jdbc.Driver
    url: jdbc:mysql://localhost:3306/test?serverTimezone=GMT%2B8
    username: root
    password: root
```

##### 2. Spring Boot 启动类中添加 `@MapperScan` 注解，扫描 Mapper 文件夹：

`MybatisPlusConfig`文件

```java
@SpringBootApplication
@MapperScan("com.xiangji.demo.mapper")
public class MybatisPlusApplication {

    public static void main(String[] args) {
        SpringApplication.run(MybatisPlusApplication.class, args);
    }

}
```



#### 5、编码

编写实体类 `User.java`（此处使用了 [Lombok](https://www.projectlombok.org/)简化代码）

```java
@Data
@TableName("tbl_user")
public class User{

    /**
     * 主键ID
     */
    private Long id;

    /**
     * 姓名
     */
    private String name;

    /**
     * 年龄
     */
    private Integer age;

    /**
     * 邮箱
     */
    private String email;

}
```

编写Mapper类 `UserMapper.java`

```java
public interface UserMapper extends BaseMapper<User> {

}
```

#### 6、 Mapper测试使用

###### 1. 查询所有

```java
@SpringBootTest
class MybatisPlusApplicationTests {

    @Autowired
    private UserMapper userMapper;

    @Test
    public void testSelect() {
        System.out.println(("----- selectAll method test ----- "));
        List<User> userList = userMapper.selectList(null);
        userList.forEach(System.out::println);
    }
}
```

控制台输出：

```bash
User(id=1, name=Jone, age=18, email=test1@baomidou.com)
User(id=2, name=Jack, age=20, email=test2@baomidou.com)
User(id=3, name=Tom, age=28, email=test3@baomidou.com)
User(id=4, name=Sandy, age=21, email=test4@baomidou.com)
User(id=5, name=Billie, age=24, email=test5@baomidou.com)
```

> **Tip：**
>
> 这里强调一下可能出现的问题
>
> 1. 在我们数据库表名与我们的Java实体类名不一样时会报错，可以使用 `@TableName("tbl_user")`

###### 2. 添加

```java
@Test
public void testAdd() {
    System.out.println(("----- add method test ------"));
    User user = new User();
    user.setName("Xiangji");
    user.setAge(18);
    user.setEmail("test6@baomidou.com");
    int i = userMapper.insert(user);
    // i 返回的是插入的条数
    // user.getId() 可以直接获取到插入后的ID值
    System.out.println("return the number added：" + i);
    System.out.println("return the primary key ID：" + user.getId());
}
```

控制台输出

```bash
return the number added：1
return the primary key ID：6
```

#### 7、ID 策略

在创建表的时候我故意没有设置主键的增长策略，可能很多小伙伴添加会出错，我们可以在实体类中使用 `@TableId` 来设置主键的策略：

```java
@Data
@TableName("tbl_user")
public class User{

    /**
     * 主键ID
     */
    @TableId(type = IdType.AUTO)
    private Long id;

    /**
     * 姓名
     */
    private String name;

    /**
     * 年龄
     */
    private Integer age;

    /**
     * 邮箱
     */
    private String email;

}
```

`MyBatisPlus` 提供了几种主键的策略：

```java
public enum IdType {
    // 数据库ID 自增
    // 该类型确保了数据库设置了ID自增，否则无效
    AUTO(0),
    // 无状态,该类型为未设置主键类型(注解里等于跟随全局,全局里约等于 INPUT)
    NONE(1),
    // 用户输入ID，该类型可以通过自己注册自动填充进行
    INPUT(2),
    // 分配ID(主键类型为Number(Long和Integer)或String)(since 3.3.0),使用接口IdentifierGenerator的方法nextId(默认实现类为DefaultIdentifierGenerator雪花算法)
    ASSIGN_ID(3),
    // 分配UUID,主键类型为String(since 3.3.0),使用接口IdentifierGenerator的方法nextUUID(默认default方法)
    ASSIGN_UUID(4);
}
```

其中 `AUTO` 表示数据库自增策略，该策略下需要数据库实现主键的自增（auto_increment)，`ASSIGN_ID` 是雪花算法，默认使用的是该策略，`ASSIGN_UUID` 是 UUID 策略，一般不会使用该策略。

这里多说一点， 当实体类的主键名为 id，并且数据表的主键名也为 id 时，此时 `MyBatisPlus` 会自动判定该属性为主键 id，倘若名字不是 id 时，就需要标注 `@TableId` 注解，若是实体类中主键名与数据表的主键名不一致，则可以进行声明：

```java
@TableId(value = "uid",type = IdType.AUTO)
private Long id;
```

还可以在配置文件中配置全局的主键策略：

```yaml
mybatis-plus:
  global-config:
    db-config:
      id-type: auto
```

这样能够避免在每个实体类中重复设置主键策略。

#### `8、小结`

在以上的结果，我们可以看到已经打印出了数据库中的全部数据。而并没有看到平时我们需要写的 **mapper.xml** 文件，只是用到了 userMapper中的 **selectList()** 方法，而 **UserMapper** 继承了 **BaseMapper** 这个接口，这个接口便是 **MybatisPlus** 提供给我们的，我们再来看下这个接口给我们提供了哪些方法。

​							![20210702174858](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210714141056.png)

具体细节可以查阅其源码自行体会，其实大概看名称都知道能代表什么意思，注释都是中文的，非常容易理解。





其实在我们平常开发过程中，主要业务逻辑都是在 `Service` 层，然后调用 `Mapper` 层的方法，而 `MyBatisPlus` 也为我们提供了通用的 `Service`：

```java
// This is the interface
public interface IUserService extends IService<User> {

}

// This is the interface implementation
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements IUserService {

}
```

这里我们看到 `UserServiceImpl` 不仅继承了公共 `ServiceImpl` 还实现了我们自己的 `IUserService` ，在我们继承了`ServiceImpl`  后其实就可以使用 `Service` 层的方法，但是在我们平常开发中往往会有一些复杂场景的数据处理，`MyBatisPlus` 提供的 `Service` 方法可能无法处理，这时候就得在 `IUserService` 定义自己的方法，来方便业务的扩展。

接下来我们就来测试一下 mybatisPlus 提供的 `Service` 方法

#### 9、Service测试使用

###### 1. 查询所有

```java
@SpringBootTest
class MybatisPlusApplicationServiceTests {

    @Autowired
    private IUserService userService;


    @Test
    public void testSelect() {
        System.out.println(("----- selectAll method test ------"));
        List<User> userList = userService.list();
        userList.forEach(System.out::println);
    }
}
```

控制台输出

```bash
User(id=1, name=Jone, age=18, email=test1@baomidou.com)
User(id=2, name=Jack, age=20, email=test2@baomidou.com)
User(id=3, name=Tom, age=28, email=test3@baomidou.com)
User(id=4, name=Sandy, age=21, email=test4@baomidou.com)
User(id=5, name=Billie, age=24, email=test5@baomidou.com)
User(id=6, name=Xiangji, age=18, email=test6@baomidou.com)
User(id=7, name=Xiangji, age=18, email=test6@baomidou.com)
```

###### 2. 自定义数据处理场景

首先在 `UserMapper` 中进行声明：

```java
public interface UserMapper extends BaseMapper<User> {

    User selectFirstByName(@Param("firstName")String name);

}
```

此时我们需要自己编写配置文件实现该方法，创建 `UserMapper.xml` 文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.xiangji.demo.mapper.UserMapper">

    <!-- 通用查询映射结果 -->
    <resultMap id="BaseResultMap" type="com.xiangji.demo.domain.User">
        <id column="id" property="id" />
        <result column="name" property="name" />
        <result column="age" property="age" />
        <result column="email" property="email" />
    </resultMap>

    <!-- 通用查询结果列 -->
    <sql id="Base_Column_List">
        id, name, age, email
    </sql>

    <select id="selectFirstByName" resultMap="BaseResultMap">
        select <include refid="Base_Column_List"/>
        from tbl_user
        where `name`= #{firstName} limit 1
    </select>

</mapper>
```

`MyBatisPlus` 默认扫描的是类路径下的 `mapper` 目录，这可以从源码中得到体现：

![20210705111023](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210714141121.png)

所以我们直接将 `Mapper` 配置文件放在该目录下就没有任何问题，可如果不是这个目录，我们就需要进行配置，比如：

```yaml
mybatis-plus:
  mapper-locations: classpath:xml/*.xml
```

编写好 `Mapper` 接口后，我们就需要定义 `Service` 方法了：

```java
public interface IUserService extends IService<User> {
    User selectFirstByName(String name);
}

@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements IUserService {
    @Override
    public User selectFirstByName(String name) {
        return baseMapper.selectFirstByName(name);
    }
}
```

可以在代码中看到直接使用的是 `baseMapper` ，没有使用 userMapper 来调用接口，查看 `ServiceImpl` 的源码：

![20210705112924](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210714141137.png)

可以看到它为我们注入了一个 `BaseMapper` 对象，而它是第一个泛型类型，也就是 `UserMapper ` 类型，所以我们可以直接使用这个 `baseMapper` 来调用 `Mapper` 中的方法。

编写测试代码：

```java
@SpringBootTest
class MybatisPlusApplicationServiceTests {

    @Autowired
    private IUserService userService;


    @Test
    public void testSelect() {
        System.out.println(("----- queryFirstByName method test ------"));
        User user = userService.queryFirstByName("Xiangji");
        System.out.println(user);
    }
}
```

控制台输出：

```bash
User(id=6, name=Xiangji, age=18, email=test6@baomidou.com)
```

## 核心功能

### 分页插件

开发中，分页是很常用的，`MyBatisPlus` 自带分页插件，只需要进行简单的配置即可实现：

```java
@Configuration
public class MybatisPlusConfig {

	@Bean
	public MybatisPlusInterceptor mybatisPlusInterceptor() {
		MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
		// 分页插件
		interceptor.addInnerInterceptor(paginationInnerInterceptor());
    }
	/**
	 * 分页插件，自动识别数据库类型
	 * https://baomidou.com/guide/interceptor-pagination.html
	 */
	public PaginationInnerInterceptor paginationInnerInterceptor() {
		PaginationInnerInterceptor paginationInnerInterceptor = new PaginationInnerInterceptor();
		// 设置数据库类型为mysql
		paginationInnerInterceptor.setDbType(DbType.MYSQL);
		// 设置最大单页限制数量，默认 500 条，-1 不受限制
		paginationInnerInterceptor.setMaxLimit(-1L);
		return paginationInnerInterceptor;
	}
}
```

我们配置好了分页功能后，就可以直接使用分页插件功能了：

```java
@Test
public void testPage() {
    // 创建Page对象，参数分别对应当前页和每页显示记录数
    Page<User> page = new Page<>(1,2);
    // 直接调用公共 Service 的page方法，参数分别对应page对象和wrapper条件
    userService.page(page, null);
    // page.getRecords() 当前查询出来的分页数据集合
    List<User> userList = page.getRecords();
    userList.forEach(System.out::println);
    System.out.println("获取总条数:" + page.getTotal());
    System.out.println("获取当前页码:" + page.getCurrent());
    System.out.println("获取总页码:" + page.getPages());
    System.out.println("获取每页显示的数据条数:" + page.getSize());
    System.out.println("是否有上一页:" + page.hasPrevious());
    System.out.println("是否有下一页:" + page.hasNext());
}
```

输出结果：

```bash
User(id=1, name=Jone, age=18, email=test1@baomidou.com)
User(id=2, name=Jack, age=20, email=test2@baomidou.com)
获取总条数:7
获取当前页码:1
获取总页码:4
获取每页显示的数据条数:2
是否有上一页:false
是否有下一页:true
```

我们还可以在分页过程中构建一些限定条件，这是就需要使用 `QueryWrapper` 来构建使用：

```java
@Test
public void testPageWrapper() {
    // 创建Page对象，参数分别对应当前页和每页显示记录数
    Page<User> page = new Page<>(1,2);
    // 直接调用公共 Service 的page方法，参数分别对应page对象和wrapper条件
    userService.page(page,new QueryWrapper<User>()
                     .between("age", 20, 30));
    // page.getRecords() 当前查询出来的分页数据集合
    List<User> userList = page.getRecords();
    userList.forEach(System.out::println);
}
```

当前查询就应该只出现年龄 20 ~30 之间的分页数据，每页显示记录数只有两条

结果输出：

```bash
User(id=2, name=Jack, age=20, email=test2@baomidou.com)
User(id=3, name=Tom, age=28, email=test3@baomidou.com)
```

### 逻辑删除

- **物理删除**：真实删除，将对应数据从数据库中删除，之后查询不到此条被删除数据
- **逻辑删除**：假删除，将对应数据中代表是否被删除字段状态修改为“被删除状态”，之后在数据库中仍旧能看到此条数据记录

在这个数据为王的时代，数据就是财富，所以一般并不会有哪个系统在删除某些重要数据时真正删掉了数据，通常都是在数据库中建立一个状态列，让其默认为 0，当为 0 时，用户可见；当执行了删除操作，就将状态列改为 1，此时用户不可见，但数据还是在表中的。

![20210705143312](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210714141214.png)

按照《阿里巴巴 Java 开发手册》第 5 章 MySQL 数据库相关的建议，我们来为数据表新增一个`is_deleted` 字段：

```sql
alter table tbl_user add column is_deleted tinyint not null;
```

在实体类中也要添加这一属性：

```java
@Data
@TableName("tbl_user")
public class User{

    /**
     * 主键ID
     */
    @TableId(type = IdType.AUTO)
    private Long id;

    /**
     * 姓名
     */
    private String name;

    /**
     * 年龄
     */
    private Integer age;

    /**
     * 邮箱
     */
    private String email;

    /**
     * 逻辑删除属性
     */
    @TableLogic
    @TableField("is_deleted")
    private Boolean deleted;

}
```

![20210705143532](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210714141235.png)

还是参照《阿里巴巴 Java 开发手册》第 5 章 MySQL 数据库相关的建议，对于布尔类型变量，不能加 is 前缀，所以我们的属性被命名为 `deleted`，但此时就无法与数据表的字段进行对应了，所以我们需要使用 `@TableField` 注解来声明一下数据表的字段名，而 `@TableLogin` 注解用于设置逻辑删除属性；此时我们执行删除操作：

```java
@Test
public void testRemove() {
    userService.removeById(7);
}
```

然后查询数据库：

​												![20210705144218](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210714141249.png)

数据ID为7的这一条已经被逻辑删除，is_deleted字段变成了1

我们再查询一下所有的数据，看看是否还能查出7条。

```java
@Test
public void testSelectAll() {
    List<User> user = userService.list();
    user.forEach(System.out::println);
}
```

执行结果：

```bash
User(id=1, name=Jone, age=18, email=test1@baomidou.com, deleted=false)
User(id=2, name=Jack, age=20, email=test2@baomidou.com, deleted=false)
User(id=3, name=Tom, age=28, email=test3@baomidou.com, deleted=false)
User(id=4, name=Sandy, age=21, email=test4@baomidou.com, deleted=false)
User(id=5, name=Billie, age=24, email=test5@baomidou.com, deleted=false)
User(id=6, name=Xiangji, age=18, email=test6@baomidou.com, deleted=false)
```

#### 配置输出日志

发现只有6天，id为7的这一条数据没有查询出来，仔细看代码我们也没有添加限定条件查询，直接查询的所有，它是如何实现的呢？我们可以输出 `MyBatisPlus` 生成的 SQL 来分析一下，在配置文件中进行配置：

```yaml
mybatis-plus:
  configuration:
    log-impl: org.apache.ibatis.logging.stdout.StdOutImpl # 输出SQL日志
```

然后我们从新查询一遍，再看输出日志：

![20210705145014](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210714141314.png)

发现给我们自动添加了删除条件，只查询未删除（0）的数据，已删除（1）的数据就直接过滤掉了。

另外， `MyBatisPlus` 默认 0 为不删除，1 为删除。若是你想修改这个规定，比如设置1 为已删除，2 为未删除，也可以进行配置：

```yaml
mybatis-plus:
  global-config:
    db-config:
      id-type: auto
      logic-delete-field: deleted # 逻辑删除属性名
      logic-delete-value: 1 # 已删除
      logic-not-delete-value: 2 # 未删除
```

这里还是建议就用默认的规则，参照前面阿里巴巴规范也是使用的默认规则。

### 自动填充

翻阅《阿里巴巴 Java 开发手册》，在第 5 章 MySQL 数据库可以看到这样一条规范：

![202107051458391](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210714141941.png)

对于一张数据表，它必须具备三个字段：

- `id` : 唯一 ID
- `gmt_create` : 保存的是当前数据创建的时间
- `gmt_modified` : 保存的是更新时间

我们改造一下数据表：

```sql
alter table tbl_user add column gmt_create datetime not null;
alter table tbl_user add column gmt_modified datetime not null;
```

然后改造一下实体类：

```java
@Data
@TableName("tbl_user")
public class User{

    /**
     * 主键ID
     */
    @TableId(type = IdType.AUTO)
    private Long id;

    /**
     * 姓名
     */
    private String name;

    /**
     * 年龄
     */
    private Integer age;

    /**
     * 邮箱
     */
    private String email;

    /**
     * 创建时间
     */
    private LocalDateTime gmtCreate;

    /**
     * 修改时间
     */
    private LocalDateTime gmtModified;

    /**
     * 逻辑删除属性
     */
    @TableLogic
    @TableField("is_deleted")
    private Boolean deleted;

}

```

对于固定字段，我们不能每次添加/修改都要手动设置时间，这样很麻烦，好在 `MyBatisPlus` 提供了字段自动填充功能来帮助我们进行管理，需要使用到的是 `@TableField` 注解：

```java
@Data
@TableName("tbl_user")
public class User{

    /**
     * 主键ID
     */
    @TableId(type = IdType.AUTO)
    private Long id;

    /**
     * 姓名
     */
    private String name;

    /**
     * 年龄
     */
    private Integer age;

    /**
     * 邮箱
     */
    private String email;

    /**
     * 创建时间
     */
    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime gmtCreate;

    /**
     * 修改时间
     */
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime gmtModified;

    /**
     * 逻辑删除属性
     */
    @TableLogic
    @TableField("is_deleted")
    private Boolean deleted;

}
```

> Tip：
>
> @TableField(fill = FieldFill.INSERT) 表示 *插入的时候自动填充*
>
> @TableField(fill = FieldFill.INSERT_UPDATE)  表示 *插入和更新的时候自动填充*

然后编写一个类实现 `MetaObjectHandler ` 接口

```java
@Slf4j
public class CreateAndUpdateMetaObjectHandler implements MetaObjectHandler {

	@Override
	public void insertFill(MetaObject metaObject) {
		//根据属性名字设置要填充的值
		if (metaObject.hasGetter("gmtCreate")) {
			if (metaObject.getValue("gmtCreate") == null) {
				this.setFieldValByName("gmtCreate", LocalDateTime.now(), metaObject);
			}
		}
	}

	@Override
	public void updateFill(MetaObject metaObject) {
		if (metaObject.hasGetter("gmtModified")) {
			if (metaObject.getValue("gmtModified") == null) {
				this.setFieldValByName("gmtModified", LocalDateTime.now(), metaObject);
			}
		}
	}

}
```

该接口中有两个未实现的方法，分别为插入和更新时的填充方法，在方法中调用 `strictInsertFill()` 方法 即可实现属性的填充，它需要四个参数：

1. `metaObject`：元对象，就是方法的入参
2. `fieldName`：为哪个属性进行自动填充
3. `fieldType`：属性的类型
4. `fieldVal`：需要填充的属性值

再到我们刚才配置分页的config配置类新增一个自动填充的bean

```java
@Configuration
public class MybatisPlusConfig {

	@Bean
	public MybatisPlusInterceptor mybatisPlusInterceptor() {
		MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
		// 分页插件
		interceptor.addInnerInterceptor(paginationInnerInterceptor());
		return interceptor;
	}

	/**
	 * 分页插件，自动识别数据库类型
	 * https://baomidou.com/guide/interceptor-pagination.html
	 */
	public PaginationInnerInterceptor paginationInnerInterceptor() {
		PaginationInnerInterceptor paginationInnerInterceptor = new PaginationInnerInterceptor();
		// 设置数据库类型为mysql
		paginationInnerInterceptor.setDbType(DbType.MYSQL);
		// 设置最大单页限制数量，默认 500 条，-1 不受限制
		paginationInnerInterceptor.setMaxLimit(-1L);
		return paginationInnerInterceptor;
	}

	/**
	 * 元对象字段填充控制器
	 * https://baomidou.com/guide/auto-fill-metainfo.html
	 */
	@Bean
	public MetaObjectHandler metaObjectHandler() {
		return new CreateAndUpdateMetaObjectHandler();
	}

}
```

此时在更新数据之前，这两个方法会先被执行，以实现属性的自动填充，通过日志我们可以进行验证：

```java
@Test
public void testUpdate() {
    User user = new User();
    user.setId(6L);
    user.setAge(20);
    userService.updateById(user);
}
```

日志输出：

```bash
2021-07-05 15:18:32.899  INFO 20264 --- [           main] c.x.d.c.CreateAndUpdateMetaObjectHandler : update开始属性填充
==>  Preparing: UPDATE tbl_user SET age=?, gmt_modified=? WHERE id=? AND is_deleted=0
==> Parameters: 20(Integer), 2021-07-05T15:15:04.077(LocalDateTime), 6(Long)
<==    Updates: 1
```

### 乐观锁

当程序中出现并发访问时，就需要保证数据的一致性。以商品系统为例，现在有两个管理员均想对同一件售价为 100 元的商品进行修改，A 管理员正准备将商品售价改为 150 元，但此时出现了网络问题，导致 A 管理员的操作陷入了等待状态；此时 B 管理员也进行修改，将商品售价改为了 200 元，修改完成后 B 管理员退出了系统，此时 A 管理员的操作也生效了，这样便使得 A 管理员的操作直接覆盖了 B 管理员的操作，B 管理员后续再进行查询时会发现商品售价变为了 150 元，这样的情况是绝对不允许发生的。

要想解决这一问题，可以给数据表加锁，常见的方式有两种：

1. 乐观锁
2. 悲观锁

悲观锁认为并发情况一定会发生，所以在某条数据被修改时，为了避免其它人修改，会直接对数据表进行加锁，它依靠的是数据库本身提供的锁机制（表锁、行锁、读锁、写锁）。

而乐观锁则相反，它认为数据产生冲突的情况一般不会发生，所以在修改数据的时候并不会对数据表进行加锁的操作，而是在提交数据时进行校验，判断提交上来的数据是否会发生冲突，如果发生冲突，则提示用户重新进行操作，一般的实现方式为 `设置版本号字段` 。

就以商品售价为例，在该表中设置一个版本号字段，让其初始为 1，此时 A 管理员和 B 管理员同时需要修改售价，它们会先读取到数据表中的内容，此时两个管理员读取到的版本号都为 1，此时 B 管理员的操作先生效了，它就会将当前数据表中对应数据的版本号与最开始读取到的版本号作一个比对，发现没有变化，于是修改就生效了，此时版本号加 1。

而 A 管理员马上也提交了修改操作，但是此时的版本号为 2，与最开始读取到的版本号并不对应，这就说明数据发生了冲突，此时应该提示 A 管理员操作失败，并让 A 管理员重新查询一次数据。

乐观锁的优势在于采取了更加宽松的加锁机制，能够提高程序的吞吐量，适用于读操作多的场景。

那么接下来我们就来模拟这一过程。

```java
@Test
public void testOptimisticLock() {
    // A、B管理员读取数据
    User A = userMapper.selectById(6L);
    User B = userMapper.selectById(6L);
    // B管理员先修改
    B.setAge(15);
    int result = userMapper.updateById(B);
    if (result == 1) {
        System.out.println("B管理员修改成功!");
    } else {
        System.out.println("B管理员修改失败!");
    }
    // A管理员后修改
    A.setAge(25);
    int result2 = userMapper.updateById(A);
    if (result2 == 1) {
        System.out.println("A管理员修改成功!");
    } else {
        System.out.println("A管理员修改失败!");
    }
    // 最后查询
    System.out.println(userMapper.selectById(6L));
}
```

输出结果：

```bash
==>  Preparing: UPDATE tbl_user SET name=?, age=?, email=?, gmt_create=?, gmt_modified=? WHERE id=? AND is_deleted=0
==> Parameters: Xiangji(String), 15(Integer), test6@baomidou.com(String), 2021-07-05T15:47:04(LocalDateTime), 2021-07-05T15:18:33(LocalDateTime), 6(Long)
<==    Updates: 1
B管理员修改成功!

==>  Preparing: UPDATE tbl_user SET name=?, age=?, email=?, gmt_create=?, gmt_modified=? WHERE id=? AND is_deleted=0
==> Parameters: Xiangji(String), 25(Integer), test6@baomidou.com(String), 2021-07-05T15:47:04(LocalDateTime), 2021-07-05T15:18:33(LocalDateTime), 6(Long)
<==    Updates: 1
A管理员修改成功!

User(id=6, name=Xiangji, age=25, email=test6@baomidou.com, gmtCreate=2021-07-05T15:47:04, gmtModified=2021-07-05T15:18:33, deleted=false)
```

**问题出现了，B 管理员的操作被 A 管理员覆盖，那么该如何解决这一问题呢？**

其实 `MyBatisPlus` 已经提供了乐观锁机制。

#### OptimisticLockerInnerInterceptor

> 当要更新一条记录的时候，希望这条记录没有被别人更新
> 乐观锁实现方式：
>
> - 取出记录时，获取当前version
> - 更新时，带上这个version
> - 执行更新时， set version = newVersion where version = oldVersion
> - 如果version不对，就更新失败

**乐观锁配置需要两步**

在之前配置分页插件以及自动填充的mybatisPlus配置类中新增乐观锁

```java
@Configuration
public class MybatisPlusConfig {

	@Bean
	public MybatisPlusInterceptor mybatisPlusInterceptor() {
		MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
		// 分页插件
		interceptor.addInnerInterceptor(paginationInnerInterceptor());
		// 乐观锁插件
		interceptor.addInnerInterceptor(optimisticLockerInnerInterceptor());
		return interceptor;
	}

	/**
	 * 分页插件，自动识别数据库类型
	 * https://baomidou.com/guide/interceptor-pagination.html
	 */
	public PaginationInnerInterceptor paginationInnerInterceptor() {
		PaginationInnerInterceptor paginationInnerInterceptor = new PaginationInnerInterceptor();
		// 设置数据库类型为mysql
		paginationInnerInterceptor.setDbType(DbType.MYSQL);
		// 设置最大单页限制数量，默认 500 条，-1 不受限制
		paginationInnerInterceptor.setMaxLimit(-1L);
		return paginationInnerInterceptor;
	}

	/**
	 * 乐观锁插件
	 * https://baomidou.com/guide/interceptor-optimistic-locker.html
	 */
	public OptimisticLockerInnerInterceptor optimisticLockerInnerInterceptor() {
		return new OptimisticLockerInnerInterceptor();
	}

	/**
	 * 元对象字段填充控制器
	 * https://baomidou.com/guide/auto-fill-metainfo.html
	 */
	@Bean
	public MetaObjectHandler metaObjectHandler() {
		return new CreateAndUpdateMetaObjectHandler();
	}

}
```

然后在实体类中新增 version 字段，并做上注解

```java
@Data
@TableName("tbl_user")
public class User{

    /**
     * 主键ID
     */
    @TableId(type = IdType.AUTO)
    private Long id;

    /**
     * 姓名
     */
    private String name;

    /**
     * 年龄
     */
    private Integer age;

    /**
     * 邮箱
     */
    private String email;

    /**
     * 创建时间
     */
    @TableField(fill = FieldFill.INSERT)
    private LocalDateTime gmtCreate;

    /**
     * 修改时间
     */
    @TableField(fill = FieldFill.INSERT_UPDATE)
    private LocalDateTime gmtModified;

    /**
     * 逻辑删除属性
     */
    @TableLogic
    @TableField("is_deleted")
    private Boolean deleted;

    @Version
    private Integer version;
}
```

> 说明:
>
> - **支持的数据类型只有:int,Integer,long,Long,Date,Timestamp,LocalDateTime**
> - 整数类型下 `newVersion = oldVersion + 1`
> - `newVersion` 会回写到 `entity` 中
> - 仅支持 `updateById(id)` 与 `update(entity, wrapper)` 方法
> - **在 `update(entity, wrapper)` 方法下, `wrapper` 不能复用!!!**

我们再为数据库新增一个version字段

```sql
alter table tbl_user add column version int not null;
```

重新执行测试代码，结果如下：

```
==>  Preparing: UPDATE tbl_user SET name=?, age=?, email=?, gmt_create=?, gmt_modified=?, version=? WHERE id=? AND version=? AND is_deleted=0
==> Parameters: Xiangji(String), 15(Integer), test6@baomidou.com(String), 2021-07-05T15:47:04(LocalDateTime), 2021-07-05T15:18:33(LocalDateTime), 1(Integer), 6(Long), 0(Integer)
<==    Updates: 1
B管理员修改成功!

==>  Preparing: UPDATE tbl_user SET name=?, age=?, email=?, gmt_create=?, gmt_modified=?, version=? WHERE id=? AND version=? AND is_deleted=0
==> Parameters: Xiangji(String), 25(Integer), test6@baomidou.com(String), 2021-07-05T15:47:04(LocalDateTime), 2021-07-05T15:18:33(LocalDateTime), 1(Integer), 6(Long), 0(Integer)
<==    Updates: 0
A管理员修改失败!

User(id=6, name=Xiangji, age=15, email=test6@baomidou.com, gmtCreate=2021-07-05T15:47:04, gmtModified=2021-07-05T15:18:33, deleted=false, version=1)
```

看结果，说明我们的乐观锁已经配置成功，B先执行修改的年龄为15，A后执行修改的年龄为25，A/B修改的version条件都为0，B执行成功后version变成了1，所以A修改执行version为0就没有这条数据，修改无效。

### Wrapper条件构造器

在分页插件中我们简单地使用了一下条件构造器（`Wrapper`），下面我们来详细了解一下。先来看看 `Wrapper` 的继承体系：

![20210705160606](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210714142202.png)

分别介绍一下它们的作用：

- `Wrapper`：条件构造器抽象类，最顶端的父类

- - `LambdaQue！ryWrapper`：用于对象封装，使用 Lambda 语法
  - `LambdaUpdateWrapper`：用于条件封装，使用 Lambda 语法
  - `QueryWrapper`：用于对象封装
  - `UpdateWrapper`：用于条件封装
  - `AbstractWrapper`：查询条件封装抽象类，生成 SQL 的 where 条件
  - `AbstractLambdaWrapper`：Lambda 语法使用 Wrapper

通常我们使用的都是 `QueryWrapper` 和 `UpdateWrapper`，若是想使用 Lambda 语法来编写，也可以使用 `LambdaQueryWrapper` 和 `LambdaUpdateWrapper`，通过这些条件构造器，我们能够很方便地来实现一些复杂的筛选操作，比如：

```java
@Test
public void testWrapper() {
    // 查询名字中包含'j'，年龄大于等于18岁及小于等于20岁
    QueryWrapper<User> wrapper = new QueryWrapper<>();
    wrapper.like("name", "j");
    wrapper.ge("age", 18);
    wrapper.le("age", 20);
    wrapper.isNotNull("email");
    List<User> list = userMapper.selectList(wrapper);
    list.forEach(System.out::println);
}
```

输出结果：

```bash
==>  Preparing: SELECT id,name,age,email,gmt_create,gmt_modified,is_deleted AS deleted,version FROM tbl_user WHERE is_deleted=0 AND (name LIKE ? AND age >= ? AND age <= ? AND email IS NOT NULL)
==> Parameters: %j%(String), 18(Integer), 20(Integer)
<==    Columns: id, name, age, email, gmt_create, gmt_modified, deleted, version
<==        Row: 1, Jone, 18, test1@baomidou.com, 2021-07-05 16:11:26, 2021-07-05 16:12:11, 0, 0
<==        Row: 2, Jack, 20, test2@baomidou.com, 2021-07-05 16:11:29, 2021-07-05 16:12:09, 0, 0
<==      Total: 2

User(id=1, name=Jone, age=18, email=test1@baomidou.com, gmtCreate=2021-07-05T16:11:26, gmtModified=2021-07-05T16:12:11, deleted=false, version=0)
User(id=2, name=Jack, age=20, email=test2@baomidou.com, gmtCreate=2021-07-05T16:11:29, gmtModified=2021-07-05T16:12:09, deleted=false, version=0)
```

我们看到输出的sql语句条件都是我们wrapper组合成的对应的语句，关于wrapper的方法，相信经常写sql的小伙伴也是很容易看懂，我们也可以查看源码或者在官方文档里面进行学习[wrapper条件构造文档](https://mp.baomidou.com/guide/wrapper.html#abstractwrapper)。



我们还可以使用链式编程

```java
@Test
public void testWrapper() {
    // 查询名字中包含'j'，年龄大于等于18岁及小于等于20岁
    QueryWrapper<User> wrapper = new QueryWrapper<User>()
        .like("name", "j")
        .ge("age", 18)
        .le("age", 20);
    List<User> list = userMapper.selectList(wrapper);
    list.forEach(System.out::println);
}
```

也可以使用 `LambdaQueryWrapper` 实现：

```java
@Test
public void testWrapper() {
    // 查询名字中包含'j'，年龄大于等于18岁及小于等于20岁
    LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<User>()
        .like(User::getName, "j")
        .ge(User::getAge, 18)
        .le(User::getAge, 20);
    List<User> list = userMapper.selectList(wrapper);
    list.forEach(System.out::println);
}
```

这种方式的好处在于对字段的设置不是硬编码，而是采用方法引用的形式，效果与 `QueryWrapper` 是一样的。

最后还有`UpdateWrapper` 与 `QueryWrapper` 不同，它的作用是封装更新内容的，比如：

```java
@Test
public void testUpdateWrapper() {
    // 修改邮箱为“temp@163.com”，名字中包含'j'，年龄大于等于18岁及小于等于20岁
    UpdateWrapper<User> wrapper = new UpdateWrapper<User>()
        .set("email","temp@163.com")
        .like("name", "j")
        .ge("age", 18)
        .le("age", 20);
    userMapper.update(null,wrapper);
}
```

输出结果

```
==>  Preparing: UPDATE tbl_user SET email=? WHERE is_deleted=0 AND (name LIKE ? AND age >= ? AND age <= ?)
==> Parameters: testUpdateWrapper@baomidou.com(String), %j%(String), 18(Integer), 20(Integer)
<==    Updates: 2
```

将名字中包含 `j` 且年龄大于等于18岁及小于等于20岁，邮箱改为 temp@163.com，`UpdateWrapper` 不仅能够封装更新内容，也能作为查询条件，所以在更新数据时可以直接构造一个 `UpdateWrapper` 来设置更新内容和条件。

## 源码

最后，如果感觉碎片化的代码片段了解得不是很透彻，可以下载源码查阅 [GitHub源码下载](https://github.com/wilbur147/xiangStudy/tree/main/lab-springBoot/MybatisPlus-1) ，下载下来后更新一下maven便可以直接启动测试，附上代码目录

​												![20210705164231](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210714144502.png)



参考：

https://mp.baomidou.com/guide

https://juejin.cn/post/6847902215504396296#heading-49

https://mp.weixin.qq.com/s/tKeOw8JiSC8G4mIqQvReYA