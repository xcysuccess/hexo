---
layout: post
title: "Block那些事"
date: 2016-01-31 21:53:03 +0800
comments: true
categories: Technic
description: "这里讨论了从Block的基本用法，实战应用，以及最后的测试案例，供大家学习参考" 
keywords: block
---

## 一. Block入门

### 1.概念
Block是iOS4.0+ 和Mac OS X 10.6+ 引进的对C语言的扩展，用来实现匿名函数的特性  
闭包是一个能够访问其他函数内部变量的函数.  

### 2.基本的用法
!["Block语法"](/images/block/1.png)  

每次写block都很绕，语法很麻烦。好吧，再此记录一下。  

**1.作为方法时**  

```objc
	- (void )testGlobalBlock:(NSString*) url
           success:(void (^)(BOOL isSuccess))success
           failure:(void (^)(BOOL isSuccess))failure;
```
**2.作为成员变量进行声明时**  

```objc
	typedef void (^SUCCESSBLOCK)(BOOL isSuccess);
	typedef void (^FAILEDBLOCK)(BOOL isSuccess);

	@interface XXEngine()
	@property(nonatomic,copy) SUCCESSBLOCK successBlock;
	@property(nonatomic,copy) FAILEDBLOCK failedBlock;
	@end
```

**3.写在函数内部**  

```objc
	void (^_block)() = ^{
        _a = 10;
    };
```
### 3.内存的5个区
这里先普及一个基本的5个概念，内存的5个区。否则在具体讲堆栈的时候怕大家会不理解。  

| 作用 | 名称 |
| --- | --- |
| 栈区（stack） | 由编译器自动分配释放，存放函数的参数值，局部变量的值等 |
| 堆区（heap） | 一般由程序员分配释放， 若程序员不释放，程序结束时可能由OS回收 | 
| 全局区（静态区）（static） | 全局变量和静态变量的存储是放在一块的，初始化的全局变量和静态变量在一块区域，未初始化的全局变量和未初始化的静态变量在相邻的另一块区域。程序结束后由系统释放 | 
| 文字常量区 | 常量字符串就是放在这里的。程序结束后由系统释放 |
| 程序代码区 | 存放函数体的二进制代码 |

<!-- more -->  
	
### 4.使用场景
1. 任务完成时回调处理
2. 消息监听回调处理
3. 错误回调处理
4. 枚举回调
5. 视图动画、变换
6. 排序


## 二. Block原理

### 1.Block的三种类型

| 作用 | 名称 |
| --- | --- |
| NSConcreteGlobalBlock | 全局静态，不访问外部变量的时候就是NSConcreteGlobalBlock |
| NSConcreteStackBlock  | 保存在栈中，出花括号会被销毁 | 
| NSConcreteMallocBlock | 保存在堆中的block,当引用计数为0的时候会被销毁| 

### 2.类型判断
如何判断在哪个区,我们分ARC和MRC两种情况

```objc
    int i = 10;
    void (^block)() = ^{printf("%d",i);} ;
    __weak void (^weakBlock)() = ^{printf("%d",i);}; //被weak修饰之后还是stack
    void (^globalStack)() = ^{};

    NSLog(@"%@", ^{printf("%d",i);});
    NSLog(@"%@", block);
    NSLog(@"%@", weakBlock);
    NSLog(@"%@", globalStack);
```

**1.ARC下**
```objc
	testARC[76639:507728] <__NSStackBlock__: 0x7fff54c9fbc8>
	testARC[76639:507728] <__NSMallocBlock__: 0x7fdaaae161c0>
	testARC[76639:507728] <__NSStackBlock__: 0x7fff54c9fbf8>
	testARC[76639:507728] <__NSGlobalBlock__: 0x10af60230>
```
**2.MRC下**
```objc	
	testARC[76639:507728] <__NSStackBlock__: 0x7fff54c9fba0>
	testARC[76639:507728] <__NSStackBlock__: 0x7fff54c9fc00>
	testARC[76639:507728] <__NSStackBlock__: 0x7fff54c9fbd0>
	testARC[76639:507728] <__NSGlobalBlock__: 0x10af60130>
```
**3.判断原则**  

**1.ARC下判断原则**  
!["Block语法"](/images/block/2.png)  

**2.MRC下判断原则**  
!["Block语法"](/images/block/3.png)  
	
