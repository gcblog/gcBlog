---
title: slice
categories:
  - go
tags:
  - go
  - slice
  - 排序
date: 2018-05-04 11:45:23
---


切片的几种常用操作，reslice, append, copy, delete, insert, 排序, 取最值等，这几种基本上可以解决日常开发的绝大部分的问题了。

### slice声明和创建

slice 本身不是数组，它指向数组的底层
作为变长数组饿替代方案，可以关联底层数组的全部或者局部
可以直接创建，一般用make(), 也可以从底层数组获取生成
如果多个slice 指向相同的底层数组，一个值的改变会影响全部

<!--more-->

```
func sliceCreate() {

	//声明 slice 如果只是声明，只有指针初始地址，容量为0
	var slice1 []int

	fmt.Printf("slice1  %p\n", slice1)

	//添加元素后，容量超出上限，内存地址重新分配
	slice1 = []int{1, 2, 3}

	fmt.Printf("slice1  %p\n", slice1)

	//声明并赋值

	var s2 = []string{"a", "b", "c"}
	fmt.Println(s2)

	var arr = [10]int{0, 1, 2, 3, 4, 5, 6, 7, 8, 9}
	fmt.Print("s2  \n", arr)

	// 取第二个元素
	slice2 := arr[1]
	fmt.Println(slice2)

	//取第5到10个元素
	slice3 := arr[5:10] // slice范围取值的时候，前面闭区间，后面开区间
	fmt.Println(slice3)

	//从第5个元素取到最后  的两种写法
	slice4 := arr[5:len(arr)]
	slice5 := arr[5:]
	fmt.Println(slice4, slice5)

	//取前5个元素
	slice6 := arr[:5]
	fmt.Print("\n", slice6)

	//使用make 创建slice
	//存放的元素是int类型，初始容量是3, 容量超过10再次开辟内存空间，并且容量翻倍
	// 如果不给初始容量，默认的初始容量就是当前元素个数
	s1 := make([]int, 3, 10)
	fmt.Print("\n", len(s1), cap(s1))
}
```

输出

```
slice1  0x0
slice1  0xc4200182c0
[a b c]
s2  
[0 1 2 3 4 5 6 7 8 9]1
[5 6 7 8 9]
[5 6 7 8 9] [5 6 7 8 9]

[0 1 2 3 4]
3 10
```

### ReSlice

```
func ResliceTest() {
	
	// reslice索引 以 被slice 为准
	// 索引不可以超过cap值

	var a = []byte{'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h'}

	// 切片区间取值 包含起始值， 不包含终止值 相当与 闭开区间
	//取出 cde
	var s1 = a[2:5]
	fmt.Println(string(s1))
	fmt.Println(len(s1), cap(s1))

	//从 s1中取出de
	var s2 = s1[1:3]
	fmt.Print("\n", string(s2))

	// 虽然s1 指向 a中的 c 到 e, 由于a的内存是连续， 可以从s1中取到数组的末尾

	var s3 = s1[0:cap(s1)]
	fmt.Print("\n s3 = ", string(s3))

}
```

```
cde
3 6

de
 s3 = cdefgh
```

### Append

可以在slice尾部追加元素
可以将一个slice追加到另一个slice
slice2 追加到slice1 后，如果容量超过slice1，则slice1容量将重新分配，内存地址也改变
```
func appendTest() {

	s1 := make([]int, 3, 6)
	fmt.Printf("%p\n", s1)

	s1 = append(s1, 1, 2, 3) //s1 还没有超出容量，内存地址不变
	fmt.Printf("%v---%p\n", s1, s1)

	s1 = append(s1, 9) //超出容量，内存地址改变
	fmt.Printf("%v %p\n", s1, s1)

	var arr = []int{1, 2, 3, 4, 5}
	var slice2 = arr[2:5] //3, 4, 5
	var slice3 = arr[1:3] //2, 3

	fmt.Println(slice2, slice3) //重叠部分是3

	slice2[0] = 9 //改变重叠部分的值， 其他所有对应的值都会改变
	fmt.Println(arr, slice2, slice3)

	slice3 = append(slice3, 1, 2, 2, 2, 2, 2) //超出slice3原本的容量，内存地址重新分配，其他对象重叠部分的元素就不会改变
	fmt.Println(slice3)
}
```

```
0xc42001c0c0
[0 0 0 1 2 3]---0xc42001c0c0
[0 0 0 1 2 3 9] 0xc420078060
[3 4 5] [2 3]
[1 2 9 4 5] [9 4 5] [2 9]
[2 9 1 2 2 2 2 2]
```

### Copy

