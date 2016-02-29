---
layout: post
title: "ios9.0的系统bug导致的objc_msgSend错误的解决方案"
date: 2015-09-24 19:34:03 +0800
comments: true
categories: Debug 
description: "对遇到系统错误的时候，我们如何去分析解决提供一种思路" 
keywords: debug,异常,fishhook,objc_msgSend
---

## 前言
看此篇文章之前请先阅读[xcode调试效率](http://www.iosxxx.com/blog/categories/debug/).  
ios9.0上遇到一个问题，`UITableView`中长按`section`，如果我们的交互中要求弹出menu菜单，那么就会出现如图所示的必现崩溃  
!["操作"](/images/ios9crash/1.png)  

### 安装`lldb`的`malloc`命令

	vim ~/.lldbinit
	command script import lldb.macosx.heap
	按一下esc 
	wq 保存退出
	
## 一. 分析

### 问题:
1.这个NSDictionary到底是什么?是`UITableView`的数据源吗？  
2.如果是数据源是否是多线程导致的呢？(`NSDictionary`在多线程下如未处理好极容易崩溃,`set`和`get`同时调用的时候,一个对象被`remove`之后野指针了，然后`get`操作会立马导致崩溃)  
3. 是否是`mrc`导致问题？  

### 验证过程:
1.由于工程比较大，建议先写一个demo去做。(大工程的一些配置选项可能导致`lldb`的某些命令无法使用），这一点耗费了我们很大的精力去分析`NSZombie`的`malloc`历史，都没有结果  

### 结果:
在工程中拆分出demo之后，由于我们根本没有`NSDictionary`，所以排除了这种情况，但是为什么会有这个问题呢。这个`NSDictionay`又是什么呢?  
<!-- more -->

## 二. 过程分析

### 1. 查看崩溃源

```objc
(lldb) command script import "lldb.macosx.heap"
(lldb) bt
* thread #1: tid = 0xbe55, 0x0000000100ded805 libobjc.A.dylib`objc_msgSend + 5, queue = 'com.apple.main-thread', stop reason = EXC_BAD_ACCESS (code=1, address=0x1)
    frame #0: 0x0000000100ded805 libobjc.A.dylib`objc_msgSend + 5
    frame #1: 0x00000001012fb1a5 CoreFoundation`-[NSDictionary descriptionWithLocale:indent:] + 373
    frame #2: 0x000000010097d3f4 Foundation`_NSDescriptionWithLocaleFunc + 64
    frame #3: 0x000000010124fe4d CoreFoundation`__CFStringAppendFormatCore + 9597
    frame #4: 0x000000010133c563 CoreFoundation`_CFStringCreateWithFormatAndArgumentsAux2 + 259
    frame #5: 0x000000010134ce0a CoreFoundation`_CFLogvEx2 + 154
    frame #6: 0x000000010134cf6b CoreFoundation`_CFLogvEx3 + 171
    frame #7: 0x0000000100a53d5e Foundation`_NSLogv + 117
    frame #8: 0x00000001009a35e2 Foundation`NSLog + 152
  * frame #9: 0x000000010183fc5e UIKit`-[UITableView reloadData] + 1853
    frame #10: 0x00000001008d436c Test`-[SectionHeader showPendingMenu](self=0x00007fab6b443250, _cmd="showPendingMenu") + 572 at FirstViewController.m:47
    frame #11: 0x00000001008d411b Test`-[SectionHeader longPress](self=0x00007fab6b443250, _cmd="longPress") + 43 at FirstViewController.m:27
    frame #12: 0x0000000101bc6b40 UIKit`_UIGestureRecognizerSendTargetActions + 153
    frame #13: 0x0000000101bc36af UIKit`_UIGestureRecognizerSendActions + 162
    frame #14: 0x0000000101bc1f01 UIKit`-[UIGestureRecognizer _updateGestureWithEvent:buttonEvent:] + 822
    frame #15: 0x0000000101bc93f3 UIKit`___UIGestureRecognizerUpdate_block_invoke809 + 79
    frame #16: 0x0000000101bc9291 UIKit`_UIGestureRecognizerRemoveObjectsFromArrayAndApplyBlocks + 342
    frame #17: 0x0000000101bbaeb4 UIKit`_UIGestureRecognizerUpdate + 2624
    frame #18: 0x00000001012899d7 CoreFoundation`__CFRUNLOOP_IS_CALLING_OUT_TO_AN_OBSERVER_CALLBACK_FUNCTION__ + 23
    frame #19: 0x0000000101289947 CoreFoundation`__CFRunLoopDoObservers + 391
    frame #20: 0x000000010127f59b CoreFoundation`__CFRunLoopRun + 1147
    frame #21: 0x000000010127ee98 CoreFoundation`CFRunLoopRunSpecific + 488
    frame #22: 0x0000000104a5aad2 GraphicsServices`GSEventRunModal + 161
    frame #23: 0x000000010170a676 UIKit`UIApplicationMain + 171
    frame #24: 0x00000001008d4c9f Test`main(argc=1, argv=0x00007fff5f32c618) + 111 at main.m:14
    frame #25: 0x00000001039e992d libdyld.dylib`start + 1
    frame #26: 0x00000001039e992d libdyld.dylib`start + 1  
```

### 2. 异常截获
!["操作"](/images/ios9crash/4.jpg)  
通过条件断点我们查看最后一次调用的地方，然后打印当前寄存器的值  

```objc
(lldb) register read
General Purpose Registers:
 rax = 0x0000000109a1b001  CoreFoundation`_CFRuntimeGetClassWithTypeID + 17
 rbx = 0x00007feb75041a70
 rcx = 0x0000000000000000
 rdx = 0x0000000000000000
 rdi = 0x00007feb75041a70
 rsi = 0x0000000109c8db63  "descriptionWithLocale:"
 rbp = 0x00007fff56c08380
 rsp = 0x00007fff56c08368
 r8 = 0x0000000080000000
 r9 = 0x000000000000001f
 r10 = 0x00007feb736203e0
 r11 = 0x0000000109ca0ef0  (void *)0x0000000109ca0f18: __NSCFDictionary
 r12 = 0x00007feb7346f180
 r13 = 0x00007fff56c08fe0
 r14 = 0x0000000000000000
 r15 = 0x000000010a98788e  "There are visible views left after reusing them all: %@"
 rip = 0x0000000109a1b010  CoreFoundation`-[NSDictionary descriptionWithLocale:]
 rflags = 0x0000000000000246
 cs = 0x000000000000002b
 fs = 0x0000000000000000
 gs = 0x0000000000000000  
```	
 
通过  
!["操作"](/images/ios9crash/2.jpg)  
我们可以去找到rdi,这个返回参数。此时去找这个参数的地址  

```objc
x00007fc93a638050: malloc(    64) -> 0x7fc93a638050 __NSCFDictionary.NSMutableDictionary.NSDictionary.NSObject.isa
stack[0]: addr = 0x7fc93a638050, type=malloc, frames:
[0] 0x000000010bb44391 libsystem_malloc.dylib`malloc_zone_malloc + 107
[1] 0x00000001091b3c36 CoreFoundation`_CFRuntimeCreateInstance + 310
[2] 0x00000001091b3712 CoreFoundation`CFBasicHashCreate + 114
[3] 0x00000001091b6674 CoreFoundation`CFDictionaryCreateMutable + 212
[4] 0x00000001097d482e UIKit`-[UITableView _setupTableViewCommon] + 137
[5] 0x00000001097d4e89 UIKit`-[UITableView initWithFrame:style:] + 205
[6] 0x000000010886e63e Test`-[FirstViewController loadView] + 254 at FirstViewController.m:0
[7] 0x000000010982f47c UIKit`-[UIViewController loadViewIfRequired] + 139
[8] 0x0000000109872c26 UIKit`-[UINavigationController _layoutViewController:] + 54  
```

`CFDictionaryCreateMutable`是关键方法，通过它我们知道了如何去分配对象，理想当然，我们得去找到系统的`CFDictionary`，这时候会发现有一个`CFDictionarySetValue`的方法  

## 三. 解决方案

### 1. hook截获
要截获`CounFoundation`库的C方法，普通的oc的`swizzle`当然是做不到的，但是幸好有`facebook`提供的`fishhook`，`fishhook`原理可以参照[iOS安全攻防（十七）：Fishhook](http://blog.csdn.net/yiyaaixuexi/article/details/19094765).  
我们现在来截获一把,!["key的值"](/images/ios9crash/3.jpeg)，惊喜的事情出现了，此时的`key`的值竟然是`0x01`，明显的错误啊！内部实现应该是将某个整形树枝给一个NSObject的指针了。  

### 2. 替换判断

```objc
void my_CFDictionarySetValue(CFMutableDictionaryRef theDict, const void *key, const void *value) {
NSLog(@"wym Calling my_CFDictionarySetValue, %p, k:%p, v:%p\n", theDict, key, value);
const void *newKey = key;
if ((int)key == 1 && [NSStringFromClass([(__bridge NSObject *)value class]) isEqualToString:@"SectionHeader"]) {
   newKey = (__bridge void *)[NSNumber numberWithUnsignedInteger:[(__bridge NSObject *)value hash]];
   //NSLog(@"%@", [NSThread callStackSymbols]);
}
orig_CFDictionarySetValue(theDict, newKey, value);
}    
```	
  
这样我们就不会崩溃了。  

### 3. 结论  
整个事情的来龙去脉已经清楚了，这里提供了只是提供一个思路去解决问题。但是实际开发中不推荐这么去做，这个方法会对app性能有影响，建议用`UIButton`去模拟规避这个问题。  

这个问题的解决并不是本人，只是对大神岳明哥的思路进行了总结，一切版权以及所有权归岳明哥所有！非经作者允许不得转载！～

## demo下载
[demo](http://share.weiyun.com/c4301c85ae5243fb57b60c9983764388)

## 参考链接：  
1.[iOS安全攻防（十七）：Fishhook](http://blog.csdn.net/yiyaaixuexi/article/details/19094765)  
2.[lldb官方命令大全](http://lldb.llvm.org/lldb-gdb.html)

  
