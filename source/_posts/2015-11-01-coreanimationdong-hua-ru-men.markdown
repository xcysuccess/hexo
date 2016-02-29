---
layout: post
title: "CoreAnimation动画入门"
date: 2015-11-01 18:16:54 +0800
comments: true
categories: animation 
description: "CoreAnimation,CAAnimation,CABasicAnimation,CAKeyframeAnimation, CATrasition,CAAnimaitionGroup, CAAnaimation" 
keywords: debug,异常
---

## 一. 动画的基础分类
![1.png](/images/animations/1.png)  
上述我们可以看到动画大体可以分为如下几类:  

属性 | 	说明 |
----|------|
CAAnaimation | 抽象类，不具备动画效果，必须用它的子类才有动画效果 |
CAAnimaitionGroup | 动画组，可以同时进行缩放，旋转 |
CAPropertyAnimation | 抽象类，本身不具备动画效果，只有子类才有 |
CABasicAnimation | 基本动画，做一些简单效果 |
CAKeyFrameAnimation | 帧动画，做一些连续的流畅的动画  |
CATrasition | 转场动画 |  

核心动画中所有类都遵守CAMediaTiming协议。

## 二. CAAnimation——简介

### 2.1 基本属性说明：

属性 | 	说明 |
----|------|
duration |	动画时长（秒为单位）（注：此处与原文有出入） |
repeatCount |	重复次数。永久重复的话设置为HUGE_VALF。 |
repeatDuration |	重复时间 |
beginTime | 指定动画开始时间。从开始指定延迟几秒执行的话，请设置为「CACurrentMediaTime() + 秒数」的形式。 |
timingFunction |	设定动画的速度变化 |
autoreverses | 动画结束时是否执行逆动画 |
removedOnCompletion | 	默认为YES，代表动画执行完毕后就从图层上移除，图形会恢复到动画执行前的状态。如果想让图层保持显示动画执行后的状态，那就设置为NO，不过还要设置fillMode为kCAFillModeForwards
fillMode | 决定当前对象在非active时间段的行为。比如动画开始之前或者动画结束之 |

### 2.2 **fillMode**解释说明:
  
属性 | 	说明 |
----|------|
kCAFillModeRemoved | 这个是默认值，也就是说当动画开始前和动画结束后，动画对layer都没有影响，动画结束后，layer会恢复到之前的状态 |
kCAFillModeForwards |	当动画结束后,layer会一直保持着动画最后的状态 |
kCAFillModeBackwards |	这个和kCAFillModeForwards是相对的,就是在动画开始前,你只要将动画加入了一个layer,layer便立即进入动画的初始状态并等待动画开始.你可以这样设定测试代码,将一个动画加入一个layer的时候延迟5秒执行.然后就会发现在动画没有开始的时候,只要动画被加入了layer,layer便处于动画初始状态  |
kCAFillModeBoth | 理解了上面两个,这个就很好理解了,这个其实就是上面两个的合成.动画加入后开始之前,layer便处于动画初始状态,动画结束后layer保持动画最后的状态. |

<!-- more -->

### 2.3 **CAMediaTimingFunction**速度控制方法:
  
属性 | 	说明 |
----|------|
kCAMediaTimingFunctionLinear（线性）| 匀速，给你一个相对静态的感觉 |
kCAMediaTimingFunctionEaseIn（渐进）| 动画缓慢进入，然后加速离开 |
kCAMediaTimingFunctionEaseOut（渐出）| 动画全速进入，然后减速的到达目的地 |
kCAMediaTimingFunctionEaseInEaseOut（渐进渐出）| 动画缓慢的进入，中间加速，然后减速的到达目的地。这个是默认的动画行为。 |  

![3.png](/images/animations/3.png)   

### 2.4 代理方法:

```objc
// 动画开始时调用
- (void)animationDidStart:(CAAnimation *)anim;
// 动画结束后调用
- (void)animationDidStop:(CAAnimation *)anim finished:(BOOL)flag;
```

## 三. CABasicAnimation

###概念:
CABasicAnimation其实可以看作一种特殊的关键帧动画,只有头尾两个关键帧  

