---
title: 从Java EE 到 MicroProfile 再到 Jakarta EE
toc: true
date: 2020-04-12 18:34:58
tags:
categories:
---

> 如果说Spring Cloud是以SpingBoot为核心和基础的微服务架构，那么MicroProfile则是将传统JavaEE轻量化以适应微服务时代的一个体系。

- 2016年6月：成立了MicroProfile项目
- 2017年9月：Oracle 选择 Eclipse 基金会作为 Java EE 的新东家，与IBM 和 Red Hat 共同做出了这个决定。
- 2018年2月：Eclipse宣布Java EE 正式改名为 Jakarta EE

当前企业都在进行数字化转型，其中构建高效可靠易扩展的IT系统架构核心就是微服务，在Java语言的微服务框架中Spring Boot和Spring Cloud凭借简单易上手，多种云原生功能的集成，异常活跃的社区，占据半壁江山，俨然已经成为微服务开发的默认标准，被公认为企业进行微服务新建和改造的首选。

那么作为传统的Java企业级架构Java EE似乎被有点冷落，发展遇到了各种问题，主要是微服务架构和云原生两大问题，为了更好的服务微服务架构2016年6月成立了MicroProfile项目，专注于微服务架构标准的制定，并加入Eclipse基金会。随后Java EE为了更好的拥抱云原生和保持创新，在2017年9月Oracle将其贡献给了Eclipse基金会了，并更名为Jakata EE。

MicroProfile项目的目标是在短周期内进行迭代和创新，以提出新的通用API和功能，获得社区的认可，发布和重复。最终该项目的输出可以提交给Jakarta EE，JCP，OpenJDK或任何相关的标准机构。

## Java EE的历史

1996年， Sun公司正式推出了Java 1.0版本，"一次编译，到处运行"的概念风靡全球技术生态圈，立刻赢得开发者的青睐，长期以来霸占着企业级开发的头把交椅。Java语言的发展和传播非常迅速，经历了JDK 1.1 和1.2 两个版本的发布之后，Sun公司随即就发布了**Java Platfrom,  Enterprise Edition**(*Java EE早期也称为J2EE，Java 2 Platform, Enterprise Edition*). **Java EE**的目标是基于**Java SE**(*Java Platform, Standard Edition*)建立一整套标准，致力于企业级Java应用的开发。

2006年底，Sun公司决定开始基于**GPL**开源绝大部分的Java虚拟机(JVM)代码，到了2007年初，整个开源过程结束，全部的JVM代码都已经开源并用于公开发行。随后OpenJDK诞生，成为Java SE版本的开源实现。直到2009年Sun公司被Oracle收购，十年的发展，Java EE版本也发布到了Java EE6，期间也部署了无数的企业应用，诞生了各种Java EE标准兼容的运行时应用服务器，知名的有Apache Tomcat， Weblogic，Websphere，Jboss，Glassfish等。

2009-2010年间，Oracle公司收购了Sun 公司，宣称会继续支持和投入Java的发展。然而2010年，Java之父James Gosling的离职和2012年跟Google的关于Java在安卓的使用法律诉讼，使得社区对于Java语言的发展信心蒙上了一层阴影。

为了给Java社区注入一剂强心剂，时隔5年之后，Oracle公司在2011年和2013年分别发布了JDK7和Java EE 7。虽然Java EE依然受到企业级的追捧，但是随着Spring的框架的不断演进，新的开发者和原先的一些企业开始转向Spring，同时Java EE自身平台变得原来越臃肿，无法满足业务的日益变化和适应云原生，微服务架构的技术需求。这一切使得Oracle坚信开源模式是对JAVA EE最好的选择。

## MicroProfile产生背景

- 官网地址：<https://microprofile.io/>
- 项目地址： <https://github.com/eclipse?utf8=%E2%9C%93&q=microprofile>

