---
index_img: https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20211129140533
title: Redis高可用集群哨兵图文教程
date: 2021-12-01 21:11:12
updated: 2021-12-01 21:11:12
tags:
  - Redis
categories:
  - 技术
comments: true
---
## 一. redis高可用方案–sentinel（哨兵模式）

当我们搭建好redis主从复制方案后会发现一个问题，那就是当主服务器宕机后，需要手动把一台从服务器切换为主服务器，这就需要人工干预，费事费力，同时在手动切过程中也会导致redis服务器写入功能不可用。所以需要一种方法可以完成Master故障后可以自动的将一个Slave切换为Master，这个时候就有了sentinel哨兵模式。

### 哨兵模式简介：

sentinel是官方提供的高可用方案，其原理是哨兵通过发送命令，等待Redis服务器响应，从而监控运行的多个Redis实例。同时 sentinel是一个分布式系统，可以在一个架构中运行多个Sentinel进程，可以做到sentinel的高可用。

![图片](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20211129140533)

#### sentinel工作过程：

- 通过向主服务器和从服务器发送ping命令，让服务器返回运行状态。
- 当哨兵监测到master宕机，会自动将一个slave切换成master，然后通过发布订阅模式通知其他的从服务器，修改配置文件，让它们切换主机。

#### 关于sentinel的三个定时任务：

- 每1秒每个sentinel对其他sentinel和redis节点执行ping操作，心跳检测。
- 每10秒每个sentinel会对master和slave执行info命令，目的是发现slave结点，确定主从关系。
- 每2秒每个sentinel通过master节点的channel交换信息（pub/sub）。master节点上有一个发布订阅的频道(sentinel:hello)。sentinel节点通过`__sentinel__:hello`频道进行信息交换(对节点的"看法"和自身的信息)，达成共识.

#### sentinel网络：

sentinel是一个分布式系统，可以在一个架构中运行多个Sentinel进程。所以监控同一个Master的Sentinel会自动连接，组成一个分布式的Sentinel网络，互相通信并交换彼此关于被监视服务器信息。

![图片](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20211129140558)

#### sentinel网络故障修复原理：

**1.主观下线：**

当主服务器发生故障时，此时一个sentinel发现了故障，系统并不会马上进行failover过程（这个现象称为主观下线），它会向网络中的其他Sentinel进行确认。

**2.客观下线：**

接着其他Sentinel也陆续发现故障，这个时候其中一个Sentinel就会发起投票。一定数量的哨兵(在配置文件中指定)确认Master被标记为主观下线，此时将Master标记为客观下线。

**3.sentinel的leader选举：**

要想完成故障切换（将故障master剔除，并将一个slave提升为master）就必须先选举一个leader。最先发现故障的sentinel向其他哨兵发起请求成为leader，其他哨兵在没有同意别的哨兵的leader请求时，就会把票投给该sentinel。当半数以上的sentinel投票通过后就认定该sentinel为leader。接下来的故障切换有该leader完成。

**4.master选举：**

leader选好后将故障master剔除，从slave中挑选一个成为master。遵照的原则如下：

- slave的优先级
- slave从master那同步的数据量，那个slave多就优先。

**5.新Master再通过发布订阅模式通知所有sentinel更新监控主机信息。**

**6.故障的主服务器修复后将成为从服务器继续工作。**

故障发生

![图片](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20211129140612)

故障切换

![图片](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20211129140620)

Master重新上线后

![图片](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20211129140633)

### 哨兵模式配置

本实验在一台机器上完成，创建不同端口的redis实例。

![图片](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20211129140652)

**1.创建redis实例**

```bash
[root@redis ~]# nohup redis-server --port 6380 >> /data/redis/log/6380.log 2>&1 & 
[root@redis ~]# nohup redis-server --port 6381 >> /data/redis/log/6381.log 2>&1 & 
[root@redis ~]# nohup redis-server --port 6382 >> /data/redis/log/6382.log 2>&1 & 

[root@redis ~]# ps -ef |grep redis
root      16421  16314  0 03:01 pts/1    00:00:00 redis-server *:6380
root      16427  16314  0 03:01 pts/1    00:00:00 redis-server *:6381
root      16431  16314  0 03:01 pts/1    00:00:00 redis-server *:6382
root      16436  16314  0 03:01 pts/1    00:00:00 grep --color=auto redis
```

**2.连接数据库并设置主从复制**

