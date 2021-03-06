---
layout: post
title: "字体入门篇"
date: 2015-08-24 16:00:19 +0800
comments: true
categories: Font 
description: "曾工作于wps负责ios版的文字排版，对文字的乱七八糟的编码常常感到绝望，感谢前人的文章和积累，这里做一下字体相关的总结。适合入门级的学习。" 
keywords: 字体,入门,coretext,font
---

建议大家先阅读一下[字符编码详解及由来](http://blog.sina.com.cn/s/blog_6966650401013e9q.html)  
好了，是不是感觉头脑里即清晰又模糊,ok,接下来我们对`ASCII`,`UNICODE`,`UTF8`来做一下解释

## 一. ASCII

### 1. 背景:
我们知道，在计算机内部，所有的信息最终都表示为一个二进制的字符串。每一个二进制位（bit）有0和1两种状态，因此八个二进制位就可以组合出256种状态，这被称为一个字节（byte）。也就是说，一个字节一共可以用来表示256种不同的状态，每一个状态对应一个符号，就是256个符号，从0000000到11111111。  
上个世纪60年代，美国制定了一套字符编码，对英语字符与二进制位之间的关系，做了统一规定。这被称为ASCII码，一直沿用至今。   
ASCII码一共规定了128个字符的编码，比如空格"SPACE"是32（二进制00100000），大写的字母A是65（二进制01000001）。这128个符号（包括32个不能打印出来的控制符号），只占用了一个字节的后面7位，最前面的1位统一规定为0。
  
### 2. 对照表:
![image](/images/font/asciifull.gif)
<!-- more -->

## 二. UNICODE

### 1. 背景:为什么需要unicode?
英语用128个符号编码就够了，但是用来表示其他语言，128个符号是不够的。比如，在法语中，字母上方有注音符号，它就无法用ASCII码表示。于是，一些欧洲国家就决定，利用字节中闲置的最高位编入新的符号。比如，法语中的é的编码为130（二进制10000010）。这样一来，这些欧洲国家使用的编码体系，可以表示最多256个符号。
但是，这里又出现了新的问题。不同的国家有不同的字母，因此，哪怕它们都使用256个符号的编码方式，代表的字母却不一样。比如，130在法语编码中代表了é，在希伯来语编码中却代表了字母Gimel (ג)，在俄语编码中又会代表另一个符号。但是不管怎样，所有这些编码方式中，0--127表示的符号是一样的，不一样的只是128--255的这一段。  
至于亚洲国家的文字，使用的符号就更多了，汉字就多达10万左右。一个字节只能表示256种符号，肯定是不够的，就必须使用多个字节表达一个符号。比如，简体中文常见的编码方式是GB2312，使用两个字节表示一个汉字，所以理论上最多可以表示256x256=65536个符号。  
 

### 2. 概念:
一种统一规范的编码，将世界上所有的符号都纳入其中，防止其各个国家进行不同的扩展，出现类似gbk这种。每一个符号都给予一个独一无二的编码，那么乱码问题就会消失。这就是Unicode，就像它的名字都表示的，这是一种所有符号的编码。    
目前实际应用的统一码版本对应于UCS-2，使用16位的编码空间。也就是每个字符占用2个字节。这样理论上一共最多可以表示216（即65536）个字符。基本满足各种语言的使用。实际上当前版本的统一码并未完全使用这16位编码，而是保留了大量空间以作为特殊使用或将来扩展。

### 3. 核心逻辑:
Unicode当然是一个很大的集合，现在的规模可以容纳100多万个符号。每个符号的编码都不一样，比如，U+0639表示阿拉伯字母Ain，U+0041表示英语的大写字母A，U+4E25表示汉字"严"。具体的符号对应表，可以查询[unicode.org](http://www.unicode.org/charts/)，或者专门的汉字对应表。  

## 三. UTF8

### 1. 背景:
外国人使用的大部分是英文，这样的话只需要一个字节，国人使用的是中文，中文是双字节。如果统一用unicode会造成资源的浪费。UTF-8就是在互联网上使用最广的一种Unicode的实现方式。**UTF-8是Unicode的实现方式之一**

### 2. 概念:
UTF-8编码是Unicode字符集的一种编码方式，其特点是使用变长字节数来存储数据。**1到4个byte**

### 3. 核心:
![image](/images/font/utf8.png)    

	单字节可编码的Unicode范围：\u0000~\u007F（0~127）
	双字节可编码的Unicode范围：\u0080~\u07FF（128~2047）
	三字节可编码的Unicode范围：\u0800~\uFFFF（2048~65535）
	四字节可编码的Unicode范围：\u10000~\u1FFFFF（65536~2097151）

在ASCII码的范围，用一个字节表示，超出ASCII码的范围就用字节表示，这就形成了我们上面看到的UTF-8的表示方法，这様的好处是当UNICODE文件中只有ASCII码时，存储的文件都为一个字节，所以就是普通的ASCII文件无异，读取的时候也是如此，所以能与以前的ASCII文件兼容。  
大于ASCII码的，就会由上面的第一字节的前几位表示该unicode字元的长度，比如110xxxxxx前三位的二进制表示告诉我们这是个2BYTE的UNICODE字元；1110xxxx是个三位的UNICODE字元，依此类推；xxx的位置由字符编码数的二进制表示的位填入。越靠右的x具有越少的特殊意义。只用最短的那个足够表达一个字符编码数的多字节串。注意在多字节串中，第一个字节的开头"1"的数目就是整个串中字节的数目。

## 四. 其他探讨
  
### 1. big endian和little endian  
big endian和little endian是CPU处理多字节数的不同方式。例如“汉”字的Unicode编码是6C49。那么写到文件里时，究竟是将6C写在前面，还是将49写在前面？如果将6C写在前面，就是big endian。如果将49写在前面，就是little endian。  
“endian”这个词出自《格列佛游记》。小人国的内战就源于吃鸡蛋时是究竟从大头(Big-Endian)敲开还是从小头(Little-Endian)敲开，由此曾发生过六次叛乱，一个皇帝送了命，另一个丢了王位。  
我们一般将endian翻译成“字节序”，将big endian和little endian称作“大尾”和“小尾”。   

### 2. 推荐一个查unicode码的网站  
[unicode的chart对照表](http://www.unicode.org/charts/),里面记录了每个字符的对应的unicode码,如果这里查不到可以去[unicode的维基百科](http://en.wikibooks.org/wiki/Unicode/Character_reference/2000-2FFF)




## 参考文献:
1.[维基百科 ASCII](https://zh.wikipedia.org/wiki/ASCII)  
2.[维基百科 UTF-8](https://zh.wikipedia.org/wiki/UTF-8)  
3.[维基百科 UTF-16](https://zh.wikipedia.org/wiki/UTF-16)  
4.[维基百科 Unicode](https://zh.wikipedia.org/wiki/Unicode)  
5.[字符编码笔记：ASCII，Unicode和UTF-8](http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html)  
6.[为什么UTF-8没有字节序问题？](http://www.guokr.com/blog/83367/)  
7.[谈谈Unicode编码，简要解释UCS、UTF、BMP、BOM等名词](http://www.fmddlmyy.cn/text6.html)
