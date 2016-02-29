---
layout: post
title: "Runloop入门篇"
date: 2015-06-02 21:17:44 +0800
comments: true
categories: Runloop
description: "Runloop入门篇" 
keywords: Runloop 
---

## 基础知识

## 一. RunLoop的概念

![NSRunLoop事件驱动模型]( /images/runloop/NSRunLoop.gif "Runloop时间驱动模型")   
runloop可以想象成一个事件驱动的圆圈，我们在执行事件、手势、时间相应等等操作的时候，需要有监听者，这时候就有了源的概念，NSRunloop只有两种源，**1.输入源** **2.时间源**  
----伪代码如下:  

```objc
while (true) {
 	[[NSRunLoop currentRunLoop] runMode:NSRunLoopCommonModes beforeDate:[NSDate distantFuture]];
}
```
#### 注意：
主线程会默认启动一个Runloop，而异步线程不会,while是为了防止RunLoop休眠，在没有“事件源”去驱动的时候，RunLoop会自动启动休眠模式

<!-- more -->
## 二. 输入源
包括三种:`NSPort`、`自定义源`、`performSelector:OnThread:delay`

### 2.1 NSPort 基于端口的源
Cocoa和 Core Foundation 为使用端口相关的对象和函数创建的基于端口的源提供了内在支持。Cocoa中你从不需要直接创建输入源。你只需要简单的创建端口对象，并使用NSPort的方法将端口对象加入到run loop。端口对象会处理创建以及配置输入源。  

NSPort一般分三种： `NSMessagePort（基本废弃）`、`NSMachPort`、 `NSSocketPort`。 系统中的NSURLConnection就是基于NSSocketPort进行通信的，所以当在后台线程中使用NSURLConnection 时，需要手动启动RunLoop, 因为后台线程中的RunLoop默认是没有启动的.  

```objc  
NSURLConnection *connection = [[NSURLConnection alloc] initWithRequest:request delegate:self startImmediately:NO];//暂时不运行  
[connection scheduleInRunLoop:[NSRunLoop currentRunLoop] forMode:NSRunLoopCommonModes];//用NSRunLoopCommonModes  
[connection start];
```
可以用`NSMachPort`作为线程之间的通讯通道。例如在主线程创建子线程时传入一个`NSPort`对象，这样主线程就可以和这个子线程通讯啦，如果要实现双向通讯，那么子线程也需要回传给主线程一个`NSPort`  

### 2.2 自定义源  
在Core Foundation程序中，必须使用CFRunLoopSourceRef类型相关的函数来创建自定义输入源，接着使用回调函数来配置输入源。Core Fundation会在恰当的时候调用回调函数，处理输入事件以及清理源。常见的触摸、滚动事件等就是该类源，由系统内部实现。
一般我们不会使用该种源，第三种情况已经满足我们的需求

### 2.3 performSelector:OnThread
Cocoa提供了可以在任一线程执行函数（perform selector）的输入源。和基于端口的源一样，perform selector请求会在目标线程上序列化，减缓许多在单个线程上容易引起的同步问题。而和基于端口的源不同的是，**perform selector执行完后会自动清除出run loop**。
此方法简单实用，使用也更广泛.  

```objc
performSelectorOnMainThread:withObject:waitUntilDone:  
performSelectorOnMainThread:withObject:waitUntilDone:modes:

performSelector:onThread:withObject:waitUntilDone:  
performSelector:onThread:withObject:waitUntilDone:modes:

performSelector:withObject:afterDelay:  
performSelector:withObject:afterDelay:inModes:

cancelPreviousPerformRequestsWithTarget:  
cancelPreviousPerformRequestsWithTarget:selector:object:
```
这些API最后两个是取消当前线程中调用，其他API是在主线程或者当前线程下的Run Loop中执行指定的@selector。

## 三. 时间源
需要注意的是`scheduledTimerWith****`开头生成的Timer会自动帮你以默认`NSDefaultRunLoopMode`模式加载到当前的Run Loop中，而其他接口生成的Timer则需要你手动使用`-addTimer:forMode`添加到Run Loop中。需要额外注意的是Timer的触发不会让Run Loop返回。  

	[[NSRunLoop currentRunLoop] addTimer:_timer forMode:NSRunLoopCommonModes];
	[[NSRunLoop currentRunLoop] runMode:NSRunLoopCommonModes beforeDate:[NSDate distantFuture]];

## 四. RunLoop观察者
我们可以通过创建CFRunLoopObserverRef对象来检测RunLoop的工作状态，它可以检测RunLoop的以下几种事件：
Run loop入口
Run loop将要开始定时
Run loop将要处理输入源
Run loop将要休眠
Run loop被唤醒但又在执行唤醒事件前
Run loop终止 

### 1. Run Loop Modes  

|   Mode   | Name	   |      Description    |
|:------------- |:---------------------:| -------------------:|
| Default      | NSDefaultRunLoopMode (Cocoa) kCFRunLoopDefaultMode (Core Foundation) | 缺省情况下，将包含所有操作，并且大多数情况下都会使用此模式，默认的运行模式，除了NSConnection对象的事件 |
| Connection   | NSConnectionReplyMode (Cocoa)        | 此模式用于处理NSConnection的回调事件 |
| Modal | NSModalPanelRunLoopMode (Cocoa)        |  模态模式，此模式下，RunLoop只对处理模态相关事件     |
| Common Modes   | NSRunLoopCommonModes (Cocoa) kCFRunLoopCommonModes (Core Foundation)      |  此模式用于配置”组模式”，一个输入源与此模式关联，则输入源与组中的所有模式相关联  	|
| Event Tracking | NSEventTrackingRunLoopMode (Cocoa)        |  此模式下用于处理窗口事件,鼠标事件等 |  

<br>**注意**: **NSRunLoopCommonModes**是一种常用的模式集合，包含**NSDefaultRunLoopMode**、**NSTaskDeathCheckMode**、**UITrackingRunLoopMode**,这个模式存在的好处就是如果现在异步线程有个timer启动，不需要在所有的runloop mode中都去加一遍，只需要直接在NSRunLoopCommonModes加一次即可  

	

## 参考连接:  
1.[iPhone开发之NSRunLoop的进一步理解](http://www.cnblogs.com/pengyingh/articles/2343920.html)  
2.[iOS多线程编程Part 1/3 - NSThread & Run Loop](http://www.hrchen.com/2013/06/multi-threading-programming-of-ios-part-1/)  
3.[苹果官方文档Run Loops](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/Multithreading/RunLoopManagement/RunLoopManagement.html)  
4.[Cocoa深入学习:NSOperationQueue、NSRunLoop和线程安全](http://blog.cnbluebox.com/blog/2014/07/01/cocoashen-ru-xue-xi-nsoperationqueuehe-nsoperationyuan-li-he-shi-yong/)
