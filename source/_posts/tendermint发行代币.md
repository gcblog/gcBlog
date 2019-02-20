---
title: tendermint发行代币
categories:
  - tendermint
tags:
  - tendermint
date: 2019-01-06 22:31:38
---

# 发行自己的代币

## 代币发行概述
状态机模型是一种图灵完备的计算模型，理论上你可以用它来实现任何应用， 代币也不例外。例如，我们可以借鉴以太坊的账户模型，设计出自己的账户 状态机：

![sm-account](https://lh3.googleusercontent.com/-1FgpvG7AFCY/XAiKngG59mI/AAAAAAAAEIY/q_xfvLxbQVQyTOj1dK1xYQ2uBul6tTFkACHMYCw/I/sm-account.png)


出于简化问题考虑，我们假设系统只发行一种代币，因此在账户模型中只需要 记录每个账户的余额即可。所有账户及其余额是整个系统的状态，只有当交易 发生时，这一状态才会发生变化。

例如，假设账户tommy有1000个代币，那么当发生一笔从mary到tommy 的500个币的转账交易后，tommy的余额将增加500个代币，同时mary的余额将 减少500个代币，这意味着整个系统在这笔交易后进入了一个新的状态。

基于我们之前的学习，很容易将账户采用哈希地址来表示，同时通过非对称 加密技术进行身份验证，从而实现一个去中心化的代币账户状态机。
<!--more-->
## 状态机实现

账户状态机的主要成员，包括记录系统状态的成员变量Accounts，以及表征交易的 成员函数issue()和transfer()：

![token-s](https://lh3.googleusercontent.com/-wPRofw9uocU/XAiKm1b7zYI/AAAAAAAAEIM/_7p0xWRoWwAwWZquVHXZ9nckYkaOYfscQCHMYCw/I/token-sm.png)


TokenApp的结构声明如下：
```
type  TokenApp struct {
  types.BaseApplication
  Accounts map[string]int
}
```
我们使用一个映射来表示系统的整个状态，其中键为哈希地址，值为账户余额。 由于crypto.Address类型其实是一个字节切片，因此我们采用其16进制表示 作为账户映射表的键。

发行交易将向指定的地址发行一定数量的代币，显然，只允许系统设定的发行人 SYSTEM_ISSUER执行该交易：
```
func (app *TokenApp) issue(issuer,to crypto.Address,value int) error {
  if !bytes.Equal(issuer,SYSTEM_ISSUER) return { errors.New("invalid issuer") }
  app.accounts[to] += value
  return nil
}
```
转账交易从转出账户减去一定数量的代币，再向转入账户增加一定数量的代币， 因此我们需要先保证转出账户有足量的余额：
```
func (app *TokenApp) transfer(from,to crypto.Address,value int) error {
  if app.accounts[from] < value {return errors.New("no enough balance")}
  app.accounts[from] -= value
  app.accounts[to] += value
  return nil
}
```

## 简单钱包实现
为了避免琐碎的密钥/地址管理，我们使用一个简单的钱包结构来管理一组私钥：

![wallet](https://lh3.googleusercontent.com/-mvk6yRKXjTk/XAiKm9yf4ZI/AAAAAAAAEII/yGNwzSguf5wGUrd_2G319GxTA3AMXnPUQCHMYCw/I/wallet.png)


为了避免输入冗长难记的地址，我们使用字符串标识钱包中的不同私钥，因此得到 如下的结构定义：
```
type Wallet struct {
  Keys map[string]crypto.PrivKey
}
```
钱包应该可以随时创建新的私钥，因此我们实现GenPrivKey()方法：
```
func (wallet *Wallet) GenPrivKey(label string) crypto.PrivKey {
  priv := kf.GenPrivKey()
  wallet.Keys[label] = priv
  return priv
}
```
GenPrivKey()方法需要传入一个字符串作为私钥的标识，以便我们可以在 以后使用该标识获取该私钥，或者该私钥对应的公钥或地址：
```
func (wallet *Wallet) GetPrivKey(label string) crypto.PrivKey {
  return wallet.Keys[label]
}
```
当然，还需要提供一个方法将钱包保存到硬盘上：
```
func (wallet *Wallet) Save(wfn string){
  bz,err := codec.MarshalJSON(wallet)
  if err != nil { panic(err) }
  ioutil.WriteFile(wfn,bz,0644)  
}
```
或者载入硬盘的钱包文件获得一个钱包实例：
```
func LoadWallet(wfn string) *Wallet{
  var wallet Wallet
  bz,err := ioutil.ReadFile(wfn)
  if err != nil { panic(err) }
  err = codec.UnmarshalJSON(bz,&wallet)
  if err != nil { panic(err) }
  return &wallet  
}
```

## 交易结构设计与实现
由于我们使用非对称密钥进行身份标识，因此在交易中需要包含身份校验所需要的 信息，例如签名、公钥和消息序列号：

![tx-struct](https://lh3.googleusercontent.com/-ot6g87ztbro/XAiKnp-91oI/AAAAAAAAEIU/Xp6ldO3EiV07P12L56nF1pusSHh_d_SHQCHMYCw/I/tx-struct.png)


我们使用一个统一的Tx结构来表示所有的交易，其中交易载荷指向一个Payload接口的 实现，该接口的三个方法可用于接收方的签名验证与交易路由：

GetSigner()：获取交易发起方地址
GetSignBytes()：获取交易载荷中用于签名的数据
GetType()：获取交易类别，状态机根据该调用返回值执行相应的动作
在账户状态机应用中，我们需要两种类型的交易：发行交易、转账交易，分别用于 向指定账户发行代币，以及在指定账户之间转移代币。不同的交易对应不同的Payload 接口实现，例如对于转账交易，其对应的TransferPayload结构的接口实现如下：
```
func (p *TransferPayload) GetSigner() crypto.Address{   return p.From  }
func (p *TranferPayload) GetSignBytes() []byte {   return json.Marshal(p) }
func (p *TransferPayload)  GetType() string{   return "transfer" }
```
交易核验

在接收端，首先应当检查交易结构中公钥的有效性，这通过校验公钥与交易发起方地址 是否一致来实现，然后则通过重算交易签名来确认签名的有效性，只有有效的交易，我们 才进行后续处理。例如，下面的代码展示了交易的验证逻辑：
```
func (app *AccountApp) validateTx(tx *Tx) error {
  addr := tx.PubKey.Address()
  if !bytes.Equals(addr,tx.Payload.GetSigner()) { return errors.New("pubkey / signer mismatch") }
  valid := tx.PubKey.VerifyBytes(tx.Payload.GetSignBytes(),tx.Signature)
  if !valid  { errors.New("bad signature") }
  return nil
}
```
交易路由

一旦交易验证有效，状态就可以根据交易类别进行分别处理了。例如：
```
switch tx.Payload.GetType(){
  case "transfer":  
    pld := tx.Payload.(TransferPayload)
    app.transfer(pld.From,pld.To,pld.Value)
  case "issue":  
    pld := tx.Payload.(IssuePayload)
    app.issue(pld.Issuer,pld.To,pld.Value)
}
```

## 交易的编解码处理

根据我们定义的交易结构，显然在RPC客户端提交交易之前，需要首先串行化为 16进制码流，在ABCI应用中同时也需要相应的解码：

![tx-code](https://lh3.googleusercontent.com/-afcL-yNBO0c/XAiKnuKkykI/AAAAAAAAEIc/Ruug_ukvbuIF-lOzAhkZICvku0En8g98wCHMYCw/I/tx-codec.png)


tendermint官方推荐的编解码器是其自产的go-amino，它类似于protobuf，最大的特点是支持 解码到接口类型 —— 这就是我们可以在Tx结构中使用接口类型的原因。

amino通过在编码码流中加入标识序列来区分不同的接口实现结构，因此解码接口之前，首先 需要注册接口以及对应的实现结构及标识名，例如，在下面的代码中，我们注册Payload接口， 然后注册其两个实现结构TransferPayload和IssuePayload，并分别使用tx/transfer 和tx/issue来标识这两个Payload接口的实现：
```
codec := amino.NewCodec()
codec.RegisterInterface((*Payload)(nil),nil)
codec.RegisterConcrete(*TransferPayload{},"tx/transfer",nil)
codec.RegisterConcrete(*IssuePayload{},"tx/issue",nil)
```
需要指出的是，当你使用amino时，并不是所有的自定义类型都需要在codec中注册，只有那些 需要解码到接口类型的结构，才需要进行注册。

现在，接收端可以对接收到的二进制码流bz进行解码了：
```
func (app *AccountApp) decodeTx(bz []byte) (*Tx,error){
  var tx Tx
  err := codec.UnmarshalBinary(bz,&tx)
  return &tx,err
}
```

## ABCI协议实现

有了基本的状态机、钱包、交易结构以及序列化手段，现在我们可以实现状态机的ABCI接口了：

![token-abci](https://lh3.googleusercontent.com/-Q2P78SG_soU/XAiKm3FLwdI/AAAAAAAAEIQ/IecHrp48bQECoWV5XuhmjqaySYMQWUFQACHMYCw/I/token-abci.png)


交易检查：CheckTx

在CheckTx()方法实现中检查交易的有效性，只有解码正确并且检查有效的交易才允许 进入交易内存池：

```
func (app *TokenApp) CheckTx(bz []byte) types.ResponseCheckTx {
  var tx Tx
  err := codec.UnmarshalBinary(bz,&tx)
  if err !=nil { return types.ResponseCheckTx{Code:1,Info: err.Error()}}
  err = app.validateTx(tx)
  if err !=nil { return types.ResponseCheckTx{Code:2,Info:err.Error()}}
  return types.ResponseCheckTx{}
}
```
交易执行：DeliverTx

在DeliverTx()方法中判断交易类型，然后执行相应的状态迁移：

```
func (app *TokenApp) DeliverTx(bz []byte) (rsp types.ResponseDeliverTx){
  var tx Tx
  codec.UnmarshalBinary(bz,&tx)
  switch tx.Payload.GetType() {
    case "transfer": 
      pld := tx.Payload.(TransferPayload)
      app.transfer(pld.From,pld.To,pld.Value)     
    case "issue": 
      pld := tx.Payload.(IssuePayload)
      app.issue(pld.Issuer,pld.To,pld.Value) 
    default: rsp.Code = 1
  }
  return
}
```
状态查询

在Query()方法中返回指定账户的余额：

```
func (app *TokenApp) Query(req types.RequestQuery) types.ResponseQuery{
  addr := crypto.Address(req.Data)
  bal, _ := codec.MarshalBinary(app.Accounts[addr.String()])
  desc: = fmt.Sprintf("balance : %v => %v",addr,app.Accounts[addr.String])
  return types.ResponseQuery{Key:req.Data,Value:bal,Log:desc}
}
```

## RPC客户端开发
tendermint内置了RPC开发接口的API封装包rpc/client，极大地简化了客户端的 开发难度：

![rpc-client](https://lh3.googleusercontent.com/-8AunBudQiis/XAiKnq_aegI/AAAAAAAAEIg/0CBgl7HKYEwgX40PXWW17L8udbqEnnRQwCHMYCw/I/rpc-client.png)


使用rpc/client包的NewHTTP()方法，我们可以得到一个HTTP实例：

`cli := client.NewHTTP("http://localhost:26657","")`
HTTP结构实现了tendermint中所有的RPC客户端接口，例如ABCI客户端接口 （ABCIClient）、历史数据访问接口（HistoryClient）等，因此我们可以直接 利用其方法向abci应用提交交易。

首先构造一个发行交易，并利用发行人私钥签名交易：
```
paylod := NewIssuePayload(issuerAddr,receiverAddr,value)
tx := NewTx(payload)
tx.Sign(issuerPrivKey)
```
然后将交易实例序列化：

`rawtx,_ := codec.MarshalBinary(tx)`
最后使用HTTP实例的BroadcastTxCommit()方法提交给节点，并 打印输出响应结果：
```
ret,_ := cli.BroadcastTxCommit(rawtx)
fmt.Printf("ret => %+v\n",ret)
```

