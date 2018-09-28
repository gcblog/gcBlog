---
title: 共识算法DPOS原理及实现
categories:
  - 共识算法
tags:
  - DPOS
  - 共识算法
date: 2018-06-29 19:31:58
---

## 原理简介

DPOS：Delegated Proof of Stake，委任权益证明
它的原理是让每一个持有币的人进行投票，由此产生n位代表 , 我们可以将其理解为n个超级节点或者矿池，而这n个超级节点彼此的权利是完全相等的。从某种角度来看，DPOS有点像是议会制度或人民代表大会制度。如果代表不能履行他们的职责（当轮到他们时，没能生成区块），他们会被除名，网络会选出新的超级节点来取代他们。EOS就是采用DPOS共识算法。

## 算法具体实现原理

假设n为21，竞选的节点有几百个，持币人对这些节点进行投票，选出票数最多的21位，由这21位轮流来出块。

<!--more-->

## 代码简单实现其原理

```
package main

import (
	"fmt"
	"math/rand"
	"time"
	"strconv"
	"crypto/sha256"
	"encoding/hex"
)

type Block struct {
	Index int
	Timestamp string
	Prehash string
	Hash string
	Data []byte

	delegate *Node// 代理 区块由哪个节点挖出
}


func GenesisBlock()  Block {
	gene := Block{0, time.Now().String(),"", "", []byte("genesis block"), nil}

	gene.Hash = string(blockHash(gene))

	return Block{}
}

func blockHash(block Block) []byte  {

	record := strconv.Itoa(block.Index) + block.Timestamp + block.Prehash + hex.EncodeToString(block.Data)

	h := sha256.New()
	h.Write([]byte(record))
	hashed := h.Sum(nil)
	return hashed
}


//节点类型
type Node struct {
	Name  string //节点名称
	Votes int    // 被选举的票数
}

func (node *Node)GenerateNewBlock(lastBlock Block, data []byte) Block  {

	var newBlock = Block{lastBlock.Index+1, time.Now().String(), lastBlock.Hash, "", data, nil}

	newBlock.Hash = hex.EncodeToString(blockHash(newBlock))
	newBlock.delegate = node
	return newBlock

}

//创建节点
var NodeArr = make([]Node,10)
func CreateNode() {

	for i := 0; i < 10; i++ {
		name := fmt.Sprintf("NODE %d num", i+1)
		NodeArr[i] = Node{name, 0}
	}

}

//简单模拟投票
func Vote()  {
	for i := 0; i < 10; i++ {
		rand.Seed(time.Now().UnixNano())
		vote := rand.Intn(10) + 1
		NodeArr[i].Votes = vote
	}
}


//选出票数最多的前3位
func SortNodes() []Node  {
	n:= NodeArr
	for i := 0; i<len(n) ;i++  {
		for j := 0; j < len(n)-1 ;j++  {
			if n[j].Votes < n[j+1].Votes {
				n[j],n[j+1] = n[j+1],n[j]
			}
		}
	}

	return n[:3]
}


func main() {

	CreateNode()
	fmt.Println(NodeArr)
	Vote()
	nodes := SortNodes()

	fmt.Println(nodes)


	//创建创世区块
	gene := GenesisBlock()

	lastBlock := gene
	for i:= 0; i< len(nodes) ;i++  {
		lastBlock =  nodes[i].GenerateNewBlock(lastBlock,[]byte(fmt.Sprintf("new block %d",i)))

	}


}

```

输出
```
竞选的节点 [{第 1 个节点 0} {第 2 个节点 0} {第 3 个节点 0} {第 4 个节点 0} {第 5 个节点 0} {第 6 个节点 0} {第 7 个节点 0} {第 8 个节点 0} {第 9 个节点 0} {第 10 个节点 0}]
选出的节点 [{第 10 个节点 8} {第 4 个节点 7} {第 3 个节点 6}]
第 10 个节点 出块 {1 2018-06-29 19:28:28.41834078 +0800 CST m=+0.000940357  0c142d83bf773e248c3438dd99423f6b289d171696b5e24573e06e2c4c445161 [110 101 119 32 98 108 111 99 107 32 48] 0xc420090000}
第 4 个节点 出块 {2 2018-06-29 19:28:28.418365811 +0800 CST m=+0.000965387 0c142d83bf773e248c3438dd99423f6b289d171696b5e24573e06e2c4c445161 439cbbac2d6a8b476b7dbffd788c0f8019b55d65c6c075eb32ce4060e3a2cd63 [110 101 119 32 98 108 111 99 107 32 49] 0xc420090018}
第 3 个节点 出块 {3 2018-06-29 19:28:28.418395274 +0800 CST m=+0.000994849 439cbbac2d6a8b476b7dbffd788c0f8019b55d65c6c075eb32ce4060e3a2cd63 86d8790c046523635a02f1316a4b85c27c7df3a762b8d7bc550bf5317adf8455 [110 101 119 32 98 108 111 99 107 32 50] 0xc420090030}

```





