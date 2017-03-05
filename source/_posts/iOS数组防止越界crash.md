---
title: iOS数组防止越界crash
date: 2016-12-24 15:45:55
category:
	- iOS技术
tags: [iOS, 越界, NSArray, NSMutableArray, crash]
---


有时候项目中总是出现一些无法预知的情况，导致数组越界是程序crash，如果这种意外情况无法避免，那么只能从侧面采取保护措施。我先从网上找答案，我想其他人也肯定遇到过相同的情况，如果有好的解决方案，直接采用就可以了。但是实际上，网上搜索的结果令人有些失望。下面还是记录一下我自己的解决方案，以及和网上解决方案的差异。

### crash的具体几种情况
- 取值：index超出array的索引范围
- 添加：插入的object为nil或者Null
- 插入：index大于count、插入的object为nil或者Null
- 删除：index超出array的索引范围
- 替换：index超出array的索引范围、替换的object为nil或者Null

### 解决思路
任何代码都需要围绕"高内聚，低耦合"的思想来实现，尤其是这种工具类的代码，更是应该对原代码入侵越少越好。一个很容易想到的方法，就是采用runtime, 把array中的以上几种情况的方法替换成自己的方法，然后再执行方法的时候加以判断。而我在网上搜到的结果全是以这种方案解决的，不排除有更好的方法我没找到。附上一个我找到的代码比较详细的[demo](https://github.com/wanyakun/YKIntercepter)。我试了一下，效果是可以达到，不过我还是毫不犹豫的拒绝这种方式。直接替换了系统的方法必然会导致更多无法预知的问题。这些问题，我在后面会讲几个我遇到的。而我准备这样解决：
<!--more-->
- 这是系统原本的调用方式
![这是系统原本的调用方式](https://raw.githubusercontent.com/suifengqjn/demoimages/master/iOS%E9%98%B2%E6%AD%A2%E6%95%B0%E6%8D%AE%E8%B6%8A%E7%95%8Ccrash/12.png)

- 这是改变之后的调用方式
![这是改变之后的调用方式](https://raw.githubusercontent.com/suifengqjn/demoimages/master/iOS%E9%98%B2%E6%AD%A2%E6%95%B0%E6%8D%AE%E8%B6%8A%E7%95%8Ccrash/13.png)

我是先勾住array自带的方法，进行判断，如果没有越界等几种情况，再继续执行它自身的方法，相当于在执行方法前多了一步判断，而网上是直接把方法替换成自己的方法了，这里还是有本质的区别。

### 具体实现原理
这里举例说明 `NSArray` 的 `addObject:` 方法，其他也类似。
#### 先定义一个静态变量
`static IMP array_old_func_imap_object = NULL;`
这个变量用来记录array自带方法的指针地址
#### 获取方法，然后记录方法的指针地址
```
Method old_func_imap_object = class_getInstanceMethod(NSClassFromString(@"__NSArrayI"), @selector(objectAtIndex:));
            array_old_func_imap_object = method_getImplementation(old_func_imap_object);
```
#### 改变原方法的指针地址，并指向自定义方法
`method_setImplementation(old_func_imap_object, [self methodForSelector:@selector(fm_objectAtIndex:)]);`
#### 自定义方法的实现
```
- (id)fm_objectAtIndex:(NSUInteger)index {
    if (index < [(NSArray*)self count]) {
        return ((id(*)(id, SEL, NSUInteger))array_old_func_imap_object)(self, @selector(objectAtIndex:), index);
    }
    NSLog(@"NArray objectAtIndex 失败--%@", [NSThread callStackSymbols]);
    return nil;
}
```
#### 最后一步
到这里已经差不多完成了，就剩最后一个问题了，就是怎么运用到项目中，让这个工具类继承自NSObject，把这个工具类写成一个单例，然后在load方法中调用单例。load 方法会在本类第一次使用的时候调用一次，所以，把这个工具类拖到项目中，不用写其他代码，就实现了以上的功能。
```
+ (void)load {
    [FMDetecter sharedInstance];
}

static dispatch_once_t onceToken;
static FMDetecter *sharedInstance;

+ (instancetype)sharedInstance {
    dispatch_once(&onceToken, ^{
        sharedInstance = [[FMDetecter alloc] init];
    });
    return sharedInstance;
}
```
这里有完整的代码，有兴趣可查看[demo](https://github.com/suifengqjn/FMArrarMonitor)

### 实际出现的问题
我用这两种方式都试了试，新建一个空项目，然后把上面几个方法都试一遍，似乎都没问题，然后我把他们公司的项目中，程序有时候卡死，还会crash，还是没法用，两种方式都有问题，找了找原因，发现NSArray和NSMutableArray的那几个方法，系统自己会调用很多很多次，极大的影响了性能，还有网友遇到了其他的问题：替换了objectAtIndex方法有输入的地方出来了软键盘按手机Home键就Crash了。简直无解，最后，还是决定写个分类，虽然low一点，毕竟还是能解决我的问题，并且不会带来新的问题。

#### 这是给NSArray添加的方法 

```
#import "NSArray+beyond.h"

@implementation NSArray (beyond)
-(id)objectAtIndexCheck:(NSUInteger)index
{
    if (index < self.count) {
        return [self objectAtIndex:index];
    }
    return nil;
}
@end
```

#### 这是给NSMutableArray添加的方法

```
#import "NSMutableArray+beyond.h"

@implementation NSMutableArray (beyond)
-(id)objectAtIndexCheck:(NSUInteger)index
{
    if (index < self.count) {
        return [self objectAtIndex:index];
    }
    NSLog(@"%@", [NSThread callStackSymbols]);
    return nil;
}
- (void)addObjectCheck:(id)anObject
{
    if (anObject != nil && [anObject isKindOfClass:[NSNull class]] == NO) {
        [self addObject:anObject];
    } else {
        NSLog(@"%@", [NSThread callStackSymbols]);
        
    }
}
- (void)insertObjectCheck:(id)anObject atIndex:(NSUInteger)index
{
    if (index <= self.count && anObject != nil && [anObject isKindOfClass:[NSNull class]] == NO) {
        [self insertObject:anObject atIndex:index];
    } else {
        NSLog(@"%@", [NSThread callStackSymbols]);
        
    }
}

- (void)removeObjectAtIndexCheck:(NSUInteger)index
{
    if (index < self.count) {
        [self removeObjectAtIndex:index];
    } else {
        NSLog(@"%@", [NSThread callStackSymbols]);
        
    }
}
- (void)replaceObjectAtIndexCheck:(NSUInteger)index withObject:(id)anObject
{
    if (index < self.count && anObject != nil && [anObject isKindOfClass:[NSNull class]] == NO) {
        [self replaceObjectAtIndex:index withObject:anObject];
    } else {
        NSLog(@"%@", [NSThread callStackSymbols]);
        
    }
}
@end
```


