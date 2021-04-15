---
title: Distributed-06-Paxos诞生
date: 2018-06-13 15:03:02
tags: Distributed
---

Google Chubby(Zookeeper是其开源版本)的作者Mike Burrows说过这个世界上只有一种一致性算法，那就是Paxos，其它的算法都是残次品。

![Leslie Lamport 本尊 from 他的博客 https://lamport.azurewebsites.net/ ](https://note.youdao.com/yws/api/personal/file/7B38405379744350B0BC0E855801DE38?method=download&shareKey=98d548a772026b3c91ea2c2692486de4)


larmport的paxos相关的三篇论文：
1. [The Part-Time Parliament](https://lamport.azurewebsites.net/pubs/lamport-paxos.pdf)
1. [Paxos made simple](https://lamport.azurewebsites.net/pubs/paxos-simple.pdf)
1. [Fast Paxos](https://www.microsoft.com/en-us/research/publication/fast-paxos/)
 

## 追本溯源

1982年，Lamport与Robert Shostak、Marshall Pease共同发表了论文The Byzantine Generals Problem(拜占庭将军问题)，提出了一计算机容错理论。在理论描述过程中，为了将所要描述的问题形象的表达出来，Lamport设想出了下面这样一个场景：

> 拜占庭帝国有许多支军队，不同军队的将军之间必须制定一个统一的行动计划，从而做出进攻或者撤退的决定，同时,各个将军在地理上都是被分隔开来的，只能依靠军队的通讯员来进行通讯。然而，在所有的通讯员中可能会存在叛徒，这些叛徒可以任意篡改消息，从而达到欺骗将军的目的。

这就是著名的“拜占廷将军问题”。从理论上来说，在分布式计算领域，试图在异步系统和不可靠的通道上来达到一致性状态是不可能的，因此在对一致性的研究过程中，都往往徦设信道是可靠的。而事实上，大多数系统都是部署在同一个局域网中的，因此消息被篡改的情况非常罕见；另一方面，由于硬件和网络原因而造成的消息不完整问题，只需一套简单的校验算法即可避免----因此，在实际工程中，可以假设不存在拜占庭问题，也即假设所有消息都是完整的，没有被篡改的。那么，在这种情况下需在什么样的算法来保证一致性呢?

Lamport在1990年提出了一个理论上的一致性解决方案，同时给出了严格的数学证明。鉴于之前采用故事类比的方式成功的阐述了“拜占庭问题”，因此这次Lamport同样用心良苦地设想出了一个场景来描述这种一致性算法需在解决的问题，及其具体的解决过程：

> 在古希腊有一个叫做Paxos的小岛，岛上采用议会的形式来通过法令，议会中的议员通过信使来进行消息的传递。值得注意的是，议员和信使都是兼职的，他们随时有可能会离开议会厅，并且信使可能会重复的传递消息，也可能一去不复返，因此，议会协议要保证在这种情况下法令仍然能够正确的产生，并且不会出现冲突。

这就是论文The Part-Time Parliament中提到的兼职议会，而Paxo算法名称也是由来也是取自论文中提到的Paxos小岛。在这个论文中，Lamport压根没有说Paxos小岛是虚构出来的，而是人开展议会的方法。因此，在这个论文中，Lamport从问题的提出到算法的推演论证，通篇贯穿发对Paxos议会历史的描述。

## 理论诞生
在讨论Paxos理论之前，我们不得不首先来介绍下Paxos算法的作者Leslie Lamport(莱斯利-兰伯特)及其对计算机科学尤其是分布式计算领域的杰出贡献。作为2013年的新科图灵奖得主，现年73岁的Lamport是计算机科学领域一位拥有杰出成就的传奇人物，其先后多次荣获ACM和IEEE以及其他各类计算机重大奖项。他对时间时钟、面包店算法、拜占庭将军问题以及Paxos算法的创造性研究，极大的推动了分布式计算领域的发展，全世界无数工程师得益于他的理论。

说起Paxos理论的发表，还有一段非常有趣的历史故事。Lamport早在1990年就已经将其对Paxos算法的研究论文The Part-Time Parliament提交给ACM TOCS 的评审委员会了，但是由于Lamport创造性地使用了故事的方式来进行算法的描述，导致当时委员会的工作人员没有一个能正确理解其对算法的描述，时任主编要求其用严谨的数据证明方式来描述该算法，否则它们将不考虑接收这篇论文。遗憾的是，Lamport并没有接收它们的建议，当然也就拒绝了对论文的修改，并撤销了对这篇论文的提交。在后来的一个会议上，Lamport还对此事情耿耿于怀：“为什么这些搞理论的人一点幽默感也没有呢？”

幸运的是，还是有人能够理解Lamport那公认的令人晦涩的算法的。1996年，来自微软的Butler Lampson在WDAG96上提出了重新审视这篇分布式论文的建议，在次年的WDAG97上，麻省理工学院的Nancy Lynch也公布了其根据Lamport的原文重新修改后的Revisiting the Paxos Algorithm, “帮助”Lamport用数学的形式化术语定义并证明了Paxos算法。于是在1998年ACM TOCS上，这篇延时了9年的论文终于被接收了，也标志Paxos算法正式被计算机科学接收并开始影响更多工程师解决分布式一致性问题。

后来在2001年，Lamport本人也作出了让步，这次他放弃了故事的描述方式，而是使用了通俗易懂的语言重新讲述了原文。并发表了Paxos Made Simple -----------当然，Lamport甚为固执地任务他自己的表述语言没有歧义，并且也足够让人明白Paxos算法，因此不需在数学来协助描述，于是整篇论文还是没有任何数学符号。好在这篇文章已经能被大多数人理解。

由于Lamport个人自负固执的性格，使得Paxos理论的诞生可谓一波三折。关于Paxos理论的诞生过程，后来也成为计算机科学领域被广泛流传的学术趣事。