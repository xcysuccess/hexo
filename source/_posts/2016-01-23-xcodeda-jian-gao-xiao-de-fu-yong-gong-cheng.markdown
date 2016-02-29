---
layout: post
title: "xcode用workspace搭建高效的复用工程"
date: 2016-01-23 11:27:32 +0800
comments: true
categories: designmode
description: "这里讨论了如何利用xcode搭建以及pod搭建复用工程，用静态库的思路做代码，增加代码的可读性和重用性" 
keywords: workspace
---

## 一. 前言

工作一直以来就很想分享一下如何利用xcode的静态库搭建复用工程。优点：  

1. 解决依赖，增加复用性，没有必要重复造轮子    
2. 增强代码可读性（极其关键）  
3. 拉分支时候可以删除无用静态库，加快编译速度
4. 只需要打开一个工作环境，需要修改、同步代码，都不需要打开新的项目、新的文件，让人可以集中心思在代码上，在不同的项目里跳来跳去很容易打断思维的。
5. 可以像同一个工程里一样，直接点击方法名查看引用库项目的代码，否则就要打开另一个项目，然后找到对应文件再找到方法。
6. 只要运行自己的项目就行，就会自动帮你编译库文件。

目前很多大公司的工程其实是十分混乱的，后来思考后觉得一个原因是大家并不了解workspace，还有一个原因是因为一些历史包袱-刚开始创建工程的人并没有做到统筹全局的规划，这种事情呢，在大公司一般也无法和KPI去挂钩，动工程的代价太大，还有引起很多问题，所以一般的思路是能继续用就继续用。  
这里要特别感谢自己以前在金山工作的时候，团队技术负责人涌哥，无论有多忙，工程目录的结构都是重中之重，不会因为时间不够而不去梳理，后期当团队有几十号人的时候，项目工程依然还是能够很清晰。


## 二. 实战操作
因为现在用`Pod`可以自动创建`workspace`,所以这里介绍一下最新的方法去玩`workspace`.
<!-- more -->

### 1. New一个`Podfile`文件
写上

```objc
	platform:ios, '7.0' 
	pod 'AFNetworking','~>3.0.4'
```
一般网络操作都少不了`AFNetworking`,加上

### 2. cd到目录，用命令行进行`pod install`
此刻`workspace`已经生成

### 3. Add静态库,如图所示
![1.png](/images/static/1.png)  
![2.png](/images/static/2.png)  

### 4. 独立化
这个是一个大的静态库，其包含的每一个库应该是能独立的（可以参照pod里面加第三方库的思路），现在我们这么做  
![3.png](/images/static/3.png)  

### 5. 编译静态库
如图，如果是红色就代表没有编译成功，不是红色代表编译成功了，'Ctrl+B'并选择Device进行编译，用模拟器编译没用，据网上说是xcode的bug
![4.png](/images/static/4.png)  

### 6. 解决依赖和header头文件的关键
![5.png](/images/static/5.png)  

### 7. 库的引入
当然，主工程里面还得把他们引入进来，千万不可忘记
![6.png](/images/static/6.png)  

### 8. 结束
大功告成，现在看我们的实战，工程里面调用了UIColorHex，在foundationex库中的方法  

```objc
@interface ViewController ()
@end

@implementation ViewController
- (void)viewDidLoad {
    [super viewDidLoad];
    self.view.backgroundColor = UIColorHex(233344);
    // Do any additional setup after loading the view, typically from a nib.
}
```
## 三. 结论
**在分享技术的时候一切没有demo的结论都是耍流氓**，这里当然要附上demo，大家如果遇到问题可以进行参照对比。

## demo地址
[https://github.com/xcysuccess/XXKit.git](https://github.com/xcysuccess/XXKit.git)



	

	