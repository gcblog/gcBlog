---
title: 共识算法POW原理及实现
categories:
  - 共识算法
tags:
  - POW
  - 共识算法
date: 2018-06-29 19:31:22
---

## POW简介

Proof of Work，工作证明。
POW共识算法主要是通过计算难度值来决定谁来出块。POW的工作量是指方程式求解，谁先解出来，谁就有权利出块。方程式是通过前一个区块的哈希值和随机值nonce来计算下一个区块的哈希值，谁先找到nonce，谁就能最先计算出下一个区块的哈希值，这种方式之所以被称为计算难度值是因为方程式没有固定解法，只能不断的尝试，这种解方程式的方式称为哈希碰撞，是概率事件，碰撞的次数越多，方程式求解的难度就会越大。比特币就是采用POW共识算法

## 算法具体实现原理

这里涉及到两个重要的概念，一个是难度系数，一个是nonce，nonce可以理解为一个随机数，就是挖矿中要找到一个符合条件的nonce值。
这里假设难度系数是4(比特币初始难度系数就是4)，将一个区块中的数据加上nonce值打包，nonce值从0开始一直递增，将这打包的数据计算hash值，hash满足最前面有4个0，就是挖矿成功。难度系数为多少，hash最前面就需要满足多少个0。

<!--more-->

## 代码简单实现其原理

```
package main

import (
	"strconv"
	"time"
	"crypto/sha256"
	"strings"
	"encoding/hex"
	"fmt"
)

//pow 挖矿算法

//定义难度系数
const difiiculty = 4

type Block struct {
	Index      int // 区块高度
	TimeStamp  int64
	Data       string //交易记录
	Hash       string
	Prehash    string
	Nonce      int
	Difficulty int //难度系数
}

//创建区块链
var BlockChain []Block

//创世区块
func GenesisBlock() *Block {

	var geneBlock = Block{0, time.Now().Unix(), "", "", "", 0, difiiculty}
	geneBlock.Hash = hex.EncodeToString(BlockHash(geneBlock))

	return &geneBlock

}



func BlockHash(block Block) []byte {
	re := strconv.Itoa(block.Index) + strconv.Itoa(int(block.TimeStamp)) + block.Data + block.Prehash +
		strconv.Itoa(block.Nonce) + strconv.Itoa(block.Difficulty)

	h := sha256.New()
	h.Write([]byte(re))
	hashed := h.Sum(nil)

	return hashed


}

func isBlockValid(block Block) bool  {
	prefix := strings.Repeat("0", block.Difficulty)
	return strings.HasPrefix(block.Hash, prefix)
}

//创建新区块 pow挖矿
func CreateNewBlock(lastBlock *Block, data string) *Block {
	var newBlock Block
	newBlock.Index = lastBlock.Index + 1
	newBlock.TimeStamp = time.Now().Unix()
	newBlock.Data = data
	newBlock.Prehash = lastBlock.Hash
	newBlock.Difficulty = difiiculty
	newBlock.Nonce = 0
	//开挖-当前区块的hash值的前面的0的个数与难度系数值相同
	for {
		//计算hash
		cuhash := hex.EncodeToString(BlockHash(newBlock))
		fmt.Println("挖矿中",cuhash)
		newBlock.Hash = cuhash
		if isBlockValid(newBlock) {

			//校验区块
			if VerflyBlock(newBlock, *lastBlock) {
				fmt.Println("挖矿成功")
				return  &newBlock
			}
		}
		
		newBlock.Nonce ++

	}
}


//校验新的区块是否合法
func VerflyBlock(newblock Block, lastBlock Block) bool  {
	if lastBlock.Index +1 !=newblock.Index {
		return false
	}
	if newblock.Prehash !=lastBlock.Hash {
		return false
	}
	return true

}

func main()  {

	var genBlock = GenesisBlock()
	
	newBlock := CreateNewBlock(genBlock,"新区块")
	fmt.Println(newBlock)

}
```

输出

```
挖矿中 ac6665903c0cd2f000e17483fbcf6e3e8fa365de2b55663e7c94167f816d1489
挖矿中 a46e18c7938ccb2d0554232f94c6e8db933fae509adafd4091f5f0b51951e6ae
挖矿中 3738b5eb5f8f956974fc767058a6d7c94da0fc406e86df2d508b9b87fc109171
挖矿中 0000694b1acaec754175f0a49a1aa190e122b58e9f58125bd18ceec898f8d811
挖矿成功
&{1 1530267247 新区块 0000694b1acaec754175f0a49a1aa190e122b58e9f58125bd18ceec898f8d811 a8df431924b17633bdf0303763661aa7a41c2608cd99f6527542e1326c718152 12167 4}
```