Java EE历来是企业级应用开发的重要框架，它在事务，安全，扩展性，并发，部署管理等各方面都建立起了完善的标准和规范，同时IMB，Redhat，Oracle等公司的各种实现Java EE标准的应用服务器，使得便捷的管理传统企业应用的开发部署，众多功能特点依然是应用开发的必备。其实谈到Java EE，绕不开的还是Spring，Java EE加上Spring统治着企业级Java应用开发，两者之间无关优劣，就像[Adam Bien](http://adam-bien.com/)说的，这是两种不同的哲学。但是Spring对于Java EE的影响是显而易见的，如Spring依赖注入是Java EE CDI构建的基础。

也正是因为Spring的快速发展和不断创新，特别是Spring Boot和Spring Cloud框架的建立使得Java开发者能快速开发微服务架构和云原生的应用。Spring Boot奉行约定大于配置的准则使得微服务开发的简单易用深入人心。Spring Cloud已经是云原生开发的重要平台，实现了服务注册和发现，负载均衡，监控等云原生的功能。既然Spring已经如此优秀，Java EE依然是微服务和云原生应用开发的选择吗？答案依然也是肯定的。为了解决这个问题，2016年6月，MicroProfile项目创立；

创立MicroProfile社区和项目的想法萌生于2016年的Devoxx UK和DevNation会议。正如前面提到的，Java EE的新版本发布持续延期，特别是在云原生和微服务架构应用方面的新技术使用场景又快速变化，让Java EE的开发者，厂商和相关组织不得不思考需要给企业级JAVA社区注入全新的微服务功能。于是MicroProfile社区就选择Eclipse基金会来管理这个项目，首先看重的是Eclipse基金会强大的开源项目的管理能力以及知识产权方面的经验。同样的Eclipse基金会目前没有微服务方面相关的项目运营，这也有助于建立在该领域的地位，因此这对MicroProfile和Eclipse基金会都是一个双赢的局面。

随后在2016年的JavaOne大会上面，本着致力于优化企业级Java微服务架构的宗旨，MicroProfile 1.0版本正式发布。参与的厂商主要包括Payara, Fujitsu, Tomitribe, IBM, Red Hat, Hammock, SmartBear, Hazelcast, Oracle和London Java Community (LJC)这里只是列举了一些，具体可以参考官网。MicroProfile只是一套标准和规范，本身并不提供实现，对应的运行时实现由各厂商提供，目前主要的实现包含：

- Open Liberty (<https://openliberty.io/>)
- Thorntail  (<https://thorntail.io/>)
- Payara (<https://www.payara.fish/upstream_builds>)
- Helidon (<https://helidon.io/>)

更具体的信息可以参考官网Wiki: <https://wiki.eclipse.org/MicroProfile/Implementation>

MicroProfile主要专注于微服务架构相关的规范比如可注入配置，故障隔离，安全(JWT), 可观测性，分布式跟踪，健康检查，OpenAPI，响应式异步REST客户端等，这些规范重要的一个目标是兼容目前业界主流的云平台如Kubernetes。现在MicroProfile项目如此优秀，那么Java EE该如何走，MicroProfile创建的相关规范和功能如何引入到Java EE？答案是Jakarta EE。

## Jakarta EE的诞生

- 官网地址：https://jakarta.ee
- 项目地址：https://github.com/eclipse-ee4j
- 改名公告：https://eclipse-foundation.blog/2018/02/26/and-the-name-is/

2017年Oracle正式决定将Java EE贡献给Eclipse基金会。起初项目被命名为**Eclipse Enterprise for Java (EE4J)**。因为考虑到Oracle实际拥有Java的商标权，为了避免法律纠纷，经过社区调查投票，新的项目正式更名为**Jakarta Enterprise Edition (Jakarta EE)**，Java EE 8是Jakarta EE的第一个版本，至此Java EE也进入了一个全新的发展纪元。

选择Eclipse的其中一个因素是它已经接手MicroProfile了，MicroProfile是一个旨在优化企业级Java以用于微服务和云架构的项目。在宣布选择Eclipse的博文中，Oracle提到他们希望“利用诸如MicroProfile之类的互补类项目”。MicroProfile原始协作者之一的Red Hat表示，他们希望将Java EE交给Eclipse以便IBM能够更容易调和Java EE和MicroProfile，这一态度同样也得到了IBM的回应。Oracle表示，在做出该行动前，首先向Red Hat和IBM寻求了帮助和支持，因为他们是三个组织中对Java EE的贡献最大的。完成此事后，Oracle接下来的工作就是细化将Java EE迁移到开源基金会的实际意义。他们最终得到了一项全面的战略，其中涵盖了技术细节、许可、品牌和过程定义，从而支持平台可以在未来继续演进。


Oracle 放弃了大部分 Java EE 知识产权之后，但由于 Oracle 目前仍保留了 Java 的商标权，所以Eclipse需要一个新的命名。当重命名在社区中被广泛讨论时，TomEE的大拿David Blevins提出，“我们不如再用`Jakarta`吧”，迅速得到了很多开发者的响应和支持。**Jakarta(雅加达)**这个名字对于Java开发者并不陌生，因为Java名称来源于印尼爪哇岛，所以印尼首都雅加达的英文名称很早就被Java社区所使用。Jakarta曾被Apache基金会用于管理各个Java子项目，诸如**Tomcat, Ant, Maven, Struts, JMeter, Velocity,Commons**等。2011年12月，在所有子项目都被迁移为独立项目后，**Jakarta**名称就不再使用了。所以，对于Java开发者来说，`J2EE, JavaEE, JEE`等都已经（或者正在）变成过去时了，随着JDK的模块化和包移除，javax名域以下（甚至很多java下面的）类和接口，都会改名。大家需要做好兼容技术准备。

Eclipse成立了Jakarta EE 专家工作组，不但有老牌中间件软件公司IBM, Oracle, Redhat, 也有Java阵营的大型软件公司SAP, Fujitsu, 也有EE专家成立的`Tomitribe, Payara`公司，而且连`Microsoft, Pivotal, Lightbend`等过去的“竞争者”，也都加入了Jakarta EE专家组，形成近年来空前强大的以软件公司为主的技术联盟。目前Oracle已经或者正在将原来JavaEE的产品文档；原JavaEE的TCK测试用例代码；Glassfish参考实现的源码，包含子项目如`Jersey, Mojarra, Tyrus, OpenMQ, EclipseLink, JsonP`等迁移到Eclipse组织。

Jakarta EE被寄予厚望，不但继承了原来JavaEE的企业级应用市场的优势，也准备在Cloud Native方向大有作为。不出意外的话，Microprofile的各项规范将会合并入下一版本的Jakarta EE规范之中。同时，也会对Docker和Kubernates等容器环境有优异的集成能力。目前打包为jakarta.*的API已经陆续开始进入maven中央仓库，下一步会发布Glassfish 5.1，使用全新的Eclipse授权声明和三方包，就标志着JakartaEE正式发布。 JakartaEE的TCK也已经开源在github之上，之前从事JavaEE应用服务器工作的朋友知道这个测试套件的商业价值。 另外JakartaEE活跃成员，同时也在为Microprofile技术规范积极共享，后续这两个Eclipse旗下的项目应该会合并。

## 运作流程

在云原生时代，我们见证的众多开源项目的成功，诸如Kubernetes，Envoy，Spring Boot，强大的社区灵活有效的运作模式对于项目的成功至关重要。反观在Java EE时代，最初Java的创建者Sun公司，后来的Oracle公司，维护着Java执行委员会JCP（Java Community Process)，控制着版本发布的节奏，其中的JSR（Java Specification Requests 规范提案）包含了20年来Java语言的规范文本和API定义，复杂繁琐的JCP审核流程，导致Java EE版本的发布和新功能的引入无比缓慢，这也是Jakarta EE试图改变的局面。

当前Jakarta EE工作组的成员包括：Fujitsu, IBM, Oracle, Lightbend, Payara, Pivotal, Red Hat, Tomitribe和Webtide。在整个管理架构中有四个委员会，分别是指导委员会，标准委员会，市场和品牌委员会，企业需求委员会，各司其职，相互协作。

在Eclipse基金会治下，工作组秉承自我管理的精髓，制定所有发布日常和计划，与此同时标准和规范的制定也完全以社区为导向，鼓励开发者，厂商和组织参与制定流程以此来反应社区的意志，通过快速迭代，在新版本中不断融入其它开源社区的功能创新，如MicroProfile，帮助开发者构建云原生应用。

Jakarta EE 规范将不会在 JCP 下，而是由 Jakarta EE 工作组定义并由规范委员会批准，JCP 将仅负责 Java SE 和 Java ME 规范。

2019年9月10日，Eclipse基金会宣布了Jakarta EE 8的完整平台和 Web 配置规范以及相关的兼容套件（TCK）的完全开源发布，Eclipse Enterprise for Java（EE4J）顶级项目将发布 Eclipse Glassfish 作为 Java EE 8 兼容实现。Jakarta EE 8 的规范与 Java EE 8 的规范完全兼容，是在 Jakarta EE 规范流程和 Eclipse 开发流程下开发的。Jakarta EE 8 同样还包含了 Java 开发者们一贯使用的相同的编程模型中的 API 和 Javadoc。Jakarta EE 8 的兼容包 TCK 与 Java EE 8 TCK 也是完全兼容的。所有的这些都意味着企业版用户将可以对程序不做任何修改就将项目迁移到 Jakarta EE 8上。

更名对应如下：

| **Old Name**                  | **New Name**                                                 |
| ----------------------------- | ------------------------------------------------------------ |
| Java EE                       | Jakarta EE                                                   |
| Glassfish                     | Eclipse Glassfish                                            |
| Java Community Process (JCP)  | Jakarta EE Working Group                                     |
| Oracle development management | Eclipse Enterprise for Java (EE4J)        Project Management Committee(PMC) |

JCP 被改为 Eclipse EE.next 工作组只是一个暂时的名字，而现在则正式更名为 Jakarta EE 工作组

Jakarta EE 工作组: https://www.eclipse.org/org/workinggroups/jakarta_ee_charter.php
EE4J：https://projects.eclipse.org/projects/ee4j/charter
PMC：https://projects.eclipse.org/projects/ee4j/pmc

## 参考资料

> - [<http://www.useopen.com/blog/2018/jakartaee_group/>]()
> - [<http://www.useopen.com/blog/2018/javaee_rename_to_jakarta/>]()
> - <https://www.jianshu.com/p/15d84533b915>