属性 | 	说明 |
----|------|
fromValue | 开始值
toValue | 终了值（絶対値）
byValue | keyPath属性的变化值（使用较少）

###动画过程说明：
随着动画的进行，在长度为duration的持续时间内，keyPath相应属性的值从fromValue渐渐地变为toValue  

###可以改变的值:
![2.png](/images/animations/2.png)   
**animationWithKeyPath的值：**  

transform.scale = 比例轉換  
transform.scale.x = 闊的比例轉換  
transform.scale.y = 高的比例轉換  
transform.rotation.z = 平面圖的旋轉  
opacity = 透明度  
margin  
zPosition  
backgroundColor  
cornerRadius  
borderWidth  
frame  
bounds  
contents  
contentsRect  
cornerRadius  
hidden  
mask  
masksToBounds  
position  
shadowColor  
shadowOffset  
shadowOpacity  
shadowRadius  

###3.1 移动动画

```objc
CABasicAnimation *animation = [CABasicAnimation animationWithKeyPath:@"position"];
    
//动画选项的设定
animation.duration = 2.5;  // 持续时间
animation.repeatCount = 1; // 重复次数
    
//起始帧和终了帧的设定
animation.fromValue = [NSValue valueWithCGPoint:self.viewRect.layer.position]; // 起始帧
animation.toValue = [NSValue valueWithCGPoint:CGPointMake(320, 480)]; // 终了帧
animation.autoreverses = YES; // 动画结束时执行逆动画
    
// 动画终了后不返回初始状态
// animation.removedOnCompletion = NO;
// animation.fillMode = kCAFillModeForwards;
    
// 添加动画
[self.viewRect.layer addAnimation:animation forKey:@"move-layer"];
```

### 3.2 旋转动画
![4.png](/images/animations/4.png)   

```objc
// 对Y轴进行旋转（指定Z轴的话，就和UIView的动画一样绕中心旋转）
CABasicAnimation *animation = [CABasicAnimation animationWithKeyPath:@"transform.rotation.y"];
    
// 设定动画选项
animation.duration = 2.5; // 持续时间
animation.repeatCount = 1; // 重复次数
    
// 设定旋转角度
animation.fromValue = [NSNumber numberWithFloat:0.0]; // 起始角度
animation.toValue = [NSNumber numberWithFloat:2 * M_PI/3]; // 终止角度
    
// 动画终了后不返回初始状态
animation.removedOnCompletion = NO;
animation.fillMode = kCAFillModeForwards;
    
// 添加动画
[self.viewRect.layer addAnimation:animation forKey:@"rotate-layer"];
```

### 3.3 缩放动画

```objc
CABasicAnimation *animation = [CABasicAnimation animationWithKeyPath:@"transform.scale"];
// 动画选项设定
animation.duration = 2.5; // 动画持续时间
animation.repeatCount = 1; // 重复次数
animation.autoreverses = YES; // 动画结束时执行逆动画
    
// 缩放倍数
animation.fromValue = [NSNumber numberWithFloat:1.0]; // 开始时的倍率
animation.toValue = [NSNumber numberWithFloat:2.0];   // 结束时的倍率
    
// 添加动画
[self.viewRect.layer addAnimation:animation forKey:NULL];
```

### 3.4 CABaseAnimation组合动画

```objc
 /* 动画1（在X轴方向移动） */
CABasicAnimation *animation1 =
[CABasicAnimation animationWithKeyPath:@"transform.translation.x"];
// 终点设定
animation1.toValue = [NSNumber numberWithFloat:80];; // 終点
    
/* 动画2（绕Z轴中心旋转） */
CABasicAnimation *animation2 =
[CABasicAnimation animationWithKeyPath:@"transform.rotation.z"];
// 设定旋转角度
animation2.fromValue = [NSNumber numberWithFloat:0.0]; // 开始时的角度
animation2.toValue = [NSNumber numberWithFloat:2 * M_PI]; // 结束时的角度
    
/* 动画组 */
CAAnimationGroup *group = [CAAnimationGroup animation];
    
// 动画选项设定
group.duration = 3.0;
group.repeatCount = 1;
group.autoreverses = YES;
    
// 添加动画
group.animations = [NSArray arrayWithObjects:animation1, animation2, nil];
[self.viewRect.layer addAnimation:group forKey:@"move-rotate-layer"];
```

