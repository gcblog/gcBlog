---
title: 对称加密算法DES原理及实现
categories:
  - 加密算法
tags:
  - 对称加密
  - DES
  - 3DES
date: 2018-06-26 19:24:12
---
## DES加密算法

DES加密算法，为对称加密算法中的一种。70年代初由IBM研发，后1977年被美国国家标准局采纳为数据加密标准，即DES全称的由来：Data Encryption Standard。对称加密算法，是相对于非对称加密算法而言的。两者区别在于，对称加密在加密和解密时使用同一密钥，而非对称加密在加密和解密时使用不同的密钥，即公钥和私钥。常见的DES、3DES、AES均为对称加密算法，而RSA、椭圆曲线加密算法，均为非对称加密算法。
 
　　DES是以64比特的明文为一个单位来进行加密的，超过64比特的数据，要求按固定的64比特的大小分组，分组有很多模式，后续单独总结，暂时先介绍DES加密算法。DES使用的密钥长度为64比特，但由于每隔7个比特设置一个奇偶校验位，因此其密钥长度实际为56比特。奇偶校验为最简单的错误检测码，即根据一组二进制代码中1的个数是奇数或偶数来检测错误。
　　
<!--more-->

## Feistel网络

DES的基本结构，由IBM公司的Horst Feistel设计，因此称Feistel网络。在Feistel网络中，加密的每个步骤称为轮，经过初始置换后的64位明文，进行了16轮Feistel轮的加密过程，最后经过终结置换后形成最终的64位密文。如下为Feistel网络的示意图：

![](http://olgjbx93m.bkt.clouddn.com/5f65e67ee5cbf1514c5614e944684cc1af2a4096.jpg)  

64位明文被分为左、右两部分处理，右侧数据和子密钥经过轮函数f生成用于加密左侧数据的比特序列，与左侧数据异或运算，运算结果输出为加密后的左侧，右侧数据则直接输出为右侧。
　　其中子密钥为本轮加密使用的密钥，每次Feistel均使用不同的子密钥。子密钥的计算，以及轮函数的细节，稍后下文介绍。由于一次Feistel轮并不会加密右侧，因此需要将上一轮输出后的左右两侧对调后，重复Feistel轮的过程，DES算法共计进行16次Feistel轮，最后一轮输出后左右两侧无需对调。

DES加密和解密的过程一致，均使用Feistel网络实现，区别仅在于解密时，密文作为输入，并逆序使用子密钥。

## DES 加密算法的使用
在使用DES加密前，需要自己实现对明文的补码和去码操作

补码
```
//实现PKCS5Padding补码
func PKCS5Padding(cipherTxt [] byte, blockSize int) []byte {
	//计算准备添加的数字
	padding := blockSize - len(cipherTxt)%blockSize

	//得到补码
	padTxt := bytes.Repeat([]byte{byte(padding)}, padding)

	//拼接原文与补码
	var byteTxt = append(cipherTxt, padTxt...)

	return byteTxt

}
```

去码
```
//PKCS5Unpadding 去码
func PKCS5UnPadding(cipherTxt []byte) []byte {
	var l = len(cipherTxt)
	var txt = int(cipherTxt[l-1])
	res := cipherTxt[:l-txt]
	return res
}
```

DES加密
```
// key 必须为8位
func EnDESEncrypt (origData []byte, key []byte) []byte {

	//校验密钥
	block, _ := des.NewCipher(key)
	//设置补码
	origData = MyDES.PKCS5Padding(origData, block.BlockSize())
	//设置CBC加密模式
	blockMode := cipher.NewCBCEncrypter(block, key)

	//加密明文
	crypted := make([]byte, len(origData))

	blockMode.CryptBlocks(crypted, origData)

	return crypted

}
```


DES解密
```
func DeDESCriypt(cript []byte, key []byte) []byte  {
	//校验key的有效性
	block,_:=des.NewCipher(key)
	//通过CBC模式解密
	blockMode:=cipher.NewCBCDecrypter(block,key)

	//实现解密
	origData:=make([]byte,len(cript))
	blockMode.CryptBlocks(origData,cript)

	//去码
	origData = MyDES.PKCS5UnPadding(origData)
	return origData
}
```

使用
```
func main()  {

	key := []byte("aswedrfg")
	var data =[]byte("hello world")
	var cipherTxt = EnDESEncrypt(data,key)
	fmt.Println("加密的结果：",hex.EncodeToString( cipherTxt))

	var origData=DeDESCriypt(cipherTxt,key)
	fmt.Println("解密后的结果为:",string(origData))
}
```

输出结果
```
加密的结果： 935ae7ca3229f6c707bb9de9db9693c7
解密后的结果为: hello world
```

## 根据DES原理自己实现加密解密过程

加密
```
func EnCrypt(key string, data []byte) []byte {

	var sum = 0
	for i := 0; i < len(key); i++ {
		sum += int(key[i])
	}
	// 对明文进行补码
	var pad = PKCS5Padding(data, len(key))
	//通过加法，实现简单加密
	for i := 0;i<len(pad);i++{
		pad[i] = pad[i]+byte(sum)
	}
	return pad
}
```

解密
```
func Decrypt(cipherTxt []byte,key string) []byte {

	fmt.Println("???",cipherTxt)
	//计算key的总和
	var sum =0
	for i:=0;i<len(key);i++ {
		sum += int(key[i])
	}

	//减法运算
	for i:=0;i<len(cipherTxt);i++{
		cipherTxt[i]=cipherTxt[i]-byte(sum)
	}
	fmt.Println("???",cipherTxt)
	//去码
	var p = PKCS5UnPadding(cipherTxt)
	return p
}
```

## 3DES

### 3DES加密
DES是一个经典的对称加密算法，但也缺陷明显，即56位的密钥安全性不足，已被证实可以在短时间内破解。为解决此问题，出现了3DES，也称Triple DES，3DES为DES向AES过渡的加密算法，它使用3条56位的密钥对数据进行三次加密。为了兼容普通的DES，3DES并没有直接使用 `加密->加密->加密` 的方式，而是采用了`加密->解密->加密` 的方式。

![](http://oscd4dgpc.bkt.clouddn.com/00f3fb2538ec90b80eb9.jpg)

当三重密钥均相同时，前两步相互抵消，相当于仅实现了一次加密，因此可实现对普通DES加密算法的兼容。

![](http://oscd4dgpc.bkt.clouddn.com/7de5d8dfcd0bd221596384.jpg)

### 3DES解密

3DES解密过程，与加密过程相反，即逆序使用密钥。是以密钥3、密钥2、密钥1的顺序执行 `解密->加密->解密`。

![](http://oscd4dgpc.bkt.clouddn.com/WX20180216-083713.png)

相比DES，3DES因密钥长度变长，安全性有所提高，但其处理速度不高。因此又出现了AES加密算法，AES较于3DES速度更快、安全性更高。



