---
title: Redis需要注意的规范
date: 2021-12-02 23:06:12
updated: 2021-12-02 23:06:12
tags:
  - Redis
categories:
  - 技术
comments: true
---
有位朋友在单身红娘婚恋类型互联网公司工作，在双十一推出下单就送女朋友的活动。

谁曾想，凌晨 12 点之后，用户量暴增，出现了一个技术故障，用户无法下单，当时老大火冒三丈！

经过查找发现 Redis 报 `Could not get a resource from the pool`。

获取不到连接资源，并且集群中的单台 Redis 连接量很高。

于是各种更改最大连接数、连接等待数，虽然报错信息频率有所缓解，但还是**持续报错** 。

后来经过线下测试，发现存放 Redis 中的**字符数据很大，平均 1s 返回数据** 。

> 基哥，可以分享下使用 Redis 的规范么？我想做一个唯快不破的真男人！

通过 Redis 为什么这么快？这篇文章我们知道 Redis 为了高性能和节省内存费劲心思。

所以，只有规范的使用 Redis，才能实现高性能和节省内存，否则再屌的 Redis 也禁不起我们瞎折腾。

Redis 使用规范围绕如下几个纬度展开：

- 键值对使用规范；
- 命令使用规范；
- 数据保存规范；
- 运维规范。

## 键值对使用规范

有两点需要注意：

1. 好的 `key` 命名，才能提供可读性强、可维护性高的 key，便于定位问题和寻找数据。
2. `value`要避免出现 `bigkey`、选择高效的序列化和压缩、使用对象共享池、选择高效恰当的数据类型。

### key 命名规范

规范的 `key`命名，在遇到问题的时候能够方便定位。Redis 属于 没有 `Scheme`的 `NoSQL`数据库。

所以要靠规范来建立其 `Scheme` 语意，就好比根据不同的场景我们建立不同的数据库。

**敲黑板**

把「业务模块名」作为前缀（好比数据库 `Scheme`），通过「冒号」分隔，再加上「具体业务名」。

这样我们就可以通过 `key` 前缀来区分不同的业务数据，清晰明了。

总结起来就是：「业务名:表名:id」

比如我们要统计属于技术类型的博主「码哥字节」的粉丝数。

```bash
set 技术类:码哥字节 100000
```

> 基哥，key 太长的话有什么问题么？

key 是字符串，底层的数据结构是 `SDS`，SDS 结构中会包含字符串长度、分配空间大小等元数据信息。

**字符串长度增加，SDS 的元数据也会占用更多的内存空间。**

所以当字符串太长的时候，我们可以采用适当缩写的形式。

### 不要使用 bigkey

> 哥，我就中招了，导致报错获取不到连接。

**因为 Redis 是单线程执行读写指令，如果出现`bigkey` 的读写操作就会阻塞线程，降低 Redis 的处理效率。**

`bigkey`包含两种情况：

- 键值对的 `value`很大，比如 `value`保存了 `2MB`的 `String`数据；
- 键值对的 `value`是集合类型，元素很多，比如保存了 5 万个元素的 `List` 集合。

虽然 Redis 官方说明了 `key`和`string`类型 `value`限制均为`512MB`。

**防止网卡流量、慢查询，`string`类型控制在`10KB`以内，`hash、list、set、zset`元素个数不要超过 5000。**

> 哥，如果业务数据就是这么大咋办？比如保存的是《金瓶梅》这个大作。

我们还可以通过 `gzip` 数据压缩来减小数据大小:

```java
/**
 * 使用gzip压缩字符串
 */
public static String compress(String str) {
    if (str == null || str.length() == 0) {
        return str;
    }

    try (ByteArrayOutputStream out = new ByteArrayOutputStream();
    GZIPOutputStream gzip = new GZIPOutputStream(out)) {
        gzip.write(str.getBytes());
    } catch (IOException e) {
        e.printStackTrace();
    }
    return new sun.misc.BASE64Encoder().encode(out.toByteArray());
}

/**
 * 使用gzip解压缩
 */
public static String uncompress(String compressedStr) {
    if (compressedStr == null || compressedStr.length() == 0) {
        return compressedStr;
    }
    byte[] compressed = new sun.misc.BASE64Decoder().decodeBuffer(compressedStr);;
    String decompressed = null;
    try (ByteArrayOutputStream out = new ByteArrayOutputStream();
    ByteArrayInputStream in = new ByteArrayInputStream(compressed);
    GZIPInputStream ginzip = new GZIPInputStream(in);) {
        byte[] buffer = new byte[1024];
        int offset = -1;
        while ((offset = ginzip.read(buffer)) != -1) {
            out.write(buffer, 0, offset);
        }
        decompressed = out.toString();
    } catch (IOException e) {
        e.printStackTrace();
    }
    return decompressed;
}
```

**集合类型**

如果集合类型的元素的确很多，我们可以将一个大集合拆分成多个小集合来保存。

### 使用高效序列化和压缩方法

为了节省内存，我们可以使用高效的序列化方法和压缩方法去减少 `value`的大小。

