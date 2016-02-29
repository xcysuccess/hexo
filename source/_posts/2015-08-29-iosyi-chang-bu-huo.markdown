---
layout: post
title: "iOS异常捕获"
date: 2015-08-29 23:48:11 +0800
comments: true
categories: Debug 
description: "Crash分为两种，一种是由EXC_BAD_ACCESS引起的，原因是访问了不属于本进程的内存地址，有可能是访问已被释放的内存；另一种是未被捕获的Objective-C异常（NSException）" 
keywords: debug,异常
---

## 前言
今天在ios高级群**237305299**，有朋友问到iOS的异常捕捉的问题，这一块以前也没有研究过，趁此机会研究了一把。并写了一个demo，如有需要可以在文章最下面去下载。 
 
在阅读文章之前，建议大家在阅读完此篇文章后可以阅读[漫谈iOS Crash收集框架](http://nianxi.net/ios/ios-crash-reporter/)，了解一下原理。  

开发iOS应用，解决Crash问题始终是一个难题。Crash分为两种，一种是由EXC_BAD_ACCESS引起的，原因是访问了不属于本进程的内存地址，有可能是访问已被释放的内存；另一种是未被捕获的Objective-C异常（NSException），导致程序向自身发送了SIGABRT信号而崩溃。其实对于未捕获的Objective-C异常，我们是有办法将它记录下来的，如果日志记录得当，能够解决绝大部分崩溃的问题。这里对于UI线程与后台线程分别说明

## 一. 系统Crash  
对于系统Crash而引起的程序异常退出，可以通过UncaughtExceptionHandler机制捕获；也就是说在程序中catch以外的内容，被系统自带的错误处理而捕获。我们要做的就是用自定义的函数替代该ExceptionHandler即可。  


## 二. 处理signal
使用Objective-C的异常处理是不能得到signal的，如果要处理它，我们还要利用unix标准的signal机制，注册SIGABRT, SIGBUS, SIGSEGV等信号发生时的处理函数。该函数中我们可以输出栈信息，版本信息等其他一切我们所想要的。 
<!-- more -->  

### 下面是一些信号说明  
1)  **SIGHUP**     
本信号在用户终端连接(正常或非正常)结束时发出,   通常是在终端的控制进程结束时,   通知同一session内的各个作业,   这时它们与控制终端不再关联。  
登录Linux时，系统会分配给登录用户一个终端(Session)。在这个终端运行的所有程序，包括前台进程组和后台进程组，一般都属于这个     Session。当用户退出Linux登录时，前台进程组和后台有对终端输出的进程将会收到SIGHUP信号。这个信号的默认操作为终止进程，因此前台进   程组和后台有终端输出的进程就会中止。不过可以捕获这个信号，比如wget能捕获SIGHUP信号，并忽略它，这样就算退出了Linux登录，   wget也   能继续下载。
此外，对于与终端脱离关系的守护进程，这个信号用于通知它重新读取配置文件。   
2)   **SIGINT**    
程序终止(interrupt)信号,   在用户键入INTR字符(通常是Ctrl-C)时发出，用于通知前台进程组终止进程。  
3)   **SIGQUIT**    
和SIGINT类似,   但由QUIT字符(通常是Ctrl-\)来控制.   进程在因收到SIGQUIT退出时会产生core文件,   在这个意义上类似于一个程序错误信号。   
4)   **SIGILL**    
执行了非法指令.   通常是因为可执行文件本身出现错误,   或者试图执行数据段.   堆栈溢出时也有可能产生这个信号。   
5)   **SIGTRAP**    
由断点指令或其它trap指令产生.   由debugger使用。   
6)   **SIGABRT**    
调用abort函数生成的信号。   
7)   **SIGBUS**    
非法地址,   包括内存地址对齐(alignment)出错。比如访问一个四个字长的整数,   但其地址不是4的倍数。它与SIGSEGV的区别在于后者是由于对合法存储地址的非法访问触发的(如访问不属于自己存储空间或只读存储空间)。   
8)   **SIGFPE**    
在发生致命的算术运算错误时发出.   不仅包括浮点运算错误,   还包括溢出及除数为0等其它所有的算术的错误。   
9)   **SIGKILL**    
用来立即结束程序的运行.   本信号不能被阻塞、处理和忽略。如果管理员发现某个进程终止不了，可尝试发送这个信号。   
10)   **SIGUSR1**    
留给用户使用   
11)   **SIGSEGV**    
试图访问未分配给自己的内存,   或试图往没有写权限的内存地址写数据.   
12)   **SIGUSR2**    
留给用户使用   
13)   **SIGPIPE**    
管道破裂。这个信号通常在进程间通信产生，比如采用FIFO(管道)通信的两个进程，读管道没打开或者意外终止就往管道写，写进程会收到SIGPIPE信号。此外用Socket通信的两个进程，写进程在写Socket的时候，读进程已经终止。   
14)   **SIGALRM**    
时钟定时信号,   计算的是实际的时间或时钟时间.   alarm函数使用该信号.   
15)   **SIGTERM**    
程序结束(terminate)信号,   与SIGKILL不同的是该信号可以被阻塞和处理。通常用来要求程序自己正常退出，shell命令kill缺省产生这个信号。如果进程终止不了，我们才会尝试SIGKILL。   
17)   **SIGCHLD**    
子进程结束时,   父进程会收到这个信号。   
如果父进程没有处理这个信号，也没有等待(wait)子进程，子进程虽然终止，但是还会在内核进程表中占有表项，这时的子进程称为僵尸进程。这种情     况我们应该避免(父进程或者忽略SIGCHILD信号，或者捕捉它，或者wait它派生的子进程，或者父进程先终止，这时子进程的终止自动由init进程   来接管)。   
18)   **SIGCONT**    
让一个停止(stopped)的进程继续执行.   本信号不能被阻塞.   可以用一个handler来让程序在由stopped状态变为继续执行时完成特定的工作.   例如,   重新显示提示符   
19)   **SIGSTOP**    
停止(stopped)进程的执行.   注意它和terminate以及interrupt的区别:该进程还未结束,   只是暂停执行.   本信号不能被阻塞,   处理或忽略.   
20)   **SIGTSTP**    
停止进程的运行,   但该信号可以被处理和忽略.   用户键入SUSP字符时(通常是Ctrl-Z)发出这个信号
21)   **SIGTTIN**    
当后台作业要从用户终端读数据时,   该作业中的所有进程会收到SIGTTIN信号.   缺省时这些进程会停止执行.   
22)   **SIGTTOU**    
类似于SIGTTIN,   但在写终端(或修改终端模式)时收到.   
23)   **SIGURG**    
有”紧急”数据或out-of-band数据到达socket时产生.   
24)   **SIGXCPU**    
超过CPU时间资源限制.   这个限制可以由getrlimit/setrlimit来读取/改变。   
25)   **SIGXFSZ**    
当进程企图扩大文件以至于超过文件大小资源限制。   
26)   **SIGVTALRM**    
虚拟时钟信号.   类似于SIGALRM,   但是计算的是该进程占用的CPU时间.   
27)   **SIGPROF**    
类似于SIGALRM/SIGVTALRM,   但包括该进程用的CPU时间以及系统调用的时间.   
28)   **SIGWINCH**    
窗口大小改变时发出.   
29)   **SIGIO**    
文件描述符准备就绪,   可以开始进行输入/输出操作.   
30)   **SIGPWR**    
Power   failure   
31)   **SIGSYS**  
非法的系统调用。
   
