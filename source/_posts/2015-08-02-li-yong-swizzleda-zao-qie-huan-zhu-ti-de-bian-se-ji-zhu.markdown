---
layout: post
title: "利用swizzle打造切换主题的变色技术"
date: 2015-08-02 21:46:10 +0800
comments: true
categories: Technic 
description: "在写app的时候，我们可能经常需要皮肤换色的功能，在这里给大家提供一个解耦合的思路，大家有更好的思路欢迎给我留言。" 
keywords: swizzle,主题,变色
---

## 一、swizzle
简单来说，swizzle有如下几个常用的方法  

### 1. 给系统类增加成员变量

```objc
#import <UIKit/UIKit.h>
#import <objc/runtime.h>
@interface UILabel (Associate)
- (nonatomic, strong) UIColor *FlashColor;
@end

#import "UILabel+Associate.h"
@implementation UILabel (Associate)
	
static char flashColorKey;
- (void) setFlashColor:(UIColor *) flashColor{
    objc_setAssociatedObject(self, &flashColorKey, flashColor, OBJC_ASSOCIATION_RETAIN_NONATOMIC);
}
- (UIColor *) getFlashColor{
   return objc_getAssociatedObject(self, &flashColorKey);
}
@end
```
<!-- more -->

### 2. 调换IMP

```objc
+ (void)swizzleInstanceMethod:(Class)class originSelector:	(SEL)originSelector otherSelector:(SEL)otherSelector
{
 	Method otherMehtod = class_getInstanceMethod(class, otherSelector);
Method originMehtod = class_getInstanceMethod(class, originSelector);
// 交换2个方法的实现
method_exchangeImplementations(otherMehtod, originMehtod);
}
```

### 3. 动态增加方法  

```objc
#if TARGET_IPHONE_SIMULATOR
#import <objc/objc-runtime.h>
#else
#import <objc/runtime.h>
#import <objc/message.h>
#endif
 
@interface EmptyClass:NSObject
@end
 
@implementation EmptyClass
@end
 
void sayHello(id self, SEL _cmd) {
    NSLog(@"Hello");
}
 
- (void)addMethod {
    class_addMethod([EmptyClass class], @selector(sayHello2), (IMP)sayHello, "v@:");
 
    // Test Method
    EmptyClass *instance = [[EmptyClass alloc] init];
    [instance sayHello2];
}
```

其中types参数为"i@:@“，按顺序分别表示:  
  
```objc
i：返回值类型int，若是v则表示void  
@：参数id(self)  
:：SEL(_cmd)  
@：id(str)
```
这些表示方法都是定义好的(Type Encodings)，关于Type Encodings的其他类型定义请参考[官方文档](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/ObjCRuntimeGuide/Articles/ocrtTypeEncodings.html)

### 4. 获取某个类的成员变量或属性

```objc
unsigned int numIvars; //成员变量个数
Ivar *vars = class_copyIvarList(NSClassFromString(@"UIView"), &numIvars);
//Ivar *vars = class_copyIvarList([UIView class], &numIvars);
    
NSString *key=nil;
for(int i = 0; i < numIvars; i++) {

   Ivar thisIvar = vars[i];
   key = [NSString stringWithUTF8String:ivar_getName(thisIvar)];  //获取成员变量的名字
   NSLog(@"variable name :%@", key);
   key = [NSString stringWithUTF8String:ivar_getTypeEncoding(thisIvar)]; //获取成员变量的数据类型
   NSLog(@"variable type :%@", key);
}
free(vars);
```

### 5. 获取成员函数

```objc
Method *meth = class_copyMethodList(NSClassFromString(@"UIView"), &numIvars);
//Method *meth = class_copyMethodList([UIView class], &numIvars);
    
for(int i = 0; i < numIvars; i++) {
   Method thisIvar = meth[i];
   
   SEL sel = method_getName(thisIvar);
   const char *name = sel_getName(sel);
   
   NSLog(@"zp method :%s", name);
}
free(meth);
```