```
func copyTest() {

	//copy 会把重叠部分的元素给覆盖
	var s1 = []int{1, 2, 3, 4, 5, 6, 7}
	var s2 = []int{7, 8, 9}
	// 短的copy到长的情况
	copy(s1, s2) //把s2 copy 到s1 中
	fmt.Println("s1---", s1)

	// 长的copy到短的情况
	var s3 = []int{1, 2, 3, 4, 5, 6, 7}
	var s4 = []int{7, 8, 9}
	copy(s4, s3)
	fmt.Println("s4---", s4)

	var s5 = []int{1, 2, 3, 4, 5, 6, 7}
	var s6 = []int{7, 8, 9}
	//将制定元素copy到指定位置
	copy(s5[2:4], s6[1:3])
	fmt.Println("s5---", s5)

}
```

```
s1--- [7 8 9 4 5 6 7]
s4--- [1 2 3]
s5--- [1 2 8 9 5 6 7]
```

### delete

go官方没有提供删除的方法，只能自己实现，实现的方法有多种，这里给出一种简洁的实现方式

```
func sliceDelete() {

	var slice = []string{"a", "b", "c", "d"}

	fmt.Println(removeString(slice, 2)
}

//删除函数
func removeString(s []string, i int) []string {

	return append(s[:i], s[i+1:]...)

}
```

输出

```
[a b d]
```

### insert

官方还是没有提供插入的方法，需要自己实现，如果用`append`实现，`append`每次操作都会复制创建一个新的底层数组，会比较耗性能，这里采用反射实现
这里需要引入`reflect`

```
func silceInsert() {

	a := []int{1, 2, 3, 4, 5}
	fmt.Println(a)
	fmt.Println("插入元素后", Insert(a, 3, 0))

	b := []string{"a", "b", "c", "d", "e"}
	fmt.Println(b)
	fmt.Println("插入元素后", Insert(b, 3, "x"))

	c := []interface{}{1, "a", true, 3.2, 'a'}
	fmt.Println(c)
	fmt.Println("插入元素后", Insert(c, 3, false))

}

func Insert(slice interface{}, pos int, value interface{}) interface{} {
	v := reflect.ValueOf(slice)
	v = reflect.Append(v, reflect.ValueOf(value))
	reflect.Copy(v.Slice(pos+1, v.Len()), v.Slice(pos, v.Len()))
	v.Index(pos).Set(reflect.ValueOf(value))
	return v.Interface()
}
```

```
[1 2 3 4 5]
插入元素后 [1 2 3 0 4 5]
[a b c d e]
插入元素后 [a b c x d e]
[1 a true 3.2 97]
插入元素后 [1 a true false 3.2 97]
```


### 排序

几种常见的排序方法：冒泡算法，选择排序，插入排序，快速排序，不过对于排序，官方倒是给出了方法，可以直接使用，需要引入`sort`

#### 升序排序

对于 int 、 float64 和 string 数组或是分片的排序， go 分别提供了 sort.Ints() 、 sort.Float64s() 和 sort.Strings() 函数， 默认都是从小到大排序。

```
func sliceSort() {

	intList := []int{2, 4, 3, 5, 7, 6, 9, 8, 1, 0}
	float8List := []float64{4.2, 5.9, 12.3, 10.0, 50.4, 99.9, 31.4, 27.81828, 3.14}
	stringList := []string{"a", "c", "b", "d", "f", "i", "z", "x", "w", "y"}

	sort.Ints(intList)
	sort.Float64s(float8List)
	sort.Strings(stringList)

	fmt.Printf("%v\n%v\n%v\n", intList, float8List, stringList)
}
```

```
[0 1 2 3 4 5 6 7 8 9]
[3.14 4.2 5.9 10 12.3 27.81828 31.4 50.4 99.9]
[a b c d f i w x y z]
```

#### 降序排序

int 、 float64 和 string 都有默认的升序排序函数， 现在问题是如果降序如何 ？ 有其他语言编程经验的人都知道，只需要交换 cmp 的比较法则就可以了， go 的实现是类似的，然而又有所不同。 go 中对某个 Type 的对象 obj 排序， 可以使用 sort.Sort(obj) 即可，就是需要对 Type 类型绑定三个方法 ： Len() 求长度、Less(i,j) 比较第 i 和 第 j 个元素大小的函数、 Swap(i,j) 交换第 i 和第 j 个元素的函数。sort 包下的三个类型 IntSlice 、 Float64Slice 、 StringSlice 分别实现了这三个方法， 对应排序的是 [] int 、 [] float64 和 [] string 。如果期望逆序排序， 只需要将对应的 Less 函数简单修改一下即可。

go 的 sort 包可以使用 sort.Reverse(slice) 来调换 slice.Interface.Less ，也就是比较函数，所以， int 、 float64 和 string 的逆序排序函数可以这么写：

