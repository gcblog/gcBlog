---
title: RSA签名验签
categories:
  - 加密算法
tags:
  - RSA
  - 签名
  - 验签
date: 2018-06-28 18:51:55
---
## 什么是数字签名

数字签名就是只有信息的发送者才能产生的别人无法伪造的一段数字串，这段数字串同时也是对信息的发送者发送信息真实性的一个有效证明。


## 签名的生成和验证

### 生成消息签名的行为

生成消息签名这一行为是由消息的发送者来完成的，也称为“对消息签名”。生成签名就是根据消息内容计算数字签名的值，这个行为意味着“我认可该消息的内容”。

<!--more-->

### 验证消息签名的行为

验证数字签名这一行为一般是由消息的接收者来完成的，但也可以由需要验证消息的第三方来完成，这里的第三方在本书中被命名为验证者。验证签名就是检查该消息的签名是否真的属于发送者,验证的结果可以是成功或者失败，成功就意味着这个签名是属于发送者的，失败则意味着这个签名不是属于发送者的。

## 公钥密码与数字签名

在数字签名中，生成签名和验证签名这两个行为需要使用各自专用的密钥来完成。发送者使用“签名密钥”来生成消息的签名，而验证者则使用“验证密钥”来验证消息的签名。数字签名对签名密钥和验证密钥进行了区分，使用验证密钥是无法生成签名的。此外，签名密钥只能由签名的人持有，而验证密钥则是任何需要验证签名的人都可以持有。


### 公钥密码机制

公钥密码包括一个由公钥和私钥组成的密钥对，其中公钥用于加密，私钥用于解密。用公钥加密所得到的密文只有
用与之对应的私钥才能正确解密。


### 数字签名

数字签名中也同样会使用公钥和私钥组成的密钥对，不过这两个密钥的用法和公钥密码是相反的，即用 **私钥加密** 相当于 **生成签名**，而用 **公钥解密** 则相当于**验证签名**。


## 数字签名算法

### RSA

RSA是提出这个算法的三人姓氏开头字母组成，可用于加密，也可以用于数字签名。

### DSA

DSA ( Digital Signature Algorithm)是一种数字签名算法，是由NIST ( National Institute of Standards and Technology,美国国家标准技术研究所)于1991年制定的数字签名规范( DSS)。DSA是Schnorr算法与ElGammal方式的变体，只能被用于数字签名。

### ECDSA

ECDSA ( Elliptic Curve Digital Signature Algorithm)是一~种利用椭圆曲线密码来实现的数字
签名算法( NIST FIPS 186-3 )。


## RSA算法实现签名和验签
```
func main()  {

	//生成私钥
	pri,_ := rsa.GenerateKey(rand.Reader, 1024)

	//生成公钥
	pub := &pri.PublicKey


	plainTxt := []byte("hello world，你好")

	//对原文进行hash散列
	h := md5.New()
	h.Write(plainTxt)
	hashed := h.Sum(nil)


	opts := rsa.PSSOptions{rsa.PSSSaltLengthAuto, crypto.MD5}

	//实现签名

	sign,_ := rsa.SignPSS(rand.Reader,pri, crypto.MD5,hashed,&opts)

	fmt.Println(hex.EncodeToString(sign))
	

	//通过公钥实现验签
	err:= rsa.VerifyPSS(pub, crypto.MD5,hashed, sign, &opts)

	//err 为空 及验签成功
	fmt.Println("err:", err)

}
```




