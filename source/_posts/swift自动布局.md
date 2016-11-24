---
title: swift自动布局
date: 2016-11-24 23:48:09
categories:
  - swfit
tags:
  - swift
  - 自动布局
---

### 苹果原生自动布局

- 自动布局核心公式

` view1.attr1 = view2.attr2 * multiplier + constant`

- 自动布局构造函数

``` 
NSLayoutConstraint(item: 视图,
	attribute: 约束属性，
	relatedBy: 约束关系，
	toItem: 参照视图,
	attribute: 参照属性,
	multiplier:乘积,
	constant:约束数值
)

```
<!--more-->

- 如果指定 宽 高 约束
  - 参照视图设置为 nil
  - 参照属性选择 .notAnAttribute
  
- 自动布局类函数

```
NSLayoutConstraint.constraints(withVisualFormat: VLF公式,
 options:[], 
 metrics: 约束数值字典[String : 数值], 
 views: 视图字典[String : 子视图]
 )
```

- VFL 可视化格式化语言
  - H 水平方向
  - V 垂直方向
  - | 边界
  - [] 包含控件的名称字符串，对应关系在`views`字典中定义
  - () 定义控件的宽/高，可以在`metrics`中指定
>tip：VFL 通常用于连续参照关系，如果遇到居中对齐，通常使用直接参照

