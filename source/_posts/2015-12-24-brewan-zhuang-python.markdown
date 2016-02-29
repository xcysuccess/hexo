---
layout: post
title: "Mac下brew安装python3"
date: 2015-12-24 19:33:56 +0800
comments: true
categories: python
description: "python安装以及遇到的一些问题" 
keywords: python
---

* list element with functor item
{:toc}

## 一. brew安装python3

### 1. 推荐安装终端
[item2](https://www.iterm2.com/)

### 2.brew安装
[官网](h ttp://brew.sh/)

### 3. 安装之前建议先下载`xcode`
里面会有很多工具，否则brew 安装会很蛋疼。 

```objc
xcode-select --install
```
	
### 4. 安装python3

```objc  
brew install python3
```

### 5. 查看当前python

```objc
➜  ~  which python3
/usr/local/bin/python3
➜  ~  which python
/usr/bin/python
➜  ~  ls -l /usr/bin/python  //查看python软链接,里面有->符号就代表有软链接
```

### 6. 移除python2,链接指向python3

```objc  
➜  ~  sudo mv /usr/bin/python /usr/bin/python2
Password:
➜  ~  which python3
/usr/local/bin/python3
➜  ~  ln -s python /usr/local/bin/python3
ln: /usr/local/bin/python3: File exists
➜  ~  ln -s /usr/local/bin/python3 /usr/bin/python
ln: /usr/bin/python: Permission denied
➜  ~  sudo ln -s /usr/local/bin/python3 /usr/bin/python
```

### 7. 最后大功告成

```objc
➜  ~  python
Python 3.4.3 (default, May  1 2015, 19:14:18) 
[GCC 4.2.1 Compatible Apple LLVM 6.1.0 (clang-602.0.49)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>> 
```

## eg: 权限错误
如果遇到`brew install`过程中有什么权限错误，可以输入命令  

```objc
sudo chown -R `whoami` /usr/local
```
## 二. Python多版本共存之pyenv
参考地址:[链接](http://seisman.info/python-pyenv.html)  
`pyenv`是一个专门为`python`准备的一个安装器,非常全面  


## 三. 参考文献
1.[Python教程](http://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000)
