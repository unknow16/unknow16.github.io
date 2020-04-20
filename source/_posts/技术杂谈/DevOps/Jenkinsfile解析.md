---
title: Jenkinsfile解析
toc: true
date: 2020-04-05 17:49:55
tags:
categories:
---





官网：https://jenkins.io/zh/

```
docker pull jenkins/jenkins:lts

docker run -d -u root --name jenkins-master -v $HOME/jenkins:/var/jenkins_home -p 8080:8080 -p 50000:50000 -p 45000:45000 jenkins/jenkins:lts

 docker run -d --name jenkins_01 -p 8081:8080 -v /home/jenkins_01:/home/jenkins_01 jenkins/jenkins:lts

docker run \
  --rm \
  -u root \
  -p 8080:8080 \
  -v $HOME/jenkins:/var/jenkins_home \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$HOME":/home \
  jenkinsci/blueocean
```



## 参考资料
> - []()
> - []()
