---
layout: post
title: "xcode调试效率"
date: 2015-06-30 21:24:45 +0800
comments: true
categories: debug
description: "调试的基本操作，Chisel，以及crashLog查看" 
keywords: xcode调试效率
---

在[苹果的官方文档](https://developer.apple.com/library/mac/documentation/IDEs/Conceptual/gdb_to_lldb_transition_guide/document/lldb-command-examples.html#//apple_ref/doc/uid/TP40012917-CH3-SW5)中列出了我们在调试中能用到的一些命令，我们在这重点讲一些常用的命令 
 
## 一、基本操作

### 1.1. 视图层次
打印视图层次	`po [self.contentView recursiveDescription]`

### 1.2. 改变某个取值

```objc
int a = 1;
//Console expr a=2
NSLog(@"实际值: %d", a);
```

### 1.3. call改变view的背景色   

	call [self.view setBackgroundColor:[UIColor redColor]]

### 1.4. 声明变量

```objc
expr int $b=2 或者 e int $b=2
//输出  po $b
```

### 1.5. 打印堆栈
ios中打印堆栈方法是 `NSThread callStackSymbols`,这里调试的时候有个简单的方法如下  

```objc
bt  或者  bt all
```

<!-- more -->

### 1.6. 更改方法返回值  
thread return NO/YES  

```objc
-(BOOL) returnYES
{
	//thread return NO ，可以更改函数返回值
	return YES;
}//方法返回结果为NO 
```

### 1.7. 多线程异常后查看历史对象的malloc分配历史

先打开**Enable Zombie Objects** 和 **Malloc Stack**  

```objc
(lldb) command script import lldb.macosx.heap
(lldb) malloc_info -S 0x7ff7206c3a70 
```

### 1.8. 寄存器查找对象  
曾经遇到过一个问题，`[self.tableview reloadData]`直接奔溃。这时候tableview其实没有问题，我们要怎么去找问题呢？

![找到文件]( /images/xcode/7.jpeg "找到日志文件")   
如上图所示，最后我们通过` malloc_info -S 0x00007fe99a629560`来查看对象分配在堆里面的具体的地址，随后在左侧打开table的所有变量，输入这个地址即能够看到这个是什么成员。
	
	
### 常见问题－打印无效
上面我们简单的学习了如何使用LLDB命令。但有时我们在使用这些LLDB命令的时候，依然可能会遇到一些问题。不明类型或者类型不匹配  

```objc
p (void)NSLog(@"%@",[self.view  viewWithTag:1001])    //记住要加void 
p (CGRect)[self.view frame]  //记住不能写成 self.view.frame，lldb的bug
```

## 二、调试进阶

### 2.1. 监听某个方法的调用
![设置断点]( /images/xcode/1.gif "设置断点")   
如果是自定义的view，比如`QQView`，想监听frame变化，直接`[QQView setFrame:]`即可
系统方法就要如图所示，x86_64系统中，rdi表示第一个参数，具体其他平台可看[inspecting-obj-c-parameters-in-gdb](https://www.clarkcox.com/blog/2009/02/04/inspecting-obj-c-parameters-in-gdb/)，里面有详细的说明  

```objc
$rdi ➡ arg0 (self)
$rsi ➡ arg1 (_cmd)
$rdx ➡ arg2
$rcx ➡ arg3
$r8 ➡ arg4
$r9 ➡ arg5
```

### 2.2. image寻址,找到崩溃行
此时会调用如下代码会崩溃  

```objc
NSArray *arr=[[NSArray alloc] initWithObjects:@"1",@"2", nil];
NSLog(@"%@",arr[2]);
    
2015-06-30 14:47:59.280 QQLLDB[6656:138560] *** Terminating app due to uncaught exception 'NSRangeException', reason: '*** -[__NSArrayI objectAtIndex:]: index 2 beyond bounds [0 .. 1]'
*** First throw call stack:
(
0   CoreFoundation                      0x000000010b2d8c65 __exceptionPreprocess + 165
1   libobjc.A.dylib                     0x000000010af71bb7 objc_exception_throw + 45
2   CoreFoundation                      0x000000010b1cf17e -[__NSArrayI objectAtIndex:] + 190
3   QQLLDB                              0x000000010aa404f6 -[ViewController viewDidLoad] + 1030
4   UIKit                               0x000000010ba75210 -[UIViewController loadViewIfRequired] + 738
5   UIKit                               0x000000010ba7540e -[UIViewController view] + 27
6   UIKit                               0x000000010b9902c9 -[UIWindow 
```

此时我们要直接找到崩溃的行数，很简单。先找到非系统bug的崩溃地址，如下图所示，显然是第三行QQLLDB,右边对应的地址是`0x000000010aa404f6`,然后输入`image lookup --address 0x000000010aa404f6`,即可看到是60行出了bug   

```objc
image lookup --address 0x000000010aa404f6  
Address: QQLLDB[0x00000001000014f6] (QQLLDB.__TEXT.__text + 1030)  
Summary: QQLLDB`-[ViewController viewDidLoad] + 1030 at ViewController.m:60
```

### 2.3. crash日志分析，读取符号表  
1.要知道如何读取符号表，我们得先伪造一份符号表的数据文件出来，代码如下  

```objc
NSArray *arr=[[NSArray alloc] initWithObjects:@"1",@"2", nil];
NSLog(@"%@",arr[2]);
```

伪造步骤：  
1.编译到真机  
2.然后进入xcode将这个打开，找到QQLLDB.app.dSYM这个文件，偷偷拷贝一份到桌面  
3.然后在product中clean一下工程  
4.在真机上打开一个编译进去的程序，会出现图 
 
### 步骤如下图所示：  
![xcode工程"]( /images/xcode/2.gif "xcode工程")  
![右键打开]( /images/xcode/3.gif "右键打开")  
![包内容]( /images/xcode/4.gif "包内容")   
![View Device Logs]( /images/xcode/5.gif "View Device Logs")   
![找到文件]( /images/xcode/6.gif "找到日志文件")  
 之后在命令行操作  
 
```objc
atos -o /Users/tomxiang/Desktop/符号表/QQLLDB.app.dSYM/Contents/Resources/DWARF/QQLLDB -l0x1000e8000 0x1000eebd4
//此时得到结果，告诉我们是ViewController的viewDidLoad的第62行崩溃
-[ViewController viewDidLoad] (in QQLLDB) (ViewController.m:62)
```

### 2.4 观察实例变量的变化
假设你有一个 UIView，不知道为什么它的 _layer 实例变量被重写了 (糟糕)。因为有可能并不涉及到方法，我们不能使用符号断点。相反的，我们想监视什么时候这个地址被写入。

首先，我们需要找到 _layer 这个变量在对象上的相对位置： 
 
```objc
(lldb) p (ptrdiff_t)ivar_getOffset((struct Ivar*)class_getInstanceVariable([MyView class], "_layer"))
(ptrdiff_t) $0 = 8
```

现在我们知道 ($myView + 8) 是被写入的内存地址：  

```objc
(lldb) watchpoint set expression -- (int *)$myView + 8
Watchpoint created: Watchpoint 3: addr = 0x7fa554231340 size = 8 state = enabled type = w
new value: 0x0000000000000000
```
这被以 wivar $myView _layer 加入到 Chisel 中。

### 2.5 打印所有方法进行log分析
![打印log"]( /images/xcode/8.jpeg "打印log") 


## 三、Chisel-facebook开源插件

### 1. 安装方法:

```objc
git clone https://github.com/facebook/chisel.git ~/.chisel
echo "command script import ~/.chisel/fblldb.py">>~/.lldbinit
安装好后，打开xcode就可以运行调试了
```

### 2. 基本命令

#### 2.1. 预览图片   

```objc
UIImage *image = [UIImage imageNamed:@"clear"];
UIImageView *imageView = [[UIImageView alloc] initWithImage:image];
[viewB addSubview:imageView];

//此时执行命令之后，图片会被苹果用自带的预览工具显示出来    
visualize image
```
#### 2.2. 边框/内容着色   

如下所示，打印得到imageView地址之后，然后用`border`命令将其边框着色`unborder`取消着色  

```objc
(lldb) p imageView
(UIImageView *) $2 = 0x00007fb8c9b15910
(lldb) border 0x00007fb8c9b15910 -c green -w 2
```
同理，`mask`是给内容着色    `unmask`  

	mask imageView -c green

#### 2.3. 关系链的继承  

```objc
(lldb) pclass image
UIImage
	| NSObject
```

#### 2.4. 打印所有属性  
`pinternals`这个命令就是打印出来的一个控件（id）类型的内部结构，详细到令人发指！甚至是你自定义的控件中的类型，譬如这个styleView就是我自定义的，内部有个iconView的属性，其中的值它也会打印出来。 

```objc	
(lldb) pinternals image
(UIImage) $8 = {  
  	NSObject = {
  	isa = UIImage  
}
_imageRef = 0x00007fc1f3330780
_scale = 2
_imageFlags = {
named = 1
imageOrientation = 0
cached = 0
hasPattern = 0
isCIImage = 0
renderingMode = 0
suppressesAccessibilityHairlineThickening = 0
hasDecompressionInfo = 0
}
}
```
    
#### 2.5. bmessage  
如果ChiselViewController没有设置`viewWillDisappear`这个方法，此时我想用断点断下来，可以这样  

```objc
(lldb) bmessage -[ChiselViewController viewWillDisappear:]
	
Setting a breakpoint at -[UIViewController viewWillDisappear:] with condition (void*)object_getClass((id)$rdi) == 0x0000000100c47060
Breakpoint 2: where = UIKit`-[UIViewController viewWillDisappear:], address = 0x0000000101a0b566
```

## 三. Crash Callstack分析 - 进⼀一步分析

属性 | 	说明 |
----|------|
0x8badf00d | 在启动、终⽌止应⽤用或响应系统事件花费过⻓长时间,意为“ate bad food”。|
0xdeadfa11 | ⽤用户强制退出,意为“dead fall”。(系统⽆无响应时,⽤用户按电源开关和HOME)|
0xbaaaaaad | ⽤用户按住Home键和⾳音量键,获取当前内存状态,不代表崩溃 |
0xbad22222 | VoIP应⽤用因为恢复得太频繁导致crash |
0xc00010ff | 因为太烫了被干掉,意为“cool off”  |
0xdead10cc | 因为在后台时仍然占据系统资源(⽐比如通讯录)被干掉,意为“dead lock” | 

## 参考链接：  
1.[当异常出现时](http://blog.csdn.net/smking/article/details/8639860)  
2.[日志记录CocoaLumberjack](https://github.com/CocoaLumberjack/CocoaLumberjack
)  
3.[在Xcode中调试程序](http://southpeak.github.io/blog/2015/01/25/gong-ju-pian-:lldbdiao-shi-qi/)  
4.[南峰子的技术博客](http://southpeak.github.io/blog/2015/01/25/gong-ju-pian-:lldbdiao-shi-qi/)  
5.[与调试器共舞 - LLDB 的华尔兹](http://objccn.io/issue-19-2/)  
6.[官方调试技巧文档](https://developer.apple.com/library/mac/technotes/tn2124/_index.html)  
7.[inspecting-obj-c-parameters-in-gdb](https://www.clarkcox.com/blog/2009/02/04/inspecting-obj-c-parameters-in-gdb/)

版权申明：我已将本文在微信公众平台的发表权「独家代理」给 iOS 开发（iOSDevTips）微信公共帐号。
扫码关注「iOS 开发」：  
![二维码](http://blog.devtang.com/images/weixin-qr.jpg)