## 四. CAKeyframeAnimation

### 概念
任何动画要表现出运动或者变化,至少需要两个不同的关键状态,而中间的状态的变化可以通过插值计算完成,从而形成补间动画,表示关键状态的帧叫做关键帧.   

属性 | 	说明 |
----|------|
path | 这是一个 CGPathRef  对象，默认是空的，当我们创建好CAKeyframeAnimation的实例的时候，可以通过制定一个自己定义的path来让  某一个物体按照这个路径进行动画。这个值默认是nil  当其被设定的时候  values  这个属性就被覆盖。 |
values | 一个数组，提供了一组关键帧的值，  当使用path的 时候 values的值自动被忽略。 |

### calculationMode计算模式
其主要针对的是每一帧的内容为一个座标点的情况,也就是对anchorPoint 和 position 进行的动画.当在平面座标系中有多个离散的点的时候,可以是离散的,也可以直线相连后进行插值计算,也可以使用圆滑的曲线将他们相连后进行插值计算. calculationMode目前提供如下几种模式  

关键字 | 属性 |
----|------|
kCAAnimationLinear | calculationMode的默认值,表示当关键帧为座标点的时候,关键帧之间直接直线相连进行插值计算 |
kCAAnimationDiscrete | 离散的,就是不进行插值计算,所有关键帧直接逐个进行显示; | 
kCAAnimationPaced | 使得动画均匀进行,而不是按keyTimes设置的或者按关键帧平分时间,此时keyTimes和timingFunctions无效; |   
kCAAnimationCubic | 对关键帧为座标点的关键帧进行圆滑曲线相连后插值计算,对于曲线的形状还可以通过tensionValues,continuityValues,biasValues来进行调整自定义,这里的数学原理是Kochanek–Bartels spline,这里的主要目的是使得运行的轨迹变得圆滑; | 
kCAAnimationCubicPaced | 看这个名字就知道和kCAAnimationCubic有一定联系,其实就是在kCAAnimationCubic的基础上使得动画运行变得均匀,就是系统时间内运动的距离相同,此时keyTimes以及timingFunctions也是无效的. |

### 4.1 使用贝赛尔曲线路径的关键帧动画

```objc	
 UIBezierPath *path = [UIBezierPath bezierPath];
[path moveToPoint:CGPointMake(-40, 100)];
[path addLineToPoint:CGPointMake(360, 100)];
[path addLineToPoint:CGPointMake(360, 200)];
[path addLineToPoint:CGPointMake(-40, 200)];
[path addLineToPoint:CGPointMake(-40, 300)];
[path addLineToPoint:CGPointMake(360, 300)];
    
CAKeyframeAnimation *moveAnimation = [CAKeyframeAnimation animationWithKeyPath:@"position"];
moveAnimation.path = path.CGPath; //是关键
moveAnimation.duration = 2.0f;
moveAnimation.rotationMode = kCAAnimationDiscrete;
//kCAAnimationPaced kCAAnimationCubic :使得keytimes无效
[moveAnimation setKeyTimes:[NSArray arrayWithObjects:[NSNumber numberWithFloat:0.0],[NSNumber numberWithFloat:0.1], [NSNumber numberWithFloat:0.50],[NSNumber numberWithFloat:0.65], [NSNumber numberWithFloat:0.7],[NSNumber numberWithFloat:1.0], nil]];
    
[self.viewRect.layer addAnimation:moveAnimation forKey:@"moveAnimation"];
```
    
### 4.2 使用基于缩放的关键帧动画