### 关键点注意
* 在以上列出的信号中，程序不可捕获、阻塞或忽略的信号有：**SIGKILL,SIGSTOP**
* 不能恢复至默认动作的信号有：SIGILL,SIGTRAP  
* 默认会导致进程流产的信号有：SIGABRT,SIGBUS,SIGFPE,SIGILL,SIGIOT,SIGQUIT,SIGSEGV,SIGTRAP,SIGXCPU,SIGXFSZ 
默认会导致进程退出的信号有:  
* SIGALRM,SIGHUP,SIGINT,SIGKILL,SIGPIPE,SIGPOLL,SIGPROF,SIGSYS,SIGTERM,SIGUSR1,SIGUSR2,SIGVTALRM 
* 默认会导致进程停止的信号有：SIGSTOP,SIGTSTP,SIGTTIN,SIGTTOU 
* 默认进程忽略的信号有：SIGCHLD,SIGPWR,SIGURG,SIGWINCH
* 此外，SIGIO在SVR4是退出，在4.3BSD中是忽略；SIGCONT在进程挂起时是继续，否则是忽略，不能被阻塞。

## 三. 实战
1.`AppDelegate.m`中  

```objc
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
// Override point for customization after application launch.
    
InstallSignalHandler();//信号量截断
	InstallUncaughtExceptionHandler();//系统异常捕获
    
return YES;
}
```
2.`SignalHandler.m`的实现  

