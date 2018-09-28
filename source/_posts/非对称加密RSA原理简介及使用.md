---
title: 非对称加密RSA原理简介及使用
categories:
  - 加密算法
tags:
  - 非对称加密
  - RSA
date: 2018-06-27 18:54:38
---

## 什么是RSA

RSA是一种公钥密码算法，它的名字是由它的三位开发者，即Ron Rivest、Adi Shamir 和 Leonard Adleman的姓氏的首字母组成的( Rivest-Shamir-Adleman )。

RSA可以被用于公钥密码和数字签名。


## RSA加密

在RSA中，明文、密钥和密文都是数字。RSA的加密过程可以用下列公式来表达：

**密文=明文<sup>E</sup> mod N** (RSA加密)

RSA的密文是对代表明文的数字的E次方求mod N的结果。换句话说，就是将
明文和自己做E次乘法,然后将其结果除以N求余数，这个余数就是密文。

加密公式中出现的两个数`E`和 `N`，到底都是什么数呢? RSA的加密是求明文的
`E`次方mod `N`，因此只要知道E和N这两个数，任何人都可以完成加密的运算。所以说，`E` 和 `N`是RSA加密的密钥，也就是说，`E` 和 `N`的组合就是公钥。

<!--more-->

## RSA解密

RSA的解密和加密一样简单，可以用下面的公式来表达:

**明文=密文 <sup>D</sup> mod N** ( RSA解密)

表示密文的数字的D次方求 mod N就可以得到明文。

这里所使用的数字N和加密时使用的数字N是相同的。数 `D` 和数 `N` 组合起来就是RSA的解密密钥，因此D和N的组合就是私钥。

在RSA中，加密和解密的形式是相同的。加密是求“明文的E次方的 mod 
N"，而解密则是求“密文的D次方的 mod N"。

## 生成密钥对

在RSA中，加密是求“明文的E次方的 mod 
N"，而解密则是求“密文的D次方的 mod N"。

由于E和N是公钥，D和N是私钥，因此求E、D和N这三个数就是生成密钥对。

## 生成私钥公钥步骤

用 Linux或者mac 上自带，命令如下，完成制作非对称加密的秘钥对（公钥和私钥）


1.生成 RSA 私钥（传统格式的）

`openssl genrsa -out rsa_private_key.pem 1024`

2.将传统格式的私钥转换成 PKCS#8 格式的（JAVA需要使用的私钥需要经过PKCS#8编码，PHP程序不需要，可以直接略过）,这个过程需要输入两遍密码，是针对私钥的密码，不要密码直接按回车

`openssl pkcs8 -topk8 -inform PEM -in rsa_private_key.pem -outform PEM`

3.生成 RSA 公钥

`openssl rsa -in rsa_private_key.pem -pubout -out rsa_public_key.pem`


此时在系统根目录已经生成了两个文件。可以直接使用`cat`命令查看文件中的内容