```objc
 CAKeyframeAnimation *bounce = [CAKeyframeAnimation animationWithKeyPath:@"transform"];
    
///设置动画
CATransform3D forward = CATransform3DMakeScale(1.3, 1.3, 1);
CATransform3D back = CATransform3DMakeScale(0.7, 0.7, 1);
CATransform3D forward2 = CATransform3DMakeScale(1.2, 1.2, 1);
CATransform3D back2 = CATransform3DMakeScale(0.9, 0.9, 1);
    
[bounce setValues:[NSArray arrayWithObjects:
                  [NSValue valueWithCATransform3D:CATransform3DIdentity],
                  [NSValue valueWithCATransform3D:forward],
                  [NSValue valueWithCATransform3D:back],
                  [NSValue valueWithCATransform3D:forward2],
                  [NSValue valueWithCATransform3D:back2],
                  [NSValue valueWithCATransform3D:CATransform3DIdentity],nil]];
    
[bounce setDuration:1];
    
UILabel *temp = (UILabel *) [self.view viewWithTag:100];
[temp.layer addAnimation:bounce forKey:NULL];
[self.viewRect.layer addAnimation:bounce forKey:NULL];
```

### 4.3 使用基于路径的关键帧动画

```objc
CGMutablePathRef path = CGPathCreateMutable();
 
CGPathMoveToPoint(path, NULL, 50.0, 120.0);
CGPathAddCurveToPoint(path, NULL, 50.0, 275.0, 150.0, 275.0, 150.0, 120.0);
CGPathAddCurveToPoint(path,NULL,150.0,275.0,250.0,275.0,250.0,120.0);
CGPathAddCurveToPoint(path,NULL,250.0,275.0,350.0,275.0,350.0,120.0);
CGPathAddCurveToPoint(path,NULL,350.0,275.0,450.0,275.0,450.0,120.0);
    
CAKeyframeAnimation *anim = [CAKeyframeAnimation animationWithKeyPath:@"position"];
[anim setPath:path];
[anim setDuration:3.0];
[anim setAutoreverses:YES];
CFRelease(path);
[self.viewRect.layer addAnimation:anim forKey:@"path"];
```

### 4.4 使用基于位置点的关键桢动画

```objc	
CAKeyframeAnimation *keyAnim = [CAKeyframeAnimation animationWithKeyPath:@"transform"];
CATransform3D rotation1 = CATransform3DMakeRotation(30 * M_PI/180, 0, 0, -1);
CATransform3D rotation2 = CATransform3DMakeRotation(60 * M_PI/180, 0, 0, -1);
CATransform3D rotation3 = CATransform3DMakeRotation(90 * M_PI/180, 0, 0, -1);
CATransform3D rotation4 = CATransform3DMakeRotation(120 * M_PI/180, 0, 0, -1);
CATransform3D rotation5 = CATransform3DMakeRotation(150 * M_PI/180, 0, 0, -1);
CATransform3D rotation6 = CATransform3DMakeRotation(180 * M_PI/180, 0, 0, -1);
    
[keyAnim setValues:[NSArray arrayWithObjects:
                   [NSValue valueWithCATransform3D:rotation1],
                   [NSValue valueWithCATransform3D:rotation2],
                   [NSValue valueWithCATransform3D:rotation3],
                   [NSValue valueWithCATransform3D:rotation4],
                   [NSValue valueWithCATransform3D:rotation5],
                   [NSValue valueWithCATransform3D:rotation6],
                   nil]];
[keyAnim setKeyTimes:[NSArray arrayWithObjects:
                     [NSNumber numberWithFloat:0.0],
                     [NSNumber numberWithFloat:0.2f],
                     [NSNumber numberWithFloat:0.4f],
                     [NSNumber numberWithFloat:0.6f],
                     [NSNumber numberWithFloat:0.8f],
                     [NSNumber numberWithFloat:1.0f],
                     nil]];
[keyAnim setDuration:4];
[keyAnim setFillMode:kCAFillModeForwards];
[keyAnim setRemovedOnCompletion:NO];
[self.viewRect.layer addAnimation:keyAnim forKey:nil];
```
  
## 五. CAGroupAnimation

### 概念
将几种动画叠加起来同时播放,Group中设置的属性优先级高于内部每一个动画的属性，比如内部`posAnim.removedOnCompletion = YES;`,但是在group中设置了`animGroup.removedOnCompletion = NO;animGroup.fillMode = kCAFillModeForwards;`内部属性即为无效


### 5.1 UIBezierPath、缩放、透明合并起来的Group动画