`protostuff`和 `kryo`这两种序列化方法，就要比 `Java`内置的序列化方法效率更高。

上述的两种序列化方式虽然省内存，但是序列化后都是二进制数据，可读性太差。

通常我们会序列化成 `JSON`或者 `XML`，为了避免数据占用空间大，我们可以使用压缩工具（snappy、 gzip）将数据压缩再存到 Redis 中。

### 使用整数对象共享池

Redis 内部维护了 0 到 9999 这 1 万个整数对象，并把这些整数作为一个共享池使用。

即使大量键值对保存了 0 到 9999 范围内的整数，在 Redis 实例中，其实只保存了一份整数对象，可以节省内存空间。

需要注意的是，有两种情况是不生效的：

1. Redis 中设置了 `maxmemory`，而且启用了 `LRU`策略（`allkeys-lru 或 volatile-lru 策略`），那么，整数对象共享池就无法使用了。

   > 这是因为 LRU 需要统计每个键值对的使用时间，如果不同的键值对都复用一个整数对象就无法统计了。

2. 如果集合类型数据采用 ziplist 编码，而集合元素是整数，这个时候，也不能使用共享池。

   > 因为 ziplist 使用了紧凑型内存结构，判断整数对象的共享情况效率低

## 命令使用规范

有的命令的执行会造成很大的性能问题，我们需要格外注意。

### 生产禁用的指令

Redis 是单线程处理请求操作，如果我们执行一些涉及大量操作、耗时长的命令，就会严重阻塞主线程，导致其它请求无法得到正常处理。

- KEYS：该命令需要对 Redis 的全局哈希表进行全表扫描，严重阻塞 Redis 主线程；

  > 应该使用 SCAN 来代替，分批返回符合条件的键值对，避免主线程阻塞。

- FLUSHALL：删除 Redis 实例上的所有数据，如果数据量很大，会严重阻塞 Redis 主线程；

- FLUSHDB，删除当前数据库中的数据，如果数据量很大，同样会阻塞 Redis 主线程。

  > 加上 ASYNC 选项，让 FLUSHALL，FLUSHDB 异步执行。

我们也可以直接禁用，用`rename-command`命令在配置文件中对这些命令进行重命名，让客户端无法使用这些命令。

### 慎用 MONITOR 命令

MONITOR 命令会把监控到的内容持续写入输出缓冲区。

如果线上命令的操作很多，输出缓冲区很快就会溢出了，这就会对 Redis 性能造成影响，甚至引起服务崩溃。

所以，除非十分需要监测某些命令的执行（例如，Redis 性能突然变慢，我们想查看下客户端执行了哪些命令）我们才使用。

### 慎用全量操作命令

比如获取集合中的所有元素（HASH 类型的 hgetall、List 类型的 lrange、Set 类型的 smembers、zrange 等命令）。

这些操作会对整个底层数据结构进行全量扫描 ，导致阻塞 Redis 主线程。

> 码哥，如果业务场景就是需要获取全量数据咋办？

有两个方式可以解决：

1. 使用 `SSCAN、HSCAN`等命令分批返回集合数据；
2. 把大集合拆成小集合，比如按照时间、区域等划分。

## 数据保存规范

### 冷热数据分离

虽然 Redis 支持使用 RDB 快照和 AOF 日志持久化保存数据，但是，这两个机制都是用来提供数据可靠性保证的，并不是用来扩充数据容量的。

不要什么数据都存在 Redis，应该作为缓存保存**热数据** ，这样既可以充分利用 Redis 的高性能特性，还可以把宝贵的内存资源用在服务热数据上。

### 业务数据隔离

不要将不相关的数据业务都放到一个 Redis 中。一方面避免业务相互影响，另一方面避免单实例膨胀，并能在故障时降低影响面，快速恢复。

### 设置过期时间

在数据保存时，我建议你根据业务使用数据的时长，设置数据的过期时间。

写入 Redis 的数据会一直占用内存，如果数据持续增多，就可能达到机器的内存上限，造成内存溢出，导致服务崩溃。

### 控制单实例的内存容量

建议设置在 2~6 GB 。这样一来，无论是 RDB 快照，还是主从集群进行数据同步，都能很快完成，不会阻塞正常请求的处理。

### 防止缓存雪崩

避免集中过期 key 导致缓存雪崩。

> 哥，什么是缓存雪崩？

当某一个时刻出现大规模的缓存失效的情况，那么就会导致大量的请求直接打在数据库上面，导致数据库压力巨大，如果在高并发的情况下，可能瞬间就会导致数据库宕机。

## 运维规范

1. 使用 Cluster 集群或者哨兵集群，做到高可用；
2. 实例设置最大连接数，防止过多客户端连接导致实例负载过高，影响性能。
3. 不开启 AOF 或开启 AOF 配置为每秒刷盘，避免磁盘 IO 拖慢 Redis 性能。
4. 设置合理的 repl-backlog，降低主从全量同步的概率
5. 设置合理的 slave client-output-buffer-limit，避免主从复制中断情况发生。
6. 根据实际场景设置合适的内存淘汰策略。
7. 使用连接池操作 Redis。