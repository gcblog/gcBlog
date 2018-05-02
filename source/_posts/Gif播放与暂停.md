---
title: Gif播放与暂停
date: 2017-11-14 10:13:08
categories:
  - Swfit
tags:
	- Swift
	- Gif
	- 动画
	- runloop
	- CADisplayLink
---

## gif播放的两种方式
### UIWebView

这应该是播放gif文件最简单的方式了，缺点：无法暂停播放

```
//1. 把gif文件 转化成 data
guard let dataPath = Bundle.main.path(forResource: "demo", ofType: "gif") else { return }
guard let gifData = NSData(contentsOfFile: dataPath) else { return }

//2. 给UIWebView 设置data
let webview = UIWebView()
webview.frame = CGRect(x: 0, y: 300, width: self.view.frame.size.width, height: 200)
webview.scalesPageToFit = true
webview.load(gifData as Data, mimeType: "image/gif", textEncodingName: String(), baseURL: NSURL() as URL)
self.view.addSubview(webview)
```
<!--more-->
### UIImageView

这里面有两个要点，一是从gif文件中获取图片数组，二是获取gif文件播放时长

```
let imageView = UIImageView()
imageView.frame = CGRect(x: 0, y: 50, width: self.view.frame.size.width, height: 200)
self.view.addSubview(imageView)

// 1. 把gif文件 转化成 data
guard let dataPath = Bundle.main.path(forResource: "demo", ofType: "gif") else { return }
guard let gifData = NSData(contentsOfFile: dataPath) else { return }

// 2. 把data 转换成CGImageSource 对象
guard let imageSource = CGImageSourceCreateWithData(gifData, nil) else { return }
// 2.1 获取图片的个数
let imageCount = CGImageSourceGetCount(imageSource)

var images = [UIImage]()
var gifDuration : TimeInterval = 0
// 3. 遍历所有的图片
for i in 0..<imageCount {
    // 3.1 取出图片
   guard let cgimage = CGImageSourceCreateImageAtIndex(imageSource, i, nil)  else { return }

    let image = UIImage(cgImage: cgimage)
    images.append(image)
    if (i == 0) {
        imageView.image = image
    }
    // 3.1 取出每张图片持续的时间
    guard let property = CGImageSourceCopyPropertiesAtIndex(imageSource, i, nil) as? NSDictionary else { continue }
    guard let gifDic = property[kCGImagePropertyGIFDictionary] as? NSDictionary else { continue }
    guard let imageDuration = gifDic[kCGImagePropertyGIFDelayTime] as? NSNumber else { continue }
    gifDuration += imageDuration.doubleValue

}

// 4.设置images属性
imageView.animationImages = images
imageView.animationDuration = gifDuration
imageView.animationRepeatCount = 1

// 5. 开始播放
imageView.startAnimating()
```

## Gif暂停实现