```bash
[root@redis ~]# redis-cli -p 6380
127.0.0.1:6380> 

[root@redis ~]# redis-cli -p 6381
127.0.0.1:6381> SLAVEOF 127.0.0.1 6380    #将6380设置为master
OK
[root@redis ~]# redis-cli -p 6382
127.0.0.1:6382> SLAVEOF 127.0.0.1 6380
OK
```

**3.搭建哨兵模式集群**

创建sentinel配置文件

```bash
[root@redis conf]# cat sentinel1.conf 
port 26300                              #指定sentinel进程端口号
sentinel monitor redis1 127.0.0.1 6380 2  #Sentinel monitor <name> <ip> <port> <quorum>

[root@redis conf]# cat sentinel2.conf 
port 26301
sentinel monitor redis1 127.0.0.1 6380 2
```

- `name` ：redis主服务名称，可以自行命名，但是在一个sentinel网络中，一个redis主服务只能有一个名称；
- `ip和port` ：redis主服务的IP地址和端口号.
- `quorum` ：表示要将这个主服务器判断为失效并下线至少需要2个sentinel同意
- `protected-mode` ：关闭保护模式（默认情况下，redis node和sentinel的protected-mode都是yes，在搭建集群时，若想从远程连接redis集群，需要将redis node和sentinel的protected-mode修改为no，若只修改redis node，从远程连接sentinel后，依然是无法正常使用的，且sentinel的配置文件中没有protected-mode配置项，需要手工添加。依据redis文档的说明，若protected-mode设置为no后，需要增加密码证或是IP限制等保护机制，否则是极度危险的。）

启动sentinel

```bash
[root@redis ~]# nohup redis-sentinel /data/redis/conf/sentinel1.conf >> /data/redis/log/sentinel1.log 2>&1 &
[1] 16522
[root@redis ~]# nohup redis-sentinel /data/redis/conf/sentinel2.conf >> /data/redis/log/sentinel2.log 2>&1 &
[2] 16526
[root@redis ~]# ps -ef |grep sentinel
root      16522  16440  0 03:55 pts/2    00:00:00 redis-sentinel *:26300 [sentinel]
root      16526  16440  0 03:55 pts/2    00:00:00 redis-sentinel *:26301 [sentinel]
```

**4.测试。模拟主节点故障，查看故障后主从环境改变**

关闭主节点

```bash
[root@redis ~]#  ps -ef |grep redis-server
root      16604  16440  0 04:12 pts/2    00:00:02 redis-server *:6381
root      16608  16440  0 04:12 pts/2    00:00:03 redis-server *:6382
root      16702  16314  0 04:46 pts/1    00:00:00 redis-server *:6380
[root@redis ~]# kill 16702
```

此时查看6381和6382的日志文件，在6382的日志文件中发现了如下内容，说明此时已经将6381切换为主节点。

![图片](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20211129140829)

再次启动6380时，6380成为slave，6381时master

![图片](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20211129141526)

最后查看一下sentinel配置文件：

```bash
cat /data/redis/conf/sentinel2.conf
port 26301
sentinel myid 74cdfbb5ae55a77ad4d05d5d9d50fd64725e192a
# Generated by CONFIG REWRITE
dir "/root"
protected-mode no
sentinel deny-scripts-reconfig yes
sentinel monitor redis1 127.0.0.1 6381 1     #主节点
sentinel config-epoch redis1 1
sentinel leader-epoch redis1 1
sentinel known-replica redis1 127.0.0.1 6382    #从节点
sentinel known-replica redis1 127.0.0.1 6380
sentinel known-sentinel redis1 127.0.0.1 26300 9539652da78b0385479a827e753deceaef864989
sentinel current-epoch 1
```

## 二. redis高可用方案–集群

使用哨兵模式，解决了主节点故障自动切换的问题，但是却不可以动态扩充redis。所以在redis3.0之后提出了集群模式。

##### redis集群设计：

redis集群采用无中心结构，每个节点保存数据和整个集群状态,每个节点都和其他所有节点连接。

![图片](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20211129141542)

特点：

- 所有的redis节点彼此互联(PING-PONG机制),内部使用二进制协议优化传输速度和带宽。
- 节点的失效是通过集群中超过半数的节点检测失效时才生效。
- 集群是一个整体，客户端与redis节点直连,不需要中间proxy层.客户端不需要连接集群所有节点,连接集群中任何一个可用节点即可。
- Redis集群预分好16384个桶，当需要在 Redis 集群中放置一个 key-value 时，根据 CRC16(key) mod 16384的值，决定将一个key放到哪个桶中。