```
func sliceSort2() {
	intList := []int{2, 4, 3, 5, 7, 6, 9, 8, 1, 0}
	float8List := []float64{4.2, 5.9, 12.3, 10.0, 50.4, 99.9, 31.4, 27.81828, 3.14}
	stringList := []string{"a", "c", "b", "d", "f", "i", "z", "x", "w", "y"}

	sort.Sort(sort.Reverse(sort.IntSlice(intList)))
	sort.Sort(sort.Reverse(sort.Float64Slice(float8List)))
	sort.Sort(sort.Reverse(sort.StringSlice(stringList)))

	fmt.Printf("%v\n%v\n%v\n", intList, float8List, stringList)
}
```


```
[9 8 7 6 5 4 3 2 1 0]
[99.9 50.4 31.4 27.81828 12.3 10 5.9 4.2 3.14]
[z y x w i f d c b a]
```

#### 结构体类型的排序
结构体类型的排序是通过使用 sort.Sort(slice) 实现的， 只要 slice 实现了 sort.Interface 的三个方法就可以。 虽然这么说，但是排序的方法却有那么好几种。首先一种就是模拟排序 [] int 构造对应的 IntSlice 类型，然后对 IntSlice 类型实现 Interface 的三个方法。
##### 1、模拟 IntSlice 排序
```
package main
 
import (
    "fmt"
    "sort"
)
 
type Person struct {
    Name string
    Age  int
}
 
// 按照 Person.Age 从大到小排序
type PersonSlice [] Person
 
func (a PersonSlice) Len() int {    	 // 重写 Len() 方法
    return len(a)
}
func (a PersonSlice) Swap(i, j int){     // 重写 Swap() 方法
    a[i], a[j] = a[j], a[i]
}
func (a PersonSlice) Less(i, j int) bool {    // 重写 Less() 方法， 从大到小排序
    return a[j].Age < a[i].Age
}
 
func main() {
    people := [] Person{
        {"zhang san", 12},
        {"li si", 30},
        {"wang wu", 52},
        {"zhao liu", 26},
    }
 
    fmt.Println(people)
 
    sort.Sort(PersonSlice(people))    // 按照 Age 的逆序排序
    fmt.Println(people)
 
    sort.Sort(sort.Reverse(PersonSlice(people)))    // 按照 Age 的升序排序
    fmt.Println(people)
 
}
```

这完全是一种模拟的方式，所以如果懂了 IntSlice 自然就理解这里了，反过来，理解了这里那么 IntSlice 那里也就懂了。
这种方法的缺点是：根据 Age 排序需要重新定义 PersonSlice 方法，绑定 Len 、 Less 和 Swap 方法， 如果需要根据 Name 排序， 又需要重新写三个函数； 如果结构体有 4 个字段，有四种类型的排序，那么就要写 3 × 4 = 12 个方法， 即使有一些完全是多余的， O__O”… 仔细思量一下，根据不同的标准 Age 或是 Name， 真正不同的体现在 Less 方法上，所以可以将 Less 抽象出来， 每种排序的 Less 让其变成动态的，比如下面一种方法。

##### 2、封装成 Wrapper

```
package main
 
import (
    "fmt"
    "sort"
)
 
type Person struct {
    Name string
    Age  int
}
 
type PersonWrapper struct {					//注意此处
    people [] Person
    by func(p, q * Person) bool
}
 
func (pw PersonWrapper) Len() int {    		// 重写 Len() 方法
    return len(pw.people)
}
func (pw PersonWrapper) Swap(i, j int){     // 重写 Swap() 方法
    pw.people[i], pw.people[j] = pw.people[j], pw.people[i]
}
func (pw PersonWrapper) Less(i, j int) bool {    // 重写 Less() 方法
    return pw.by(&pw.people[i], &pw.people[j])
}
 
func main() {
    people := [] Person{
        {"zhang san", 12},
        {"li si", 30},
        {"wang wu", 52},
        {"zhao liu", 26},
    }
 
    fmt.Println(people)
 
    sort.Sort(PersonWrapper{people, func (p, q *Person) bool {
        return q.Age < p.Age    // Age 递减排序
    }})
 
    fmt.Println(people)
    sort.Sort(PersonWrapper{people, func (p, q *Person) bool {
        return p.Name < q.Name    // Name 递增排序
    }})
 
    fmt.Println(people)
 
}
```

这种方法将 [] Person 和比较的准则 cmp 封装在了一起，形成了 PersonWrapper 函数，然后在其上绑定 Len 、 Less 和 Swap 方法。 实际上 sort.Sort(pw) 排序的是 pw 中的 people， 这就是前面说的， go 的排序未必就是针对的一个数组或是 slice， 而可以是一个对象中的数组或是 slice 。

##### 3、进一步封装

