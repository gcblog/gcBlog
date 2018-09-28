---
title: etcd在golang中的使用
categories:
  - 区块链
tags:
  - etcd
  - 分布式
date: 2018-07-05 19:35:05
---

## 什么是etcd

ETCD是用于共享配置和服务发现的分布式，一致性的KV存储系统
etcd作为一个受到ZooKeeper与doozer启发而催生的项目，除了拥有与之类似的功能外，更专注于以下四点。
	1.	简单：基于HTTP+JSON的API让你用curl就可以轻松使用。
	2.	安全：可选SSL客户认证机制。
	3.	快速：每个实例每秒支持一千次写操作。
	4.	可信：使用Raft算法充分实现了分布式。
	
!<--more-->
	
## 安装

在如下路径创建文件夹
`$ mkdir -p $GOPATH/src/github.com/coreos`

下载etcd包
`$ git clone https://github.com/coreos/etcd.git`

下载完后，然后依次执行下面命令
```
$ cd etcd
$ ./build
$ ./bin/etcd
```

## 代码实现

```
package main

import (
	"time"
	"github.com/coreos/etcd/clientv3"
	"fmt"
	"context"
	"github.com/coreos/etcd/mvcc/mvccpb"
	"ketang/netWork/0604_Socket/Tool"
)

var (
	dialTimeout = 5 * time.Second
	requestTimeout = 2 * time.Second
	endPoints = []string{"127.0.0.1:2379"} //etcd 默认接受数据的端口2379
)
//添加 删除 查找 前缀 延时

var etcd *clientv3.Client
func main()  {
	fmt.Println(Tool.GetLocalIp())
	var err error
	etcd, err =clientv3.New(clientv3.Config{
		Endpoints:endPoints,
		DialTimeout:dialTimeout,


	})
	if err != nil {
		fmt.Println(err)
	}
	
	//添加
	err = putValue("a", "abc")
	fmt.Println(err)

	//查找
	result := getValue("a")
	fmt.Println(result)

	//删除
	cnt := delValue("a")
	fmt.Println("delete:", cnt)


	err = putValue("b1", "abc1")
	err = putValue("b2", "abc2")
	err = putValue("b3", "abc3")

	//按前缀查找
	result = getValueWIthPrefix("b")
	fmt.Println(result)
	for _,item := range result {
		fmt.Println(string(item.Key),string(item.Value))
	}

	//按前缀删除
	cnt2 := delValueWithPrefix("b")
	fmt.Println("批量删除：", cnt2)



	//事务处理
	putValue("user1", "zhangsan")
	_,err = etcd.Txn(context.TODO()).
		If(clientv3.Compare(clientv3.Value("user1"),"=", "zhangsan")).
		Then(clientv3.OpPut("user1", "zhangsan")).
		Else(clientv3.OpPut("user1", "lisi")).Commit()

	fmt.Println(err)
	result = getValue("user1")
	fmt.Println("user1:", string(result[0].Value))


	//lease 设置有效时间
	resp, err:= etcd.Grant(context.TODO(), 1)
	_,err = etcd.Put(context.TODO(), "username","wangwu",clientv3.WithLease(resp.ID))

	time.Sleep(3 * time.Second)

	v := getValue("username")
	fmt.Println("lease:",v)


	//watch监听的使用
	putValue("w", "hello")
	go func() {
		rch := etcd.Watch(context.Background(),"w")
		for wresp := range  rch {
			for _,ev := range wresp.Events {
				fmt.Printf("watch>>w  %s %q %q\n", ev.Type,ev.Kv, ev.Kv)
			}
		}
	}()

	putValue("w", "hello world!")



	//监听某个key在一定范围内 value的变化
	//putValue("fo0", "a")
	go func() {
		//监听范围 [fo0-fo3)
		rch := etcd.Watch(context.Background(), "fo0", clientv3.WithRange("fo3"))

		for wresp := range  rch {
			for _,ev := range wresp.Events {
				fmt.Printf("watch range  --   %s %q %q\n", ev.Type,ev.Kv, ev.Kv)
			}
		}
	}()

	putValue("fo0", "b")
	putValue("fo1", "b")
	putValue("fo2", "c")
	putValue("fo2.5", "c")
	putValue("fo3", "c")

	
	time.Sleep(10 * time.Second)

}


//添加键值对
func putValue(key, value string)  error  {
	_, err := etcd.Put(context.TODO(),key, value)
	return err
}

//查询
func getValue(key string) []*mvccpb.KeyValue  {
	resp, err := etcd.Get(context.TODO(), key)
	if err != nil {
		return  nil
	} else {
		return resp.Kvs
	}
}

// 返回删除了几条数据
func delValue(key string) int64  {
	res,err := etcd.Delete(context.TODO(),key)
	if err != nil {
		return 0
	} else {
		return res.Deleted
	}

}


//按照前缀删除
func delValueWithPrefix(prefix string) int64  {
	res,err := etcd.Delete(context.TODO(),prefix,clientv3.WithPrefix())
	if err != nil {
		fmt.Println(err)
		return 0
	} else {
		return res.Deleted
	}
}

func getValueWIthPrefix(prefix string) []*mvccpb.KeyValue {
	resp, err := etcd.Get(context.TODO(), prefix, clientv3.WithPrefix())
	if err != nil {
		return  nil
	} else {
		return resp.Kvs
	}
}

```


