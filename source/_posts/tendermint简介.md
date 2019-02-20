---
title: tendermint简介
categories:
  - tendermint
tags:
  - tendermint
date: 2018-12-10 14:28:21
---

## 1.1 tendermint简介
tendermint是一个开源的完整的区块链实现，可以用于公链或联盟链，其官方定位 是面向开发者的区块链共识引擎：

与其他区块链平台例如以太坊或者EOS相比，tendermint最大的特点是其差异化的定位： 尽管包含了区块链的完整实现，但它却是以SDK的形式将这些核心功能提供出来，供开发者 方便地定制自己的专有区块链：

![tendermint](https://lh3.googleusercontent.com/-7tK7wrbhhT8/XAiIonVqubI/AAAAAAAAEFI/S_7isBa52LAk1s-_J-QCvvev28w581AdwCHMYCw/I/tendermint.png)


tendermint的SDK中包含了构造一个区块链节点旳绝大部分组件，例如加密算法、共识算法、 区块链存储、RPC接口、P2P通信等等，开发人员只需要根据其应用开发接口 （Application Blockchain Communication Interface）的要求实现自己 的应用即可。

ABCI是开发语言无关的，开发人员可以使用自己喜欢的任何语言来开发基于tendermint的 专用区块链。不过由于tendermint本身是采用go语言开发的，因此用go开发ABCI应用的一个额外好处 就是，你可以把tendermint完整的嵌入自己的应用，干净利落地交付一个单一的可执行文件。
<!--more-->
## 1.2 tendermint的共识算法
在技术方面，tendermint引以为傲的是其共识算法 —— 世界上第一个可以应用于公链的拜占庭 容错算法。tendermint曾于2016年国际区块链周获得最具创新奖，并在Hyperledger的雨燕（Burrow） 等诸多产品中被采纳为共识引擎。你可以点击 [这里](https://forum.cosmos.network/t/list-of-projects-in-cosmos-tendermint-ecosystem/243) 查看其应用案例。

tendermint采用的共识机制属于一种权益证明（ Proof Of Stake）算法，一组验证人 （Validator）代替了矿工（Miner）的角色，依据抵押的权益比例轮流出块：

![consensus-pos](https://lh3.googleusercontent.com/-IXSEtA-dXcY/XAiIohkad-I/AAAAAAAAEFQ/7pD-ir6VXEIThFTDRL0SHkwwD6_jriy9ACHMYCw/I/consensus-pos.png)


由于避免了POW机制，tendermint可以实现很高的交易吞吐量。根据官方的说法，在 合理（理想）的应用数据结构支持下，可以达到42000交易/秒，引文参考 [这里](https://github.com/tendermint/tendermint/wiki/Benchmarks)。 不过在现实环境中，部署在全球的100个节点进行共识沟通，实际可以达到1000交易/秒。

tendermint同时是拜占庭容错的（Byzantine Fault Tolerance），因此对于3f+1个 验证节点组成的区块链，即使有f个节点出现拜占庭错误，也可以保证全局正确共识的达成。同时 在极端环境下，tendermint在交易安全与停机风险之间选择了安全，因此当超过f个验证节点发生 故障时，系统将停止工作。

什么是拜占庭错误？简单的说就是任何错误：既包括节点宕机、也包括恶意节点的欺骗和攻击。

tendermint共识机制的另一个特点就是其共识的最终确定性：一旦共识达成就是真的达成， 而不是像比特币或以太坊的共识是一种概率性质的确定性，还有可能在将来某个时刻失效。 因此在tendermint中不会出现区块链分叉的情况。

## 1.3 tendermint vs. 以太坊
tendermint的定位决定了在最终交付的节点软件分层中，应用程序占有相当部分的分量。 让我们通过与以太坊的对比来更好地理解这一点：

![td-vs-ethe](https://lh3.googleusercontent.com/-oPtxJ3NzSfo/XAiIokGDwnI/AAAAAAAAEFM/YMXMTtwH6qA3UoKGgVgBUVFfWJ0ApVmowCHMYCw/I/td-vs-ether.png)


在上图中，tendermint结构中的abci应用和以太坊结构中的智能合约，都是由用户代码实现的。 显然，ABCI应用大致与EVM+合约的组合相匹配。

在以太坊中，节点是一个整体，开发者提供的智能合约则运行在受限的虚拟机环境中；而在 tendermint中，并不存在虚拟机这一层，应用程序是一个标准的操作系统进程，不受任何 的限制与约束 —— 听起来这很危险，但当你考虑下使用tendermint的目的是构建专有的区块链 时，这种灵活性反而更有优势了。

事实上，tendermint留下的应用层空间如此之大，以至于你完全可以在ABCI应用中实现一个 EVM，然后提供solidity合约开发能力，这就是超级账本的 [Burrow](https://cn.hyperledger.org/projects/hyperledger-burrow) 做的事情。

## 获得配套代码资料
关注微信公众号`区块链001`, 回复`tendermint`获得