## 二、基础换主题思路
主题我们可以将其划分为**颜色**和**图片**两大块.

### 1. 颜色的替换  
我们可以设置不同的key，将key对应的颜色值作为value，放在苹果的plist表中。当切换新主题的时候，我们去加载不同的plist即可

### 2. 图片替换 
图片的话只要设置好对应目录即可。给出相同的名字，然后从网上下载好资源，本地建立一样名称的文件夹，并设置好即可。切换主题的时候，从不同目录去加载。

## 三、优化技巧

按照上诉方法大家在进行编码的时候是不是会发现代码很凌乱，通知或者委托到处在。这里提供一个简单的技巧。  
我们利用swizzle在对系统控件加入一个`reloadAppearance `方法,用来设将我们给系统设置好的属性给其赋值。

### 第一步. 在UIView中

```objc
#import <objc/runtime.h>
@implementation UIView (Custom)
#pragma mark - ISkinProtocol

- (void)reloadAppearance
{
    // 遍历所有view reloadAppearance
    for (UIView* subView in self.subviews) {
        [subView reloadAppearance];
    }
    
    [self setNeedsDisplay];
}
```

### 第二步. 由于其他子View如`UILabel`都继承自`UIView`,这里只举一个例子，`UILabel`的写法:

```objc
#import <UIKit/UIKit.h>
#import <objc/runtime.h>
@interface UILabel (Custom)
- (nonatomic, assgin) int skinNormalTextColor;
@end

#import "UILabel+Custom.h"
@implementation UILabel (Custom)
	
- (void)reloadAppearance
{
[super reloadAppearance];//遍历子view
self.skinTextColorNormal = self.skinTextColorNormal;//核心点，会设置上textColor的颜色
}
static NSString *skinNormalTextColorKey = @"skinNormalTextColorKey";
- (int)skinTextColorNormal
{
    NSNumber* skinTextColorNormal = objc_getAssociatedObject(self, skinNormalTextColorKey);
    return [skinTextColorNormal intValue];
}

- (void)setSkinTextColorNormal:(int)skinTextColorNormal
{
    objc_setAssociatedObject(self, skinNormalTextColorKey, [NSNumber numberWithInt:skinTextColorNormal], OBJC_ASSOCIATION_RETAIN);
    
    if (skinTextColorNormal != kColorInvalid) {
        [self setTextColor:__QQGLOBAL_COLOR_USEDEFAULT(skinTextColorNormal, self.skinIsSetDefault)];
    }
}
@end
```

### 第三步. 使用方法:
在`Appdelegate`中提供一个`reload`的`public`方法即可:  

    [self.navigationcontroller.view reloadAppearance];
这样就会从View递归自动遍历到`UILabel`,`UIImageView`那些控件了。


## 小结：
之前在网上看到的思路在代码编写的时候，用了发送通知或者delegate的方法去通知界面刷新，但是只处理了图片，在颜色上没有很好的处理。而且代码显得十分臃肿。有效的利用`swizzle`我们使我们的代码的结构和可读性大大增强，减少代码量。

## demo下载链接:
[变色demo](http://share.weiyun.com/967753e3f6f2dd34226301b9bfb7ccb9)

## 参考链接：  
1.[IOS使用 swizzle 解决一些错误](http://blog.csdn.net/li6185377/article/details/12744577)  
2.[Objective-C Runtime 运行时之四：Method Swizzling](http://blog.jobbole.com/79580/)  
3.[Objective-C的hook方案（一）: Method Swizzling](http://blog.csdn.net/yiyaaixuexi/article/details/9374411)  
4.[Objective-C Runtime Reference](https://developer.apple.com/library/mac/#documentation/Cocoa/Reference/ObjCRuntimeRef/Reference/reference.html)  
5.[iOS运行时获取对象的成员变量和成员方法](http://blog.csdn.net/studyrecord/article/details/18841849)