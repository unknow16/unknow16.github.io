---
title: Spring Boot-17-Win下打包镜像到远程Docker服务器
date: 2018-07-24 17:13:41
tags: Spring Boot
---

## 环境准备
#### 1. 开启Docker服务端远程访问API
要想在windows中操作远程linux中的docker服务进程，那前提是必须开启docker远程API。

* Centos-7
```
# 编辑docker配置文件
vi /usr/lib/systemd/system/docker.service

# 将ExecStart开头的行，修改为如下：
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock

# 刷新配置
sudo systemctl daemon-reload

# 重新启动docker守护进程
sudo systemctl restart docker

# 确认是否成功。
ps -ef | grep docker

# 测试可以连接到docker api 
curl 127.0.0.1:2375/info
```

* Ubuntu-16.04

```
# 编辑docker配置文件
sudo vi /lib/systemd/system/docker.service

# 将ExecStart开头的行，修改为如下：
ExecStart=/usr/bin/dockerd -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock

# 后边操作同CentOS-7一样。。。
```

#### 2. 客户端配置docker_host
* win

新增系统环境变量DOCKER_HOST
```
# 变量名：DOCKER_HOST
# 变量值：tcp://192.168.108.129:2375
# 上面的ip要改成你docker服务器ip
```

## 应用配置docker插件
#### 1. 开发Spring Boot应用

#### 2. 添加maven插件配置
```
<!-- spring boot 打docker镜像插件 -->
<plugin>
	<groupId>com.spotify</groupId>
	<artifactId>docker-maven-plugin</artifactId>
	<version>1.0.0</version>
	<configuration>
		<imageName>${project.artifactId}</imageName>
		<dockerDirectory>src/main/docker</dockerDirectory>
		<resources>
			<resource>
				<targetPath>/</targetPath>
				<directory>${project.build.directory}</directory>
				<include>${project.build.finalName</include>
			</resource>
		</resources>
	</configuration>
</plugin>
```
#### 2. 编写Dockerfile文件
新建src/java/docker源文件夹，并要其下创建Dockerfile文件

```
FROM openjdk:8-jdk-alpine
VOLUME /tmp
ADD http://hd.139xy.cn/img/hjq/integration-docker-0.0.1-SNAPSHOT.jar app.jar
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","/app.jar"]
```
其中要注意：ADD命令中两个参数，源文件和目的文件，实际就是将我们可执行的jar文件，添加到镜像中。

其中源文件支持两种方式：
1. 和docker进程在同一主机的文件系统的绝对和相对目录
2. URL


## 构建项目
#### 1. 打包构建镜像
配置好maven插件后，打开cmd，执行以下命令：

```
mvn clean package docker:build -DskipTests
```
然后慢慢等待，直到最后build成功。

#### 2. 查看构建好的镜像
登陆linux，输入#docker images 

发现自己的项目已经被编译成镜像了，可以启动容器运行镜像了，也相当于完成了项目的云部署