##### redis集群节点分配和数据分配

**节点分配：**

Redis集群预分好16384个桶，采用哈希槽 (hash slot)的方式来平均分配16384个slot 。以三个节点为例，

- 节点1：0－5460;
- 节点2：5461－10922;
- 节点3：10923－16383.

若存入一个值，按照哈希槽算法得到6587，那么就会将数据存入节点2。取数据时也是从节点2上取。

**当新增一个节点时：**

采用从各个节点的前面各拿取一部分槽到新节点上，如添加节点4，哈希槽就为，0-1364,5461-6826,10923-12287。

##### redis集群的主从模式

为了保证数据高可用，集群应建立在主从基础之上。一个主节点对应一个从节点。主节点提供数据存取，从节点提供数据读取，当主节点故障后，就会有这个从节点选取一个来充当主节点，从而保证集群正常运行。

> 但是在一个集群中，一对主从节点同时故障，那么集群将失去服务能力。

##### redis集群搭建：

redis集群中至少应该有奇数个节点，所以至少有三个节点，每个节点至少有一个备份节点，所以本次实验使用6个节点（主节点、备份节点由redis-cluster集群确定）。

实验在两台机器进行，每台机器启动三个基于不同端口redis实例，6个实例两两对应主从。

![图片](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20211129141640)

![图片](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20211129141646)

**1.创建一个目录redis_cluster,用来存放每个实例所用的配置文件.**

```bash
[root@redis redis]# mkdir redis_cluster
[root@redis redis]# mkdir -p redis_cluster/7001
[root@redis redis]# cp -r conf/redis.conf redis_cluster/
```

**2.修改配置文件，将修改好的文件复制5份分别放置不同redis配置目录下。**

修改配置文件

```bash
vim 7001/redis.conf修改一下几项
bind 192.168.126.162  (本机IP)
port 7001                                    #redis端口
daemonize yes                               #redis在后台启动
logfile "/data/redis/log/logs"
pidfile /var/run/redis_7001.pid
cluster-enabled yes                    #开启集群功能
cluster-config-file nodes-7001.conf    #集群配置文件
cluster-node-timeout 5000          #姐点之间通讯时间
appendonly yes                      #开启AOF持久化方式
```

创建目录7002 到7006，其中7004到7005创建在`192.168.126.161`上。再把配置文件复制到`700*`目录下，修改配置文件中端口号和ip。

```bash
[root@redis redis_cluster]# cp -rp 7001 7002
[root@redis redis_cluster]# cp -rp 7001 7003

[root@redis redis_cluster]# vim 7002/redis.conf          #修改端口为7002 
[root@redis redis_cluster]# vim 7003/redis.conf 
将配置文件目录拷贝到192.168.126.161上
[root@redis redis_cluster]# scp -rp 7001 192.168.126.161:/usr/local/redis/redis_cluster/

修改配置文件（端口号和ip）
[root@centosm redis_cluster]# ls
7004  7005  7006
```

**3.启动redis，可以使用脚本启动**

```bash
[root@redis redis_cluster]# ls
7001  7002  7003  start.sh
[root@redis redis_cluster]# cat start.sh     #另一台机器相同操作
cd 7001
redis-server redis.conf
cd ../7002
redis-server redis.conf
cd ../7003
redis-server redis.conf
#192.168.126.162
[root@redis redis_cluster]# ps -ef |grep redis
root       1757      1  2 12:36 ?        00:00:00 redis-server 192.168.126.162:7001 [cluster]
root       1762      1  1 12:36 ?        00:00:00 redis-server 192.168.126.162:7002 [cluster]
root       1767      1  1 12:36 ?        00:00:00 redis-server 192.168.126.162:7003 [cluster]


#192.168.126.161
[root@centosm redis_cluster]# ps -ef |grep redis
root      11906      1  0 23:07 ?        00:00:00 redis-server 192.168.126.161:7004 [cluster]
root      11911      1  0 23:07 ?        00:00:00 redis-server 192.168.126.161:7005 [cluster]
root      11913      1  0 23:07 ?        00:00:00 redis-server 192.168.126.161:7006 [cluster]
```

**4.开始创建集群**