感觉方法 2 已经很不错了， 唯一一个缺点是，在 main 中使用的时候暴露了 sort.Sort 的使用，还有就是 PersonWrapper 的构造。 为了让 main 中使用起来更为方便， me 们可以再简单的封装一下， 构造一个 SortPerson 方法， 如下：
```
package main
 
import (
    "fmt"
    "sort"
)
 
type Person struct {
    Name string
    Age  int
}
 
type PersonWrapper struct {
    people [] Person
    by func(p, q * Person) bool
}
 
type SortBy func(p, q *Person) bool
 
func (pw PersonWrapper) Len() int {    		// 重写 Len() 方法
    return len(pw.people)
}
func (pw PersonWrapper) Swap(i, j int){         // 重写 Swap() 方法
    pw.people[i], pw.people[j] = pw.people[j], pw.people[i]
}
func (pw PersonWrapper) Less(i, j int) bool {    // 重写 Less() 方法
    return pw.by(&pw.people[i], &pw.people[j])
}
 
// 封装成 SortPerson 方法
func SortPerson(people [] Person, by SortBy){
    sort.Sort(PersonWrapper{people, by})
}
 
func main() {
    people := [] Person{
        {"zhang san", 12},
        {"li si", 30},
        {"wang wu", 52},
        {"zhao liu", 26},
    }
 
    fmt.Println(people)
 
    sort.Sort(PersonWrapper{people, func (p, q *Person) bool {
        return q.Age < p.Age    // Age 递减排序
    }})
 
    fmt.Println(people)
 
    SortPerson(people, func (p, q *Person) bool {
        return p.Name < q.Name    // Name 递增排序
    })
 
    fmt.Println(people)
 
}
```
在方法 2 的基础上构造了 SortPerson 函数，使用的时候传过去一个 [] Person 和一个 cmp 函数。

##### 4、另一种思路

```
package main
 
import (
    "fmt"
    "sort"
)
 
type Person struct {
    Name        string
    Weight      int
}
 
type PersonSlice []Person
 
func (s PersonSlice) Len() int  { return len(s) }
func (s PersonSlice) Swap(i, j int)     { s[i], s[j] = s[j], s[i] }
 
type ByName struct{ PersonSlice }    // 将 PersonSlice 包装起来到 ByName 中
 
func (s ByName) Less(i, j int) bool     { return s.PersonSlice[i].Name < s.PersonSlice[j].Name }    // 将 Less 绑定到 ByName 上
 
 
type ByWeight struct{ PersonSlice }    // 将 PersonSlice 包装起来到 ByWeight 中
func (s ByWeight) Less(i, j int) bool   { return s.PersonSlice[i].Weight < s.PersonSlice[j].Weight }    // 将 Less 绑定到 ByWeight 上
 
func main() {
    s := []Person{
        {"apple", 12},
        {"pear", 20},
        {"banana", 50},
        {"orange", 87},
        {"hello", 34},
        {"world", 43},
    }
 
    sort.Sort(ByWeight{s})
    fmt.Println("People by weight:")
    printPeople(s)
 
    sort.Sort(ByName{s})
    fmt.Println("\nPeople by name:")
    printPeople(s)
 
}
 
func printPeople(s []Person) {
    for _, o := range s {
        fmt.Printf("%-8s (%v)\n", o.Name, o.Weight)
    }
}
```
对结构体的排序， 暂时就到这里。 第一种排序对只根据一个字段的比较合适， 另外三个是针对可能根据多个字段排序的。方法 4 我认为每次都要多构造一个 ByXXX ， 颇为不便， 这样多麻烦，不如方法 2 和方法 3 来的方便，直接传进去一个 cmp。 方法2、 3 没有太大的差别， 3 只是简单封装了一下而已， 对于使用者来说， 可能会更方便一些，而且也会更少的出错。


### 取最值

这个取最大值和最小值也是需要自己实现, 引入包`math`
```
func sliceMinMax() {
	intList := []int{2, 4, 3, 5, 7, 6, 9, 8, 1, 0}

	fmt.Println(intList)
	//取最大值
	fmt.Println("maxValue = ", getMaxValue(intList))

	//取最小值
	fmt.Println("minValue = ", getMinValue(intList))

}

func getMaxValue(slice []int) int {
	var maxValue float64 = float64(slice[0])
	for _, v := range slice {
		maxValue = math.Max(float64(maxValue), float64(v))
	}
	return int(maxValue)
}

func getMinValue(slice []int) int {
	var maxValue float64 = float64(slice[0])
	for _, v := range slice {
		maxValue = math.Min(float64(maxValue), float64(v))
	}
	return int(maxValue)
}
```

```
[2 4 3 5 7 6 9 8 1 0]
maxValue =  9
minValue =  0
```





