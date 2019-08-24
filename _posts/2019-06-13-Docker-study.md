---
layout: post
title: 'Docker学习'
tags: [code]
---

## Docker简介

### 容器是什么

**一句话概括容器：容器就是将软件打包成标准化单元，以用于开发、交付和部署。**

- **容器镜像是轻量的、可执行的独立软件包** ，包含软件运行所需的所有内容：代码、运行时环境、系统工具、系统库和设置。
- **容器化软件适用于基于Linux和Windows的应用，在任何环境中都能够始终如一地运行。**
- **容器赋予了软件独立性**　，使其免受外在环境差异（例如，开发和预演环境的差异）的影响，从而有助于减少团队间在相同基础设施上运行不同软件时的冲突。

### Docker是什么？

说实话关于Docker是什么并太好说，下面我通过四点向你说明Docker到底是个什么东西。

- **Docker 是世界领先的软件容器平台。**
- **Docker** 使用 Google 公司推出的 **Go 语言** 进行开发实现，基于 **Linux 内核** 的cgroup，namespace，以及AUFS类的**UnionFS**等技术，**对进程进行封装隔离，属于操作系统层面的虚拟化技术。** 由于隔离的进程独立于宿主和其它的隔离的进 程，因此也称其为容器。**Docke最初实现是基于 LXC.**
- **Docker 能够自动执行重复性任务，例如搭建和配置开发环境，从而解放了开发人员以便他们专注在真正重要的事情上：构建杰出的软件。**
- **用户可以方便地创建和使用容器，把自己的应用放入容器。容器还可以进行版本管理、复制、分享、修改，就像管理普通的代码一样。**

### 容器和虚拟机的区别

**容器是一个应用层抽象，用于将代码和依赖资源打包在一起。** **多个容器可以在同一台机器上运行，共享操作系统内核，但各自作为独立的进程在用户空间中运行** 。与虚拟机相比， **容器占用的空间较少**（容器镜像大小通常只有几十兆），**瞬间就能完成启动** 。

**虚拟机是一个物理硬件层抽象，用于将一台服务器变成多台服务器。** 管理程序允许多个虚拟机在一台机器上运行。每个都包含一整套操作系统、一个或多个应用、必要的二进制文件和库资源，因此 **占用大量空间** 。而且虚拟机启动也十分缓慢。

### 三个重要概念

#### 镜像

**Docker 镜像是一个特殊的文件系统，除了提供容器运行时所需的程序、库、资源、配置等文件外，还包含了一些为运行时准备的一些配置参数（如匿名卷、环境变量、用户等）**

#### 容器

镜像（Image）和容器（Container）的关系，就像是面向对象程序设计中的 类 和 实例 一样，镜像是静态的定义，**容器是镜像运行时的实体。容器可以被创建、启动、停止、删除、暂停等** 。

#### 仓库

镜像构建完成后，可以很容易的在当前宿主上运行，但是， **如果需要在其它服务器上使用这个镜像，我们就需要一个集中的存储、分发镜像的服务，Docker Registry就是这样的服务。**

##  Docker安装

**配置阿里云镜像**

右击docker任务栏图标--->在设置settings里--->守护进程Daemon---> Registry mirrors下方输入框中添加自己的阿里云镜像，形如这种链接  https://xxxxxx.mirror.aliyuncs.com  点击apply应用即可。

**运行测试docker**

cmd 运行下面的hello-world命令如果有一长串输出证明已经安装好docker镜像

```dockerfile
docker run hello-world;
```

## Docker初步使用

**使用docker安装一个ubuntu环境并使用echo命令打印hello docker！**

```dockfile
docker run ubuntu echo hello docker!
# 输出hello docker！
```

**使用命令查看正在运行的docker容器**

```dockerfile
docker ps
```

**查看docker所有安装的镜像**

```dockerfile
docker images
```

使用docker安装并使用nginx 默认80端口改成8080端口，访问localhost:8080界面就可以看到nginx经典欢迎页面

```dockerfile
# -d表示挂起后台运行 -p做端口映射
docker run -p 8080:80 -d nginx
```

