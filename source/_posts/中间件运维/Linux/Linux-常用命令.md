---
title: Linux-常用命令
toc: true
date: 2019-09-04 14:55:28
tags:
categories:
---





## 新增用户

```
// 新建用户，并新增同名组，且生成一个home目录
useradd minfy

// 设置密码
passwd minfy

// 删除用户
userdel minfy

// 查看修改用户命令
usermod --help

// 切换用户，su是switch user的缩写
su minfy
```
会在/etc/passwd文件中增加用户信息

## 用户组

```
// 组的添加
groupadd testgroup    

// 组的删除
groupdel testgroup    
```
组的增加和删除信息会在/etc/group文件中体现出来

## 修改文件所有权

```
// 将file.txt文件所有权改为minfy
chown minfy ./file.txt

// 将file.txt文件所有权组改为minfy
chown :minfy ./file.txt

// 同时修改文件的所有者和用户组
chown minfy:minfy ./file.txt

// 同时修改子目录文件
chown -R minfy:minfy ./file.txt
```

## 修改文件权限
```
// 所有人添加执行权限
chmod a+x ./app.sh

// 将/usr/local/dir及子目录文件权限都设置成744
chmod -R 744 /usr/local/dir
```

## 参考资料
> - []()
> - []()
