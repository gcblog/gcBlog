---
title: go数组
date: 2018-04-18 14:59:13
tags: [数组, go]
categories: 
- go
---


### 数组

* 在Go语言中数组是一个值类型（value type）。是真真实实的数组，而不是一个指向数组内存起始位置的指针，也不能和同类型的指针进行转化，这一点严重不同于C语言。所有的值类型变量在赋值和作为参数传递时都将产生一次复制动作。如果将数组作为函数的参数类型，则在函数调用时该参数将发生数据复制。因此，在函数体中无法修改传入的数组的内容，因为函数内操作的只是所传入数组的一个副本。
* 定义数组的格式：var <varName> [n]<type>，n>=0
* 数组长度也是类型的一部分，因此具有不同长度的数组为不同类型
* 注意区分指向数组的指针和指针数组
* 数组在Go中为值类型
* 数组之间可以使用==或!=进行比较，但不可以使用<或>(比较的前提是数组个数相同，并且元素类型相同)
* 可以使用new来创建数组，此方法返回一个指向数组的指针
* Go支持多维数组

<!--more-->
#### 创建数组

```
func createArr() {

	// 指定数组长度
	// 创建数组如果给定数组长度，数组个数就不能超过这个长度
	var arr1 = [5]int{1, 3, 5, 2, 9}
	fmt.Println(arr1)

	//不指定数组长度
	//Go 语言会根据元素的个数来设置数组的大小
	var arr2 = []int{1, 2}
	var arr3 = [...]int{2, 3, 4}
	fmt.Println(arr2)
	fmt.Println(arr3)

	//创建变量或者常量 可以简写
	arr4 := []int{5}
	fmt.Println(arr4)
	//初始化对指定元素赋值
	//第三个元素 初始化为1
	var arr5 = [5]int{3: 1}
	fmt.Println(arr5)

	// 获取数组的长度和容量
	fmt.Println(len(arr1))
	fmt.Println(cap(arr1))

}
```
结果
```
[1 3 5 2 9]
[1 2]
[2 3 4]
[5]
[0 0 0 1 0]
5
5
```

#### 编辑数组

```
func editArr() {

	var arr1 = []int{1, 2}
	fmt.Print(len(arr1), cap(arr1))
	fmt.Println(arr1)

	// 给arr1 添加元素 3，4
	var arr2 = append(arr1, 3, 4)
	fmt.Println("arr1 = ", arr1)
	fmt.Println("arr2 = ", arr2)
	fmt.Print(len(arr1), cap(arr1))

	//改变数组中元素的值
	arr1[0] = 9
	fmt.Println(arr1)
}
```
结果
```
2 2[1 2]
[1 2 3 4]
4 4[9 2 3 4]
```

#### 指针&数组

```
func pointArr() {
	//数组指针   它是一个指针，指向数组的地址
	a := []int{5: 1}
	var p *[]int = &a
	fmt.Println(a)
	fmt.Println(p) //比a 多了一个取地址符

	// 指针数组 数组里存放的是指针地址，不是实际的值
	var x, y = 4, 5
	arr := []*int{&x, &y}
	fmt.Println(arr)
}
```
结果
```
[0 0 0 0 0 1]
&[0 0 0 0 0 1]
[0xc4200142b0 0xc4200142b8]
```

#### range遍历数组

```
func rangeArr() {

	var arr = []int{1, 2, 3, 4, 5}
	for i, v := range arr {
		fmt.Println(i, v)
	}
}
```
结果
```
0 1
1 2
2 3
3 4
4 5
```
#### new 关键字创建数组

```
func newarr() {

	var a = [5]int{}
	a[1] = 2
	fmt.Println(a)

	p := new([5]int) //可以通过下标赋值
	p[1] = 2
	fmt.Println(p)

}
```
结果
```
[0 2 0 0 0]
&[0 2 0 0 0]
```

#### 数组传递

在函数内改变数组不会影响原本数组的值，因为是指类选，会进行一次拷贝
```
func main() {

	var testArr = [5]int{1, 2, 3, 4, 5}

	fmt.Println(testArr)

	modifyarr(testArr)

	fmt.Println("In main", testArr)

}

func modifyarr(arr [5]int) {
	arr[0] = 10
	fmt.Println("In modify", arr)
}
```
结果
```
[1 2 3 4 5]
In modify [10 2 3 4 5]
In main [1 2 3 4 5]
```

