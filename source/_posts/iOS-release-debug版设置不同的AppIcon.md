---
title: iOS release,debug版设置不同的AppIcon
tags:
  - iOS
  - 配置
categories:
  - iOS配置
author:
  - name: 千寻墨
    url: 'http://www.jianshu.com/users/527ecf8c8753/latest_articles'
date: 2016-04-13 18:41:56
---

也许你也遇到过这种情况，产品经理或者测试让你给装个测试包，一会装正式环境的，一会又装测试环境的，一会又装个灰度环境的。弄来弄去的有时候自己都搞不清楚测试包是什么环境下的。

通过判断debug还是release环境，我们可以用很多种方法来区分这两种环境，这里最简单而且效果非常好的一种方法。不同的环境下采用不同的图标，这样软件一安装上就可以非常明显的分辨出来。

#### 第一步

创建一个新的AppIcon

![image](https://raw.githubusercontent.com/suifengqjn/demoimages/master/AppIcon/1.png)

<!-- more -->

#### 第二步

给两个AppIcon分别加入不同的图片

![image](https://raw.githubusercontent.com/suifengqjn/demoimages/master/AppIcon/2.png)


#### 第三步

在build Setting 里面搜索 icon，找到Asset catalog AppIcon set name

![image](https://raw.githubusercontent.com/suifengqjn/demoimages/master/AppIcon/3.png)

#### 第四步

分别给debug和release设置不同的AppIcon。到这里就全部完成了，快去试试吧。
![image](https://raw.githubusercontent.com/suifengqjn/demoimages/master/AppIcon/4.png)