搭建集群的话，如果redis版本小于5.0就需要使用一个工具redis-trib（脚本文件），这个工具在redis解压文件的源代码里。因为这个工具是一个ruby脚本文件，所以这个工具的运行需要ruby的运行环境，所以需要安装ruby

```bash
yum install ruby -y

yum install rubygems -y

gem install redis
```

当前redis版本大于5.0，所以不用ruby，可以直接创建。

```bash
redis-cli --cluster create 192.168.126.162:7001 192.168.126.162:7002 192.168.126.162:7003 192.168.126.161:7004 192.168.126.161:7005 192.168.126.161:7006 --cluster-replicas 1
```

`–cluster-replicas 1`：主从比例为1:1

```bash
[root@redis redis_cluster]# redis-cli --cluster create 192.168.126.162:7001 192.168.126.162:7002 192.168.126.162:7003 192.168.126.161:7004 192.168.126.161:7005 192.168.126.161:7006 --cluster-replicas 1
>>> Performing hash slots allocation on 6 nodes...      #对6个节点进行哈希槽位分配，实际分配三个主节点即可。
Master[0] -> Slots 0 - 5460
Master[1] -> Slots 5461 - 10922
Master[2] -> Slots 10923 - 16383
Adding replica 192.168.126.161:7006 to 192.168.126.162:7001  #三个主节点7001 7004 7002
Adding replica 192.168.126.162:7003 to 192.168.126.161:7004   #三个从节点7006 7005 7003
Adding replica 192.168.126.161:7005 to 192.168.126.162:7002
M: fb89e991f2fca476964195f496428c0de3e57f76 192.168.126.162:7001
   slots:[0-5460] (5461 slots) master
M: 54788eed17c99719f0d9e49b4933f8fc6e900cd9 192.168.126.162:7002
   slots:[10923-16383] (5461 slots) master
S: 0d4d849e80a4f12e546fa3df7fcec42cb65951b2 192.168.126.162:7003
   replicates 4fbffafb9088e65f60526147f4bff5260ea897f0
M: 4fbffafb9088e65f60526147f4bff5260ea897f0 192.168.126.161:7004
   slots:[5461-10922] (5462 slots) master
S: 6f44fa89577fd53ff8d703390e3908b7db5cb88c 192.168.126.161:7005
   replicates 54788eed17c99719f0d9e49b4933f8fc6e900cd9
S: 3f224e631bffba6d3978412df83c11b9d53f5799 192.168.126.161:7006
   replicates fb89e991f2fca476964195f496428c0de3e57f76
Can I set the above configuration? (type 'yes' to accept): yes
>>> Nodes configuration updated
>>> Assign a different config epoch to each node
>>> Sending CLUSTER MEET messages to join the cluster
Waiting for the cluster to join
......
>>> Performing Cluster Check (using node 192.168.126.162:7001)
M: fb89e991f2fca476964195f496428c0de3e57f76 192.168.126.162:7001
   slots:[0-5460] (5461 slots) master
   1 additional replica(s)
S: 0d4d849e80a4f12e546fa3df7fcec42cb65951b2 192.168.126.162:7003
   slots: (0 slots) slave
   replicates 4fbffafb9088e65f60526147f4bff5260ea897f0
M: 54788eed17c99719f0d9e49b4933f8fc6e900cd9 192.168.126.162:7002
   slots:[10923-16383] (5461 slots) master
   1 additional replica(s)
S: 6f44fa89577fd53ff8d703390e3908b7db5cb88c 192.168.126.161:7005
   slots: (0 slots) slave
   replicates 54788eed17c99719f0d9e49b4933f8fc6e900cd9
S: 3f224e631bffba6d3978412df83c11b9d53f5799 192.168.126.161:7006
   slots: (0 slots) slave
   replicates fb89e991f2fca476964195f496428c0de3e57f76
M: 4fbffafb9088e65f60526147f4bff5260ea897f0 192.168.126.161:7004
   slots:[5461-10922] (5462 slots) master
   1 additional replica(s)
[OK] All nodes agree about slots configuration.   #所有节点同意分配hash槽
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.        #分配完毕，创建完成
```

最后集群主从对应关系

![图片](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20211129141719)

可以看出主从节点在两个节点随机分配，且一对对应主从服务不会分配到同一台机器上。即使一台机器损坏，也不会影响redis继续提供服务。

*来源：blog.csdn.net/wdwangye/article/details/113084351*

