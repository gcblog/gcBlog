---
title: tendermint加密算法
categories:
  - tendermint
tags:
  - tendermint
  - secp256k1
  - ed25519
date: 2018-12-18 16:59:32
---


## 身份识别机制概述

无论是中心化系统，还是去中心化系统，都有一个基本的问题：如何表征与验证用户的身份。

在中心化系统中，这一问题是基于统一存储的用户表来实现的：每个用户在表中都有 一条对应的记录，而系统则通过验证用户输入的用户名和口令是否与用户表中的记录一致来识别 用户的身份：

![id-plan-compare](https://lh3.googleusercontent.com/-rw73ACdT8sI/XAiJ0GfZ27I/AAAAAAAAEGQ/QFDJDZxLhCgFLYmTBzLjBAf825oBFzAAACHMYCw/I/id-plan-compare-1.png)


区块链则采用了另外一种不需要集中存储的方案来解决这一问题：每个用户由一对公/私钥来 标识，可以将公钥视为用户名，而私钥视为用户的口令。当用户提交数据时，必须使用自己 的私钥进行签名，这样其他人就可以利用其公钥验证签名数据是否真的来自于该用户。

不过由于公钥比较长，通常会对公钥进行一定的哈希计算，并进行必要的截短，作为区块链上 用户的标识，即我们通常所说的地址。

tendermint提供了两种非对称加密算法的实现：比特币/以太坊采用的secp256k1椭圆曲线算法， 以及tendermint推荐的相对较新的ed25519加密算法，在我们的应用中都可以用来实现身份识别。
<!--more-->
## 经典算法secp256k1

Secp256k1是指比特币中使用的ECDSA(椭圆曲线数字签名算法)曲线的参数，公/私钥就对应于 该曲线上的点。

在比特币流行之前secp256k1几乎无人使用，但现在已经是无人不知了。secp256k1的参数由于 是精心选择的，因此它的计算会比随机参数的曲线快30%，具有较短的密钥，同时也能显著降低 算法中存在后门的风险。

tendermint内置了secp256k1的实现包crypto/secp256k1，其中的PrivKeySecp256k1和 PubKeySecp256k1分别实现了私钥和公钥接口：

![key-address](https://lh3.googleusercontent.com/-1VROJMF7QdQ/XAiJ0KcyefI/AAAAAAAAEGc/NPEtkVID1QEGILwfbcXPuGzUSPZ_KUN5gCHMYCw/I/key-address.png)

私钥对应于曲线上点的X坐标。使用secp256k1包的GenPrivKey()方法可以生成一个32 字节长的随机私钥，例如：
```
priv := secp256k1.GenPrivKey()
fmt.Printf("private key => %v\n",priv)
```
或者将一个特定的密文转换为私钥，例如：
```
priv := secp256k1.GenPrivKeySecp256k1([]byte("your secret byte slice"))
fmt.Printf("private key => %v\n",priv)
```

公钥对应与曲线上点的Y坐标，因此从私钥可以推导出公钥，调用私钥的GetPubKey() 方法获得其对应的公钥实例：
```
pub := priv.GetPubKey()
fmt.Printf("public key => %v\n",pub)
```
tendermint的实现是返回压缩公钥，因此公钥长度是32+1=33字节 —— 额外的1个字节标识Y 在上部还是下部。 地址的计算方法和比特币一样，都是对公钥进行两重哈希运算（sha256->ripemd160）， 最后得到20字节长的地址，但tendermint的地址没有像比特币那样添加网络前缀。

调用公钥实例的Address()方法获取公钥对应的地址，例如：
```
addr := pub.Address()
fmt.Printf("address => %v\n",addr)
```
尽管secp256k1在区块链领域已经非常流行，但是它也有一些问题。

首先是计算效率的问题。Montgomery ladder是一种可以快速简捷地进行椭圆曲线计算的 方法，但是secp256k1不支持，这使得它的计算效率无法通过采用该算法得到提升。

其次是secp256k1不具备数理完备性。在某些边界情况下，secp256k1有可能产生错误的结果， 因此这要求在算法实现时非常小心。

最后是secp256k1对扭曲攻击的抵抗力不够强。扭曲攻击是指攻击者从类似的曲线上抽点 来欺骗算法。secp256k1必须在签名和验证过程中检查攻击者提供的点是否真的在曲线上， 这使得安全性和效率打了折扣。

## 下一代算法ed25519

tendermint推荐的ed25519算法属于下一代的EdDSA签名算法，与secp256k1相比，ed25519 的计算能快30%，安全性更高，密钥和签名也更短一些。下表列出了两者的对比：

类型	Secp256k1	Ed25519
私钥长度	32 字节	64 字节
公钥长度	33 字节	32 字节
签名长度	~71 字节	64 字节
安全目标	2128	2128
安全测试通过率	7/11	11/11

tendermint的ed25519实现包是crypto/ed25519，其中的PrivKeyEd25519和 PubKeyEd25519分别实现了私钥和公钥接口：

![ed25519](https://lh3.googleusercontent.com/-HPN4U1hdDS8/XAiJ0A61uPI/AAAAAAAAEGU/pgqGTicZKvIv2N6Dqz4gKJPWxMtsRm9VwCHMYCw/I/ed25519.png)


使用ed25519包的GenPrivKey()方法可以生成一个64字节长的随机私钥，例如：
```
priv := ed25519.GenPrivKey()
fmt.Printf("private key => %v\n",priv)
```
或者将一个特定的密文转换为私钥，例如：
```
priv := ed25519.GenPrivKeyFromSecret([]byte("your secret byte slice"))
fmt.Printf("private key => %v\n",priv)
```
调用私钥的GetPubKey()方法获得其对应的公钥实例：
```
pub := priv.GetPubKey()
fmt.Printf("public key => %v\n",pub)
```
地址的计算方法，就是对公钥进行sha256哈希计算，然后截取前20个字节。 调用公钥实例的Address()方法获取公钥对应的地址，例如：
```
addr := pub.Address()
fmt.Printf("address => %v\n",addr)
```

## 数据签名与认证

非对称密钥有一个很有用的特性，就是私钥签名，可以用公钥进行认证：

![key-verify](https://lh3.googleusercontent.com/-s0CFT2YZEEc/XAiJ0BCdgaI/AAAAAAAAEGY/-z0-4GYH2JITtRD6GmB279nTYqnStLblACHMYCw/I/key-verify.png)

发送方首先使用私钥签名要发送的数据，例如msg的内容：
```
privTommy := secp256k1.GenPrivKey()
msg := []byte("some text to send")
sig := privTommy.Sign(msg)
```
由于secp256k1和ed25519都实现了PrivKey接口，因此你可以任选其一生成 你的私钥。

Sign()方法返回的是一个字节切片，通常和被签名的数据一起发送出去， 当然，接收方不一定持有发送方的公钥，因此通常也会把公钥一并发过去。

例如，我们可以使用如下的结构来声明要发送的签名数据：
```
type Letter struct {
  Msg []byte
  Signature []byte
  PubKey secp256k1.PubKeySecp256k1
}
```
将签名数据序列化为字节码流，然后通过网络发送出去，或者拷贝给接收方：
```
letter := Letter{msg,sig,pubSender.(kf.PubKeySecp256k1)}
bz,err := json.Marshal(letter)
if err !=nil { panic(err) }
fmt.Printf("encoded letter => %x\n",bz)
```
在上面的代码中，我们使用了json编码器，当然你可以使用任何其他可用的编解码器， 例如gob。

接收方首先解码接收到的字节码流：
```
var received Letter
err = json.Unmarshal(bz,&received)
if err !=nil { panic( err)}
fmt.Printf("decoded letter => %+v\n",received)
```
然后就可以使用信件中发送方的公钥验证签名了：
```
valid := received.PubKey.VerifyBytes(received.Msg,received.Signature)
fmt.Printf("validated => %t\n",valid)
```

## 获得配套代码资料
关注微信公众号`区块链001`, 回复`tendermint`获得