```objc
void SignalExceptionHandler(int signal)
{
    NSMutableString *mstr = [[NSMutableString alloc] init];
    [mstr appendString:@"Stack:\n"];
    void* callstack[128];
    int i, frames = backtrace(callstack, 128);
    char** strs = backtrace_symbols(callstack, frames);
    for (i = 0; i <frames; ++i) {
        [mstr appendFormat:@"%s\n", strs[i]];
    }
    [SignalHandler saveCreash:mstr];

}

void InstallSignalHandler(void)
{
    signal(SIGHUP, SignalExceptionHandler);
    signal(SIGINT, SignalExceptionHandler);
    signal(SIGQUIT, SignalExceptionHandler);
    
    signal(SIGABRT, SignalExceptionHandler);
    signal(SIGILL, SignalExceptionHandler);
    signal(SIGSEGV, SignalExceptionHandler);
    signal(SIGFPE, SignalExceptionHandler);
    signal(SIGBUS, SignalExceptionHandler);
    signal(SIGPIPE, SignalExceptionHandler);
}
```

有关错误类型可以看上面的说明，SignalExceptionHandler是信号出错时候的回调。当有信号出错的时候，可以回调到这个方法

3.`UncaughtExceptionHandler.m`的实现  

```objc
	void HandleException(NSException *exception)
	{
	    // 异常的堆栈信息
	    NSArray *stackArray = [exception callStackSymbols];
	    // 出现异常的原因
	    NSString *reason = [exception reason];
	    // 异常名称
	    NSString *name = [exception name];
	    NSString *exceptionInfo = [NSString stringWithFormat:@"Exception reason：%@\nException name：%@\nException stack：%@",name, reason, stackArray];
	    NSLog(@"%@", exceptionInfo);
	    [UncaughtExceptionHandler saveCreash:exceptionInfo];
	}

	void InstallUncaughtExceptionHandler(void)
	{
	    NSSetUncaughtExceptionHandler(&HandleException);
	}
```
4.**测试**--踩坑关键
**这里最关键的一步，SignalHandler不要在debug环境下测试。因为系统的debug会优先去拦截。我们要运行一次后，关闭debug状态。应该直接在模拟器上点击我们build上去的app去运行。而UncaughtExceptionHandler可以在调试状态下捕捉**   

```objc
	- (IBAction)buttonClick:(UIButton *)sender {
	//1.信号量
	    Test *pTest = {1,2};
	    free(pTest);//导致SIGABRT的错误，因为内存中根本就没有这个空间，哪来的free，就在栈中的对象而已
	    pTest->a = 5;
	}
	- (IBAction)buttonOCException:(UIButton *)sender
	{
	    //2.ios崩溃
	    NSArray *array= @[@"tom",@"xxx",@"ooo"];
	    [array objectAtIndex:5];
	}
```

!["文件结构"](/images/iosCrashException/1.png)
!["操作"](/images/iosCrashException/2.png)  

## 四. Crash Callstack分析 - 进⼀一步分析

属性 | 	说明 |
----|------|
0x8badf00d | 在启动、终⽌止应⽤用或响应系统事件花费过⻓长时间,意为“ate bad food”。|
0xdeadfa11 | ⽤用户强制退出,意为“dead fall”。(系统⽆无响应时,⽤用户按电源开关和HOME)|
0xbaaaaaad | ⽤用户按住Home键和⾳音量键,获取当前内存状态,不代表崩溃 |
0xbad22222 | VoIP应⽤用因为恢复得太频繁导致crash |
0xc00010ff | 因为太烫了被干掉,意为“cool off”  |
0xdead10cc | 因为在后台时仍然占据系统资源(⽐比如通讯录)被干掉,意为“dead lock” | 
 
## 五. demo地址
[iOSCrashUncaught下载](https://github.com/xcysuccess/iOSCrashUncaught)

## 六. 参考文献
1.[程序crash后的调试技巧](http://www.yifeiyang.net/iphone-development-skills-of-debugging-articles-3-crash-after-debugging-skills-program/#sec7)  
2.[iOS开发socket程序被SIGPIPE信号Terminate的问题](http://arthurchen.blog.51cto.com/2483760/736181)  
3.[美女念茜](http://nianxi.net/ios/ios-crash-reporter/)  
4.[如何定位Obj-C野指针随机Crash(一)：先提高野指针Crash率](http://bugly.qq.com/blog/?p=200)  
5.[如何定位Obj-C野指针随机Crash(二)：让非必现Crash变成必现](http://bugly.qq.com/blog/?p=308)  
6.[如何定位Obj-C野指针随机Crash(三)：加点黑科技让Crash自报家门](http://bugly.qq.com/blog/?p=335)
