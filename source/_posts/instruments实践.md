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


