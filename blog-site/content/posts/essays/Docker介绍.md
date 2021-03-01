---
title: "Docker介绍"
date: 2020-04-07
draft: false
tags: ["使用介绍", "Java", "docker"]
slug: "docker-introduction"
---

### docker是什么
Docker 属于 Linux 容器的一种封装，提供简单易用的容器使用接口。它是目前最流行的 Linux 容器解决方案。
Docker 将应用程序与该程序的依赖，打包在一个文件里面。运行这个文件，就会生成一个虚拟容器。程序在这个虚拟容器里运行，就好像在真实的物理机上运行一样。有了 Docker，就不用担心环境问题。
总体来说，Docker 的接口相当简单，用户可以方便地创建和使用容器，把自己的应用放入容器。容器还可以进行版本管理、复制、分享、修改，就像管理普通的代码一样.

### docker用途
提供一次性的环境。比如，本地测试他人的软件、持续集成的时候提供单元测试和构建的环境。
提供弹性的云服务。因为 Docker 容器可以随开随关，很适合动态扩容和缩容。
组建微服务架构。通过多个容器，一台机器可以跑多个服务，因此在本机就可以模拟出微服务架构。

### 安装
1.安装依赖,docker依赖于系统的一些必要的工具

```shell script
yum install -y yum-utils device-mapper-persistent-data lvm2
```
2.添加软件源, 阿里云镜像(在阿里云镜像站上面可以找到docker-ce的软件源，使用国内的源速度比较快)
```shell script
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
```
[3.安装docker-ce(社区版,免费)](https://blog.csdn.net/zhuzz1030/article/details/80097553)
```shell script
yum clean allyum makecache fastyum -y install docker-ce
```
4.启动服务
````shell script
 #service 命令的用法
$ sudo service docker start

#systemctl 命令的用法
$ sudo systemctl start docker

````
5.查看安装版本
````docker
docker version
````
6.测试,检查 docker 是否正确安装并运行 hello-world 镜像
````linux
docker run hello-world
````
7.Docker 需要用户具有 sudo 权限，为了避免每次命令都输入sudo，可以把用户加入 Docker 用户组
````linux
sudo usermod -aG docker $USER
````

### docker架构
![docker架构](/myblog/post/images/essays/docker架构.png)
docker架构分为三部分客户端,宿主机,注册中心

例如：输入 `docker run mysql:5.6`命令,docker程序会从docker_host去找对应的镜像,如果,又mysql不存在则会去从register下载,下载完成后docker会自动给该镜像分配一个contains,mysql就会运行起来

#### docker基本操作
例如安装运行`nginx`为例

```shell
// docker run 包括下载镜像(pull),创建容器(create),运行容器(start) 可用dock -h查看帮助
// --rm 表明这是一个临时的容器,关闭的话会自动删除
// --name 容器名称
// -p 外部服务器端口映射docker容器端口
docker run --rm --name myNginx -p 80:80 nginx:版本

// 查看容器日志
docker logs myNginx

// 进入容器
docker exec -it myNginx bash


## docker常用命令

// 查看当前运行的镜像
docker ps
//查看所有镜像
docker ps -a
//停止容器
docker stop 容器名称
// 删除镜像
docker rmi 镜像名称
//下载镜像 若不指定版本为最新版本
docker pull 镜像:版本
//查看当前本地仓库的镜像
docker images
//查看远程仓库镜像
docker search 镜像名
```


### 参考资料
**入门参考:** https://baijiahao.baidu.com/s?id=1626633654476933953&wfr=spider&for=pc
**安装docker菜鸟教程:** https://www.runoob.com/docker/centos-docker.install.html
**docker官网:** https://www.docker.com/
**docker文档:** https://docs.docker.com/install/linux/docker-ce/centos/
**阮一峰dockerのblog:** http://www.ruanyifeng.com/blog/2018/02/docker-tutorial.html
**参考blog**: https://cyc2018.github.io/CS-Notes/#/notes/Docker
