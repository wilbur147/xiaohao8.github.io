---
index_img: https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805104804277.png
title: Jenkins安装部署完整篇教程（图文）
date: 2023-07-17 19:23:33
updated: 2023-07-17 19:23:33
tags:
  - Jenkins
categories:
  - 运维部署
comments: true
---
## Jenkins+Gitlab自动化部署

### 使用场景

测试环境频繁发布让人烦不胜烦，干脆自己搭建一个自动化部署的应用，一劳永逸！ 



整个发布流程是，程序员提交gitlab代码后，gitlab webhook通知Jenkins触发构建Jenkins使用maven打包过后，然后Jenkins登录远程SSH服务器，执行Jar包启动

### 准备工作

- 需要准备一台服务器。
- Docker环境 [Docker安装教程](https://www.cnblogs.com/fuzongle/p/12781828.html)

### 安装步骤

#### 一、下载Jenkins镜像文件

启动Docker后，下载Jenkins镜像文件

```bash
docker pull jenkins/jenkins
```



![Snipaste_2022-08-05_10-12-47](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/Snipaste_2022-08-05_10-12-47.png) 

#### 二、创建挂载目录

创建Jenkins挂载目录并授权权限

```bash
mkdir -p /root/docker/data/jenkins/jenkins_home

chmod 777 /root/docker/data/jenkins/jenkins_home

# 普通用户启动时，如果报没有权限，就执行：chown -R 1000 /root/docker/data/jenkins/jenkins_home
# root用户启动时，如果报没有权限，就执行：chown -R root:root /root/docker/data/jenkins/jenkins_home
```



#### 三、创建并启动Jenkins容器

```bash
# 普通Jenkins用户启动
docker run \
-d \
-p 8777:8080 \
-p 50000:50000 \
--name jenkins \
--restart always \
-v /root/docker/data/jenkins/jenkins_home:/var/jenkins_home \
jenkins/jenkins

# root用户启动，有docker方式编排时可以将宿主机的docker直接映射进来就不用单独安装了
docker run -d -uroot -p 8777:8080 -p 50000:50000 --name jenkins \
-v /root/docker/data/jenkins/jenkins_home:/var/jenkins_home \
-v /home:/home \
-v /etc/localtime:/etc/localtime \
-v /var/run/docker.sock:/var/run/docker.sock \
-v /usr/bin/docker:/usr/bin/docker \
jenkins/jenkins
```



> **-d 后台运行镜像**
>
> **-p 8777:8080 将镜像的8080端口映射到服务器的8777端口。**
>
> **-v /root/docker/data/jenkins/jenkins_home:/home/jenkins_home**
>
> - **`/root/docker/data/jenkins/jenkins_home`目录为容器jenkins工作目录映射至服务器的目录，我们将目录挂载到这个位置，方便后续更新镜像后继续使用原来的工作目录。**
> - **`/var/jenkins_home` 容器jenkins的工作目录**
>
> **--name jenkins 给容器起一个别名**

![image-20220805102739348](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805102739348.png) 

#### 四、查看Jenkins是否启动成功

```bash
docker ps -a
```

执行命令，出现如下图状态 `Up` 说明启动成功

![image-20220805102939604](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805102939604.png) 

#### 五、查看容器日志

```bash
docker logs jenkins
```

![image-20220805103050379](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805103050379.png) 

#### 六、配置镜像加速

```bash
cd /root/docker/data/jenkins/jenkins_home/
```

![image-20220805104055022](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805104055022.png) 

**修改 vim hudson.model.UpdateCenter.xml里的内容**

**修改前**

![image-20220805104405188](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805104405188.png) 



将 url 修改为 清华大学官方镜像：https://mirrors.tuna.tsinghua.edu.cn/jenkins/updates/update-center.json

**修改后**

![image-20220805104522133](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805104522133.png) 

#### 七、访问Jenkins

输入你的服务器IP+端口， 本篇示例端口为：8777

![image-20220805104804277](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805104804277.png) 

**管理员密码获取方法，编辑initialAdminPassword文件查看，把密码输入登录中的密码即可，开始使用**

编辑 `vim /root/docker/data/jenkins/jenkins_home/secrets/initialAdminPassword`

![image-20220805105039791](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805105039791.png) 

**至此，安装篇完成**

### **Jenkins基本配置**

进入网址后会有新手引导，我们依次操作

`选择安装推荐的插件` >>> `创建管理员账户` >>> `实例配置选择保存并完成`

![image-20220805114010116](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805114010116.png) 

这一步非常慢，需要耐心等待



#### 时区配置

设置Jenkins时区为北京时间

`Manage Jenkins` >>> `Script Console` >>> `执行脚本`

```shell
System.setProperty('org.apache.commons.jelly.tags.fmt.timeZone', 'Asia/Shanghai')
```

![image-20220805160410176](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805160410176.png) 

#### 安装构建和部署所需的插件

![image-20220805160529828](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805160529828.png) 



- **安装MAVEN插件**

![image-20220805160824454](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805160824454.png) 

- **安装GitLab插件**

![image-20220805161437311](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805161437311.png) 

- **安装SSH插件和Publish Over SSH插件**

Publish Over SSH：因为我的项目是使用 Docker 启动 Springboot 服务；所以在 Jenkins 需要操作目标服务器，进行Dockerfile 启动

![image-20220805161845567](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805161845567.png) 

- **安装 Docker 插件**

![image-20220805162530296](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805162530296.png) 

**最后安装完所有插件后，重启一下Jenkins（也可以命令重启）**

![image-20220805162748749](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805162748749.png) 

手动重启，直接使用URL重启即可：`http://192.168.124.51:8777/restart`，点击是，进行重启

#### 添加凭证

这里就是相当于配置一些 ssh，gitlab等 连接的凭证

![image-20220805163756200](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805163756200.png) 

![image-20220805163819682](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805163819682.png) 

![image-20220805163830905](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805163830905.png) 

#### Jenkins密钥连接时的，凭证密钥生成

> # 首先进入Jenkins容器内部，用 Jenkins 用户名登录
> `docker exec -it -u jenkins jenkins /bin/bash`
>
> # 进入 /var/jenkins_home 目录
> `cd /var/jenkins_home/`
>
> # 生成密钥，一路回车，生成文件路径 /var/jenkins_home/.ssh
> `ssh-keygen -t rsa -C "wkc120796@163.com"`
>
> # 将id_rsa.pub 放入到gitee或者gitlab或者github的ssh密钥中
> # 然后在Jenkins配置私钥
> # 配好了需要在后台 `/var/jenkins_home/workspace` 目录手动拉取一下远程仓库，会自动在.ssh目录生成 `known_hosts` 文件
> # 拉取完了就可以配置jinkins进行创建任务了



**输入 ssh 连接的相关账户密码**

![image-20220805164259966](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805164259966.png) 

#### 配置SSH remote hosts

**注：配置SSH连接Dockerfile所在服务器的相关信息，并添加凭证，最后测试连接并保存；**

![image-20220805164831548](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805164831548.png) 

![image-20220805164902318](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805164902318.png) 

![image-20220805165015842](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805165015842.png) 

#### 全局工具配置

##### JDK配置

![image-20220805165448571](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805165448571.png) 

![image-20220805165943877](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805165943877.png) 

需要登录 Orcale账号密码

![image-20220805170259010](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805170259010.png) 

> 账号 1 :amador.sun@foxmail.com密码:1211WaN!
>
> 账号 2 :amador.sun@qq.com密码:1211WaN!
>
> 账号 3 :hellooracle123@qq.com密码:1211WaN!
>
> 账号 4 :javacno.1@qq.com密码:1211WaN!
>
> 账号 5 :oracle-01@qq.com密码:1211WaN!
>
> 账号 6 :oracle-02@qq.com密码:1211WaN!

##### Git 安装配置

还是当前页面，配置git

![image-20220805170439144](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805170439144.png) 

##### MAVEN安装配置

![image-20220805170542170](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805170542170.png) 

**手动安装**

maven官方下载地址如下：（注意：maven下载地址翻到本文最下面）

> https://maven.apache.org/download.cgi

![image-20220808175818828](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220808175818828.png) 

下载后上传至服务器，解压后修改conf中 setting文件后复制到docker容器中

- 解压 `tar -xzvf apache-maven-3.8.6-bin.tar.gz`
- 修改 `apache-maven-3.8.6/conf/settings.xml` 配置仓库加速

```xml
<!-- 添加仓库 localRepository --> 
<localRepository>/var/jenkins_home/repository</localRepository>

<!-- 添加加速 --> 
<mirror>
     <id>alimaven</id>
     <name>aliyun maven</name>
     <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
     <mirrorOf>central</mirrorOf>
</mirror>
```

- 复制到docker `docker cp /root/docker/data/apache-maven-3.8.6 jenkins:/maven/`
- 进入容器后，创建软连接 `ln -s /maven/bin/mvn /usr/local/bin/`
- 执行 `mvn -v` 查看版本

**手动安装下，需要Jenkins配置MAVEN环境**

1. **系统环境配置**

![19d49315-eab7-4005-bc75-e97af6742614](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/19d49315-eab7-4005-bc75-e97af6742614.png)

2. **tool配置**

![8ec6f5a4-fd00-4323-9be8-d0cdddcdbd8f](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/8ec6f5a4-fd00-4323-9be8-d0cdddcdbd8f.png)

##### Docker 安装配置

![image-20220805170702263](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805170702263.png) 

### Jenkins任务创建

#### 1. 新建maven项目任务

![image-20220805170835148](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805170835148.png) 

![image-20220805170944228](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805170944228.png) 

#### 2. 配置git仓库

![image-20220805171659258](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805171659258.png) 

**【内网推荐使用Http地址去clone项目】**

#### 3. 配置Build Triggers 构建触发器

【构建触发器中配置，会获取到**URL**和**Token**,这两个东西需要记录下来，供gitlab配置webhook使用】

![image-20220805173526993](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805173526993.png) 

记录URL

```bash
http://192.168.92.130:8777/project/iotBuild
```

 URL和Token都需要填写到GitLab中，去配置webhooks！！！

 

**点击高级后，最下方可以点击生成Token**

记录Token

```bash
120e649995c048afe3c981507dcad71a
```

![image-20220805173756147](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805173756147.png) 



#### 4. 构建环境的配置根据自己需求配置

![image-20220809130917518](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220809130917518.png)  



#### 5. Pre Steps 配置前置步骤

> 可以在拉取代码后和maven构建前执行的步骤
>
> 一般用于解决清理空间或者手动执行一些git命令

#### 6. Build 构建的配置

jenkins构建项目，本处是以maven插件实现的。

因此，配置

```
clean package -Dmaven.test.skip=true -T 6
```

> `-T`为可选，服务器配置高可以搞上，代表多线程并行编译，速度更快
>
> `-T 6`：代表用6个线程构建
>
> `-T 6C`：代表根据CPU核数分配线程（1核分配6个线程）

#### 7. Post Steps 即jenkins构建完成后一步配置

【本处选择，只在jenkins构建成功后，再执行这一步】

【因为最后的构建成功的maven项目的jar包是以docker启动服务为目的，所以最后的docker操作，一定是在jenkins容器以外的服务器上运行的，可能是本机宿主机，也可能是远程的服务器】

【所以本处选择，在远程的SSH执行shell脚本】

【因此，必须要求，**Publish Over SSH插件以及它的相关配置**】

![image-20220805175629982](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805175629982.png)  

shell命令如下：

**Docker方式**

```shell
#=====================================================================================
#=================================定义初始化变量======================================
#=====================================================================================

#操作/项目路径(Dockerfile存放的路径),一般是项目中写的Dockerfile
BASE_PATH=/root/docker/data/jenkins/jenkins_home/workspace/iot_build_docker

# jenkins构建好的源jar路径  
SOURCE_PATH=/root/docker/data/jenkins/jenkins_home/workspace/iot_build_docker

#【docker 镜像】【docker容器】【Dockerfile同目录下的jar名字[用它build生成image的jar]】【jenkins的workspace下的项目名称】
#这里都以这个命名[微服务的话，每个服务都以iot-project这种格式命名]
#注意统一名称！！！！！
SERVER_NAME=iot-standalone

#容器id  [grep -w 全量匹配容器名] [awk 获取信息行的第一列，即容器ID]  [无论容器启动与否，都获取到]
CID=$(docker ps -a | grep -w "$SERVER_NAME" | awk '{print $1}')

#镜像id  [grep -w 全量匹配镜像名] [awk 获取信息行的第三列，即镜像ID]
IID=$(docker images | grep -w "$SERVER_NAME" | awk '{print $3}')

#源jar完整地址  [jenkins构建成功后，会在自己的workspace/项目/target 下生成maven构建成功的jar包，获取jar包名的完整路径]
#例如：/root/docker/data/jenkins/jenkins_home/iotBuild/iot-standalone/target/iot-standalone.jar
SOURCE_JAR_PATH=$(find "$SOURCE_PATH/$SERVER_NAME/target/"  -name "*$SERVER_NAME*.jar" )

DATE=`date +%Y%m%d%H%M%S`


#=====================================================================================
#============================对原本已存在的jar进行备份================================
#=====================================================================================



# 备份
# function backup(){
#     if [ -f "$BASE_PATH/$SERVER_NAME.jar" ]; then
#         echo "=========================>>>>>>>$SERVER_NAME.jar 备份..."
#             mv $BASE_PATH/$SERVER_NAME.jar $BASE_PATH/backup/$SERVER_NAME-$DATE.jar
#         echo "=========================>>>>>>>备份老的 $SERVER_NAME.jar 完成"
# 
#     else
#         echo "=========================>>>>>>>老的$BASE_PATH/$SERVER_NAME.jar不存在，跳过备份"
#     fi
# }



#=====================================================================================
#=========================移动最新源jar包到Dockerfile所在目录=========================
#=====================================================================================


 
# 查找源jar文件名，进行重命名，最后将源文件移动到项目环境
function transfer(){
       
         
    echo "=========================>>>>>>>源文件完整地址为 $SOURCE_JAR_PATH"

        
    echo "=========================>>>>>>>重命名源文件"
        mv $SOURCE_JAR_PATH  $SOURCE_PATH/$SERVER_NAME/target/$SERVER_NAME.jar

    echo "=========================>>>>>>>最新构建代码 $SOURCE_PATH/$SERVER_NAME/target/$SERVER_NAME.jar 迁移至 $BASE_PATH"
        cp $SOURCE_PATH/$SERVER_NAME/target/$SERVER_NAME.jar $BASE_PATH 

    echo "=========================>>>>>>>迁移完成Success"

}
 


#=====================================================================================
#==================================构建最新镜像=======================================
#=====================================================================================


 
# 构建docker镜像
function build(){
    
    #无论镜像存在与否，都停止原容器服务，并移除原容器服务
    echo "=========================>>>>>>>停止$SERVER_NAME容器，CID=$CID"
    docker stop $CID

    echo "=========================>>>>>>>移除$SERVER_NAME容器，CID=$CID"
    docker rm $CID

    #无论如何，都去构建新的镜像
    if [ -n "$IID" ]; then
        echo "=========================>>>>>>>存在$SERVER_NAME镜像，IID=$IID"


        echo "=========================>>>>>>>移除老的$SERVER_NAME镜像，IID=$IID"
        docker rmi $IID

        echo "=========================>>>>>>>构建新的$SERVER_NAME镜像，开始---->"
        cd $BASE_PATH
        docker build -t $SERVER_NAME .
        echo "=========================>>>>>>>构建新的$SERVER_NAME镜像，完成---->"

    else
        echo "=========================>>>>>>>不存在$SERVER_NAME镜像，构建新的镜像，开始--->"


        cd $BASE_PATH
        docker build -t $SERVER_NAME .
        echo "=========================>>>>>>>构建新的$SERVER_NAME镜像，结束--->"
    fi
}
 

#=====================================================================================
#==============================运行docker容器，启动服务===============================
#=====================================================================================

# 运行docker容器
function run(){
    # backup
    transfer
    build

    docker run --name $SERVER_NAME -itd --net=host -e TZ=Asia/Shanghai $SERVER_NAME 
    
}
 
#入口
run
```

**Dockerfile**

```dockerfile
FROM openjdk:11
MAINTAINER aa@qq.com
COPY ./iot-standalone.jar mydockerapp.jar

RUN echo 'Asia/Shanghai' >/etc/timezone

EXPOSE 8088
CMD ["java", "-jar", "mydockerapp.jar", "--spring.profiles.active=test"]
```

**纯 Jar包 启动方式**

```shell
#操作/项目路径，选择你自己喜欢的路径，比如平时方便看控制台日志啥的
BASE_PATH=/root/app

# jenkins构建的项目路径
SOURCE_PATH=/root/docker/data/jenkins/jenkins_home/workspace/iotBuild

#Jar服务名称
SERVER_NAME=iot-standalone

#源jar完整地址  [jenkins构建成功后，会在自己的workspace/项目/target 下生成maven构建成功的jar包，获取jar包名的完整路径]
#例如：/root/docker/data/jenkins/jenkins_home/workspace/iotBuild/iot-standalone/target/iot-standalone.jar
SOURCE_JAR_PATH=$(find "$SOURCE_PATH/$SERVER_NAME/target/"  -name "$SERVER_NAME.jar" )

DATE=`date +%Y%m%d%H%M%S`

#=====================================================================================
#============================对原本已存在的jar进行备份================================
#=====================================================================================



# 备份
function backup(){
	 if [ ! -d "$BASE_PATH/backup/" ];then
    		mkdir $BASE_PATH/backup
    else
        echo "$BASE_PATH/backup/ 文件夹已经存在,不需要创建"
    fi
    
    if [ -f "$BASE_PATH/$SERVER_NAME.jar" ]; then
        echo "=========================>>>>>>>$SERVER_NAME.jar 备份..."
            mv $BASE_PATH/$SERVER_NAME.jar $BASE_PATH/backup/$SERVER_NAME-$DATE.jar
        echo "=========================>>>>>>>备份老的 $SERVER_NAME.jar 完成"

    else
        echo "=========================>>>>>>>老的$BASE_PATH/$SERVER_NAME.jar不存在，跳过备份"
    fi
}



#=====================================================================================
#=========================移动最新源jar包到BASE_PATH所在目录=========================
#=====================================================================================


 
# 查找源jar文件名，进行重命名，最后将源文件移动到项目环境
function transfer(){
       
         
    echo "=========================>>>>>>>源文件完整地址为 $SOURCE_JAR_PATH"

        
    echo "=========================>>>>>>>重命名源文件"
        mv $SOURCE_JAR_PATH  $SOURCE_PATH/$SERVER_NAME/target/$SERVER_NAME.jar

    echo "=========================>>>>>>>最新构建代码 $SOURCE_PATH/$SERVER_NAME/target/$SERVER_NAME.jar 迁移至 $BASE_PATH"
        cp $SOURCE_PATH/$SERVER_NAME/target/$SERVER_NAME.jar $BASE_PATH 

    echo "=========================>>>>>>>迁移完成Success"

}

#=====================================================================================
#==============================运行Jar服务，启动服务===============================
#=====================================================================================

function runjar() {
	#无论JAR服务存在与否，都停止服务
    echo "=========================>>>>>>>停止$SERVER_NAME服务"
    kill -9 `cat $BASE_PATH/PID`

    echo "=========================>>>>>>>启动$SERVER_NAME服务"
    nohup java -jar  $BASE_PATH/$SERVER_NAME.jar "--spring.profiles.active=test" >> $BASE_PATH/log.log 2>&1 &
	echo "$!" > $BASE_PATH/PID 
}

# 运行JAR服务
function run(){
    backup
    transfer
    runjar  
}
 
#入口
run
```



**附上一个参考的完整配置**

![2023_172137_140.210.195.227](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/2023_172137_140.210.195.227.jpeg)



#### 8.GitLab配置

打开gitlab，并进入要自动部署的项目，点击左侧设置（setting）

![image-20220805175943308](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805175943308.png) 

将从jenkins获取到的URL和Token,填写在此处

【根据自己的需求，勾选webhook的触发事件都有哪些】

![image-20220805180239490](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220805180239490.png)  

最后点击添加

添加后，即可在下方看到刚刚添加的webhook！！

 

然后即可点击Test,选择刚刚勾选的绑定的触发事件 ，即可回到jenkins查看测试效果！！！



## 问题解决

### git 拉取源码出现 Host key verification failed.

在jenkins里面-系统设置-系统信息里面有个user.name可以看下当前jenkins使用的用户是什么，很多朋友以为默认就是jenkins用户。我们并没有为jenkins生成过密钥对,也没有将他的公钥拷到目标服务器上.

![image-20220808114927619](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220808114927619.png) 

**解决方案**

进入jenkins的容器，创建一个用于登录的公私钥，然后将公钥分配给远程主机即可

1. 生成公私密钥，做免密登录

```bash
# 是Dokcker容器的就进入容器内部,root方式进入
docker exec -it jenkins /bin/bash

# 生成密钥，一路回车
ssh-keygen -t rsa
```

2. gitlab配置公钥

![image-20220808171719839](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220808171719839.png) 

查看公钥复制到gitlab

![image-20220808171820945](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220808171820945.png) 

3. 收到clone一下项目测验

![image-20220808172123089](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220808172123089.png) 

拉取项目测试一下



4. Jenkins设置SSH私钥

查看私钥

![image-20220808171942575](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220808171942575.png) 

**Jenkins配置**

![image-20220808172318303](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220808172318303.png) 

5. 在任务配置源码这里，选择 ssh git方式

![image-20220808172508714](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220808172508714.png) 

再次执行就能成功拉取了

### git多个子仓库拉取不到最新代码

Jenkins 默认拉取多子仓库代码语句为 `git submodule update --init --recursive`

**问题原因：**

git submodule update获取代码的时候是和子工程的git路径和这里的commit id有关联的，获取的就是对应的git路径下截止这个commit id的所有代码，之后的代码是不会获取到的。

**解决方案**

在 Jenkins中配置前置shell

![image-20220809130352042](https://cdn.jsdelivr.net/gh/wilbur147/cdnPictureBed/article/image-20220809130352042.png) 

命令语句

```shell
git submodule foreach git checkout master && git submodule foreach git pull origin master
```

