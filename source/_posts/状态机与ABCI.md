---
title: 三状态机与ABCI
categories:
  - tendermint
tags:
  - tendermint
  - 状态机
  - ABCI
date: 2018-12-17 19:13:21
---

# 状态机与ABCI

## 状态机
tendermint采用的分布式计算模型为状态机复制（State Machine Replication），其基本 思路就是通过在多个节点间通过同步输入序列来保证各节点状态机的同步。

状态机是一种在通信软件、游戏、工业控制等领域应用非常广泛的计算模型，用来抽象地表示一个 系统的演化过程。状态机由一组（有限或无限的）状态以及激发状态迁移的外部输入组成， 对于确定性状态机而言，在某一个时刻一定处于一个确定的状态，而在一个状态下 针对特定输入的状态迁移也是确定的。

现在让我们看一个计数器的状态以及其变化情况，在某一个特定时刻其状态为特定的数值：

counter state machine
![sm-counte](https://lh3.googleusercontent.com/-4X19ZKNBNgA/XAiKDw17mvI/AAAAAAAAEGg/xhsVl_BXkwwJLl3Qq_RkCiJvZTCXqr7KQCHMYCw/I/sm-counter.png)

显然，计数器有无限个状态（1,2,3...），但只有三个触发动作：

* inc - 递增
* dec - 递减 
* reset - 复位
<!--more-->
当涉及到区块链时，这些来自状态机外部的触发动作通常被称为交易/Transaction， 将被永久性地保存在区块链上，成为区块链不可篡改特性的一个基石。

## 状态机复制

状态机复制是指在多个节点中的状态机保持一致，彼此互为副本，无论客户端访问哪一个 节点，都能得到同样的状态；无论客户端向哪一个节点提交交易，也都能保证各节点可以 最终过渡到一致的新状态。

尽管是显而易见的，但依然值得指出，只有确定性状态机才可以利用状态机 复制模型实现分布式共识。

显然，当任意一个节点收到交易请求时，首先需要与其他节点进行协调，确认达成一致意见后， 再分别于不同的节点执行同样的交易序列 —— 状态机复制就是通过在各节点之间保持交易序列 （状态机的外部输入）的一致次序来保证最终状态的一致性的，而节点间协调的过程，就是我们 所说的共识算法。

下图反映了tendermint作为共识引擎时，RPC客户端、tendermint程序和abci应用 三者之间（简化）的交互时序：
![sm-replica](https://lh3.googleusercontent.com/-dG1enIwN3nU/XAiKEeUnSUI/AAAAAAAAEGo/tXTtVMIQimk5ZPU3I2xWEBbkzy72UyCAgCHMYCw/I/sm-replica.png)

当RPC客户端提交一个新的交易后，该交易首先进入tendermint的交易池，然后tendermint将与 其他节点就要执行哪些交易的问题通过p2p协议进行协调，达成共识后，tendermint才会通知应用状态机 执行交易更新状态。

显然，在整个状态机复制模型中，作为状态机的ABCI应用是被动的，它只需要响应来自共识引擎 的ACBI消息，并执行相应的动作即可。 


## ABCI 接口概述
tendermint将ABCI协议交互过程进行了封装，开发者只需要实现Application接口，等待tendermint 在合适的时机调用就可以了：

![uml-abci-application](https://lh3.googleusercontent.com/-z6DvEYgdtCk/XAiKPjuPYHI/AAAAAAAAEG0/T9OJLxTPJUgMZ0YVSGaK956aEPIn8aavQCHMYCw/I/uml-abci-application.png)


上图列出了Application接口约定的方法，每个方法对应于一个特定的ABCI消息：

Info：当tendermint与ABCI应用建立初始连接时，将发送Info请求消息尝试获取应用状态对应的区块 高度、状态哈希等信息，以确定是否需要重放（replay）区块交易。

Query：当Rpc客户端发出abci_query调用时，tendermint将通过Query请求消息转发给ABCI应用，并 将响应结果转发回Rpc客户端。

CheckTx：当tendermint从Rpc接口或p2p端口收到新的交易时，首先会通过CheckTx请求消息提给ABCI应用 进行初步检查，确认该交易是否合规。只有ABCI应用确认有效的消息才会进入tendermint的交易 池等待下一步的共识确认。

InitChain：当创建创世块时，tendermint会发送InitChain请求消息给ABCI应用，可以在此刻进行 应用状态机的状态初始化。

BeginBlock/EndBlock：当tendermint就新区块交易达成共识后，将通过BeginBlock请求开始启动ABCI应用 的交易执行流程，并以EndBlock请求作为交易执行流程的结束。

DeliverTx：在BeginBlock和EndBlock请求之间，tendermint会为区块中的每一个交易向ABCI应用 发出一个DeliverTx请求消息，这是应用状态机更新的时机。

Commit：作为执行交易的最后一步，tendermint会发送Commit请求，并在获取响应后持久化区块状态。

## 交易检查：CheckTx
为了减轻共识环节的工作负担，对于通过rpc接口提交的交易，tendermint引入了 交易检查环节，只有检查成功的交易才能够进入交易池等待确认，否则直接拒绝：

![abci-checktx](https://lh3.googleusercontent.com/-1-zUz1B_UnY/XAZR4IstqJI/AAAAAAAAEEs/MO9w6CYPll407QDP_NhoTBMIWvpDi8-mgCHMYCw/I/abci-checktx.png)


例如，下面的代码检查交易是否为指定的三种交易之一（0x01 - inc，0x02 - dec ,0x03 - reset）， 否则拒绝：
```
func (app *EzApp) CheckTx(tx []byte) types.ResponseCheckTx{
  if tx[0]  < 0x04 { 
    return types.ResponseCheckTx{}
  }
  return types.ResponseCheckTx{Code:1,Log:"bad tx rejected"}
}
```
`CheckTx()`方法返回的是一个ResponseCheckTx结构，其组成与ResponseDeliverTx 相同：

![rps-checktx](https://lh3.googleusercontent.com/-aKE1lVKJtVs/XAZR4DggZUI/AAAAAAAAEEo/-uX8irWbscQIALtcyIZAMVabCFEmaUFMACHMYCw/I/rps-checktx.png)


同样，结构中只有Code是必须的，为0值时表示交易执行成功，不同的非0值的失败含义由 应用自行定义。

现在，当我们对修改过的ABCI应用试图提交如下的交易时，将发生错误：

`~$ curl http://localhost:26657/broadcast_tx_commit?tx=0x78`
结果如下：

![rsp-checktx](https://lh3.googleusercontent.com/-p5rpZCNDpXQ/XAZR4Bqmi3I/AAAAAAAAEEw/XZZ6X7rbiwktKB8FvreKmHOpxjkKcfTbQCHMYCw/I/rsp-checktx.png)


交易的唯一性要求

为了对抗重放攻击，以及避免节点间反弹的重复消息引起共识过程死循环，tendermint在 触发CheckTx()之前会使用一个缓存拒绝近期已经出现过的交易。因此当你试图重复提交 一个交易时，将提示该交易已经存在的错误：tx already exists in cache。

解决的办法就是为交易附加一个序列号，以保证交易的唯一性。例如，当我们使用0x01表示 inc交易时，可以如下的方式多次提交交易：
```
~$ curl http://localhost:26657/broadcast_tx_commit?tx=0x0101
~$ curl http://localhost:26657/broadcast_tx_commit?tx=0x0102
~$ curl http://localhost:26657/broadcast_tx_commit?tx=0x0103
```
交易序列号的设计完全取决于特定的应用，tendermint只是简单地拒绝已经在缓存中的交易。

## 交易执行：DeliverTx

计数器应用只有一个要维护的状态（计数值），因此第一版的实现很简单，我们 只需要在DeliverTx方法中检查交易类型，然后增减或复位成员变量Value即可：
```
type CounterApp struct{
  types.BaseApplication
  Value int64
}

func (app *CounterApp) DeliverTx(tx []byte) types.ResponseDeliverTx{
  switch tx[0] {
    case 0x01: app.Value += 1
    case 0x02: app.Value -= 1
    case 0x03: app.Value = 0
    default: return types.ResponseDeliverTx{Code:0,Log:"bad tx"}
  }
  info := fmt.Printf("value updated : %v",app.Value)
  return types.ResponseDeliverTx{Info: info}
}
```
`DeliverTx()`方法的返回结果是一个ResponseDeliverTx结构，其定义如下：

![rsp-delivertx](https://lh3.googleusercontent.com/-_ExelDcyWOk/XAiKQiZg7fI/AAAAAAAAEHI/g5CtcJs0p0M2SIajwa5OL7UYFSOK9q4bwCHMYCw/I/rsp-delivertx.png)


结构中只有Code是必须的，为0值时表示交易执行成功，不同的非0值的失败含义由 应用自行定义。

### 状态初始化：InitChain

当tendermint准备构建创始区块时，将向abci应用发送InitChain消息，同时在该消息 请求中可以携带创世文件genesis.json中应用特定的初始化状态数据（app_byte）， 因此是应用进行状态初始化的好时机：

![req-initchain](https://lh3.googleusercontent.com/-rlJ2-JdfY1M/XAiKPuw9QXI/AAAAAAAAEG4/QBHZ5JLZ56wxCnmJRHjgr3LF3ecxOWh7wCHMYCw/I/req-initchain.png)


例如，下面的代码将计数状态从100开始：
```
func (app *CounterApp) InitChain(req types.RequestInitChain) types.ResponseInitChain{
  app.Value = 100
  return types.ResponseInitChain{}
}
```
在上面的代码中我们直接在InitChain()实现中设定了技术初始值100，这当然是可以的，不过 更好的办法是借助于创世文件genesis.json，在该文件中声明应用状态的初始值。

例如，下面展示了genesis.json中的内容：
```
{
  "genesis_time": "2018-10-30T00:42:47.699648591Z",
  "chain_id": "test-chain-wx5RA8",
  ...
  "app_hash": "",
  "app_state": {"counter":100}
}
```
app_state字段的内容将以原始字节码的形式在InitChain请求的AppStateBytes字段传入， 因此，我们改为如下的实现：
```
func (app *CounterApp) InitChain(req types.RequestInitChain) types.ResponseInitChain{
  var state map[string]int
  json.Unmarshal(req.AppStateBytes,&state)
  app.Value = state["counter"]
  return types.ResponseInitChain{}
}
```

## 应用状态查询：Query

RPC客户端可以利用节点旳abci_query调用查询ABCI应用维护的状态，该调用允许在请求 中设定查询条件，以过滤潜在的查询结果：

![abci-query](https://lh3.googleusercontent.com/-42MySWeNchA/XAiKQ-oU3WI/AAAAAAAAEHs/07MlHJ9VlM8_00OFFixEwi0nrAgemzo_wCHMYCw/I/abci-query.png)


对于我们的计数状态机而言，由于只有一个状态Value，因此直接返回它的当前值即可：
```
func (app *CounterApp) Query(req types.RequestQuery) types.ResponseQuery{
  val := fmt.Sprintf("%d",app.Value)
  return types.ResponseQuery{Value: []byte(val)}
}
```
Query()的参数是一个RequestQuery结构，用来指定查询条件，由于我们只有一个状态 值得返回，因此暂时先忽略它。

Query()的返回结果是一个ResponseQuery结构，其定义如下：

![rsp-query](https://lh3.googleusercontent.com/-i6Jt3CF9RYI/XAiKQ1TWBiI/AAAAAAAAEHg/l60WYvZogpsb53SY2J7gfDPPELU8EOIbgCHMYCw/I/rsp-query.png)


同样，结构中只有Code是必须的，为0值时表示交易执行成功，不同的非0值的失败含义由 应用自行定义即可。对于计数应用，我们使用Value字段来返回当前状态值。

现在可以使用abci_query来查询应用状态了：

`~$ curl http://localhost:26657/abci_query`
abci_query返回的value是base64编码的：

![rsp-query-base64](https://lh3.googleusercontent.com/-Coi53j0PGeg/XAiKP9CoWQI/AAAAAAAAEG8/xZGmjD4j_IkZP5IttfLO3_eqiMqO9817wCHMYCw/I/rsp-query-base64.png)


我们可以用base64解码value字段的值：

`~$ echo MQ== |  base64 -d`

## 应用状态的历史
现在观察RequestQuery的结构，注意其Height字段：

![req-query](https://lh3.googleusercontent.com/-B1cLCas3Egw/XAiKQbzFnrI/AAAAAAAAEHE/-EbkrsVWayAdEhI0NbfWs8TAxh4PrNVzgCHMYCw/I/req-query.png)


对于状态机而言，当一个新的区块产生，其中的交易就会导致状态的迁移，因此状态是 与区块存在着对应关系 —— 在不同的区块高度，对应着状态机的不同状态：

![state-and-block](https://lh3.googleusercontent.com/-PMaNN5xMr5c/XAiKQZxy1gI/AAAAAAAAEHA/R6v9EUgQVSQNTNAcmAXCJ59MTUzTY8ACACHMYCw/I/state-and-block.png)


因此对于ABCI应用而言，应当记录状态的历史变化，每一个区块高度，对应着不同版本的状态：
```
type CounterApp struct {
  types.BaseApplication
  Value int64
  Version int64
  History map[int64]int64
}
```
我们在Commit()方法中递增Version（以便和区块高度保持一致），并记录状态历史：
```
func (app *CounterApp) Commit() types.ResponseCommit{
  app.Version += 1
  app.History[app.Version] = app.Value
  return types.ResponseCommit{}
}
```
现在，我们可以根据RequestQuery中的Height值来返回对应版本的状态了，高度0意味着要 返回最新区块的状态：
```
func (app *CounterApp) Query(req types.RequestQuery) types.ResponseQuery {  
  height := req.Height
  if req.Height == 0 {  height = app.Version }
  val := fmt.Sprintf("%d",app.History[height])
  return types.ResponseQuery{Value: []byte(val),Height: height}
}
```
我们现在可以查询历史状态了：

`$ curl http://localhost:26657/abci_query?height=1`
结果如下：

![rsp-query-version](https://lh3.googleusercontent.com/-iaqEjfZ3gwU/XAiKQwD8ksI/AAAAAAAAEHk/4UdFTnR0u5EnV9NJPCOo19a0y3igU5AXQCHMYCw/I/rsp-query-version.png)

## 应用/区块链握手机制

现在我们的计数应用已经在区块链中有了一些交易记录，也实现了基本的状态更新逻辑， 那么思考一个问题：如果重新启动节点（及ABCI应用），然后再次查询最后区块的状态值， 结果是8还是0？

![block-replay](https://lh3.googleusercontent.com/-DYs7pd3LHbg/XAiKQ9lLU5I/AAAAAAAAEHc/c0fhZwbSepkIylfq9sCYoyQeDyP2QnUygCHMYCw/I/block-replay.png)


这个问题是有意义的，毕竟我们只是在内存里记录了状态值以及其历史，重新启动后， 内存中的状态数据已经丢失了。

你可以自己尝试一下，不出意外的话，还是会和重新启动之前一样，你得到的状态值 依然是8。

这是因为当tendermint节点连接ABCI应用后，会有一个握手同步的处理，tendermint会 向ABCI应用发送Info消息获取应用状态的最后区块高度，如果ABCI应用返回的区块高度 小于tendermint节点旳最后区块高度，tendermint就会认为ABCI应用漏掉了这些区块并 进行重放：

![abci-handshake](https://lh3.googleusercontent.com/-w86qK175SHk/XAiKQojIWsI/AAAAAAAAEHQ/dr8Sp4Cl52ELAii04ICZmYFJC7LSKbkygCHMYCw/I/abci-handshake.png)


显然，由于我们没有明确处理Info消息，因此握手时tendermint会认为我们的计数应用 的区块高度为0，所以它会重放所有的区块，这意味着当重放结束，我们的计数器值还是 会回到8。

容易理解，我们不期望每次重新启动节点（及ABCI应用）都重放所有区块，那会非常耗时 并且体验很差。因此我们可以在Info()方法中返回状态机处理过的最后区块高度，你知道 它对应于我们的Version成员：
```
func (app *CounterApp) Info(req types.RequestInfo) types.ResponseInfo{
  return types.ResponseInfo{LastBlockHeight:app.Version}
}
```
Info()的返回值是一个ResponseInfo结构，其成员如下：

![rsp-info](https://lh3.googleusercontent.com/-ibUWynGaIbg/XAiKQm0WlNI/AAAAAAAAEHM/SJmTwgZLEDkoY1E4j20t6lE5QF-q8mJaQCHMYCw/I/rsp-info.png)

## 应用状态的哈希值
根据tendermint的约定，ABCI应用在Commit()调用时返回当前状态的哈希，会由tendermint 写入下一个区块头中的app_hash部分，作为区块链状态的一部分：

![app-hash](https://lh3.googleusercontent.com/-5_t-Zmml6MI/XAiKQsWX1wI/AAAAAAAAEHU/LpjL2tceE9werlFhK0Lgv8KyMU5ekkC2gCHMYCw/I/app-hash.png)


apphash用来表示某个区块时的应用状态特征，哈希函数自然是一个很好的选择，但tendermint 只要求这个值能够表达状态，因此我们理论上可以使用任何与状态相关的确定性的值。例如， 对于计数应用而言，我们可以直接使用计数器的值，因为这个状态本身很简单，不需要提取指纹了。
```
func (app *CounterApp) Commit() types.ResponseCommit{
  app.Version += 1
  app.Hash = []byte(fmt.Sprintf("%d",app.Value))
  app.History[app.Version] = app.Value  
  return types.ResponseCommit{Data:app.Hash}
}
```
与此同时，在tendermint与abci应用握手时，如果涉及到区块重放，也会检查区块头记录的AppHash 与前一区块的Commit()返回的结果是否一致，因此，我们调整Info()来返回记录的当前状态的哈希：
```
func (app *CounterApp) Info(req types.RequestInfo) types.ResponseInfo{
  return types.ResponseInfo{LastBlockHeight:app.Version,LastBlockAppHash:app.Hash }
}
```

### 应用状态持久化
到目前位置，我们的应用状态都是保存在内存里，每次重新启动都会从零开始。 现在考虑把状态持久化到硬盘上。这包括两部分工作：在Commit()中保存状态到硬盘、 在创建应用实例时载入硬盘保存的状态。

理论上你可以使用任何存储机制，例如SQL数据库、NoSQL数据库、文件系统等等。 不过为了便于观察，我们采用JSON格式的平文件记录计数值：
```
func (app *CounterApp) Commit() types.ResponseCommit{
  app.Version += 1
  app.History[app.Version] = app.Value
  state,err := json.Marshal(app) 
  if err !=nil { panic(err) }
  ioutil.WriteFile("./counter.state",state,0644)
  return types.ResponseCommit{}
}
```
同时，我们在实例化计数应用时，载入之前的状态：
```
func NewCounterApp() *CounterApp {
  app := CounterApp{ History:map[int64]int{} }
  state, err := ioutil.ReadFile("./counter.state")
  if err != nil { return &app }
  err = json.Unmarshal(state,&app)
  if err != nil { return &app }
  return &app
}
```
现在我们的状态已经持久化，可以查看一下counter.state的内容：

![state-persist](https://lh3.googleusercontent.com/-vuFW5-Wup34/XAiKQxixafI/AAAAAAAAEHY/4nywQ_HUvAo-nD8jEznbHoG2V0QyP7qJwCHMYCw/I/state-persist.png)

## 获得配套代码资料
关注微信公众号`区块链001`, 回复`tendermint`获得