下面使用命令更换这个经典页面

```dockerfile
# 使用vi命令新建index.html文件
vi index.html

# index.html文件内容如下
<html>
<h1>hello docker!</h1>
</html>

# 在使用命令将index.html文件拷贝进去，这里的XXXXXX替换成启动的nginx containerId(通过nginx ps命令查看)
docker cp index.html XXXXXX://usr/share/nginx/html
```

再次访问 localhost:8080 即可看到修改后的index.html页面

**使用docker命令停止docker容器**

```dockerfile
# 这里的XXXXXX替换成需要停止的containerId(通过docker ps命令查看)
docker stop XXXXXX
```

在这里停止掉刚刚启动的nginx容器之后，再次使用命令启动nginx容器，访问localhost:8080发现还是经典的nginx欢迎页面而不是我们修改后的页面，这是因为docker在容器所做的改动都是临时的没有被保存下来。

我们再次使用命令将文件拷贝过去，然后使用命令提交更改，操作类似git

```dockerfile
# 这里的XXXXXX替换成containerId(通过docker ps命令查看)
# nginx-fun可替换成任意内容 这个是指定REPOSITORY名字
docker commit -m "commit message" XXXXXX nginx-fun
```

```dockerfile
# 使用docker images命令查看输出下面内容 跟之前相比多了一个我们提交的nginx-fun
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
nginx-fun           latest              76bdf9aa026c        7 seconds ago       109MB
nginx               latest              719cd2e3ed04        32 hours ago        109MB
ubuntu              latest              7698f282e524        3 weeks ago         69.9MB
hello-world         latest              fce289e99eb9        5 months ago        1.84kB
```

使用命令删除image

```dockerfile
# XXXXXX替换为对应的IMAGE ID(docker images命令查看)
docker rmi XXXXX
```

使用命令查看docker容器历史

```dockerfile
docker ps -a
```

使用命令清除docke容器历史

```dockerfile
# XXXXXX代替CONTAINER ID 
docker rm XXXXXXX
```

**使用命令进入container内部**

```dockerfile
# XXX 为containerId通过 docker ps命令查看
docker exec -it XXXX bash
```



**命令总结**

| 命令          | 用途                          |
| ------------- | ----------------------------- |
| docker pull   | 获取image                     |
| docker build  | 创建image                     |
| docker images | 列出所有image                 |
| docker run    | 运行container                 |
| docker ps     | 列出运行中的container         |
| docker rm     | 删除container                 |
| docker rmi    | 删除image                     |
| docker cp     | 在host和container之间拷贝文件 |
| docker commit | 保存改动为新的image           |

## 使用Dockerfile

### 初步使用

```shell
# 新建一个文件夹
mkdir d1
# 进入文件夹并新建一个Dockerfile文件
cd d1
touch Dockerfile
# 注意文件名按照统一约定为Dockerfile
```

文件内容如下

```dockerfile
# 要生成的新镜像构建在alpine最新版本镜像的基础之上
FROM alpine:latest  
# 使用这个命令是用来生成镜像署名作者
MAINTAINER debugjoker 
# CMD指令的主要功能是在build完成后，给docker run启动到容器时提供默认命令或参数，这些默认值可以包含可执行的命令
CMD echo 'hello docker!'
```

编译当前目录下的Dockerfile文件

```shell
# -t参数是添加一个标签 .是build当前目录下的所有文件
docker build -t hello-docker .
# 看到下面的输出表示成功
# Successfully built e6141dc4ee41
# Successfully tagged hello-docker:latest
```

我们也可以使用```docker images``` 命令查看，之后通过```docker run hello-docker```命令运行即可看到输出 “hello docker!” 成功

### 进阶使用

```shell
# 新建一个文件夹
mkdir dockerfile2
# 进入文件夹并新建一个Dockerfile文件
cd dockerfile2
touch Dockerfile
# 注意文件名按照统一约定为Dockerfile
```