```objc	
-(void) startAnimations
{
    CAAnimationGroup *group = [self createGroupAnimation];
    [self.viewRect.layer addAnimation:group forKey:NULL];
}

- (CAAnimationGroup *)createGroupAnimation
{
    //贝塞尔曲线路径
    UIBezierPath *movePath = [UIBezierPath bezierPath];
    [movePath moveToPoint:CGPointMake(10.0, 10.0)];
    [movePath addQuadCurveToPoint:CGPointMake(100, 300) controlPoint:CGPointMake(300, 100)];
    
    //路径动画
    CAKeyframeAnimation * posAnim = [CAKeyframeAnimation animationWithKeyPath:@"position"];
    posAnim.path = movePath.CGPath;
    posAnim.removedOnCompletion = YES;
    
    //缩放动画
    CABasicAnimation *scaleAnim = [CABasicAnimation animationWithKeyPath:@"transform"];
    scaleAnim.fromValue = [NSValue valueWithCATransform3D:CATransform3DIdentity];
    scaleAnim.toValue = [NSValue valueWithCATransform3D:CATransform3DMakeScale(0.1, 0.1, 1.0)];
    scaleAnim.removedOnCompletion = YES;
    
    //透明动画
    CABasicAnimation *opacityAnim = [CABasicAnimation animationWithKeyPath:@"opacity"];
    opacityAnim.fromValue = [NSNumber numberWithFloat:1.0];
    opacityAnim.toValue = [NSNumber numberWithFloat:0.1];
    opacityAnim.removedOnCompletion = NO;
    
    //动画组
    CAAnimationGroup *animGroup = [CAAnimationGroup animation];
    animGroup.animations = [NSArray arrayWithObjects:posAnim, scaleAnim, opacityAnim, nil];
    animGroup.duration = 1;
    animGroup.removedOnCompletion = NO;
    animGroup.fillMode = kCAFillModeForwards;
    
    return animGroup;
}
```

## 六. CATransition

### 概念

关键字 | 属性 |
----|------|
startProgress | 动画开始(在整体动画的百分比)  
endProgress   | 动画停止(在整体动画的百分比)

### 关键字属性
`CATransition`关键字:  
type  

关键字 | 属性 |
----|------|
kCATransitionFade | 淡出  
kCATransitionMoveIn | 覆盖原图  
kCATransitionPush | 推出  
kCATransitionReveal | 底部显出来  

subtype  

关键字 | 属性 |
----|------|
kCATransitionFromRight |  右 |
kCATransitionFromLeft | 左 |
kCATransitionFromTop | 上 |
kCATransitionFromBottom | 下 | 

还有一种设置动画类型的方法，不用setSubtype，只用setType.  
[animation setType:@"suckEffect"];这里的suckEffect就是效果名称，可以用的效果主要有：

关键字 | 属性 |
----|------|
fade | 交叉淡化过渡
push | 新视图把旧视图推出去
moveIn | 新视图移到旧视图上面
reveal | 将旧视图移开,显示下面的新视图
cube | 立方体翻滚效果
oglFlip | 上下左右翻转效果
suckEffect | 收缩效果，如一块布被抽走
rippleEffect | 水滴效果
pageCurl | 向上翻页效果
pageUnCurl | 向下翻页效果
cameraIrisHollowOpen | 相机镜头打开效果
cameraIrisHollowClos | 相机镜头关闭效果  

### 6.1 改变导航栏文字

```objc
CATransition *animation = [CATransition animation];
animation.duration = 0.5;
animation.type = kCATransitionFade;//过渡效果
animation.subtype = kCATransitionFromRight;//过渡方向
animation.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut];//样式
[self.navigationController.navigationBar.layer addAnimation: animation forKey: @"changeTextTransition"];
self.navigationController.navigationBar.topItem.title = @"new标题";
```

### 6.2 切换视图

