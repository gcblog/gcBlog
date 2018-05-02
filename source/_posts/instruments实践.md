---
title: instruments实践
date: 2017-01-01 17:24:39
tags: [iOS, instruments, crash, 性能]
categories: 
- iOS技术
---

前段时间写项目，突然跳到某一个页面crash了，然后我重新又进来几次，然后又没问题了，以为这是个"意外"，也没在意，一段时间后，又发生了一次crash，还是同一个页面，我意识到这不是偶然了，然后开始找原因。关于instruments的使用也看过很多次了，但是一直没怎么用，正好这次用它解决了一个问题，顺便记录一下。在猜测某个页面有问题的情况下，我一般的思路是这样的：

### 第一步：查看dealloc
看这个页面有没有被释放，在dealloc打上断点，发现页面确实已经释放了，排除这个可能，进行下一步。
<!--more-->
### 第二步：查看内存,cpu
![查看运行状态的内存cpu](https://raw.githubusercontent.com/suifengqjn/demoimages/master/instruments/1.png)

我先后两次进入到这个页面，发现刚进入这个页面的瞬间内存飚的很高，然后瞬间又降下来，到这里问题已经变得明显了，这个页面确实存在问题，下面就需要深入的寻找问题所在。

### 第三步：Profile
直接在上图的右上角点击 Profile in instruments,
![查看运行状态的内存cpu](https://raw.githubusercontent.com/suifengqjn/demoimages/master/instruments/2.png)
进入instruments后，我还是重复之前的操作，进入这个问题页面，还是发现了内存飙高的情况，然后我在下面的堆栈里找，发现怎么也找不到到底是哪个方法引起的。
![查看运行状态的内存cpu](https://raw.githubusercontent.com/suifengqjn/demoimages/master/instruments/3.png)
然后我发现上面的内存图上有条线可以拖动，然后我就把它拖到内存最高的地方，然后再去下面找，发现还是找不到。
![查看运行状态的内存cpu](https://raw.githubusercontent.com/suifengqjn/demoimages/master/instruments/4.png)
最后，我尝试在内存飙高的同时，点击左上角暂停运行，然后再下面的堆栈中继续找，这回终于找到了，是我自己的一个类渲染UI的时候导致的内存问题，找到具体的原因，解决起来也就轻松了。光看果然是没有用的，重要的还是要实践。

leak 右侧几个属性选项
![几个属性含义](https://raw.githubusercontent.com/suifengqjn/demoimages/master/instruments/7.png)

Separate by Thread（建议选择）   按照线程分类查看哪些占用cpu最多
Invert Call Tree（不建议选择）：调用树倒返过来，将习惯性的从根向下一级一级的显示，如选上就会返过来从最底层调用向一级一级的显示。如果想要查看那个方法调用为最深时使用会更方便些。
Hidden System Librares (建议选择)   隐藏系统类库方法
Flatten Recursion （一般不选）：选上它会将调用栈里递归函数作为一个入口。
Hide Missing Symbols（建议选择）：隐藏丢失的符号，比如应用或者系统的dSYM文件找不到的话，在详情面板上是看不到方法名的，只能看一些读不明的十六进值，所以对我们来说是没有意义的，去掉了会使阅读更清楚些。