```dockerfile
FROM ubuntu
MAINTAINER debugjoker
# RUN指令会在当前镜像的顶层执行任何命令，并commit成新的（中间）镜像
# 这个是更换ubuntu镜像
RUN sed -i 's/archive.ubuntu.com/mirrors.ustc.edu.cn/g' /etc/apt/sources.list
RUN apt-get update
RUN apt-get install -y nginx
# 本地index.html文件拷贝到container中/var/www/html的位置
COPY index.html /var/www/html
# ENTRYPOINT命令设置在容器启动时执行命令，如果有多个ENTRYPOINT指令，那只有最后一个生效
ENTRYPOINT ["/usr/sbin/nginx", "-g", "daemon off;"]
# EXPOSE指令告诉容器在运行时要监听的端口
EXPOSE 80
```

```shell
# 在dockerfile2目录下新建一个index.html文件并随意填写内容
touch index.html
# 编译dockerfile文件
docker build -t debugjoker-nginx .
# 查看一下有没有build成功
docker images
# 运行
 docker run -d -p 80:80 debugjoker-nginx
# 查看nginx有没有成功部署在容器内
curl http://localhost
```

| 命令       | 用途         |
| ---------- | ------------ |
| FROM       | base image   |
| RUN        | 执行命令     |
| ADD        | 添加文件     |
| COPY       | 拷贝文件     |
| CMD        | 执行命令     |
| EXPOSE     | 暴露接口     |
| WORKDIR    | 指定路径     |
| MAINTAINER | 维护者       |
| ENV        | 设定环境变量 |
| ENTRYPOINT | 容器入口     |
| USER       | 指定用户     |
| VOLUME     | mount point  |

# 镜像分层

Docker 设计时，就充分利用 **Union FS**的技术，将其设计为 **分层存储的架构** 。 镜像实际是由多层文件系统联合组成。

**镜像构建时，会一层层构建，前一层是后一层的基础。每一层构建完就不会再发生改变，后一层上的任何改变只发生在自己这一层。**　比如，删除前一层文件的操作，实际不是真的删除前一层的文件，而是仅在当前层标记为该文件已删除。在最终容器运行的时候，虽然不会看到这个文件，但是实际上该文件会一直跟随镜像。因此，在构建镜像的时候，需要额外小心，每一层尽量只包含该层需要添加的东西，任何额外的东西应该在该层构建结束前清理掉。

分层存储的特征还使得镜像的复用、定制变的更为容易。甚至可以用之前构建好的镜像作为基础层，然后进一步添加新的层，以定制自己所需的内容，构建新的镜像。

# 数据卷

按照 Docker 最佳实践的要求，**容器不应该向其存储层内写入任何数据** ，容器存储层要保持无状态化。**所有的文件写入操作，都应该使用数据卷（Volume）、或者绑定宿主目录**，在这些位置的读写会跳过容器存储层，直接对宿主(或网络存储)发生读写，其性能和稳定性更高。数据卷的生存周期独立于容器，容器消亡，数据卷不会消亡。因此， **使用数据卷后，容器可以随意删除、重新 run ，数据却不会丢失。**

# 使用Docker运行一个Java Web应用

1. 准备war包

   这里使用开源的jpress的war包

2. 下载tomcat镜像

   使用命令 ```docker pull tomcat``` 下载最新版本的tomcat，对版本有要求的可指定版本

3. 新建一个目录如tomcat_jpress，并将war包放进去，在新建一个Dockerfile文件，内容如下

   ```dockfile
   FROM tomcat
   MAINTAINER your_name your_email
   COPY jpress.war /usr/local/tomcat/webapps
   ```
   
   
   /usr/local/tomcat这个目录可以通过镜像官网的对应说明查看

   ![官网图片](https://i.loli.net/2019/06/17/5d06fb5d4b4b935094.png)
   
4. build docekr镜像

   ```dockerfile
   docker build -t jpress:latest .
   ```

5. 运行docker镜像

   ```dockerfile
   docekr run -d -p 8888:8080 jpress
   ```

6. 访问localhost:8888/jpress/ 即可看到相关页面