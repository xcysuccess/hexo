---
layout: post
title: "iOS中的SVG与iconFont详解"
date: 2015-10-07 16:52:41 +0800
comments: true
categories: font 
description: "iOS中的SVG与iconFont详解" 
keywords: iconfont,svg 
---

## 一. SVG

### 1. 概念：
SVG 是使用 XML 来描述二维图形和绘图程序的语言。

### 2. 什么是SVG？
1. SVG 指可伸缩矢量图形 (Scalable Vector Graphics)  
2. SVG 用来定义用于网络的基于矢量的图形  
3. SVG 使用 XML 格式定义图形  
4. SVG 图像在放大或改变尺寸的情况下其图形质量不会有所损失  
5. SVG 是万维网联盟的标准  
6. SVG 与诸如 DOM 和 XSL 之类的 W3C 标准是一个整体  

### 3. 优势
1. SVG 可被非常多的工具读取和修改（比如记事本）<!-- more -->  
2. SVG 与 JPEG 和 GIF 图像比起来，尺寸更小，且可压缩性更强。  
3. SVG 是可伸缩的  
4. SVG 图像可在任何的分辨率下被高质量地打印  
5. SVG 可在图像质量不下降的情况下被放大  
6. SVG 图像中的文本是可选的，同时也是可搜索的（很适合制作地图）  
7. SVG 可以与 Java 技术一起运行  
8. SVG 是开放的标准  
9. SVG 文件是纯粹的 XML  

### 4. 查看 SVG 文件
今天，所有浏览器均支持 SVG 文件，不过需要安装插件的 Internet Explorer 除外。插件是免费的，比如 Adobe SVG Viewer  

### 5. svg实战篇
1.解码  

```objc
NSString *mySVGString = [[NSString alloc] initWithContentsOfFile:pathOfSVGFile encoding:NSStringEncodingConversionExternalRepresentation error:&error];  
mySVGString的值:  

<?xml version="1.0" encoding="utf-8"?>
<!-- Generator: Adobe Illustrator 15.0.0, SVG Export Plug-In . SVG Version: 6.00 Build 0)  -->
<!DOCTYPE svg PUBLIC "-//W3C//DTD SVG 1.1//EN" "http://www.w3.org/Graphics/SVG/1.1/DTD/svg11.dtd">
<svg version="1.1" id="Layer_1" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink" x="0px" y="0px"
 width="1024px" height="768px" viewBox="0 0 1024 768" enable-background="new 0 0 1024 768" xml:space="preserve">
<path fill="#FFFFFF" stroke="#000000" stroke-width="5" stroke-miterlimit="10" d="M176.17,369.617
c0,0,335.106-189.361,214.894,38.298c-120.212,227.659,129.787,282.978,178.723,42.553C618.724,210.042,834.681,87.702,790,307.915"
/>
</svg>
```

SVG 路径 - `path`  
<path> 元素用于定义一个路径。  
下面的命令可用于路径数据： 

```objc
* M = moveto  
* L = lineto  
* H = horizontal lineto  
* V = vertical lineto  
* C = curveto  
* S = smooth curveto  
* Q = quadratic Bézier curve  
* T = smooth quadratic Bézier curveto  
* A = elliptical Arc  
* Z = closepath  
 ```
 
注意：以上所有命令均允许小写字母。大写表示绝对定位，小写表示相对定位。  
更多请参考请看**[SVG关键字说明](http://www.runoob.com/svg/svg-path.html)**  

### 6. 第三方库
既然有这么多关键字，我们自己写dom解析的工作量太大，故而思路转向去寻找第三方开源库  

**[Github-SVG解析库](https://github.com/SVGKit/SVGKit)**  
**Framework安装包大小:`6.4MB`**

## 二. iconFont

### 1. 功能介绍
在网页的世界里，图标字体的应用日趋广泛，国内以阿里系的产品为应用典型，他们更是参考谷歌搭建了国内首个图标字体平台（[http://www.iconfont.cn](http://www.iconfont.cn)），除了提供在线图标字体制作、本地下载之外，还支持CDN在线存储与使用。  

### 2. 制作好之后，我们把ttf加载到工程
修改`Supporting Files/Info.plist`文件，增加`Fonts provided by application`键值,把文件名填入。比如`Item0`对应的是`fonthello.ttf`

### 3. 如果想转换成UIImageview

```objc
 UIGraphicsBeginImageContextWithOptions(size, NO, 0.0f);
[textContent drawInRect:textRect withAttributes:@{NSFontAttributeName : font,NSForegroundColorAttributeName : iconColor,NSBackgroundColorAttributeName : bgColor}];
//Image returns
UIImage * image = UIGraphicsGetImageFromCurrentImageContext();
UIGraphicsEndImageContext();
```

此时我们可以得到UIImage，即UIImageview  

### 4. 可能某时我们想要圆角效果
这里有四种方法[改变成圆角](http://blog.csdn.net/u012282115/article/details/39007019),实践得出结论，1，4的方法耗时严重。第2个方法过于死板，不适合任意角度设置。第三个方法是比较好的，而且最贵的的是可以在*异步*执行  

```objc
- (UIImage*)iconImageWithWidth:(UIImage*) image width:(double)width cornerRadius:(double)radius
{
    UIGraphicsBeginImageContextWithOptions(CGSizeMake(width, width), NO, 0);
    CGContextRef context = UIGraphicsGetCurrentContext();
    
    CGSize imgSize = image.size;
    CGFloat mid = width / 2;
    
    CGContextBeginPath(context);
    
    CGContextMoveToPoint(context, 0, mid);
    CGContextAddArcToPoint(context, 0, 0, mid, 0, radius);
    CGContextAddArcToPoint(context, width, 0, width, mid, radius);
    CGContextAddArcToPoint(context, width, width, mid, width, radius);
    CGContextAddArcToPoint(context, 0, width, 0, mid, radius);
    CGContextClosePath(context);                                              //4
    
    CGContextClip(context);                                                    //5
    
    if (imgSize.width > imgSize.height) {
        double rate = imgSize.width / imgSize.height;
        //CGContextDrawImage(context, CGRectMake(floor(-(rate-1)/2*width), 0, floor(rate*width), floor(width)), self.CGImage);
        [image drawInRect:CGRectMake(floor(-(rate-1)/2*width), 0, floor(rate*width), floor(width))];
    } else {
        double rate = imgSize.height / imgSize.width;
        //CGContextDrawImage(context, CGRectMake(0, floor(-(rate-1)/2*width), floor(width), floor(rate*width)), self.CGImage);
        [image drawInRect:CGRectMake(0, floor(-(rate-1)/2*width), floor(width), floor(rate*width))];
    }
    
    UIImage* img = UIGraphicsGetImageFromCurrentImageContext();
    UIGraphicsEndImageContext();
    
    return img;
}
```

### 5. 大小性能
1. 关于大小
	之前网上有一个人做了测试，同时存在2x和3x图片的情况下  
	![iconfont大小]( /images/iconfont/1.png "iconfont大小")  
	这个根据自己项目而定，但是也能极大程度去优化了  
	  
2. 性能  
	加载速度非常快，可以忽略不计,*iphone5手机*平均加载一个在5ms左右