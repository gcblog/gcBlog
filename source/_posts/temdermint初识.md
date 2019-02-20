---
title: temdermint初识
categories:
  - temdermint
tags:
  - temdermint
date: 2018-12-16 22:29:43
---

## 2.1 概述
tendermint虽然定位于引擎，但它其实是一个完整的区块链实现。在这一部分 的课程中，我们将使用一个最小化的ABCI应用，来熟悉tendermint的主要组成 部分，以及使用tendermint进行去中心化应用开发的主要流程和工具。

下图列出了tendermint应用的主要构成部分：

![td-arch](https://lh3.googleusercontent.com/-9h1wL3AOaT0/XAiI0CiYoSI/AAAAAAAAEFY/GhAg0H7tV28RgTVW-kATnVCc3CJ0zFiWwCHMYCw/I/td-arch.png)


tendermint提供了一个预构建的同名可执行程序，我们将学习如何使用这个程序 来初始化节点配置文件并启动节点。这个程序是完整的节点实现，除了通过P2P协议 与其他节点交换共识，同时还提供了RPC接口供客户端提交交易或者查询应用状态。

我们将创建一个最小化的ABCI应用，tendermint可执行程序通过ABCI接口与 应用程序交互，例如要求应用执行交易、或者转发来自RPC接口的状态查询请求。
<!--more-->
## 2.2 节点初始化

tendermint节点程序的行为非常依赖于配置文件，使用其init子命令 可以获得一组默认的初始化文件。 例如，在1#终端输入如下命令创建初始化文件：

`~$ tendermint init`
init子命令将在~/.tendermint目录下创建两个子目录data和config，分别用于 保存区块链数据和配置文件。

在data目录下将包含如下的数据文件，均为leveldb格式：

blockstore.db：区块链数据库
evidence.db：节点行为数据
state.db：区块链状态数据
tx_index.db：交易索引数据，
在config子目录下将包含如下的配置文件：

config.toml：节点软件配置文件
node_key.json：节点密钥文件，用于p2p通信加密
priv_validator.json：验证节点密钥文件，用于共识签名
genesis.json：创世文件
节点配置文件config.toml用来设置节点软件的运行参数，例如RPC监听端口等。 我们修改consensus.create_empty_blocks为false，即不出无交易的空块：

[consensus]
`create_empty_blocks = false`
重新初始化

在我们开发ABCI应用的过程中，往往需要对应用中的状态结构等信息进行调整，再次重新启动 后就可能导致原有的链数据和新的状态结构不兼容，因此需要时不时地重新初始化区块链数据。

当然你可以完全删除~/.tendermint目录，然后重新执行tendermint init命令。不过官方 的建议是使用unsafe_reset_all子命令来做这个事情，这个命令可以保留现有的配置而仅删除 数据文件。例如：

`~$ tendermint unsafe_reset_all`

## 2.3 节点启动与停止
初始化之后，我们就可以启动节点了。在1#终端执行node子命令启动tendermint节点：

`~$ tendermint node`
可以看到tendermint在反复尝试abci应用的默认监听地址tcp://127.0.0.1:26658：

![try-abci](https://lh3.googleusercontent.com/-86l_z4_4cOI/XAiI0AAqovI/AAAAAAAAEFc/CSAySg1umpQuUUlxharp6lXx3fqsjxm6wCHMYCw/I/try-abci.png)


显然，tendermint要求一个配套的abci应用才能正常工作，我们将在下一节解决这个 问题。

在目前这种状态下，如果需要退出tendermint的执行，可以切换到2#终端，使用pkill 命令终止其运行：

`~$ pkill -9 tendermint`

## 2.4 编写最小化应用
tendermint开发包中已经包含了一个基本的ABCI应用实现类BaseApplication， 可以完成与tendermint节点旳基本交互：

![abci-serve](https://lh3.googleusercontent.com/-K-O5r4n20WY/XAiI0EFf2KI/AAAAAAAAEFU/VLxjrV5m3bwWOdn6p_BJRqmAhC0Q9WbWACHMYCw/I/abci-server.png)


tendermint节点程序可以通过socket通信访问ABCI应用，因此我们使用abci/server 包的NewServer()函数创建一个SocketServer实例来启动这个应用。

例如，下面的代码在tendermint尝试连接的默认端口26658启动abci应用：
```
package main

import (
  "fmt"
  "github.com/tendermint/tendermint/abci/types"
  "github.com/tendermint/tendermint/abci/server"
)

func main(){
  app := types.NewBaseApplication()
  svr,err := server.NewServer(":26658","socket",app)
  if err != nil { panic(err) }
  svr.Start()
  defer svr.Stop()
  fmt.Println("abci server started")
  select {}
}
```
将上述代码保存为~/repo/go/src/diy/c2/mini-app.go，然后在2#终端 进入c2目录并启动该应用：
```
~$ cd ~/repo/go/src/diy/c2
~/repo/go/src/diy/c2$ go run mini-app.go
```
现在回到1#终端重新启动tendermint节点：

`~$ tendermint node`
你可以看到这次tendermint节点启动成功了：

![node-started](https://lh3.googleusercontent.com/-y7rXlpkNGqw/XAiI0na91uI/AAAAAAAAEFw/TdIknhDNvl4mqDdgwlmiOn4cuONHG9XaACHMYCw/I/node-started.png)


由于我们只有一个节点，因此tendermint会抱怨连接不到其他的节点，it‘s ok。

## 2.5 RPC开发接口

在一个典型的（非理想化的）去中心化应用的开发中，除了需要开发链上应用 （例如ABCI应用或者以太坊中的智能合约），往往还需要开发传统的网页应用 /桌面应用/手机应用，以方便那些不可能自己部署节点旳用户：

![light-user-case](https://lh3.googleusercontent.com/-ecRo-qEvcKo/XAiI0c0ZmxI/AAAAAAAAEFg/toYaNUzLpYIaCYH4dA_Gyn4zPo2BZeBDwCHMYCw/I/light-user-case.png)


和以太坊一样，tendermint的节点也提供了RPC接口供这些传统应用代码访问节点功能， 例如提交交易或者查询节点状态，其默认的RPC监听端口是26657。

首先确保1#终端和2#终端分别运行着tendermint和abci应用，然后我们切换到3# 终端，输入如下命令提交交易0x68656c6c6f —— 对应于字符串hello的16进制表示：

`~$ curl http://localhost:26657/broadcast_tx_commit?tx=0x68656c6c6f`
响应结果类似于下图，其中check_tx和deliver_tx来自于abci应用，而交易哈希 和区块高度则由tendermint节点内部处理得出：

![rpc-ret](https://lh3.googleusercontent.com/-hUNne3_TT-s/XAiI0XLRMsI/AAAAAAAAEFs/RTT_P7hvznoEfKZeKFmzWe2CCnbdOhV0wCHMYCw/I/rpc-ret.png)


事实上，由于BaseApplication对于交易数据没有任何的限制，因此我们可以提交 任意有效的16进制表示，而这些交易都将成功地打包到区块里。

让我们看一下这个区块的内容，在3#终端输入如下命令：

`~$ curl http://localhost:26657/block?height=2`
注意结果中的Txs字段，它包含了该区块中所有交易的base64编码：

![rpc-block](https://lh3.googleusercontent.com/-nIJ0D0tG1mQ/XAiI0pPnFbI/AAAAAAAAEFo/W0JagkLkNdwziOQYThLJ8lQAGsZBplgKQCHMYCw/I/rpc-block.png)


我们可以使用命令行工具base64简单地进行验证：

`~$ echo aGVsbG8= | based64 -d`
可以访问这里 查看tendermint区块结构的详细说明。

也可以通过哈希查看交易内容，在3#终端输入如下命令（注意，你的哈希可能与此不同）：

`~$ curl http://localhost:26657/tx?hash=0x2CF24DBA5FB0A30E26E83B2AC5B9E29E1B161E5C`
得到如下的结果：

![rpc-tx](https://lh3.googleusercontent.com/-itMMg2kAhY0/XAiI0Qhia-I/AAAAAAAAEFk/Qp5Xk5adNDYPCgFL9_zjmzrtgWF4jFY4wCHMYCw/I/rpc-tx.png)

## 获得配套代码资料
关注微信公众号`区块链001`, 回复`tendermint`获得