---
title: '#BlockChain Tendermint共识'
date: 2021-09-01 14:29:56
tags:
    - BlockChain
---
# Tendermint 

一种拜占庭共识算法

不同环境 安全 一致性 复用

Core + ABCI， 是对共识层以下的一层封装

![note image](https://pic3.zhimg.com/80/v2-ce91a17d2eaf8c6a472c404663006b7e_1440w.jpg)

---
## 1. 核心模块
### ABCI app
- socket 链接
- 实现ABCI 接口
    - checkTX
    - DeliverTX
    - Commit
- 能处理 ABCI message

### Tendermint Core
- 共识
- P2P 网络层
- RPC 区块链接口
- 交易缓存，队列

---


## 1. 算法 - 阶段

![note image](https://pic1.zhimg.com/80/v2-ebcaa425624856f3b88109e24baf5a20_1440w.jpg)

NewHeight
- 进入到下一轮 共识
- round robin 选择 proposer （voting power）

**Propose**
- 判定 有没有lock 的 block
    - 没有 -> gossip 广播proposal
    - 有 -> 直接propose locked block + proof of block

**preVote**
- LOCK 判定
    - 有 就对lock的block投票
    - 没有 就对当前轮
- 同时收集 prevote 的投票 打包 进入 PoLC

- 如果自己有 Lock-Block，这时又收到一个新的针对另外一个块的 PoLC，并且满足LastLockRound < PoLC-Round < 当前 Round，则解锁 Lock-Block。 

- 如果 timeout 期间没收到 proposal，或者收到的 proposal 是无效的，那么就投 nil 票。
在 Prevote 阶段不会锁住任何 block。

**preCommit**
- Prevote 超时或者收到的 Prevote 的 nil 票超过 2/3 时 进入此阶段

- 如果此时收到了 +2/3 的 prevote 投票，就广播一条 precommit 投票，同时，把自己锁在当前的 block 上（把之前的都释放掉）。 LastLockRound 置为当前 Round

- 如果收到 +2/3 的 nil 投票，那么就释放锁。投 precommit
- 收到 +2/3 投票进入 commit， 否则 下一轮 propose

Commit
- 节点必须收到该 block
- 节点必须等待，直到收到 2/3 的 节点 commit 信息。

---
## Feature

- propose + preVote + preCommit 称为Round
- commit 前可能会经过多个round
- PoLC - proof of lock change - 表示特定 块+高度+轮数 上 prevote 投票集合
- 锁定机制：一旦验证人预投票了一个区块，那么该验证人就会被锁定在这个区块。然后：
    1. 该验证人必须在预提交的区块进行预投票。
    2. 当前一轮预提议和预投票没成功提交区块时，该验证人就会被解锁，然后进行对新块的下一轮预提交。

---
## 优势 bft-raft
1. 同一个高度不会有多个快，不会分叉

## 与PBFT
1. 相同点：
 1）同属BFT体系。
 2）抗1/3拜占庭节点攻击。
 3）三阶段提交，第一阶段广播交易（区块），后两阶段广播签名（确认）。
 4）两者都需要达到法定人数才能提交块。

2. 不同点：
    1. Tendermint与PBFT的区别主要是在超过1/3节点为拜占庭节点的情况下。

    当拜占庭节点数量在验证者数量的1/3和2/3之间时，PBFT算法无法提供保证，使得攻击者可以将任意结果返回给客户端。而Tendermint共识模型认为必须超过2/3数量的precommit确认才能提交块。举个例子，如果1/2的验证者是拜占庭节点，Tendermint中这些拜占庭节点能够阻止区块的提交，但他们自己也无法提交恶意块。而在PBFT中拜占庭节点却是可以提交块给客户端。
    简单的说，就是比特币的网络存在分叉的可能，而Tendermint不会发生这种情况。

    2. 另一个不同点在于拜占庭节点概念不同，PBFT指的是节点数，而Tendermint代表的是节点的权益数，也就是投票权力。

    3. 最后一点，PBFT需要预设一组固定的验证人，而Tendermint是通过要求超过2/3法定人数的验证人员批准会员变更，从而支持验证人的动态变化。


[Reference详解](https://cloud.tencent.com/developer/article/1446865)



