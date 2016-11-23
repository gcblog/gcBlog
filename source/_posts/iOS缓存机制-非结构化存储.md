---
title: iOS缓存机制-非结构化存储
date: 2016-03-12 19:43:33
tags: [iOS, 缓存, 性能]
categories: 
- iOS技术
---

对于一个优秀的app来说，缓存机制必不可少。图片，视频，音频等等各种类型的文件，怎么样去更好的管理这些数据，这对于我们开发者以及用户都是息息相关的。闲话不多说，先来看看几个github开源中牛逼的几个缓存框架。

它们的使用方式都很类似，都是通过键值对(key-value)的形式进行存取，跟`NSUserDefaults`用法类似。

以下排名按照性能由低到高：
### 1.EGOCache
 
 只提供磁盘缓存，没有内存缓存，一个比较简易的缓存框架。

### 2.TMCache
 
 最初由 Tumblr 开发，但现在已经不再维护了。TMMemoryCache 实现有很多 NSCache 并没有提供的功能，比如数量限制、总容量限制、存活时间限制、内存警告或应用退到后台时清空缓存等。TMMemoryCache 在设计时，主要目标是线程安全，它把所有读写操作都放到了同一个 concurrent queue 中，然后用 dispatch_barrier_async 来保证任务能顺序执行。它错误的用了大量异步 block 回调来实现存取功能，以至于产生了很大的性能和死锁问题。它有磁盘缓存和内存缓存两部分组成，个人感觉一般好点的缓存库都会由磁盘缓存和内存缓存两部分组成，在我们第一次存储一个文件到磁盘的时候，缓存库会帮我们复制一份到内存缓存中，可以让我们在下次使用该文件的时候可以更快的找到。我上家公司就是用的这个缓存框架，挺好用的，也没出什么问题。
 
### 3.PINCache 
 
 是 Tumblr 宣布不在维护 TMCache 后，由 Pinterest 维护和改进的一个内存缓存。它的功能和接口基本和 TMCache 一样，但修复了性能和死锁的问题。它同样也用 dispatch_semaphore 来保证线程安全，但去掉了dispatch_barrier_async，避免了线程切换带来的巨大开销，也避免了可能的死锁。相当于是TMCache的优化版，而且一直有更新。
 
<!-- more -->
 
### 4.YYCache
 
 YYMemoryCache相对于 PINMemoryCache 来说，去掉了异步访问的接口，尽量优化了同步访问的性能，用 OSSpinLock 来保证线程安全。另外，缓存内部用双向链表和 NSDictionary 实现了 LRU 淘汰算法,性能好一点。TMDiskCache, PINDiskCache, SDWebImage 等缓存，都是基于文件系统的，即一个 Value 对应一个文件，通过文件读写来缓存数据。他们的实现都比较简单，性能也都相近，缺点也是同样的：不方便扩展、没有元数据、难以实现较好的淘汰算法、数据统计缓慢。YYDiskCache 也是采用的 SQLite 配合文件的存储方式，在 iPhone 6 64G 上的性能基准测试结果见下图。在存取小数据 (NSNumber) 时，YYDiskCache 的性能远远高出基于文件存储的库；而较大数据的存取性能则比较接近了。但得益于 SQLite 存储的元数据，YYDiskCache 实现了 LRU 淘汰算法、更快的数据统计，更多的容量控制选项。
 
 ![image](https://raw.githubusercontent.com/suifengqjn/demoimages/master/KVCahce/1.png)
 
 YYCache相比较与前面几个，性能是最好的一个，但是差距也不是太大，而且他的使用需要依赖库，个人感觉用PinCache就可以了。
 下面是我通过PinCache改进的KVCache，在PinCahce的基础上增加了以下几个功能。
 
 ```
 1. 可以设置缓存上限，设置一定时间内定时清理缓存
 2. 增加缓存类型，jpg，MP4，MP3，gif，等自动保存相应类型
 3. 增加缓存区块，不同模块的文件可以缓存到各自对应的区域
 4. 以上所有参数都可以根据自己的需求自行设置
 ```
 [demo地址](https://github.com/suifengqjn/KVCache)
 
 [github Blog同步更新](http://gcblog.gtihub.io)
 


