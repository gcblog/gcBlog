---
title: iOS公共库
tags:
  - iOS
  - 公共库
  - workspace
categories:
  - iOS技术
author:
  - name: 千寻墨
    url: 'http://www.jianshu.com/users/527ecf8c8753/latest_articles'
date: 2016-04-06 16:59:52
---


### 第一步

>打开Xcode，file->new->WorkSpace,我们使用WorkSpace来管理工程和依赖库

![image](https://raw.githubusercontent.com/suifengqjn/demoimages/master/%E5%85%AC%E5%85%B1%E4%BE%9D%E8%B5%96%E5%BA%93/1.png)

<!-- more -->

### 第二步

>file->new->project->Application,创建我们的工程，创建好后关闭工程

![image](https://raw.githubusercontent.com/suifengqjn/demoimages/master/%E5%85%AC%E5%85%B1%E4%BE%9D%E8%B5%96%E5%BA%93/2.png)

### 第三步

>file->new->project->Framework&Library->Cocoa Touch Framework,创建依赖库

![image](https://raw.githubusercontent.com/suifengqjn/demoimages/master/%E5%85%AC%E5%85%B1%E4%BE%9D%E8%B5%96%E5%BA%93/3.png)

### 第四步

>新建一个Tool类，然后在Amy.h里面引入，引入的方式：`#import<Amy/Tool.h>`

![image](https://raw.githubusercontent.com/suifengqjn/demoimages/master/%E5%85%AC%E5%85%B1%E4%BE%9D%E8%B5%96%E5%BA%93/4.png)

### 第五步

>回到我们的workspace目录，workspace，依赖库和工程最好放到同一目录下面。

![image](https://raw.githubusercontent.com/suifengqjn/demoimages/master/%E5%85%AC%E5%85%B1%E4%BE%9D%E8%B5%96%E5%BA%93/5.png)

### 第六步

>打开workspace，通过 `file-> AddFile to` 依次把工程和依赖添加到workspace中。

![image](https://raw.githubusercontent.com/suifengqjn/demoimages/master/%E5%85%AC%E5%85%B1%E4%BE%9D%E8%B5%96%E5%BA%93/6.png)

### 第七步

>在workspace中选中工程，然后点击Build Phases，选择 link Binary With Libraries 点击 + 号，添加Amy.framework

![image](https://raw.githubusercontent.com/suifengqjn/demoimages/master/%E5%85%AC%E5%85%B1%E4%BE%9D%E8%B5%96%E5%BA%93/7.png)

![image](https://raw.githubusercontent.com/suifengqjn/demoimages/master/%E5%85%AC%E5%85%B1%E4%BE%9D%E8%B5%96%E5%BA%93/8.png)
	
### 第八步

>在我们的工程中可以引入依赖使用了。
引入方式：`#import<Amy/Tool.h>`
![image](https://raw.githubusercontent.com/suifengqjn/demoimages/master/%E5%85%AC%E5%85%B1%E4%BE%9D%E8%B5%96%E5%BA%93/9.png)



