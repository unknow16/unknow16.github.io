---
title: OpenShift-v3架构概述
toc: true
date: 2019-10-21 17:03:26
tags:
categories:
---

OpenShift v3是一个分层系统，旨在尽可能准确地公开底层Docker格式的容器映像和Kubernetes概念，并着重于开发人员轻松编写应用程序。

与OpenShift v2不同，在模型的所有方面创建之后，配置的灵活性就会更大。取消了将应用程序作为独立对象的概念，以便更灵活地组合“服务”，从而允许两个Web容器重用数据库或将数据库直接暴露于网络边缘。

## 什么是层？

Docker服务为打包和创建基于Linux的轻量级容器映像提供了抽象。 Kubernetes提供集群管理并在多个主机上协调容器。

OKD附加：

- 开发人员的源代码管理，构建和部署 
- 在镜像流经您的系统时进行大规模管理和推广
- 大规模的应用程序管理 
- 团队和用户跟踪，用于组织大型开发人员组织 
- 支持集群的网络基础架构



## 什么是OKD架构？

OKD具有一个基于微服务的架构，该架构包含可协同工作的较小的，分离的单元。它在Kubernetes集群的顶部运行，有关对象的数据存储在etcd（可靠的集群键值存储）中。这些服务按功能细分： 

- REST API，公开每个核心对象。 
- 读取这些API的控制器将更改应用于其他对象，并报告状态或写回该对象。

用户调用REST API来更改系统状态。控制器使用REST API读取用户所需的状态，然后尝试使系统的其他部分同步。例如，当用户请求构建时，他们将创建一个“构建”对象。构建控制器看到已经创建了新的构建，并在集群上运行一个过程以执行该构建。构建完成后，控制器将通过REST API更新构建对象，并且用户会看到其构建已完成。

控制器模式意味着OKD中的许多功能都是可扩展的。可以独立于映像的管理方式或部署方式来自定义运行和启动构建的方式。控制器执行系统的“业务逻辑”，采取用户操作并将其转换为现实。通过自定义这些控制器或用您自己的逻辑替换它们，可以实现不同的行为。从系统管理的角度来看，这还意味着该API可用于按重复的时间表编写常见的管理操作脚本。这些脚本也是监视更改并采取措施的控制器。 OKD能够以这种一流的行为来自定义集群。

为了实现这一点，控制器利用对系统的可靠更改流将其对系统的看法与用户正在执行的操作同步。该事件流将更改从etcd推送到REST API，然后在更改发生后立即推送到控制器，因此更改可以非常快速，有效地在系统中扩散。但是，由于故障随时可能发生，因此控制器还必须能够在启动时获取系统的最新状态，并确认一切都处于正确的状态。这种重新同步非常重要，因为这意味着即使出现问题，操作员也可以重新启动受影响的组件，并且系统会在继续之前再次检查所有内容。由于控制器始终可以使系统保持同步，因此该系统最终应收敛到用户的意图。



## OKD是怎样安全的？

OKD和Kubernetes API对提供凭据的用户进行身份验证，然后根据其角色对其进行授权。开发人员和管理员均可通过多种方式进行身份验证，主要是OAuth令牌和X.509客户端证书。 OAuth令牌使用JSON Web算法RS256签名，这是具有SHA-256的RSA签名算法PKCS＃1 v1.5。

开发人员（系统的客户端）通常从oc之类的客户端程序或通过其浏览器向Web控制台进行REST API调用，并使用OAuth承载令牌进行大多数通信。基础结构组件（如节点）使用由系统生成的包含其身份的客户端证书。在容器中运行的基础结构组件使用与其service account关联的令牌来连接到API。

授权是在OKD策略引擎中处理的，该引擎定义了诸如“创建容器”或“列表服务”之类的动作，并将它们分组为策略文档中的角色。角色通过用户或组标识符绑定到用户或组。当用户或service account尝试执行某个操作时，策略引擎会在允许其继续执行之前检查分配给该用户的一个或多个角色（例如，群集管理员或当前项目的管理员）。

