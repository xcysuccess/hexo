---
layout: post
title: "iOS异步读写总结之NSDictionary"
date: 2015-10-12 21:56:06 +0800
comments: true
categories: debug 
description: "多线程异步读写崩溃总结" 
keywords: nsdictionary,异步,多线程 
---

## 前言

在介绍`异步读写`之前，先介绍一下多线程下崩溃的容易崩溃的原因，以及如何进行**`同步读，异步写`**

## 一. 多线程常见崩溃

### MRC下:

```objc
-(id) getSafeObjectForKey:(NSString *) bindingKey
{
    @synchronized(self.dic) {
        if(bindingKey){
            return [self.dic objectForKey:bindingKey];
        }
    }
    return nil;
}

-(void) setSafeObject:(id) object forKey:(NSString*) bindingKey
{
    @synchronized(self.dic) {
        if(bindingKey && object){
            [self.dic setObject:object forKey:bindingKey];
        }
    }
}

-(void) testSyncizedDictionary
{
    for (int i = 0; i < 1000000; i++) {
        dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
            //多线程写
            [self setSafeObject:[NSString stringWithFormat:@"86+111111111   %i", i] forKey:KEY];
        });
        //主线程读
        NSString *result = [self getSafeObjectForKey:KEY];
        NSLog(@"get string: %@, length : %lu", result, result.length);
    }
}
```

!["NSDictionary"](/images/nsdictionary/1.png)  <!-- more -->  

原因分析:程序还没执行到`NSLog`那一行的时候，`get`出来的一个值,在多线程中执行了`set`方法执行导致`key`对应的`result`对释放掉了。此时执行到`NSLog`,导致了崩溃.  

### 修改方案:

```objc
	//主线程读
	NSString *result = [[self getSafeObjectForKey:KEY] retain];
	NSLog(@"get string: %@, length : %lu", result, result.length);
	[result release];
```

在读取的时候`retain`一下，后续在`release`,还是会崩溃。因为在retain之前很可能野掉了，此时retain也只是对野掉的的指针

## 二. 最终修改方案

### MRC下

```objc
#pragma mark- 用GCD的方式去实现
-(id) getRightObjectForKey:(NSString *) bindingKey
{
    if (!bindingKey) {
        return nil;
    }
    __block id result = nil;
    dispatch_sync(self.ioQueue, ^{
        result = [[_dic objectForKey:bindingKey] retain]; //关键点1
    });
    
    return [result autorelease]; //关键点2

}

-(void) setRightObject:(id) object forKey:(NSString*) bindingKey
{
    bindingKey = [bindingKey copy];
    dispatch_barrier_async(self.ioQueue, ^{
        if(bindingKey && object){
            [_dic setObject:object forKey:bindingKey];
        }
    });
}

-(void) testSyncizedDictionaryRight
{

    for (int i = 0; i < 1000000; i++) {
        dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
            //多线程写
            [self setRightObject:[NSString stringWithFormat:@"86+111111111   %i", i] forKey:KEY];
        });
        //主线程读
        NSString *result = [[self getRightObjectForKey:KEY] retain];
        NSLog(@"get string: %@, length : %lu", result, result.length);
        [result release];
    }
}
```

请看关键点1和关键点2，如果没有`retain`和`autorelease`,还是会崩溃。原因同上。
  
### ARC下:

怎么操作都不崩溃。不过还是建议写法  

```objc
@property(nonatomic,strong) NSMutableDictionary *dic;  
@property(nonatomic,strong) dispatch_queue_t ioQueue;
  
self.ioQueue = dispatch_queue_create("ioQueue", DISPATCH_QUEUE_CONCURRENT);  
self.dic = [NSMutableDictionary dictionaryWithCapacity:1];
   
#pragma mark- 用GCD的方式去实现
-(id) getRightObjectForKey:(NSString *) bindingKey
{
    if (!bindingKey) {
        return nil;
    }
    __block id result = nil;
    dispatch_sync(self.ioQueue, ^{
        result = [_dic objectForKey:bindingKey];     	    });
    
    return result;
}

-(void) setRightObject:(id) object forKey:(NSString*) bindingKey
{
    bindingKey = [bindingKey copy];
    dispatch_barrier_async(self.ioQueue, ^{
        if(bindingKey && object){
            [_dic setObject:object forKey:bindingKey];
        }
    });
}

-(void) testSyncizedDictionaryRight
{	    
    for (int i = 0; i < 1000000; i++) {
        dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
            //多线程写
            [self setRightObject:[NSString stringWithFormat:@"86+111111111   %i", i] forKey:KEY];
        });
        //主线程读
        NSString *result = [self getRightObjectForKey:KEY];
        NSLog(@"get string: %@, length : %lu", result, result.length);
    }
}
```	

## 三. 异步读写

创建一个串行队列。然后gcd结合锁去执行  
 
```objc
@property(nonatomic,strong) dispatch_queue_t asynModelQueue;
	
self.asynModelQueue = dispatch_queue_create("AsynModelQueue", DISPATCH_QUEUE_SERIAL);

#pragma mark- 用GCD去异步读写
-(void) getAsyncRightObjectForKey:(NSString *) bindingKey block:(void(^)(NSMutableDictionary* modelDicCache)) modelCallBack
{
    if (!bindingKey && modelCallBack) {
        modelCallBack(nil);
        return;
    }
    __block id result = nil;
    dispatch_async(self.asynModelQueue, ^{
        @synchronized(self) {//@synchronized是为了多一层保护
            result = [_dic objectForKey:bindingKey];     //由于self.bindingInfo重写过，此处不能用self.bindingInfo，否则造成死锁
            modelCallBack(result);
        }
    });
}

-(void) setAsyncRightObject:(id) object forKey:(NSString*) bindingKey
{
    bindingKey = [bindingKey copy];
    dispatch_barrier_async(self.asynModelQueue, ^{
        @synchronized(self) {//@synchronized是为了多一层保护
            if(bindingKey && object){
                [_dic setObject:object forKey:bindingKey];
            }
        }
    });
}

-(void) testAsyncSyncizedDictionaryRight
{
    for (int i = 0; i < 1000000; i++) {
        dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0), ^{
            //多线程写
            [self setRightObject:[NSString stringWithFormat:@"86+111111111   %i", i] forKey:KEY];
        });
        //主线程读
        NSString *result = [self getRightObjectForKey:KEY];
        NSLog(@"get string: %@, length : %lu", result, result.length);
    }
}
```	

## demo下载 
[demo](http://share.weiyun.com/51e4fa7790da21af0cdaefbc1a451bd4)