**3.ARC与MRC区别**  
ARC引用外部对象会自动copy，MRC则不会。所以MRC要将栈对象变成堆对象只要执行一次copy即可。

### 3.循环引用

**1.造成循环引用的实例**  

```objc
	@implementation Person
	{
	    int _a;
	    void (^_block)();
	}
	- (void)test
	{
	  void (^_block)() = ^{
	        _a = 10;
	    };
	}
	@end
```

此时_a在用clang编译后可以很明显看到有会self的身影，所以此时我们可以将其理解成self.a（便于记忆),此时Person持有block,block持有self，造成了循环引用  

**2. 解决方式**  

```objc
	- (void)test
	{
	  __weak typeof(self) weakSelf = self;
	  void (^_block)() = ^{
	        weakSelf->a = 10;
	    };
	}
```
**3.系统Block**  

**1.经验谈之GCD/UIView的系统动画的Block中，为何可以写self？**  
因为self并没有对其进行持有，循环引用的原理是两个对象之间相互有持有关系，现在仅仅是GCD持有self，但是self并没有持有GCD，所以是没有问题的。（当GCD对象为其成员变量时才具有持有的关系）


### 4.__block原理

**1.我们先看如下代码**  

```objc
	void test()
	{
	    __block int a;
	    ^{
	        a = 10;
	    };
	}
```

**2.用`clang -re-write testBlock.c`命令后**  

看关键代码如下:  

```objc
	struct __Block_byref_a_0 {
	  void *__isa;
	__Block_byref_a_0 *__forwarding;
	 int __flags;
	 int __size;
	 int a;
	};

	struct __test_block_impl_0 {
	  struct __block_impl impl;
	  struct __test_block_desc_0* Desc;
	  __Block_byref_a_0 *a; // by ref
	  __test_block_impl_0(void *fp, struct __test_block_desc_0 *desc, __Block_byref_a_0 *_a, int flags=0) : a(_a->__forwarding) {
	    impl.isa = &_NSConcreteStackBlock;
	    impl.Flags = flags;
	    impl.FuncPtr = fp;
	    Desc = desc;
	  }
	};
	static void __test_block_func_0(struct __test_block_impl_0 *__cself) {
	  __Block_byref_a_0 *a = __cself->a; // bound by ref

	        (a->__forwarding->a) = 10;
	    }
	static void __test_block_copy_0(struct __test_block_impl_0*dst, struct __test_block_impl_0*src) {_Block_object_assign((void*)&dst->a, (void*)src->a, 8/*BLOCK_FIELD_IS_BYREF*/);}

	static void __test_block_dispose_0(struct __test_block_impl_0*src) {_Block_object_dispose((void*)src->a, 8/*BLOCK_FIELD_IS_BYREF*/);}

	static struct __test_block_desc_0 {
	  size_t reserved;
	  size_t Block_size;
	  void (*copy)(struct __test_block_impl_0*, struct __test_block_impl_0*);
	  void (*dispose)(struct __test_block_impl_0*);
	} __test_block_desc_0_DATA = { 0, sizeof(struct __test_block_impl_0), __test_block_copy_0, __test_block_dispose_0};
	void test()
	{
	    __attribute__((__blocks__(byref))) __Block_byref_a_0 a = {(void*)0,(__Block_byref_a_0 *)&a, 0, sizeof(__Block_byref_a_0)};
	;
	    ((void (*)())&__test_block_impl_0((void *)__test_block_func_0, &__test_block_desc_0_DATA, (__Block_byref_a_0 *)&a, 570425344));
	}
```

**3.结论**  

我们观察__Block_byref_a_0结构体，这个结构体中含有isa指针，所以也是一个对象，它是用来包装局部变量a的。当block被copy到堆中时，__Person__test_block_impl_0的拷贝辅助函数__Person__test_block_copy_0会将__Block_byref_a_0拷贝至堆中，所以即使局部变量所在堆被销毁，block依然能对堆中的局部变量进行操作。其中__Block_byref_a_0成员指针__forwarding用来指向它在堆中的拷贝


## 三. 利用Block如何实现一对多的

### 1.一种是直接Block
这里我直接copy一段`AFNetworking`的代码，这个库写的太好了，大家有空可以参考一下

