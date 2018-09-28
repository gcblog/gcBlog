---
title: DSA和ECC签名验签
categories:
  - 加密算法
tags:
  - DSA
  - ECC
  - 签名
  - 验签
date: 2018-06-28 18:52:24
---

## DSA

DSA 是专业用于数字签名和验签，并且只有这个作用, 不能用于加密和解密。
在安全性上，DSA和RSA差不多，但是速度比RSA快很多。

<!--more-->

### DSA签名和验签

```
func main()  {

	//设置私钥使用的参数
	var param dsa.Parameters
	dsa.GenerateParameters(&param, rand.Reader, dsa.L1024N160)

	//创建私钥
	var pri dsa.PrivateKey
	pri.Parameters = param
	dsa.GenerateKey(&pri, rand.Reader)


	//生成公钥
	pub := pri.PublicKey


	//签名
	message := []byte("hello")
	r,s,_ := dsa.Sign(rand.Reader, &pri, message)


	//验证
	if dsa.Verify(&pub, message, r, s){
		fmt.Println("验签成功")
	}
	
}
```

## ECC
椭圆曲线加密算法，即：Elliptic Curve Cryptography，简称ECC，是基于椭圆曲线数学理论实现的一种非对称加密算法。相比RSA，ECC优势是可以使用更短的密钥，来实现与RSA相当或更高的安全。

椭圆曲线在密码学中的使用，是1985年由Neal Koblitz和Victor Miller分别独立提出的。
比特币就是用ECC来做签名和验签。

### 优点

* 安全性高。160位ECC加密安全性相当于1024位RSA加密，210位ECC加密安全性相当于2048位RSA加密。
* 计算量小,处理速度快,在私钥的处理速度上(解密和签名),ECC远比RSA、DSA快得多
* 存储空间占用小,ECC的密钥尺寸和系统参数与RSA、DSA相比要小得多, 所以占用的存储进空间小得多
* 带宽要求低使得ECC具有广泛得应用前景


### 椭圆曲线

一般情况下，椭圆曲线可用下列方程式来表示，其中a,b,c,d为系数。
> E:y<sup>2</sup>=ax<sup>3</sup>+ bx<sup>2</sup>+cx+d

例如，当a=1,b=0,c=-2,d=4时，所得到的椭圆曲线为:

> E:y<sup>2</sup>=x<sup>3</sup>-2x+4

该椭圆曲线E的图像如图X-1所示，可以看出根本就不是椭圆形。　　

![](http://olgjbx93m.bkt.clouddn.com/20180117-211235.png)

### ECC签名和验签

```
func main()  {

	message := []byte("hello")

	//设置生成的私钥为256位
	privatekey,_ := ecdsa.GenerateKey(elliptic.P256(), rand.Reader)

	//创建公钥
	publicKey := privatekey.PublicKey

	//hash散列明文
	digest := sha256.Sum256(message)


	//用私钥签名
	r,s,_:= ecdsa.Sign(rand.Reader,privatekey, digest[:])

	//设置私钥的参数类型
	param := privatekey.Curve.Params()

	//获取私钥的长度（字节）
	curveOrderBytes:=param.P.BitLen()/8

	//获得签名返回的字节
	rByte,sByte := r.Bytes(), s.Bytes()

	//创建数组合并字节
	signature := make([]byte,curveOrderBytes*2)
	copy(signature[:len(rByte)], rByte)
	copy(signature[len(sByte):], sByte)

	//现在signature中就存放了完整的签名的结果

	//验签
	digest = sha256.Sum256(message)
	//获得公钥的字节长度
	curveOrderBytes= publicKey.Curve.Params().P.BitLen()/8

	//创建大整数类型保存rbyte,sbyte
	r,s = new(big.Int),new(big.Int)

	r.SetBytes(signature[:curveOrderBytes])
	s.SetBytes(signature[curveOrderBytes:])


	//开始认证
	e:=ecdsa.Verify(&publicKey,digest[:],r,s)
	if e== true {
		fmt.Println("验签成功")
	}
}
```




