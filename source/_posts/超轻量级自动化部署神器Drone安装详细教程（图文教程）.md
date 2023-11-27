---
index_img: https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210719173546.png
title: 超轻量级自动化部署神器Drone安装详细教程（图文教程）
date: 2021-08-09 23:30:37
updated: 2021-08-09 23:30:37
categories:
  - 技术
comments: true
---
## 前言

现在我们在日常开发或者生产环境中，往往会经常弄一套自动化部署方案来节约时间成本。现在比较流行的一种就是`Gitlab+Jenkins`实现方案，但是这种方案占用内存还是比较大，需要的服务器内存也得8G左右，不然很难流畅运行，而且部署起来也不快。最近小翔发现了一款神器 [Drone](https://www.drone.io/) ,轻量级的CI/CD工具，我拿来结合 Gogs 使用所消耗的内存占用都不到1G，这里就给大家聊聊这款工具。

## Drone简介

Drone是一款基于go编写的容器技术持续集成工具，可以直接使用YAML配置文件即可完成自动化构建、测试、部署任务。



## 优势：

drone引入pipline的概念，整个build过程由多个stage组成，每一个stage都是docker。

- 各stage间可以通过共享宿主机的磁盘目录, 实现build阶段的数据共享和缓存基础数据, 实现加速下次build的目标
- 各stage也可以共享宿主机的docker环境，实现共享宿主机的docker image, 不用每次build都重新拉取base image，减少build时间
- 可以并发运行。多个build可以并发运行，单机并发数量由服务器cpu数决定。 由开发者负责打包image和流程控制。Docker-in-docker,这一点非常重要，一切都在掌握之中。相比jenkins的好处是，所有的image都是开发者提供，不需要运维参与在CI服务器上部署各种语言编译需要的环境。
  是DevOps的最佳实践！



## 使用教程

因为本篇文章是用 `Gogs` 的git版本管理存储代码，安装可以参考我上一篇文章， [Gogs安装部署](https://www.weiye.link/200.html/)



### Drone下载安装

使用Docker安装几秒就完成

- 下载Drone和Runner的镜像

```bash
# Drone的Server
docker pull drone/drone:1
# Drone的Runner
docker pull drone-runner-docker:1
```

- 这里有个Server和Runner的概念，我们先来理解下；

- - Server：为Drone的管理提供了Web页面，用于管理从Git上获取的仓库中的流水线任务。
  - Runner：一个单独的守护进程，会轮询Server，获取需要执行的流水线任务，之后执行。

#### 1. 安装 `Drone server`

```bash
docker run \
  -v /www/wwwroot/data/docker/drone:/data \
  -e DRONE_AGENTS_ENABLED=true \
  -e DRONE_GOGS_SERVER=https://gogs.weiye.link \
  -e DRONE_RPC_SECRET=dronerpc666 \
  -e DRONE_SERVER_HOST=192.168.31.114:3080 \
  -e DRONE_SERVER_PROTO=http \
  -e DRONE_USER_CREATE=username:weiye,admin:true \
  -e TZ="Asia/Shanghai" \
  -p 3080:80 \
  --restart=always \
  --detach=true \
  --name=drone \
  drone/drone:1
```

- 这里的配置参数比较多，下面统一解释下；

- - DRONE_GOGS_SERVER：用于配置Gogs服务地址，可以直接是IP `http://192.168.31.114:10080`
  - DRONE_RPC_SECRET：Drone的共享秘钥，用于验证连接到server的rpc连接，server和runner需要提供同样的秘钥。
  - DRONE_SERVER_HOST：用于配置Drone server外部可访问的地址。
  - DRONE_SERVER_PROTO：用于配置Drone server外部可访问的协议，必须是http或https。
  - DRONE_USER_CREATE：创建一个管理员账号，该账号需要在Gogs中注册好。

#### 2. 安装`drone-runner-docker`

- 当有需要执行的任务时，会启动临时的容器来执行流水线任务

```bash
docker run -d \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -e DRONE_RPC_PROTO=http \
  -e DRONE_RPC_HOST=192.168.31.114:3080 \
  -e DRONE_RPC_SECRET=dronerpc666 \
  -e DRONE_RUNNER_CAPACITY=2 \
  -e DRONE_RUNNER_NAME=runner-docker \
  -e TZ="Asia/Shanghai" \
  -p 3000:3000 \
  --restart always \
  --name runner-docker \
  drone/drone-runner-docker:1
```

- 这里的配置参数比较多，下面统一解释下。

- - DRONE_RPC_PROTO：用于配置连接到Drone server的协议，必须是http或https。
  - DRONE_RPC_HOST：用于配置Drone server的访问地址，runner会连接到server获取流水线任务并执行。
  - DRONE_RPC_SECRET：用于配置连接到Drone server的共享秘钥。
  - DRONE_RUNNER_CAPACITY：限制runner并发执行的流水线任务数量。
  - DRONE_RUNNER_NAME：自定义runner的名称。

### Drone使用

打开Drone， IP + 3080 登录Gogs的管理员账号就可以进入控制台了，访问地址：http://192.168.31.114:3080/

![image-20210719161938832](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210719161939.png)

- 登录进去后，我们可以看到已经有了gogs中的项目，如果没有，可以点击右上角的 `SYNC` 按钮。

![image-20210719162735591](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210719162735.png)

- 配置仓库，使Drone创建的容器能够挂载到宿主机上
- 选中仓库点击 `ACTIVATE`后， `设置仓库`  -> `勾选 Trusted` -> `SAVE` 保存。

![image-20210719163140175](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210719163140.png)

- 保存成功后我们进入 Gogs 管理面板，打开刚才操作的这个项目查看 WEB钩子是否配置成功

![image-20210719163327207](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210719163327.png)

### （可选） 添加Secret

drone保证了安全访问，我们不用在配置文件明文输出**密码**等**敏感值**，可以添加 `Secret` ，如果觉得没有必要也不用添加，直接在后面的 `drone.yml`文件中配置明文密码就行。

![image-20210719174819283](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210719174819.png)



### Drone.yml详解

- 当我们向Git仓库Push代码时，会自动触发Web钩子，然后Drone就会从Git仓库Clone代码，再通过项目目录下的`.drone.yml`配置，执行相应的流水线，接下来我们来看看这个脚本是如何写的。

在仓库的根目录下创建`.drone.yml`配置文件
配置文件参考如下配置：

```yaml
kind: pipeline # 定义对象类型，还有secret和signature两种类型
type: docker # 定义流水线类型，还有kubernetes、exec、ssh等类型
name: drone-study # 定义流水线名称

steps: # 定义流水线执行步骤，这些步骤将顺序执行
  - name: package # 1. 流水线名称（maven打包）
    pull: if-not-exists
    image: maven:ibmjava-alpine # 定义创建容器的Docker镜像
    volumes: # 将容器内目录挂载到宿主机，仓库需要开启Trusted设置
      - name: maven-cache
        path: /root/.m2 # 将maven下载依赖的目录挂载出来，防止重复下载
        # 挂载宿主机的目录
      - name: maven-build
        path: /app/build/drone-study # 将应用打包好的Jar和执行脚本挂载出来
    commands: # 定义在Docker容器中执行的shell命令,这里是复制到maven镜像容器里面的，区分开项目就行了
      - mvn clean package -Dmaven.test.skip=true
      - cp target/drone.jar /app/build/drone-study/drone.jar
      - cp Dockerfile /app/build/drone-study/Dockerfile
      - cp run.sh /app/build/drone-study/run.sh
      - cp restart.sh /app/build/drone-study/restart.sh

  - name: build-start # 2. 流水线名称（ssh默认人工操作打包好的jar包）
    image: appleboy/drone-ssh # ssh工具镜像
    when:
      branch:
        include:
          - master
        exclude:
          - dev
    settings:
      host: weiye.link # 远程连接地址,可以是IP可以是域名
      username: root # 远程连接账号
#      password: 123456 #明文密码
#      password:
#        from_secret: ssh_password # 从Secret中读取SSH密码
      key:
        from_secret: ssh_key # 从Secret中读取SSH密钥
      port: 22 # 远程连接端口
      command_timeout: 10m # 远程执行命令超时时间
      script_stop: false # 设置为false，遇到第一次错误会继续运行后面的命令
      script:
        - cd /www/wwwroot/data/maven/build/drone-study # 进入宿主机构建目录,可根据自己选择目录
        - chmod +x run.sh # 更改为可执行脚本
        - ./run.sh # 运行脚本打包应用镜像并运行
  - name: re-start
    image: appleboy/drone-ssh
    when:
      branch:
        include:
          - dev
        exclude:
          - master
    settings:
      host: weiye.link
      username: root
      key:
        from_secret: ssh_key
      port: 22
      command_timeout: 20m
      script_stop: false
      script:
        - cd /www/wwwroot/data/maven/build/drone-study
        - chmod +x restart.sh # 更改为可执行脚本
        - ./restart.sh # 重启docker容器
  - name: notify # 3. 通知（这里使用了钉钉通知，可以使用微信通知、邮件通知等）
    pull: if-not-exists
    image: guoxudongdocker/drone-dingtalk:latest
    settings:
      token:
        from_secret: dingtalk_token
      type: markdown
      message_color: true
      message_pic: true
      sha_link: true
    when:
      status: [failure, success]

volumes: # 定义流水线挂载目录，用于共享数据
  - name: maven-build
    host:
      path: /www/wwwroot/data/maven/build/drone-study # 从宿主机中挂载的目录
  - name: maven-cache
    host:
      path: /www/wwwroot/data/maven/cache

```



> **注意：**
>
> 1. 确定好自己宿主机挂载的目录
> 2. `build-start` 和 `re-start` 我判断了主分支（master）和子分支（dev）提交执行不同的步骤，这两个是对冲的，每次只会执行其中一个，主要是优化不用每次都重新构建镜像，通过挂载jar包方式启动，宿主机里的jar包更换后直接重启容器就行了，节省了一部分时间。不需要的可以直接删掉`when` 判断就行了
>
> 可以学习一下 drone 的判断语法： [https://docs.drone.io/pipeline/docker/syntax/conditions/#by-branch](https://docs.drone.io/pipeline/docker/syntax/conditions/#by-branch)

- `run.sh`执行脚本可以实现打包应用和运行容器镜像，配置如下：

```sh
#!/usr/bin/env bash
# 定义应用组名
group_name='xiang'
# 定义应用名称
app_name='drone-study'
# 定义应用版本
app_version='v1'
docker stop ${app_name}
echo '----stop container----'
docker rm ${app_name}
echo '----rm container----'
docker rmi ${group_name}/${app_name}:${app_version}
echo '----rm image----'
# 打包编译docker镜像
docker build -t ${group_name}/${app_name}:${app_version} .
echo '----build image----'
docker run -it -p 6003:6003 --name ${app_name} \
-d ${group_name}/${app_name}:${app_version}
echo '----start container----'

```

- `restart.sh`执行脚本可以实现重启容器，配置如下：

```sh
#!/usr/bin/env bash
# 定义应用名称
app_name='drone-study'

docker restart ${app_name}
echo '----restart container----'
```



- `Dockerfile`文件配置如下：

```dockerfile
# Docker image for springboot file run
# VERSION 1.0
# Author: Xiang
# 基础镜像使用java
FROM openjdk:8-jdk-alpine
ADD drone.jar app.jar
EXPOSE 6003
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","app.jar"]
```



- 文件编写完成后，git push到仓库中，gogs会通知drone进行部署，drone找到`.drone.yml`配置文件，就会按照配置文件中的步骤进行构建了，部署期间可以在drone中查看到每一步的部署情况

![image-20210719173546620](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210719173546.png)

- 钉钉通知成功

![image-20210720095042292](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/20210720095042.png)



## 答疑

1. **yml文件是否必须存放在git repository根目录**

   是的，必须在根目录

2. **yml文件名是否必须是.drone.yml**

   默认是文件名是.drone.yml，也可以能过drone web管理界面修改yml的文件名

3. **git提交代码如何让drone跳过本次提交，不执行pipeline**

   提交代码时通过备注增加[CI SKIP]跳过，如： git commit –m “first commit [CI SKIP]”

4. **drone web界面repository设置项Project settings中没有Trusted**

   Drone Server启动时要指定DRONE_USER_CREATE参数，用来设置管理员帐号，只有用管理员帐号打开drone web界面才可以看到和设置Trusted

5. **钉钉通知Token如何获取**

   可以百度，都挺简单的，获取到token后再添加到drone控制台的 `Secrets`中，取名 `dingtalk_token`

## 总结

总体使用下来还是很不错的，构建速度上面可以再优化，使用脚本来定义流水线任务无疑更简单、更直观。Drone更加轻量级，内存占用少且响应速度快。如果团队人员不是很大的话，还是很推荐大家使用的！



参考资料：

- 官方文档：https://docs.drone.io/

项目源码地址：

 https://github.com/wilbur147/xiangStudy/tree/main/lab-drone

























