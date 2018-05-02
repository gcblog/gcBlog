---
title: PhotoBatch-文件重命名(2)
date: 2017-12-04 16:50:20
tags: [macOS, macOSApp, 文件重命名]
categories:
- macOS
---

上一篇文章已经写了如何获取文件夹路径，今天实现photoBatch的第一个简单功能，图片批量重命名，当然也可以对任何文件进行重命名

### 搭建界面

搭了个简单的界面，如下图：
![布局用了purelayout框架](https://raw.githubusercontent.com/suifengqjn/demoimages/master/PhotoBatch/2_1.png)
我们需要得到重命名后的文件名前缀，以及文件格式和是否保留原文件，给这3个变量一个默认值。

### 文件夹处理

获取到文件路径后处理：需要对每个文件的路径进行处理，如果是以
`file://`开头，需要把前面的`file://`去掉，这种地址无法处理。这里是对文件夹路径的处理，处理完的路径再加上文件名就是完整的路径。

<!--more-->

```
- (void)dealFiles:(NSArray *)filepaths
{
    self.dealingLabel.stringValue = [filepaths.firstObject description];
    
    NSMutableArray *arr = [NSMutableArray new];
    // 对文件夹路径进行处理
    for (NSString *path in filepaths) {
        if ([[path description] hasPrefix:@"file:///"]) {
            NSString *newpath = [[path description] substringFromIndex:7];
            if ([newpath hasSuffix:@"/"]) {
                newpath  = [newpath substringToIndex:newpath.length - 1];
            }
            [arr addObject:newpath];
            
        } else {
            if ([[path description] hasSuffix:@"/"]) {
                NSString *tempStr = [path description];
                [arr addObject:[tempStr substringToIndex:tempStr.length - 1]];
            } else {
                [arr addObject:[path description]];
            }
            
        }
    }
    
    self.folderPaths = filepaths;
    
    NSMutableArray *allFiles = [NSMutableArray new];
    for (NSString *docuPath in self.folderPaths) { // 遍历所有文件夹 获取所有文件个数
        NSArray *files = [XCFileManager listFilesInDirectoryAtPath:docuPath deep:NO];//这里遍历得到的只是文件名
        [allFiles addObjectsFromArray:files];
    }

    NSAlert *alert = [[NSAlert alloc] init];
    [alert setMessageText:@"文件获取成功"];
    [alert setInformativeText:[NSString stringWithFormat:@"文件总数：%ld 个", allFiles.count]];
    [alert beginSheetModalForWindow:self.view.window completionHandler:^(NSModalResponse returnCode) {
    }];
}
```

### 文件批量重命名

遍历所有文件夹下所有文件，`NSFileManager` 并没有重命名的方法，如果要保留原文件，则执行`copy`操作，如果不保留原文件，则执行`move`操作。下面是重命名代码的实现。

```
- (IBAction)StartAction:(NSButton *)sender {
    
    
    NSMutableArray *allFiles = [NSMutableArray new];
    for (NSString *docuPath in self.folderPaths) {
        NSArray *files = [XCFileManager listFilesInDirectoryAtPath:docuPath deep:NO];//这里遍历得到的只是文件名
        for (NSString *filename in files) {
            [allFiles addObject:[NSString stringWithFormat:@"%@/%@", docuPath, filename]];
        }
    }
    if (allFiles.count == 0) {
        return;
    }
    NSString *resultFilePath = [NSString stringWithFormat:@"%@/%@", self.folderPaths.firstObject, @"result"];
    
    NSError *err = nil;
    [XCFileManager createDirectoryAtPath:resultFilePath error:&err];
    NSString *prefixName = _reNameView.prefixInput.stringValue;
    if (!prefixName || prefixName.length == 0) {
        prefixName = @"img_";
    }
    NSString *suffixName = _reNameView.suffixInput.stringValue;
    if(!suffixName || suffixName.length == 0) {
        suffixName = @"";
    }
    NSInteger index = 1;
    NSString *suffix = @"";
    for (NSString *path in allFiles) {
//        // 如果遇到 没有文件名的文件，直接过滤
        if ([path componentsSeparatedByString:@"."].count < 2) {
            continue;
        }
        if (suffixName.length == 0) {
            suffix = [[path componentsSeparatedByString:@"."].lastObject description];
        } else {
            suffix = suffixName;
        }
        self.dealingLabel.stringValue = [path description];
        
        NSString *movePath = [NSString stringWithFormat:@"%@/%@%ld.%@", resultFilePath, prefixName, index,suffix];
        if (_reNameView.checkSaveBtn.state == 1) {
           [XCFileManager moveItemAtPath:path toPath:movePath overwrite:NO];
        } else {
            [XCFileManager moveItemAtPath:path toPath:movePath overwrite:YES];
        }
        
        index ++;
    }
    
    self.dealingLabel.stringValue = @"处理完成";
    
}
```
[demo地址:https://github.com/macOSApp/photoBatch](https://github.com/macOSApp/photoBatch)


