---
title: Distributed-07-Paxos协议运行过程
date: 2018-06-13 15:03:12
tags: Distributed
---

## 前言
> 二阶段提交还是三阶段提交都无法很好的解决分布式的一致性问题，直到Paxos算法的提出，Paxos协议由**Leslie Lamport**最早在1990年提出，目前已经成为应用最广的分布式一致性算法。

Paxos协议在节点宕机恢复、消息无序或丢失、网络分化的场景下能保证决议的一致性，是被讨论最广泛的一致性协议。

## 节点角色
Paxos 协议中，有三类节点

#### Proposer:提案者
Proposer 可以有多个，Proposer 提出议案(value)。

所谓 value，在工程中可以是任何操作，例如“修改某个变量的值为某个值”、“设置当前 primary 为某个节点”等等。Paxos 协议中统一将这些操作抽象为 value。

不同的 Proposer 可以提出不同的甚至矛盾的 value，例如某个 Proposer 提议“将变量 X 设置为 1”，另一个 Proposer 提议“将变量 X 设置为 2”，但对同一轮 Paxos 过程，最多只有一个 value 被批准。

#### Acceptor:批准者
Acceptor 有 N 个，Proposer 提出的 value 必须获得超过半数(N/2+1)的
Acceptor 批准后才能通过。

Acceptor 之间完全对等独立。

#### Learner:学习者
Learner 学习被批准的 value。所谓学习就是通过读取各个 Proposer 对 value 的选择结果，如果某个 value 被超过半数 Proposer 通过，则 Learner 学习到了这个 value。

这里类似 Quorum 议会机制，某个 value 需要获得 W=N/2 + 1 的 Acceptor 批准，Learner 需要至少读取 N/2+1 个 Accpetor，至多读取 N 个 Acceptor 的结果后，能学习到一个通过的 value。

## 约束条件
上述三类角色只是逻辑上的划分，实践中一个节点可以同时充当这三类角色。有些文章会添加一个Client角色，作为产生议题者，实际不参与选举过程。

Paxos中 proposer 和 acceptor 是算法的核心角色，paxos 描述的就是在一个由多个 proposer 和多个 acceptor 构成的系统中，如何让多个 acceptor 针对 proposer 提出的多种提案达成一致的过程，而 learner 只是“学习”最终被批准的提案。

Paxos协议流程还需要满足几个约束条件：
- Acceptor必须接受它收到的第一个提案；
- 如果一个提案的v值被大多数Acceptor接受过，那后续的所有被接受的提案中也必须包含v值（v值可以理解为提案的内容，提案由一个或多个v和提案编号组成）；
- 如果某一轮 Paxos 协议批准了某个 value，则以后各轮 Paxos 只能批准这个value；

## 消息通信类型
每轮 Paxos 协议分为准备阶段和批准阶段，在这两个阶段 Proposer 和 Acceptor 有各自的处理流程。

Proposer与Acceptor之间的交互主要有4类消息通信，这4类消息对应于paxos算法的两个阶段4个过程：

#### Phase 1
1. proposer向网络内超过半数的acceptor发送prepare消息
1. acceptor正常情况下回复promise消息
#### Phase 2
1. 在有足够多acceptor回复promise消息时，proposer发送accept消息
1. 正常情况下acceptor回复accepted消息

## 确定一个议案的过程

### Phase 1 准备阶段
1. Proposer 生成全局唯一且递增的ProposalID，向 Paxos 集群的所有机器发送 Prepare请求，这里不携带value，只携带N即ProposalID 。

2. Acceptor 收到 Prepare请求 后，判断：收到的ProposalID 是否比之前已响应的所有提案的N大：
* 如果是，则：
    - (1) 在本地持久化 N，可记为Max_N。
    - (2) 回复请求，并带上已Accept的提案中N最大的value（若此时还没有已Accept的提案，则返回value为空）。
    - (3) 做出承诺：不会Accept任何小于Max_N的提案。 

* 如果否：不回复或者回复Error。

### Phase 2 选举阶段
#### P2a：Proposer 发送 Accept

经过一段时间后，Proposer 收集到一些 promise 回复消息，有下列几种情况：

1. 回复数量 > 一半的Acceptor数量，且所有的回复的value都为空，则Porposer发出accept请求，并带上自己指定的value。
1. 回复数量 > 一半的Acceptor数量，且有的回复value不为空，则Porposer发出accept请求，并带上回复中ProposalID最大的value(作为自己的提案内容)。
1. 回复数量 <= 一半的Acceptor数量，则尝试更新生成更大的ProposalID，再转P1a执行。

#### P2b：Acceptor 应答 Accept
Accpetor 收到 Accpet请求 后，判断：

1. 收到的N >= Max_N (一般情况下是 等于)，则回复提交成功，并持久化N和value。
1. 收到的N < Max_N，则不回复或者回复提交失败。

#### P2c: Proposer 统计投票
经过一段时间后，Proposer 收集到一些 Accept 回复提交成功，有几种情况：
1. 回复数量 > 一半的Acceptor数量，则表示提交value成功。此时，可以发一个广播给所有Proposer、Learner，通知它们已commit的value。
1. 回复数量 <= 一半的Acceptor数量，则 尝试 更新生成更大的 ProposalID，再转P1a执行。
1. 收到一条提交失败的回复，则尝试更新生成更大的 ProposalID，再转P1a执行。

#### 如何产生唯一的编号呢？
在《Paxos made simple》中提到的是让所有的Proposer都从不相交的数据集合中进行选择，例如系统有5个Proposer，则可为每一个Proposer分配一个标识j(0~4)，则每一个proposer每次提出决议的编号可以为5*i + j(i可以用来表示提出议案的次数)。

