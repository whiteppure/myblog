---
title: "SpringBoot整合docker"
date: 2020-08-30
draft: false
tags: ["springboot", "docker"]
slug: "springboot-docker"
---

## MacOS上安装docker

### 下载
国内下载网站: [http://get.daocloud.io](http://get.daocloud.io) *不推荐下载docker版本太旧了*

官网下载: [https://docs.docker.com/get-started/#download-and-install-docker](https://docs.docker.com/get-started/#download-and-install-docker)

或用`homebrew`进行下载安装
```
 brew install --cask --appdir=/Applications docker
```
### 配置镜像
由于网速原因，可以配置一下国内的镜像加速器
- 中科大镜像: [https://docker.mirrors.ustc.edu.cn](https://docker.mirrors.ustc.edu.cn)
- 网易: [https://hub-mirror.c.163.com](https://hub-mirror.c.163.com)
- 阿里云: [https://<你的ID>.mirror.aliyuncs.com](https://<你的ID>.mirror.aliyuncs.com)
- 七牛云加速器: [https://reg-mirror.qiniu.com](https://reg-mirror.qiniu.com)

阿里云获取镜像地址: [https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors](https://cr.console.aliyun.com/cn-hangzhou/instances/mirrors)
登陆后，左侧菜单选中镜像加速器就可以看到你的专属地址了;
![获取阿里云docker地址](/myblog/posts/images/application/获取阿里云docker地址.jpg)

配置docker镜像地址
![docker配置](/myblog/posts/images/application/docker配置.jpg)
```
{
  "debug": true,
  "experimental": false,
  "registry-mirrors": [
    "https://docker.mirrors.ustc.edu.cn"
  ]
}
```
在终端执行`docker info`命令
![docker信息](/myblog/posts/images/application/docker信息.jpg)

出现上图所示，docker镜像加速器配置成功

参考[菜鸟教程](https://www.runoob.com/docker/docker-mirror-acceleration.html)

## Springboot整合docker

### 创建测试
HelloController.class
```
/**
 * <p>
 * Hello Controller
 * </p>
 */
@RestController
@RequestMapping
public class HelloController {
    @GetMapping
    public String hello() {
        return "Hello,From Docker";
    }
}
```
application.yml
```
server:
  port: 8080
  servlet:
    context-path: /demo

```
### Dockerfile
首先创建一个名字叫`Dockerfile`的文件,路径任意
![DockerFile](/myblog/posts/images/application/dockerFile.jpg)
```
# 基础镜像
FROM openjdk:8-jdk-alpine

# 作者信息
MAINTAINER "whitepure"

# 添加一个存储空间 其效果是在主机 /var/lib/docker 目录下创建了一个临时文件，并链接到容器的/tmp
VOLUME /tmp

# 暴露8080端口
EXPOSE 8080

# 添加变量，获取target下的jar包； 如果使用dockerfile-maven-plugin，则会自动替换这里的变量内容
ARG JAR_FILE=target/demo-docker.jar

# 往容器中添加jar包
ADD ${JAR_FILE} app.jar

# 启动镜像自动运行程序
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/urandom","-jar","/app.jar"]
```
### pom
在 pom 文件加入 docker 插件
```
    <properties>
        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
        <project.reporting.outputEncoding>UTF-8</project.reporting.outputEncoding>
        <java.version>1.8</java.version>
        <dockerfile-version>1.4.9</dockerfile-version>
    </properties>

 <build>
        <finalName>demo-docker</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>dockerfile-maven-plugin</artifactId>
                <version>${dockerfile-version}</version>
                <configuration>
                    <repository>${project.build.finalName}</repository>
                    <tag>${project.version}</tag>
                    <buildArgs>
                        <JAR_FILE>target/${project.build.finalName}.jar</JAR_FILE>
                    </buildArgs>
                </configuration>
<!--                <executions>-->
<!--                    &lt;!&ndash;设置运行 mvn package 的时候自动执行docker build&ndash;&gt;-->
<!--                    <execution>-->
<!--                        <id>default</id>-->
<!--                        <phase>package</phase>-->
<!--                        <goals>-->
<!--                            <goal>build</goal>-->
<!--                        </goals>-->
<!--                    </execution>-->
<!--                </executions>-->
            </plugin>
        </plugins>
    </build>
```
使用 maven 打包
![docker打包](/myblog/posts/images/application/docker打包.jpg)

### docker镜像测试
前往 Dockerfile 目录，打开命令行执行;

>执行 docker build 命令，docker就会根据 Dockerfile 里你定义好的命令进行构建新的镜像;
-t代表要构建的镜像的 tag ，.代表当前目录，也就是 Dockerfile 所在的目录
```
docker build -t demo-docker .
```

查看docker镜像列表
```
docker images
```

运行该镜像
> 使用镜像 demo-docker ，将容器的 8080 端口映射到主机的 9090 端口
```
docker run -d -p 9090:8080 demo-docker
```

访问[http://localhost:9090/demo](http://localhost:9090)
![hellodocker](/myblog/posts/images/application/hellodocker.jpg)

如果要停止docker镜像，首先获取镜像id,然后在用stop命令停止运行镜像
> docker ps -a: 显示所有的容器，包括未运行的
```
whitepure@MacBook-Pro demo-docker % docker ps -a
CONTAINER ID   IMAGE      ...    NAMES
421fcff1be87   demo-docker   ...  affectionate_nightingale
whitepure@MacBook-Pro demo-docker % docker stop 421fcff1be87
421fcff1be87
```

如果要删除镜像，也需要先获取镜像id，在用rm命令进行删除
```
whitepure@MacBook-Pro demo-docker % docker ps -a
CONTAINER ID   IMAGE         ...    NAMES
421fcff1be87   demo-docker    ...     affectionate_nightingale
whitepure@MacBook-Pro demo-docker % docker rm 421fcff1be87
421fcff1be87
```



