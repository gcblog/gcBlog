---
title: mongodb+golang+mac
categories:
  - go
tags:
  - mongodb
  - 数据库
  - 分布式
date: 2018-07-09 20:01:50
---

### MongoDB简介

MongoDB是一个介于关系数据库和非关系数据库之间的产品，是非关系数据库当中功能最丰富，最像关系数据库的。他支持的数据结构非常松散，是类似json的bson格式，因此可以存储比较复杂的数据类型。

### NoSql数据库的CAP理论

CAP理论，指的是在一个分布式系统中，Consistency（一致性）、 Availability（可用性）、Partition tolerance（分区容错性），三者不可得兼。
<!--more-->
![](https://raw.githubusercontent.com/suifengqjn/demoimages/master/blog/cap.png)

* 一致性（C）：在分布式系统中的所有数据备份，在同一时刻是否同样的值。（等同于所有节点访问同一份最新的数据副本）
* 可用性（A）：在集群中一部分节点故障后，集群整体是否还能响应客户端的读写请求。（对数据更新具备高可用性）
* 分区容错性（P）：以实际效果而言，分区相当于对通信的时限要求。系统如果不能在时限内达成数据一致性，就意味着发生了分区的情况，必须就当前操作在C和A之间做出选择。

### 安装MongoDB

用OSX 的 brew 来安装 mongodb
`brew install mongodb` 或者 `sudo brew install mongodb`

如果要安装支持 TLS/SSL 命令如下：
`brew install mongodb --with-openssl`

查看是否安装成功
`mongod -version`
如果显示版本信息，说明已经安装成功

### 启动mongo

mongo默认会在根目录 /data/db 下启动服务，所以需要先创建此路径。
`sudo mkdir -p /data/db`

然后开启服务
`mongod`

或者开启指定路径的服务
`mongod --dbpath=xxx`

新开一个终端，输入命令
`mongo`
进入mongo系统

再次输入命令测试
`show dbs`
显示如下

```
> show dbs
admin   0.000GB
config  0.000GB
local   0.000GB
```
即已成功运行

### 基本命令

显示数据库
`show dbs`
显示数据表
`show collections`
选择或者创建 mydb数据表
`use mydb`

#### 增

如果没有 user表 会自动创建
`db.user.save({"name":"zhangsan","age":2})`

#### 删

删除数据表
`db.user.drop()`

删除数据库
`use mydb`
`db.dropDatasase()`

删除记录
`db.user.remove({"name":"zhangsan"})`

#### 查

查所有
`db.user.find()`
查 age=2
`db.user.find({"age":2})`

#### 改

只改动某一项值

```
db.user.update({"_id" : ObjectId("5b41c89323c223baaa7d4ef1")},{$set:{"name":"wangwu"}})
```

如果没有set，相当于覆盖这条记录

### golang中使用mongo

![](https://raw.githubusercontent.com/suifengqjn/demoimages/master/blog/mongo.png)

### golang demo


```
package main

import (
	"gopkg.in/mgo.v2"
	"gopkg.in/mgo.v2/bson"
	"fmt"
)

type Person struct {
	ID    bson.ObjectId `_id`
	Name string
	Age int
}

const  (
	dbUrl = "127.0.0.1:27017"
)

func main(){

	session,err := mgo.Dial(dbUrl)

	if err != err {
		println(err)
	}
	defer session.Close()
	session.SetMode(mgo.Monotonic, true) //设置一致性模式

	//选择数据库
	db := session.DB("mydb")

	//选择数据表
	collection := db.C("user")

	//增  单条插入
	p1 := Person{bson.NewObjectId(),"lisi2", 12}
	p2 := Person{bson.NewObjectId(),"lisi2", 12}

	err = collection.Insert(&p1, &p2)

	//批量插入
	var arr []interface{}
	arr = append(arr, p1)
	arr = append(arr, p2)
	err = collection.Insert(arr...)
	if err != err {
		println(err)
	}

	//查第一条
	result := Person{}
	collection.Find(bson.M{"age":12}).One(&result)
	fmt.Println("restlt",result)

	//查多条
	results := []Person{}
	collection.Find(bson.M{"age":12}).All(&results)
	fmt.Println("results",results, len(results))


	//查表中数据总数
	count, _:= collection.Find(nil).Count()
	fmt.Println(count)

	//查所有
	arr2 := make([]Person, 0)
	iterNew := collection.Find(nil).Iter()
	err = iterNew.All(&arr2)
	if err != nil {
		panic(err)
	}
	fmt.Println("arr2",arr2)

	//改(不加set就是覆盖)
	collection.Update(bson.M{"name":"zhangsan"}, bson.M{"$set":bson.M{"age":22}})
	collection.Update(bson.M{"age":22},bson.M{"$set":bson.M{"age":23}})
	collection.Update(bson.M{"name":"lisi"}, bson.M{"name":"lisi2","age":22})

	//批量更新
	collection.UpdateAll(bson.M{"age":22}, bson.M{"$set":bson.M{"age":44}})


	//删

	//删除符合条件的第一条
	//collection.Remove(bson.M{"name": "lisi2"})

	//删除所有
	_, err = collection.RemoveAll(bson.M{"name": "lisi"})
	if err != err {
		println(err)
	}



	//根据ID删除
	var wangwu = Person{}
	collection.Find(bson.M{"name":"wangwu"}).One(&wangwu)
	fmt.Println(wangwu, wangwu.ID.Hex(), wangwu.ID.String())
	err =collection.RemoveId(wangwu.ID)
	fmt.Println("delete",err)
	
	//删除集合
  //collection.DropCollection()


}

```

