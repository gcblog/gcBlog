---
title: 共识算法POS原理及实现
categories:
  - 共识算法
tags:
  - POS
  - 共识算法
date: 2018-06-29 19:31:41
---

## POS简介

POS：Proof of Stake，股权证明
类似于财产储存在银行，这种模式会根据你持有数字货币的量和时间，分配给你相应的利息。 
简单来说，就是一个根据你持有货币的量和时间，给你发利息的一个制度，在股权证明POS模式下，有一个名词叫币龄，每个币每天产生1币龄，比如你持有100个币，总共持有了30天，那么，此时你的币龄就为3000，这个时候，如果你发现了一个POS区块，你的币龄就会被清空为0。你每被清空365币龄，你将会从区块中获得0.05个币的利息(假定利息可理解为年利率5%)，那么在这个案例中，利息 = 3000 * 5% / 365 = 0.41个币，这下就很有意思了，持币有利息。以太坊就是采用POS共识算法。

## 算法具体实现原理

每个旷工都有出块(即挖矿)的权力，只要出块成功，就有系统给出的奖励，这里不需要通过复杂的计算来挖矿，问题只在于谁来出块，股权越大，出块的概率就越大，反之，则相反。POS有很多变种，股权可以是持有币的数量，或者支付的数量等等。

<!--more-->

## 代码简单实现其原理

```
package main

import (
	"time"
	"strconv"
	"crypto/sha256"
	"math/rand"
	"fmt"
	"encoding/hex"
)

//实现pos挖矿的原理

type Block struct {
	Index int
	Data string //
	PreHash string
	Hash string
	Timestamp string
	//记录挖矿节点
	Validator *Node

}

func genesisBlock() Block  {

	var genesBlock  = Block{0, "Genesis block","","",time.Now().String(),&Node{0, 0, "dd"}}
	genesBlock.Hash = hex.EncodeToString(BlockHash(&genesBlock))
	return genesBlock
}

func BlockHash(block *Block) []byte  {
	record := strconv.Itoa(block.Index) + block.Data + block.PreHash + block.Timestamp + block.Validator.Address
	h := sha256.New()
	h.Write([]byte(record))
	hashed := h.Sum(nil)

	return hashed
}

//创建全节点类型
type Node struct {
	Tokens int //持币数量
	Days int //持币时间
	Address string //地址
}


//创建5个节点
//算法的实现要满足 持币越多的节点越容易出块
var nodes = make([]Node, 5)
//存放节点的地址
var addr = make([]*Node, 15)


func InitNodes()  {

	nodes[0] = Node{1, 1, "0x12341"}
	nodes[1] = Node{2, 1, "0x12342"}
	nodes[2] = Node{3, 1, "0x12343"}
	nodes[3] = Node{4, 1, "0x12344"}
	nodes[4] = Node{5, 1, "0x12345"}

	cnt :=0
	for i:=0;i<5;i++ {
		for j:=0;j<nodes[i].Tokens * nodes[i].Days;j++{
			addr[cnt] = &nodes[i]
			cnt++
		}
	}

}

//采用Pos共识算法进行挖矿
func CreateNewBlock(lastBlock *Block, data string) Block{

	var newBlock Block
	newBlock.Index = lastBlock.Index + 1
	newBlock.Timestamp = time.Now().String()
	newBlock.PreHash = lastBlock.Hash
	newBlock.Data = data


	//通过pos计算由那个村民挖矿
	//设置随机种子
	rand.Seed(time.Now().Unix())
	//[0,15)产生0-15的随机值
	var rd =rand.Intn(15)

	//选出挖矿的旷工
	node := addr[rd]
	//设置当前区块挖矿地址为旷工
	newBlock.Validator = node
	//简单模拟 挖矿所得奖励
	node.Tokens += 1
	newBlock.Hash = hex.EncodeToString(BlockHash(&newBlock))
	return newBlock
}

func main()  {

	InitNodes()

	//创建创世区块
	var genesisBlock = genesisBlock()

	//创建新区快
	var newBlock = CreateNewBlock(&genesisBlock, "new block")

	//打印新区快信息
	fmt.Println(newBlock)
	fmt.Println(newBlock.Validator.Address)
	fmt.Println(newBlock.Validator.Tokens)

}

```

输出
```
{1 new block 72e8838ad3bb761c7d3ba42c4e6bad86409dd3f4ce958c890409c4b9ddf44171 e4f9575cfb14ee146810862c9e5cc78ebff185f5888f428dbb945bd9060b31f7 2018-06-29 19:29:04.827332898 +0800 CST m=+0.000837770 0xc42007e0a0}
0x12341
2
```