```objc
	- (NSURLSessionDataTask *)dataTaskWithHTTPMethod:(NSString *)method
                                       URLString:(NSString *)URLString
                                      parameters:(id)parameters
                                  uploadProgress:(nullable void (^)(NSProgress *uploadProgress)) uploadProgress
                                downloadProgress:(nullable void (^)(NSProgress *downloadProgress)) downloadProgress
                                         success:(void (^)(NSURLSessionDataTask *, id))success
                                         failure:(void (^)(NSURLSessionDataTask *, NSError *))failure
	{
	    NSError *serializationError = nil;
	    NSMutableURLRequest *request = [self.requestSerializer requestWithMethod:method URLString:[[NSURL URLWithString:URLString relativeToURL:self.baseURL] absoluteString] parameters:parameters error:&serializationError];
	    if (serializationError) {
	        if (failure) {
		#pragma clang diagnostic push
		#pragma clang diagnostic ignored "-Wgnu"
	            dispatch_async(self.completionQueue ?: dispatch_get_main_queue(), 			  ^{
	                failure(nil, serializationError);
	            });
		#pragma clang diagnostic pop
	        }

	        return nil;
	    }

	    __block NSURLSessionDataTask *dataTask = nil;
	    dataTask = [self dataTaskWithRequest:request
	                          uploadProgress:uploadProgress
	                        downloadProgress:downloadProgress
	                       completionHandler:^(NSURLResponse * __unused response, id responseObject, NSError *error) {
	        if (error) {
	            if (failure) {
	                failure(dataTask, error);
	            }
	        } else {
	            if (success) {
	                success(dataTask, responseObject);
	            }
	        }
	    }];

	    return dataTask;
	}
```

### 2.利用对象做Block
!["Block语法"](/images/block/4.png)  

现在有`RequestObject`和`XXNetEngine`两个文件.  

**1.RequestObject**  

```objc
	#import <Foundation/Foundation.h>
	typedef void (^ReqSuccessBlock)(BOOL isSuccess);
	typedef void (^FailedBlock)(BOOL isSuccess);

	@interface RequestObject : NSObject
	@property(nonatomic,copy) ReqSuccessBlock successBlock;
	@property(nonatomic,copy) FailedBlock failedBlock;
	@property(nonatomic,assign) NSInteger CMD;
	@property(nonatomic,assign) NSInteger seq;
	@end	
```

**2.XXNetEngine**  

```objc
	#import "XXNetEngine.h"
	#import "RequestObject.h"

	@interface XXNetEngine()
	@property (nonatomic, strong) NSMutableDictionary *requestDict;
	@end


	@implementation XXNetEngine

	-(instancetype)init
	{
	    if(self = [super init]){
	        self.requestDict = [NSMutableDictionary dictionary];
	    }
	    return self;
	}
	+(instancetype) sharedInstance
	{
	    static XXNetEngine* instance = nil;
	    
	    static dispatch_once_t onceToken;
	    dispatch_once(&onceToken, ^{
	        instance = [[XXNetEngine alloc] init];
	    });
	    
	    return instance;
	}

	-(void)requestData:(RequestObject*) reqObject
	      successBlock:(void (^)(BOOL isSuccess))successBlock
	      failureBlock:(void (^)(BOOL isSuccess))failureBlock
	{
	    if ( reqObject && successBlock && failureBlock ) {
	        dispatch_async(dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_LOW, 0), ^{
	            //requestSeq是一个随机数
	            int requestSeq = arc4random() % 100;;
	            
	            //网络操作
	            reqObject.successBlock = successBlock;
	            reqObject.failedBlock = failureBlock;
	            reqObject.seq = requestSeq;
	            
	            [self addRequestItem:reqObject seq:requestSeq];
	            
	            //模拟一个2秒的网络操作
	            dispatch_after(dispatch_time(DISPATCH_TIME_NOW, (int64_t)(2 * NSEC_PER_SEC)), dispatch_get_main_queue(), ^{
	                [self didReceiveData:reqObject.seq];
	            });
	        });
	    }
	}

	- (void) didReceiveData:(NSInteger) seq
	{
	    RequestObject *request = [self getRequestItem:seq];
	    if ( request ) {
	        [self removeRequestItem:seq];
	        
	        dispatch_async(dispatch_get_main_queue(), ^{
	            if ( NULL != request.successBlock ) {
	                request.successBlock(YES);
	            }
	        });
	    }

	}


	//用命令字做参数传递
	- (RequestObject *) getRequestItem:(NSInteger)seq
	{
	    RequestObject *request = nil;
	    @synchronized(self) {

	        NSNumber *numSeq = [NSNumber numberWithInteger:seq];
	        request = [self.requestDict objectForKey:numSeq];
	    }
	    
	    return request;
	}

	- (void) addRequestItem:(RequestObject *)request seq:(NSInteger)seq
	{
	    @synchronized(self) {
	        if ( request && (0!=seq) ) {
	            NSNumber *numSeq = [NSNumber numberWithInteger:seq];
	            [self.requestDict setObject:request forKey:numSeq];
	        }
	    }
	}

	- (void) removeRequestItem:(NSInteger)seq
	{
	    @synchronized(self) {
	        NSNumber *numSeq = [NSNumber numberWithInteger:seq];
	        [self.requestDict removeObjectForKey:numSeq];
	    }
	}

	@end
```

