---
title: 对称加密算法AES原理及实现
categories:
  - 加密算法
tags:
  - AES
  - 对称加密
date: 2018-06-27 18:52:14
---


AES是作为DES的替代标准出现的，全称Advanced Encryption Standard，即：高级加密标准。AES加密算法，经历了公开的选拔，最终2000年，由比利时密码学家Joan Daemen和Vincent Rijmen设计的Rijndael算法被选中，成为了AES标准。
　　AES明文分组长度为128位，即16个字节，密钥长度可以为16个字节、24个字节、或32个字节，即128位密钥、192位密钥、或256位密钥。
　　
<!--more-->
　　
## 总体结构

AES中没有使用Feistel网络，其结构称为SPN结构。
　　
和DES相同，AES也由多个轮组成，其中每个轮分为SubBytes、ShiftRows、MixColumns、AddRoundKey 4个步骤，即：字节代替、行移位、列混淆和轮密钥加。根据密钥长度不同，所需轮数也不同，128位、192位、256位密钥，分别需要10轮、12轮和14轮。第1轮之前有一次AddRoundKey，即轮密钥加，可以视为第0轮；之后1至N-1轮，执行SubBytes、ShiftRows、MixColumns、AddRoundKey；最后一轮仅包括：SubBytes、MixColumns、AddRoundKey。
　　
AES总体结构示意图：
　　
![结构示意图](http://olgjbx93m.bkt.clouddn.com/201801010001.png)

## 分组密码

密码算法可以分为分组密码和流密码两种

* **分组密码（block cipher）**是每次只能处理特定长度的一块数据的一类密码算法，这里的“一块”就称为分组（block）。一个分组的比特数就称为分组长度（block lenght）。

    例如 DES和3DES的分组长度都是64比特。AES的分组长度为128比特。

* **流密码（stream cipher）**是对数据流进行连续处理的一类密码算法。流密码中一般以1比特、8比特、或32比特等为单位进行加密和解密。

分组密码处理完一个分组就结束了，因此不需要通过内部状态来记录加密的进度；相对地，流密码是对一串数据进行连续处理，因此需要保持内部状态。

## 模式

分组密码算法只能加密固定长度的分组，但是我们需要加密的明文长度可能会超过分组密码的分组长度，这时就需要对分组密码算法进行迭代，以便将一段很长的明文全部加密。而迭代的方法就称为分组密码的模式（mode）。

* ECB模式：Electronic CodeBook mode（电子密码模式）
* CBC模式：Cipher Block Chaining mode（密码分组链接模式）
* CFB模式：Cipher FeedBack mode（密文反馈模式）
* OFB模式：Output FeedBack mode（输出反馈模式）
* CTR模式：CounTeR mode（计数器模式）

ECB模式存在很高的风险，下面举例后面4中模式的使用.
加密的过程中使用了随机流，所以每次加密的密文都不一样

### CBC模式
```
func main() {


	key := "1234567890asdfgh"
	data := "hollo, world!"

	cry := AesCBCEncrypt([]byte(data), []byte(key))
	fmt.Println(hex.EncodeToString(cry))

	oriData := AESCBCDECriypt(cry, []byte(key))
	fmt.Println(string(oriData))

}
// AES也是对称加密 AES 是 DES 的替代品
// AES 密钥长度 只能是 16、24、32 字节
//加密
func AesCBCEncrypt(org []byte, key []byte) []byte  {

	//校验密钥
	block,_ := aes.NewCipher(key)

	//按照公钥长度 进行分组补码
	org = PKCS7Padding(org, block.BlockSize())

	//设置CBC的加密模式
	blockMode := cipher.NewCBCEncrypter(block, key)

	//加密处理
	crypted := make([]byte, len(org))
	blockMode.CryptBlocks(crypted, org)

	return crypted
}

//解密
func AESCBCDECriypt(criptText []byte, key []byte) []byte  {

	//校验key的有效性
	block,_:=aes.NewCipher(key)
	//通过CBC模式解密
	blockMode:=cipher.NewCBCDecrypter(block,key)

	//实现解密
	origData:=make([]byte,len(criptText))
	blockMode.CryptBlocks(origData,criptText)

	//去码
	origData = PKCS7UnPadding(origData)
	return origData
}





//PKCS5 分组长度只能为8
//PKCs7 分组长度 1- 255

func PKCS7Padding(org []byte, blockSize int) []byte  {

	pad := blockSize-len(org)%blockSize
	padArr := bytes.Repeat([]byte{byte(pad)}, pad)
	return  append(org, padArr...)

}

func PKCS7UnPadding(cryptText []byte) []byte  {
	
	length := len(cryptText)
	lastByte := cryptText[length - 1]
	return cryptText[:length-int(lastByte)]
	
}
```

输出
```
ffa22c136fd3e944255d43e255c98ecc
hollo, world!
```

### CFB模式
```
func main()  {


	key := []byte("1234567890asdfgh")
	data := []byte("abc hello world!")
	cry := AESCFBEncrypt(data, key)

	fmt.Println(hex.EncodeToString(cry))
	//fmt.Println(base64.StdEncoding.EncodeToString(cry))

	ori := AESCFBDecrypt(cry, key)
	fmt.Println(string(ori))

}

//CFB分组模式加密
func AESCFBEncrypt(oriData []byte, key []byte) []byte  {

	//校验密钥
	block,_ := aes.NewCipher(key)

	//拆分iv和密文
	cipherText := make([]byte, aes.BlockSize + len(oriData))

	iv := cipherText[:aes.BlockSize]

	//向iv切片数组初始化 reader（随机内存流）
	io.ReadFull(rand.Reader, iv)

	//设置加密模式CFB
	stream := cipher.NewCFBEncrypter(block,iv)

	//加密
	stream.XORKeyStream(cipherText[aes.BlockSize:], oriData)

	return  cipherText


}

//解密
func AESCFBDecrypt(cryptText []byte, key []byte) []byte {

	//校验密钥
	block,_ := aes.NewCipher(key)

	//拆分iv 和密文
	iv := cryptText[:aes.BlockSize]
	cipherText := cryptText[aes.BlockSize:]


	//设置解密模式
	stream := cipher.NewCFBDecrypter(block, iv)

	var des = make([]byte, len(cipherText))

	//解密
	stream.XORKeyStream(des, cipherText)

	return des
}

```
输出
```
92e5c5d7bc54b337a7edbb548ee1a62c8c3c079b71f465a3f0566c0d74b8d513
abc hello world!
```

### OFB模式
```
func main()  {

	key := []byte("1234567890asdfgh")
	data := []byte("abcd hello world!")
	cry := AESOFBEncrypt(data, key)

	fmt.Println(hex.EncodeToString(cry))

	ori := AESOFBDecrypt(cry, key)
	fmt.Println(string(ori))
}

//AES OFB分组加密模式  CTR也是一样
func AESOFBEncrypt(plaintxt []byte, key []byte) []byte  {

	//校验密钥
	block,_ := aes.NewCipher(key)

	cipherText := make([]byte, aes.BlockSize + len(plaintxt))

	iv := cipherText[:aes.BlockSize]

	//向iv切片数组初始化 reader（随机内存流）
	io.ReadFull(rand.Reader, iv)

	//设置加密模式CFB
	stream := cipher.NewOFB(block,iv)

	//加密
	stream.XORKeyStream(cipherText[aes.BlockSize:], plaintxt)

	return  cipherText

}


//解密
func AESOFBDecrypt(cryptText []byte, key []byte) []byte {

	//校验密钥
	block,_ := aes.NewCipher(key)

	//拆分iv 和 密文
	iv := cryptText[:aes.BlockSize]
	plaintxt := make([]byte, len(cryptText)-aes.BlockSize)


	//设置解密模式
	stream := cipher.NewOFB(block, iv)

	//解密
	stream.XORKeyStream(plaintxt, cryptText[aes.BlockSize:])

	return plaintxt
}

```
输出
```
9ee409f8513e3fcba2f1ba726da0b2a5d80251efa073544220b44c8e8fee18fce4
abcd hello world!
```

### CTR模式
```
func main()  {

	key := []byte("1234567890asdfgh")
	data := []byte("abcd hello world!")
	cry := AESCTREncrypt(data, key)

	fmt.Println(hex.EncodeToString(cry))

	ori := AESCTRDecrypt(cry, key)
	fmt.Println(string(ori))
}

//AES CTR分组加密模式
func AESCTREncrypt(plaintxt []byte, key []byte) []byte  {

	//校验密钥
	block,_ := aes.NewCipher(key)

	cipherText := make([]byte, aes.BlockSize + len(plaintxt))

	iv := cipherText[:aes.BlockSize]

	//向iv切片数组初始化 reader（随机内存流）
	io.ReadFull(rand.Reader, iv)

	//设置加密模式CTR
	stream := cipher.NewCTR(block,iv)

	//加密
	stream.XORKeyStream(cipherText[aes.BlockSize:], plaintxt)

	return  cipherText

}


//解密
func AESCTRDecrypt(cryptText []byte, key []byte) []byte {

	//校验密钥
	block,_ := aes.NewCipher(key)

	//拆分iv 和 密文
	iv := cryptText[:aes.BlockSize]
	plaintxt := make([]byte, len(cryptText)-aes.BlockSize)


	//设置解密模式
	stream := cipher.NewCTR(block, iv)

	//解密
	stream.XORKeyStream(plaintxt, cryptText[aes.BlockSize:])

	return plaintxt
}

```
输出
```
c645da2d14896d2b75d41a538a5a3efe3c7721f51f2eb2e92b0c5b8ba141caf534
abcd hello world!
```



