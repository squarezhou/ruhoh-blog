---
title: 文件编码笔记
date: '2013-07-31'
description:
categories: Tech
tags: [编码,GBK,UTF8,BOM]
---
	最近总觉得看的东西太多，记住的太少。自己写不出，于是COPY别人的当笔记好了，囧～

###ASCII###

American Standard Code for Information Interchange，ANSI 制定，用1个字节中的后7位共表示128个不同的英文字符，包括空格、标点符号、数字、大小写字母。这128个符号（包括32个不能打印出来的控制符号），只占用了一个字节的后面7位，最前面的1位统一规定为0。

###GB系列###

**GB2312**

国家标准，小于127的字符的意义与原来相同，两个大于127的字符连在一起时表示一个汉字，高字节（0xA1～0xF7），低字节（0xA1~0xFE）。在 ASCII 里本来就有的数字、标点、字母都统统重新编了两个字节长的编码，这就是常说的”全角”字符，而原来在127号以下的那些就叫”半角”字符了。

**GBK**

GB2312 扩展而来，不再要求低字节一定是127号之后的内码，只要第一个字节是大于127就固定表示这是一个汉字的开始。

  
**GB18030**

GBK 扩展，加了几千个新的少数民族的字。

###UNICODE###

Universal Multiple-Octet Coded Character Set，简称 UCS，废了所有的地区性编码方案，重新搞一个包括了地球上所有文化、所有字母和符号的编码。UCS有两种格式：UCS-2和UCS-4。顾名思义，UCS-2就是用两个字节编码，UCS-4就是用4个字节（实际上只用了31位，最高位必须为0）编码，目前UCS-2能满足需求，所以默认都使用它。

**Little-endian和Big-endian**

big endian和little endian是CPU处理多字节数的不同方式。

我们知道，Unicode码可以采用UCS-2格式直接存储。例如“汉”字的Unicode编码是6C49。那么写到文件里时，究竟是将6C写在前面，还是将49写在前面？如果将6C写在前面，就是Big-endian。如果将49写在前面，就是Little-endian。因此，第一个字节在前，就是"大头方式"（Big-endian），第二个字节在前就是"小头方式"（Little-endian）。

Unicode规范中定义，每一个文件的最前面分别加入一个表示编码顺序的字符，这个字符的名字叫做"零宽度非换行空格"（ZERO WIDTH NO-BREAK SPACE），用FEFF表示。这正好是两个字节，而且FF比FE大1。

如果一个文本文件的头两个字节是FE FF，就表示该文件采用大头方式；如果头两个字节是FF FE，就表示该文件采用小头方式。

###UTF8###

互联网的普及，强烈要求出现一种统一的编码方式。UTF-8就是在互联网上使用最广的一种Unicode的实现方式。其他实现方式还包括UTF-16（字符用两个字节或四个字节表示）和UTF-32（字符用四个字节表示），不过在互联网上基本不用。重复一遍，这里的关系是，UTF-8是Unicode的实现方式之一。

UTF-8最大的一个特点，就是它是一种变长的编码方式。它可以使用1~4个字节表示一个符号，根据不同的符号而变化字节长度。

UTF-8的编码规则很简单，只有二条：

1）对于单字节的符号，字节的第一位设为0，后面7位为这个符号的unicode码。因此对于英语字母，UTF-8编码和ASCII码是相同的。  
2）对于n字节的符号（n>1），第一个字节的前n位都设为1，第n+1位设为0，后面字节的前两位一律设为10。剩下的没有提及的二进制位，全部为这个符号的unicode码。

从UCS-2到UTF-8的编码方式如下：

UCS-2编码(16进制)|UTF-8 字节流(二进制)
-|-
0000 0000-0000 007F|0xxxxxxx
0000 0080-0000 07FF|110xxxxx 10xxxxxx
0000 0800-0000 FFFF|1110xxxx 10xxxxxx 10xxxxxx
0001 0000-0010 FFFF|11110xxx 10xxxxxx 10xxxxxx 10xxxxxx

**BOM**

UTF-8以字节为编码单元，没有字节序的问题。UTF-16以两个字节为编码单元，在解释一个UTF-16文本前，首先要弄清楚每个编码单元的字节序。例如“奎”的Unicode编码是594E，“乙”的Unicode编码是4E59。如果我们收到UTF-16字节流“594E”，那么这是“奎”还是“乙”？

Unicode规范中推荐的标记字节顺序的方法是BOM。BOM不是“Bill Of Material”的BOM表，而是Byte Order Mark。BOM是一个有点小聪明的想法：

在UCS编码中有一个叫做"ZERO WIDTH NO-BREAK SPACE"的字符，它的编码是FEFF。而FFFE在UCS中是不存在的字符，所以不应该出现在实际传输中。UCS规范建议我们在传输字节流前，先传输字符"ZERO WIDTH NO-BREAK SPACE"。

这样如果接收者收到FEFF，就表明这个字节流是Big-Endian的；如果收到FFFE，就表明这个字节流是Little-Endian的。因此字符"ZERO WIDTH NO-BREAK SPACE"又被称作BOM。

UTF-8不需要BOM来表明字节顺序，但可以用BOM来表明编码方式。字符"ZERO WIDTH NO-BREAK SPACE"的UTF-8编码是EF BB BF（读者可以用我们前面介绍的编码方法验证一下）。所以如果接收者收到以EF BB BF开头的字节流，就知道这是UTF-8编码了。

Windows就是使用BOM来标记文本文件的编码方式的。

参考：  
1. [字符编码详解(基础)](http://www.laruence.com/2009/08/22/1059.html)  
2. [字符编码笔记：ASCII，Unicode和UTF-8](http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html)  
3. [谈谈Unicode编码，简要解释UCS、UTF、BMP、BOM等名词](http://www.fmddlmyy.cn/text6.html)