**3.输出结果**  

```objc
	2016-02-02 00:14:59.824 testARC[1232:45931] XXNetEngine successBlock
```

## 四. Delegate、Notification、Block

### 1.优缺点

| 名称 | 优点 | 缺点 |
| --- | --- | --- |
| Notification | 1.使用简单，代码精简。2.解决了同时向多个对象监听相应的问题。3.传值方便快捷，Context自身携带相应的内容。 | 1.使用完毕后，要时刻记得注销通知，否则将出现不可预见的crash。2.key不够安全，编译器不会监测是否被通知中心正确处理。3.调试的时候动作的跟踪将很难进行。4.当使用者向通知中心发送通知的时候，并不能获得任何反馈信息。5.需要一个第三方的对象来做监听者与被监听者的中介。 |
| Delegate | 1.减少代码的耦合性，使事件监听和事件处理相分离。2.清晰的语法定义，减少维护成本，较强的代码可读性。3.不需要创建第三方来监听事件和传输数据。 | 1.实现委托的代码过程比较繁琐。2.当实现跨层传值监听的时候将加大代码的耦合性，并且程序的层次结构将变的混乱。3.当对多个对象同时传值响应的时候，委托的易用性将大大降低。 |
| Block | 1.语法简洁，实现回调不需要显示的调用方法，代码更为紧凑。2.增强代码的可读性和可维护性。3.配合GCD优秀的解决多线程问题。 | 1.Block中得代码将自动进行一次retain操作，容易造成内存泄露2.Block内默认引用为强引用，容易造成循环引用。|

### 2.个人见解
如果方法过多，一般大于3个的时候，用delegate，因为此时用block，代码将很难维护。比如UITableViewDelegate等等UI组建，都是用的delegate。其他方式用block。  
Notification能不用的时候尽量不用，缺点太多明显，增加调试难度。在工程大的情况下，极其难以维护。当然有些情况下也是必不可少的，比如观察者模式.



## 五. 测试

### 1.测试一