## 代码实现加密以及解密过程
```

var privateKey = []byte(`-----BEGIN RSA PRIVATE KEY-----
MIICXQIBAAKBgQCQRttulLHUA3DkckkD7Bco5fY0nBDe8RlDZuIV2pu3Ry4qgZNL
d7OiYkgTcow0LIXeW4HpLJZI9oxCS3p4Y+w3AAWOjmpPXZfc3NAiW0iboLa6qld0
TfWogHurC2ArSkONEGzGzdgZrBUDGt8s+sdKmRxxLjPiWq1HQhmywNv3BQIDAQAB
AoGAPKJ64Ct/3QGhNXFOfGaBiT+0TIH2mSusmWYoyFR6svkoTtbsJ4BkL2+sqPew
MtEvZbcBjxSdCIcNhWMhUm10PTur6mOhcAABxTjdFEbIbJRHVlrsDYkyGPLOaaem
UOZeTAtnNQVAnbQpXIwLmwkSSmbJPyvFc534/c7fMkHg1RUCQQDAL5kgijYHox/1
ybqKoBxKrlIjwCpgJ7XIIXRu+AyCLNvzRRviIGGQQ5Q605hSKB+j/6lYKO/kjiNA
Jh5pYPr7AkEAwC7VLAjLQeHD/QD8SaEwQr9WgE3WxF0LuS+AI767Kluw2N2RSDhw
nfNaBhFe1j7mUQLP4C/HIFFjmi1+HiQ1/wJBAKHXM3dAjJFP4HkmAO3+OPT26Xrr
t4OzzRQUgC12u2ngBvVMrFd3d1F6Z1hGmc4Ntd9wS5ZPGv14aNz7fL63CYMCQQC9
T+TxwZ/nwCu+GLBtH3lY5v6g2QyM1lNsEpyZmZLpwPTOTESG7gIRtdyiSY4wYjmi
57A6WRZAgawp/lJUArulAkA3LKFGfViQVjRWkoIYN65R87L6DohHH1LXVg4wUUtF
IXwDFSCbHsQko+vzIlBtSXr5/hO+1CkZLtI0tisWHPCi
-----END RSA PRIVATE KEY-----`)

var publicKey = []byte(`-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQCQRttulLHUA3DkckkD7Bco5fY0
nBDe8RlDZuIV2pu3Ry4qgZNLd7OiYkgTcow0LIXeW4HpLJZI9oxCS3p4Y+w3AAWO
jmpPXZfc3NAiW0iboLa6qld0TfWogHurC2ArSkONEGzGzdgZrBUDGt8s+sdKmRxx
LjPiWq1HQhmywNv3BQIDAQAB
-----END PUBLIC KEY-----`)

func main()  {


	data := []byte("hello world")

	cry := RSAEncrypt(data)

	fmt.Println(hex.EncodeToString(cry))


	ori := RSADecrypt(cry)
	fmt.Println(string(ori))


}

func RSAEncrypt(plaintxt []byte) []byte  {

	//公钥加密
	block, _:= pem.Decode(publicKey)

	//解析公钥
	pubInterface, _ := x509.ParsePKIXPublicKey(block.Bytes)

	//加载公钥
	pub := pubInterface.(*rsa.PublicKey)

	//加密明文
	bits,_ := rsa.EncryptPKCS1v15(rand.Reader, pub, plaintxt)

	//bits为最终的密文
	return bits
}

func RSADecrypt(cryptTxt []byte) []byte  {

	block,_:= pem.Decode(privateKey)

	priv,_:= x509.ParsePKCS1PrivateKey(block.Bytes)

	bits,_ := rsa.DecryptPKCS1v15(rand.Reader, priv, cryptTxt)

	return bits
}
```

输出
```
4bf96c2a3d83a71745ebc15441046f2ca0bc2cffab73a0a452bcb5c3b58ee717585e779df5046a31f71a6abf650e097f86e4ee95cb14ddd121794763a556987a965be2ec9eb47ff1cf17adcc166bc349a2471f24b924d50927b41bb61d4c0f357949a05e1f00db870582e00e7234e35923e6f1eae3021892590458af3b790f8a
hello world
```


## 代码实现生成公钥私钥来加密和解密
```
func main()  {

	prikey := CreatePrivateKey()
	pubKey := CreatePublic(prikey)
	
	// 加密和解密
	ori:=[]byte("hello world!!!")
	//通过oaep函数实现公钥加密
	//第一个参数作用，将不同长度的明文，通过hash散列实现相同的散列值，此过程就是生成密文摘要
	cipherTxt,_ :=rsa.EncryptOAEP(md5.New(), rand.Reader, &pubKey, ori, nil)

	fmt.Println(cipherTxt)

	//解密
	plainTxt,_ := rsa.DecryptOAEP(md5.New(), rand.Reader, prikey,cipherTxt, nil)
	fmt.Println(string(plainTxt))
	
}

// 创建私钥
func CreatePrivateKey() *rsa.PrivateKey  {

	// 长度为1024 位 的私钥
	pri,_:= rsa.GenerateKey(rand.Reader, 1024)
	return pri

}
// 生成公钥
func CreatePublic(prikey *rsa.PrivateKey) rsa.PublicKey  {
	return prikey.PublicKey
}
```