```objc
CATransition * transition = [CATransition animation];
transition.duration = 1.0;//动画间隔
transition.type = kCATransitionReveal;//oglFlip 主要种类，决定动画效果
transition.startProgress = 0.0;//开始
transition.endProgress = 1.0;//结束
transition.subtype = kCATransitionFromRight;//次要种类，决定动画方向
transition.timingFunction = [CAMediaTimingFunction functionWithName:kCAMediaTimingFunctionEaseInEaseOut];//时间函数
[self.viewRect.layer addAnimation:transition forKey:@"ToNext"];
 
srand((unsigned)time(0));
int i = rand() % 5;
    
self.view.backgroundColor = _arrayColors[i];
[[self.view layer] addAnimation:transition forKey:@"animation"];
```

##七. CALayer上动画的暂停和恢复
暂停:  

```objc
-(void)pauseLayer:(CALayer*)layer
{
	CFTimeInterval pausedTime = [layer convertTime:CACurrentMediaTime() fromLayer:nil];

	// 让CALayer的时间停止走动
	layer.speed = 0.0;
	// 让CALayer的时间停留在pausedTime这个时刻
	layer.timeOffset = pausedTime;
}
```
	
恢复  

```objc
-(void)resumeLayer:(CALayer*)layer
{
	CFTimeInterval pausedTime = layer.timeOffset;
	// 1. 让CALayer的时间继续行走
	layer.speed = 1.0;
	// 2. 取消上次记录的停留时刻
	layer.timeOffset = 0.0;
	// 3. 取消上次设置的时间
	layer.beginTime = 0.0;    
	// 4. 计算暂停的时间(这里也可以用CACurrentMediaTime()-pausedTime)
	CFTimeInterval timeSincePause = [layer convertTime:CACurrentMediaTime() fromLayer:nil] - pausedTime;
	// 5. 设置相对于父坐标系的开始时间(往后退timeSincePause)
	layer.beginTime = timeSincePause;
}
```

## 八. UIView的属性:

### 8.1 属性

关键字 | 属性 |
----|------|
frame | 视图框架 |
center | 视图位置  |
bounds | 视图⼤⼩ |
backgroundColor | 背景颜⾊ |
alpha | 视图透明度|
transform | 视图转换 |

### 8.2 UIView动画的设置

⽅法名 | 作⽤ |
----|------|
+(void)setAnimationDuration:(NSTimeInterval)duration; | 动画持续时间 |
+(void)setAnimationDelay:(NSTimeInterval)delay; | 动画开始前延时 |
+(void)setAnimationCurve:(UIViewAnimationCurve)curve; | 动画的速度曲线 |
+(void)setAnimationRepeatAutoreverses:(BOOL)repeatAutoreverses; | 动画反转 |
+(void)setAnimationRepeatCount:(float)repeatCount; | 动画反转的次数 |
+(void)setAnimationDelegate:(id)delegate; | 设置动画的代理 |
+(void)setAnimationWillStartSelector:(SEL)selector; | 动画开始的代理⽅法 |
+(void)setAnimationDidStopSelector:(SEL)selector; | 动画结束的代理⽅法 |
+(void)setAnimationBeginsFromCurrentState:(BOOL)fromCurrentState; | 动画从当前状态继续执⾏ |  

## 九. 参考文献
1.[iOS基础 - 核心动画](http://www.cnblogs.com/monicaios/p/3521575.html)   
2.[iOS官方文档guide](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/CoreAnimation_guide/Introduction/Introduction.html#//apple_ref/doc/uid/TP40004514)  
3.[iOS官方文档demo](https://developer.apple.com/library/ios/documentation/Cocoa/Conceptual/Animation_Types_Timing/Articles/PropertyAnimations.html)  
4.[iOS官方搜索地址](https://developer.apple.com/search/?q=animationwithkeypath)  
5.[CABasicAnimation的基本使用方法（移动·旋转·放大·缩小）](http://blog.csdn.net/iosevanhuang/article/details/14488239)  
6.[谈谈iOS Animation](http://geeklu.com/2012/09/animation-in-ios/)  
7.[CAKeyframeAnimation 关键帧动画的用法](http://blog.csdn.net/samuelltk/article/details/9048325)  
8.[CoreAnimation](http://www.jianshu.com/p/ee2d3a8b2d67)

## demo下载地址
[CoreAnimationDemo-向晨宇撰写](https://github.com/xcysuccess/coreanimation)