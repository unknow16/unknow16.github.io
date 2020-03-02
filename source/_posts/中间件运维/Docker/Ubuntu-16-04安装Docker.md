---
title: Ubuntu-16.04安装Docker
toc: true
date: 2019-10-22 11:05:56
tags:
categories:
---









1. 由于apt官方库里的docker版本可能比较旧，所以先卸载可能存在的旧版本

```
$ sudo apt-get remove docker docker-engine docker-ce docker.io
```

2. 更新apt包索引：

```
$ sudo apt-get update
```

3. 安装以下包以使apt可以通过HTTPS使用存储库（repository）：

````
$ sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common
````

4. 添加Docker官方的GPG密钥：

```
$ curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
```

5. 使用下面的命令来设置stable存储库：

```
$ sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
```

6. 再更新一下apt包索引：

```
$ sudo apt-get update
```

7. 安装指定版本

- 在生产系统上，可能会需要应该安装一个特定版本的Docker CE，而不是总是使用最新版本，列出可用的版本：

```
$ apt-cache madison docker-ce
```

- 选择要安装的特定版本，第二列是版本字符串，第三列是存储库名称，它指示包来自哪个存储库，以及扩展它的稳定性级别。要安装一个特定的版本，将版本字符串附加到包名中，并通过等号(=)分隔它们：

```
$ sudo apt-get install docker-ce=18.06.3~ce~3-0~ubuntu -y
```



8. 直接安装最新版本的Docker CE：

```
$ sudo apt-get install -y docker-ce
```

9. 验证docker，查看docker服务是否启动：

```
$ systemctl status docker
```











## 参考资料
> - []()
> - []()
