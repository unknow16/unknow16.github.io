---
title: 从Java EE 到 Jakarta EE
toc: true
date: 2021-04-16 14:05:23
tags:
categories:
---

## 相关名词
- JCP: 全称Java Community Process，是一个开放的国际组织，主要由Java开发者以及被授权者组成，维护Java相关规范，审核Java规范提案。JCP官网：https://jcp.org/
- JSR: 全称Java Specification Requests，译为Java规范提案。
- TCK: 全称Technology Compatibility Kit，用于测试基于API和规范文档对应实现的测试代码，成功通过TCK测试是实现是对应规范的兼容实现
- Glassfish：成功通过TCK测试的规范实现，在Java EE被捐给Eclipse基金会之前，称Oracle Glassfish，是Java EE的兼容实现，现在称Eclipse Glassfish，是Jakarta EE的兼容实现。

## Java EE的历史
1. 1996年， Sun公司正式推出了Java 1.0版本，"一次编译，到处运行"的概念风靡全球技术生态圈，立刻赢得开发者的青睐，长期以来霸占着企业级开发的头把交椅。Java语言的发展和传播非常迅速，经历了JDK 1.1 和1.2 两个版本的发布之后，Sun公司随即就发布了Java企业级版本，简称J2EE。它的目标是基于Java SE建立一整套标准，致力于企业级Java应用的开发。
2. 2006年5月，“J2EE”一词被弃用，并选择了Java EE这个名称。同样，作为Java SE 5（2004）的一部分，数字2也从J2SE中删除了。
3. 2006年底，Sun公司决定开始基于GPL开源绝大部分的Java虚拟机(JVM)代码，到了2007年初，整个开源过程结束，全部的JVM代码都已经开源并用于公开发行。随后OpenJDK诞生，成为Java SE版本的开源实现。http://openjdk.java.net/
4. 直到2009年Sun公司被Oracle收购，十年的发展，Java EE版本也发布到了Java EE6，期间也部署了无数的企业应用，诞生了各种Java EE标准兼容的运行时应用服务器，知名的有Apache Tomcat，Oracle Weblogic，Oracle Jboss，IBM Websphere等。
5. 2009-2010年间，Oracle公司收购了Sun 公司，宣称会继续支持和投入Java的发展。然而2010年，Java之父James Gosling的离职和2012年跟Google的关于Java在安卓的使用法律诉讼，使得社区对于Java语言的发展信心蒙上了一层阴影。
6. 为了给Java社区注入一剂强心剂，时隔5年之后，Oracle公司在2011年和2013年分别发布了JDK7和Java EE 7。虽然Java EE依然受到企业级的追捧，但是随着Spring的框架的不断演进，新的开发者和原先的一些企业开始转向Spring，同时Java EE自身平台变得原来越臃肿，无法满足业务的日益变化和适应云原生，微服务架构的技术需求。这一切使得Oracle坚信开源模式是对JAVA EE最好的选择。

- 2015年7月3日：Java EE 7规范为JSR-342，https://jcp.org/en/jsr/detail?id=342
- 2017年9月18日：Java EE 8规范为JSR-366，https://jcp.org/en/jsr/detail?id=366

## MicroProfile产生背景
传统的Java EE发展遇到了各种问题，主要是微服务架构和云原生两大问题，为了更好的服务微服务架构，2016年6月Java EE社区大公司联合成立了MicroProfile项目，专注于微服务架构标准的制定，并加入Eclipse基金会。随后Java EE为了更好的拥抱云原生和保持创新，在2017年9月Oracle将其贡献给了Eclipse基金会了，并更名为Jakata EE。

MicroProfile只是一套标准和规范，本身并不提供实现，对应的运行时实现由各厂商提供，另外主要专注于微服务架构相关的规范比如可注入配置，故障隔离，安全(JWT), 可观测性，分布式跟踪，健康检查，OpenAPI，响应式异步REST客户端等，这些规范重要的一个目标是兼容目前业界主流的云平台如Kubernetes。现在MicroProfile项目如此优秀，那么Java EE该如何走，MicroProfile创建的相关规范和功能如何引入到Java EE？答案是Jakarta EE。

MicroProfile项目的目标是在短周期内进行迭代和创新，以提出新的通用API和功能，获得社区的认可，发布和重复。最终该项目的输出可以提交给Jakarta EE，JCP，OpenJDK或任何相关的标准机构。

- 官网：https://microprofile.io/
- 项目地址: https://projects.eclipse.org/projects/technology.microprofile
- 项目wiki: https://wiki.eclipse.org/MicroProfile
- 项目源码：https://github.com/eclipse?utf8=%E2%9C%93&q=microprofile
- 规范实现：https://wiki.eclipse.org/MicroProfile/Implementation

## Jakarta EE的诞生
2017年Oracle正式决定将Java EE贡献给Eclipse基金会。Oracle放弃了大部分 Java EE 知识产权之后，但由于 Oracle 目前仍保留了 Java 的商标权，所以Eclipse需要一个新的命名，经过社区调查投票，新的项目正式更名为Jakarta EE，成立Eclipse Enterprise for Java (EE4J)父项目进行管理。

目前Oracle已经或者正在将原来JavaEE的产品文档；原JavaEE的TCK测试用例代码；Glassfish参考实现的源码，包含子项目如`Jersey, Mojarra, Tyrus, OpenMQ, EclipseLink, JsonP`等迁移到Eclipse组织。

Jakarta EE被寄予厚望，不但继承了原来JavaEE的企业级应用市场的优势，也准备在Cloud Native方向大有作为。不出意外的话，Microprofile的各项规范将会合并入下一版本的Jakarta EE规范之中。同时，也会对Docker和Kubernates等容器环境有优异的集成能力。

在云原生时代，我们见证的众多开源项目的成功，诸如Kubernetes，Envoy，Spring Boot，强大的社区灵活有效的运作模式对于项目的成功至关重要。反观在Java EE时代，最初Java的创建者Sun公司，后来的Oracle公司，维护着Java执行委员会JCP（Java Community Process)，控制着版本发布的节奏，其中的JSR（Java Specification Requests 规范提案）包含了20年来Java语言的规范文本和API定义，复杂繁琐的JCP审核流程，导致Java EE版本的发布和新功能的引入无比缓慢，这也是Jakarta EE试图改变的局面。在Eclipse基金会治下，工作组秉承自我管理的精髓，制定所有发布日常和计划，与此同时标准和规范的制定也完全以社区为导向，鼓励开发者，厂商和组织参与制定流程以此来反应社区的意志，通过快速迭代，在新版本中不断融入其它开源社区的功能创新，如MicroProfile，帮助开发者构建云原生应用。

Jakarta EE 规范将不会在 JCP 下，而是由 Jakarta EE 工作组定义并由规范委员会批准，JCP 将仅负责 Java SE 和 Java ME 规范。

2019年9月10日：发布Jakarta EE 8，完全兼容Java EE 8，主要是进行了代码迁移及包改名
2020年11月20日：发布Jakarta EE 9

---
- 官网：https://jakarta.ee/
- EE4J项目地址：https://projects.eclipse.org/projects/ee4j
- EE4J项目源码地址：https://github.com/eclipse-ee4j
- Jakarta EE规范文档：https://github.com/eclipse-ee4j/jakartaee-platform/releases
- 改名公告：https://eclipse-foundation.blog/2018/02/26/and-the-name-is/

## 参考资料
> - [http://www.useopen.com/categories/jakartaee/](http://www.useopen.com/categories/jakartaee/)
> - []()