由于在群集上运行的每个容器都与一个service account相关联，因此还可以将secrets与这些service account相关联，并将其自动传递到容器中。这使基础架构能够管理用于拉出和推送映像，构建和部署组件的secrets，并且还允许应用程序代码轻松利用这些机密。



## TLS 支持

与REST API以及etcd等主组件与API服务器之间的所有通信通道均由TLS保护。TLS使用X.509服务器证书和公钥基础结构为服务器提供强大的加密，数据完整性和身份验证。默认情况下，为OKD的每个部署创建一个新的内部PKI。内部PKI使用2048位RSA密钥和SHA-256签名。还支持公共主机的自定义证书。

OKD使用Golang的crypto / tls标准库实现，并且不依赖于任何外部加密和TLS库。此外，客户端依赖于外部库进行GSSAPI身份验证和OpenPGP签名。GSSAPI通常由MIT Kerberos或Heimdal Kerberos提供，它们都使用OpenSSL的libcrypto。 OpenPGP签名验证由libgpgme和GnuPG处理。

不安全版本SSL 2.0和SSL 3.0不受支持并且不可用。默认情况下，OKD服务器和oc客户端仅提供TLS 1.2。可以在服务器配置中启用TLS 1.0和TLS 1.1。服务器和客户端都喜欢具有身份验证的加密算法和完美的前向保密性的现代密码套件。具有不推荐使用和不安全算法（例如RC4、3DES和MD5）的密码套件将被禁用。某些内部客户端（例如LDAP身份验证）的TLS 1.0到1.2的限制设置较少，并且启用了更多的密码套件。

| TLS Version | OKD Server                                                   | `oc` Client                                                  | Other Clients                                                |
| :---------- | :----------------------------------------------------------- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| SSL 2.0     | Unsupported                                                  | Unsupported                                                  | Unsupported                                                  |
| SSL 3.0     | Unsupported                                                  | Unsupported                                                  | Unsupported                                                  |
| TLS 1.0     | No [[1](https://docs.okd.io/3.11/architecture/index.html#_footnotedef_1)] | No [[1](https://docs.okd.io/3.11/architecture/index.html#_footnotedef_1)] | Maybe [[2](https://docs.okd.io/3.11/architecture/index.html#_footnotedef_2)] |
| TLS 1.1     | No [[1](https://docs.okd.io/3.11/architecture/index.html#_footnotedef_1)] | No [[1](https://docs.okd.io/3.11/architecture/index.html#_footnotedef_1)] | Maybe [[2](https://docs.okd.io/3.11/architecture/index.html#_footnotedef_2)] |
| TLS 1.2     | **Yes**                                                      | **Yes**                                                      | **Yes**                                                      |
| TLS 1.3     | N/A [[3](https://docs.okd.io/3.11/architecture/index.html#_footnotedef_3)] | N/A [[3](https://docs.okd.io/3.11/architecture/index.html#_footnotedef_3)] | N/A [[3](https://docs.okd.io/3.11/architecture/index.html#_footnotedef_3)] |

以下是OKD服务器和oc客户端已启用密码套件的列表，按优先顺序排列：

- `TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305`
- `TLS_ECDHE_RSA_WITH_CHACHA20_POLY1305`
- `TLS_ECDHE_ECDSA_WITH_AES_128_GCM_SHA256`
- `TLS_ECDHE_RSA_WITH_AES_128_GCM_SHA256`
- `TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384`
- `TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384`
- `TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA256`
- `TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA256`
- `TLS_ECDHE_ECDSA_WITH_AES_128_CBC_SHA`
- `TLS_ECDHE_ECDSA_WITH_AES_256_CBC_SHA`
- `TLS_ECDHE_RSA_WITH_AES_128_CBC_SHA`
- `TLS_ECDHE_RSA_WITH_AES_256_CBC_SHA`
- `TLS_RSA_WITH_AES_128_GCM_SHA256`
- `TLS_RSA_WITH_AES_256_GCM_SHA384`
- `TLS_RSA_WITH_AES_128_CBC_SHA`
- `TLS_RSA_WITH_AES_256_CBC_SHA`

## 参考资料

> - []()
> - []()