快来看看大家现在对Block的理解吧
[Block测试题, 从唐巧大神博客看到的转载链接](http://blog.parse.com/learn/engineering/objective-c-blocks-quiz/)

### 2.测试二
现在ViewController中有一个XXEngine对象

**1.XXEngine代码如下**  

```objc
	typedef void (^SUCCESSBLOCK)(BOOL isSuccess);
	typedef void (^FAILEDBLOCK)(BOOL isSuccess);

	@interface XXEngine()
	@property(nonatomic,copy) SUCCESSBLOCK successBlock;
	@property(nonatomic,copy) FAILEDBLOCK failedBlock;
	@end

	@implementation XXEngine

	-(instancetype)init
	{
	    if (self = [super init]) {
	        NSLog(@"XXEngine init");
	    }
	    return self;
	}

	-(void)dealloc
	{
	    NSLog(@"XXEngine dealloc");
	}

	- (void )testGlobalBlock:(NSString*) url
	           success:(void (^)(BOOL isSuccess))success
	           failure:(void (^)(BOOL isSuccess))failure
	{
	    //设置时间
	    double delayInSeconds = 2.0;
	    dispatch_time_t delayInNanoSeconds =
	    dispatch_time(DISPATCH_TIME_NOW, delayInSeconds * NSEC_PER_SEC);
	    
	    dispatch_queue_t concurrentQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
	    dispatch_after(delayInNanoSeconds, concurrentQueue, ^(void){
	        NSLog(@"网络操作完成");
	        if(success){
	            NSLog(@"%@",success);
	            success(YES);
	        }
	    });
	}

	- (void )testMallocBlock:(NSString*) url
	                 success:(void (^)(BOOL isSuccess))success
	                 failure:(void (^)(BOOL isSuccess))failure
	{
	    self.successBlock = success;
	    self.failedBlock = failure;
	    
	    //设置时间
	    double delayInSeconds = 2.0;
	    dispatch_time_t delayInNanoSeconds =
	    dispatch_time(DISPATCH_TIME_NOW, delayInSeconds * NSEC_PER_SEC);
	    
	    dispatch_queue_t concurrentQueue = dispatch_get_global_queue(DISPATCH_QUEUE_PRIORITY_DEFAULT, 0);
	    dispatch_after(delayInNanoSeconds, concurrentQueue, ^(void){
	        NSLog(@"网络操作完成");
	        if(self.successBlock){
	            NSLog(@"%@",self.successBlock);
	            self.successBlock(YES);
	        }
	    });
	}
```

**2.我们在ViewController中做如下处理(1)**  

```objc
	- (IBAction)testXXEngine:(UIButton *)sender {
	    XXEngine *xxEngine = [[XXEngine alloc] init];
	    [xxEngine testGlobalBlock:nil success:^(BOOL isSuccess) {
	        NSLog(@"2秒已到");
	    } failure:^(BOOL isSuccess) {
	        NSLog(@"2秒结束");
	    }];
		}
```

此时的输出结果大家能想到吗？这里公布答案：  

```objc
	2016-02-01 23:10:24.100 testARC[699:16506] XXEngine init
	2016-02-01 23:10:24.101 testARC[699:16506] XXEngine dealloc
	2016-02-01 23:10:26.101 testARC[699:16615] 网络操作完成
	2016-02-01 23:10:26.102 testARC[699:16615] <__NSGlobalBlock__: 0x102bbd110>
	2016-02-01 23:10:26.102 testARC[699:16615] 2秒已到
```

**3.我们在ViewController中做如下处理(2)**  

```objc
	- (IBAction)testXXEngineMalloc:(id)sender {
    XXEngine *xxEngine = [[XXEngine alloc] init];
    [xxEngine testMallocBlock:nil success:^(BOOL isSuccess) {
	        NSLog(@"2秒已到");
	    } failure:^(BOOL isSuccess) {
	        NSLog(@"2秒结束");
	    }];
	}
```
此时的输出结果大家能想到吗？这里公布答案:  

```objc
	2016-02-01 23:11:16.416 testARC[699:16506] XXEngine init
	2016-02-01 23:11:18.417 testARC[699:17079] 网络操作完成
	2016-02-01 23:11:18.417 testARC[699:17079] <__NSGlobalBlock__: 0x102bbd190>
	2016-02-01 23:11:18.417 testARC[699:17079] 2秒已到
	2016-02-01 23:11:18.417 testARC[699:17079] XXEngine dealloc
```
**4.结论**  

在（1）中，success是一个NSGlobalBlock对象，dispatch对success做了持有操作，GCD我们可以认为是一个系统单例对象，此时XXEngine虽然被释放了，但是success被GCD持有了一份，也就是引用技术+1，所以success这个block不会被释放。GCD持有的对象会被拷贝到堆中，现在GCD执行完成，堆中对象要进行释放，所以知道success完成后，GCD释放。Block可以回调的。  

在（2）中，GCD对XXEngine进行了持有，也就是引用技术+1，此时ViewController执行完对XXEngine引用技术-1，但是还是无法释放XXEngine，等到GCD结束之后，对XXEngine引用技术-1，此时会进行XXEngine的dealloc

## 六. Demo下载
[testBlock](http://share.weiyun.com/2c416b16d2edb92e70382f40a1f7d910)


## 七. 参考资料
1.[Delegate,Notification,Block](http://www.jianshu.com/p/1d92342c795c)  
2.[AFNetworking 3.0迁移指南](http://www.jianshu.com/p/047463a7ce9b)  
3.[Block技巧与底层解析](http://www.jianshu.com/p/51d04b7639f1)  
4.[谈Objective-C Block的实现](http://blog.devtang.com/blog/2013/07/28/a-look-inside-blocks/)-->