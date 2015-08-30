---
layout: post
title: 布莱叶盲文与二进制码
category: 编码基础
tags: code
description: 
---

### 布莱叶盲文的由来

巴比尔文字系统由凸起的点和划的组合来表示文字，由于并非使用与字母表相应的点划编码串来表示字母，而是用与读音相似的编码串表示，所以一个单词就使用了很多的码子，不适合长文本的编写。布莱叶改进了巴比尔的文字系统，创建了布莱叶盲文。

### 布莱叶盲文

在布莱叶盲文中，2x3的点码单元中一个或者多个凸起的点表示每个文字中用到的符号。

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2003%20Braille%20and%20Binary%20Codes/屏幕快照%202014-09-17%20上午9.17.18.png)

例如下面的布莱叶盲文

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2003%20Braille%20and%20Binary%20Codes/屏幕快照%202014-09-17%20上午9.17.53.png)

其中，第1，3，5点是凸起的，而第2，4，6点的位置是平的。

这些点码都是二进制的，每组有6个点，而且有平和凸两种状态，因此6个可平可凸的点的组合就是2x2x2x2x2x2，也就是64.

让我们来开始解析布莱叶盲文。

这是最基本的小写字母表

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2003%20Braille%20and%20Binary%20Codes/屏幕快照%202014-09-17%20上午9.18.01.png)

词组“you and me”用布莱叶盲文表示的组合：

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2003%20Braille%20and%20Binary%20Codes/屏幕快照%202014-09-17%20上午9.23.31.png)

> 注意，单词中每个字母所对应的点码单元之间用小块空白分开；单词之间租用大的空格分开。

二级布莱叶盲文使用缩写来保存树型结构和提高阅读速度。

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2003%20Braille%20and%20Binary%20Codes/屏幕快照%202014-09-17%20上午9.27.16.png)

“you and me”使用二级布莱叶盲文就可以表示为

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2003%20Braille%20and%20Binary%20Codes/屏幕快照%202014-09-17%20上午9.27.23.png)

目前为止，已经描述了31个码子——单词间的大空格（即没有凸点的点码单元）以及总共3排每排10个的字母和单词码字。

现在完善布莱叶编码

首先第一组a到j的码字加6号凸点的组合

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2003%20Braille%20and%20Binary%20Codes/屏幕快照%202014-09-17%20上午9.46.37.png)

然后，a到j的码字，去掉1号和2号凸起的组合。

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2003%20Braille%20and%20Binary%20Codes/屏幕快照%202014-09-17%20上午9.46.43.png)

目前已经定义了51个码字了，再补充3，4，5，6号点遗漏的码字

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2003%20Braille%20and%20Binary%20Codes/屏幕快照%202014-09-17%20上午9.47.32.png)

表示字符串缩写“ble”的码字当不作为单词的一部分时，作为数字表示符。转换a到j的码字为数字1到0.

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2003%20Braille%20and%20Binary%20Codes/屏幕快照%202014-09-17%20上午9.47.40.png)

最后添加剩余的7个码字

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2003%20Braille%20and%20Binary%20Codes/屏幕快照%202014-09-17%20上午9.48.01.png)

将6号凸起的点作为大写字母标示符

### 从盲文窥探编码的本质

综上所述，我们将6位二进制码（其实是6个点）所能表示的全部64种可能的编码都罗列了一遍。而且这64组编码中有很大一部分，根据上下文的不同将有着双重身份。尤其值得注意的是数字表示符和取消“数字表示状态”的字母标示符。它们改变了后面编码的意义——从表示字母到表示数字，又从表示数字回到表示字母。像这样的编码通常被称为“优先码”（precedence codes）或者“换挡码”（shift codes）。它们改变着作用域内编码的含义，直到作用域结束。

大写字母标示符表示紧随它的字母（而且仅仅是紧随它的字母）应该被译为大写。类似这样的编码被称为“逃逸码”（escape codes）。逃逸码让你“逃离”对编码串单调的、一成不变的解析，而转入一种新的解析方式中。

使用二进制对书面语言进行编码时，换挡码和逃逸码是相当常见的。

