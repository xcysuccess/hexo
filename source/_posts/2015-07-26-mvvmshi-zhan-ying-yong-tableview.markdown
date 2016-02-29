---
layout: post
title: "MVVM实战应用之TableView的代码优化"
date: 2015-07-26 18:36:55 +0800
comments: true
categories: DesignMode
description: "我一直觉得设计模式只是服务于业务场景，其实在绝大多数场景下MVC完全足够，如果说页面过于复杂，我才会去抽离Controller层的代码，实现逻辑分离。所以在写代码的时候我并不会一开始就用十分复杂的结构，只是在需要扩展的时候才会去重构" 
keywords: MVVM
---

## 介绍一下MVC和MVVM
这篇文章先，然后结合一个UITableView的实例来进行分析

在这里插一下个人观点，我一直觉得设计模式只是服务于业务场景，其实在绝大多数场景下`MVC`完全足够，如果说页面过于复杂，我才会去抽离`Controller `层的代码，实现逻辑分离。所以在写代码的时候我并不会一开始就用十分复杂的结构，只是在需要扩展的时候才会去重构。因为可能这一业务功能就这么多了，过两个版本这个功能就不需要了，所以我们得根据业务去衡量。

## 一、MVC
从写代码开始，用的最多的应该就属MVC和单例这两种设计模式了。
  
### 1. 什么是MVC？
Model-View-Controller,Model持有数据,View显示与用户交互的界面,而View Controller调解Model和View之间的交互。  
!["MVC结构图"]( /images/mvvm/1.gif "MVC结构图")  
**`View:`**基本的视图的代码，可以用`xib`或者`storyboard`,或者手写代码。实际开发中一般会自定义出一个View,并设置好内部的`UILabel`,`UIButton`等组件。  
**`Model:`**实体对象，最基本的原子单位  
**`Controller:`**Controller是app的“胶水代码”：协调模型和视图之间的所有交互。控制器负责管理他们所拥有的视 图的视图层次结构,还要响应视图的`loading`、`appearing`、`disappearing`等等 

### 2. MVC缺点
厚重的`View Controller`很难维护（由于其庞大的规模)，包含几十个属性，使他们的状态难以管理；遵循许多协议（`protocol`），导致协议的响应代码和`controller`的逻辑代码混淆在一起。
<!-- more -->

### 3. 快速优化，增加中间层
**`Control:`**用来做网络交互和数据存储用，核心点在于数据的管理  
**`Component:`**自定义的组件  
这种层可以根据业务的存在继续细分，这里不作过多解释。

## 二、MVVM
!["MVVM结构图"]( /images/mvvm/2.gif "MVVM结构图")  
**`ViewModel:`**MVVM就是一个经过优化的MVC，这意味着它可以兼容，也本质上还是一个MVC结构.  
是一个放置用户输入验证逻辑，视图显示逻辑，由于展示逻辑（presentation logic）放在了ViewModel中（比如model的值映射到一个格式化的字符串），视图控制器本身就会不再臃肿。
理解成Controller和Model之间的一座桥梁即可


## 三、TableView实战
!["TableView结构"]( /images/mvvm/3.png "TableView结构")  

这里着重介绍一下数据源`TableViewDataSource`和`QQViewModel`
  
###**1.`QQViewModel:`**  

**.h**文件  

```objc
#import <Foundation/Foundation.h>
@class QQModel;

@interface QQViewModel : NSObject

- (instancetype)initWithQQViewModel:(QQModel *)qqModel;

@property (nonatomic, strong) QQModel *qqModel;
@property(copy,nonatomic) NSString *content;

@end  
```
	
**.m**文件  

```objc	
#import "QQViewModel.h"
#import "QQModel.h"
	
@implementation QQViewModel

- (instancetype)initWithQQViewModel:(QQModel *)qqModel
{	
	 if (self = [super init]){  
	   
	   self.qqModel = qqModel;
   	self.content = [NSString stringWithFormat:@"%@View",qqModel.title];
}
return self;
}
@end  
```

如上述所示，`QQModel`是一个最基本的原子单位，`QQViewModel`是包括了`QQModel`，以及`View`所需要展示的一些字段，我们可以将这些字段的逻辑都封装在`QQViewModel`中，减少`Controller`的臃肿。`View`层只要直接拿到`ViewModel`即可，不用`Controller`在转哦概念图做转换。上述的`content`就是我们需要的字段，其他可根据View自行扩充。  

### 2. `TableViewDataSource:`
在请求到网络数据后，刷新数据源  

```objc
/* 设置表格的数据源 */
- (void)setupTableViewDataSource
{
    TableViewCellConfigureBlock configureCell = ^(QQTableCell *cell, id item){
        [cell configureForData:item];
    };
    _qqDataSource = [[QQDataSource alloc] initWithItems:_publicModelArray
                                         cellIdentifier:QQCELLIDENDIFIFY
                                     configureCellBlock:configureCell];
    
    _qqTableView.dataSource = _qqDataSource;
}
```

这是`QQDataSource`的核心就是通过`block`将建立起一个直接映射到`Cell`上的作用，起到一个数据透传的作用。动态地通过`item`去设置`cell`的值。在`QQDataSource`中的处理:  

```objc
- (UITableViewCell *)tableView:(UITableView *)tableView cellForRowAtIndexPath:(NSIndexPath *)indexPath {
    
    NSLog(@"indexPath.row,indexPath.section:(%ld,%ld)",indexPath.row,indexPath.section);
    QQTableCell *cell = [tableView dequeueReusableCellWithIdentifier:self.cellIdentifier forIndexPath:indexPath];
    
    id item = [self itemAtIndexPath:indexPath];
    self.configureCellBlock(cell,item);
    
    return cell;
}
```

## 小结：
在前几年的工作中，每次写新功能大概就是这个套路，先新建几个文件夹，然后分好上述的几层。再深层次优化就是做一些公共组件之类的，设置一些父类，抽出公共代码，比如公共的网络连接库，SuperController之类。  

## demo下载链接: 
[https://github.com/xcysuccess/TableViewMVVM](https://github.com/xcysuccess/TableViewMVVM)  

## 参考链接：  
1.[scaling-isomorphic-javascript-code](http://blog.nodejitsu.com/scaling-isomorphic-javascript-code/)  
2.[https://github.com/lizelu/MVVM](https://github.com/lizelu/MVVM)  
3.[http://objccn.io/issue-13-1/](https://github.com/lizelu/MVVM)

