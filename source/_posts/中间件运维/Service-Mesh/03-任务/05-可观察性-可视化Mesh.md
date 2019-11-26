---
title: 05-可观察性-可视化Mesh
toc: true
date: 2019-11-16 12:26:29
tags:
categories:
---























安装Kiali附件



1. 创建一个Secret

- 定义Kiali的用户名和密码

```
$ KIALI_USERNAME=$(read -p 'Kiali Username: ' uval && echo -n $uval | base64)
$ KIALI_PASSPHRASE=$(read -sp 'Kiali Passphrase: ' pval && echo -n $pval | base64)
```

- 如果使用Z shell、zsh，则用下面命令

```
$ KIALI_USERNAME=$(read '?Kiali Username: ' uval && echo -n $uval | base64)
$ KIALI_PASSPHRASE=$(read -s "?Kiali Passphrase: " pval && echo -n $pval | base64)
```

- 创建secret

```
apiVersion: v1
kind: Secret
metadata:
  name: kiali
  namespace: istio-system
  labels:
    app: kiali
type: Opaque
data:
  username: $KIALI_USERNAME
  passphrase: $KIALI_PASSPHRASE
```



## 参考资料

> - []()
> - []()
