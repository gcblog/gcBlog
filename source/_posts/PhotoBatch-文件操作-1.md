---
title: PhotoBatch-文件操作(1)
date: 2017-11-26 16:02:41
tags: [macOS, macOSApp, 文件处理]
categories:
- macOS
---

最近开始自学mac app开发，网上资料很少，大致看了一下官方文档，mac开发主要框架就是AppKit，我有几年iOS的开发经验，在加上官方文档和网上一些零碎的资料，慢慢学习应该是问题不大。准备自己动手制作一个批量图片处理软件，记录一下自己的学习过程，一方面帮助自己对学到知识的整理，也可以给学习macOS的人一些参考。

### 文件拖拽

* 需要实现的效果：将文件或者文件夹拖到到我们的app内，获得其绝对路径

#### 新建 macOS 工程

跟新建iOS项目工程几乎一致。

![新建工程](https://raw.githubusercontent.com/suifengqjn/demoimages/master/PhotoBatch/1.png)
<!--more-->
#### 自定义PBDragView

在iOS中，最核心的的框架就是`Foundation`和`UIKit`, 在macOS中，就是`Foundation`和`AppKit`, 对于iOS中大部分控件，都是把前缀又`UI`换成了`NS`, 他们看上去很类似，但是使用的时候在很多细节上却又大不相同，这里推荐一篇博客[从 UIKit 到 AppKit(https://www.objccn.io/issue-14-5)](https://www.objccn.io/issue-14-5), 讲述了这两个框架的一些异同。

我们需要自定义一个 `PBDragView` 继承自`NSView`，然后当有文件或者文件夹拖动到这个View中的时候，在内部实现文件拖入拖出等方法。

#### 注册支持的文件类型

```
- (void)awakeFromNib
{
    [super awakeFromNib];
    // 设置支持的文件类型
    [self registerForDraggedTypes:@[NSPasteboardTypePDF, NSPasteboardTypePNG, NSPasteboardTypeURL, NSPasteboardTypeFileURL]];
}
``` 

#### 实现文件拖动的几个方法

```

- (NSDragOperation)draggingEntered:(id<NSDraggingInfo>)sender
{
    if (self.delegate && [self.delegate respondsToSelector:@selector(dragEnter)]) {
        [self.delegate dragEnter];
    }
    
    return NSDragOperationGeneric;
}

- (void)draggingExited:(id<NSDraggingInfo>)sender
{
    if (self.delegate && [self.delegate respondsToSelector:@selector(dragExit)]) {
        [self.delegate dragExit];
    }
}

- (BOOL)performDragOperation:(id<NSDraggingInfo>)sender
{
    // 获取所有的路径
    NSArray *arr =  [[sender draggingPasteboard] propertyListForType:NSFilenamesPboardType];
    if (self.delegate && arr.count > 0 && [self.delegate respondsToSelector:@selector(dragFileComplete:)]) {
        [self.delegate dragFileComplete:arr];
    }
    return YES;
}
```
#### 在SB中使用PBDragView

![sb的效果图](https://raw.githubusercontent.com/suifengqjn/demoimages/master/PhotoBatch/2.png)

运行程序，将文件或者文件夹拖入整个app界面，就可以获取到所有的文件路径。


### 文件选择

* 需要实现的效果：点击按钮，弹出文件选择框，可以选择文件或者文件夹

#### 添加点击按钮

在SB中添加按钮，然后添加点击事件
![添加按钮](https://raw.githubusercontent.com/suifengqjn/demoimages/master/PhotoBatch/3.png)


#### 文件选择功能实现

```
    NSOpenPanel *openPanel = [NSOpenPanel openPanel];
    [openPanel setPrompt: @"打开"];
    [openPanel setCanChooseDirectories:YES]; //设置允许打开文件夹
    [openPanel setAllowsMultipleSelection:YES]; // 会否允许打开多个目录
    [openPanel setCanChooseFiles:YES];  //设置允许打开文件
    [openPanel setCanCreateDirectories:YES]; // 允许新建文件夹
    [openPanel setCanDownloadUbiquitousContents:NO]; //是否处理还未下载成功的文档
    [openPanel setCanResolveUbiquitousConflicts:NO]; //是否处理有冲突的文档
    openPanel.allowedFileTypes = [NSArray arrayWithObjects: @"jpg", @"doc",@"txt",@"jpeg",@"png",@"tiff", nil]; //设置允许打开的文件类型
    [openPanel beginSheetModalForWindow:[NSApplication sharedApplication].keyWindow completionHandler:^(NSModalResponse result) {
        
        NSArray *filePaths = [openPanel URLs];
        NSLog(@"-----%@", filePaths);
        
    }];
```
效果图：
![弹出系统的文件选择框](https://raw.githubusercontent.com/suifengqjn/demoimages/master/PhotoBatch/4.png)

[demo地址:https://github.com/macOSApp/photoBatch](https://github.com/macOSApp/photoBatch)


