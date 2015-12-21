---
layout: post
title: 二进制加法器
category: 摘录
tags: snippet
description: 
---

加法是算数运算中最基本的运算，因此如果想搭建一个计算机，那么首先就要造出可以计算两个数的和的器件。当解决了这个问题的时候，就会发现，原来加法就是计算机唯一要做的工作。如果我们造出加法器，同样地，就可以利用加法实现减法、乘法和除法，计算按揭贷款，引导火箭飞到火星、下棋、以及填写我们的话费账单。

这一章我们创建的加法器和现代计算器和计算机比起来，将会很庞大、很笨拙、很慢，而且运转起来噪声不断。但有意思的是，我们用来制作加法器的全部零件，都是像开关、灯泡、导线、电池、逻辑门、已经预先连接在各种逻辑门中的继电器等这些在之前的章节中学过的非常简单的电子器件。这个加法器所包含的所有器件早在120多年前就已经被发明出来了。我们并不用在房间中实际搭建什么，相反地，可以在纸上以及我们的头脑中来搭建这个加法器。

这一章的加法器完全用于二进制计算，用到一排开关表示需要相加的两个数，计算结果通过一排灯泡来显示。

### 加法表和进位表

要设计二进制加法器需要了解二进制的加法表。

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2012%20A%20Binary%20Adding%20Machine/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202014-09-24%20%E4%B8%8A%E5%8D%881.44.12.png)

以上加法器重新写为下面带有前导零的形式，这样每个结果都是一个2位的值。

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2012%20A%20Binary%20Adding%20Machine/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202014-09-24%20%E4%B8%8A%E5%8D%881.44.20.png)

因为我们的加法器中加法和进位是分别进行的，所以将加法位（sum bit)和进位位(carry bit)分成两个表格是非常方便的。

第一个表示加法的。

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2012%20A%20Binary%20Adding%20Machine/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202014-09-24%20%E4%B8%8A%E5%8D%881.44.34.png)

第二个表示进位的。

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2012%20A%20Binary%20Adding%20Machine/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202014-09-24%20%E4%B8%8A%E5%8D%881.44.39.png)

### 加法器控制面板

加法器的控制面板如下图所示。

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2012%20A%20Binary%20Adding%20Machine/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202014-09-24%20%E4%B8%8A%E5%8D%881.44.50.png)

经过理智的考虑后，我们决定让搭建起来的二进制加法器最高能够执行的加法长度位8位。也就是说，我们想要相加的二进制数，其范围是从0000-0000到1111-1111，即十进制数的0到255.两个8位二进制和最大可位1－1111-1110，即510.

加法器的其他部分是以各种形式连接起来的逻辑门。开关将触发逻辑门中继电器来点亮相应的灯泡。

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2012%20A%20Binary%20Adding%20Machine/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202014-09-24%20%E4%B8%8A%E5%8D%881.45.18.png)


### 进位表和与门

上一章中提到我们将隐藏继电器，转而用逻辑门来表示电路。因为这个8位二进制加法器中用到的继电器不少于144个，如果将全部电路图展现出来，一定会令人崩溃。相反地，我们利用逻辑门来分阶段地处理这个问题。当看到进位结果表时，或许你已经看出来逻辑门和二进制加法的相关性了。

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2012%20A%20Binary%20Adding%20Machine/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202014-09-24%20%E4%B8%8A%E5%8D%881.45.46.png)

这和上一章的输出结果是一样的。

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2012%20A%20Binary%20Adding%20Machine/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202014-09-24%20%E4%B8%8A%E5%8D%881.45.51.png)

因此，可以用与门计算二进制加法的进位。

### 加法表和异或门

到此，我们着实取得了一些进展。下面我们要做的就是利用继电器来实现下表。

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2012%20A%20Binary%20Adding%20Machine/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202014-09-24%20%E4%B8%8A%E5%8D%881.46.00.png)

首先我们要知道，或门和我们想要的结果很相似，除了右下角的结果。

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2012%20A%20Binary%20Adding%20Machine/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202014-09-24%20%E4%B8%8A%E5%8D%881.46.05.png)

与非门同样和我们想要的结果很相似，除了左上角的结果。

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2012%20A%20Binary%20Adding%20Machine/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202014-09-24%20%E4%B8%8A%E5%8D%881.46.09.png)

下面我们将或门和与非门连接到相同的输入上，如下图所示。

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2012%20A%20Binary%20Adding%20Machine/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202014-09-24%20%E4%B8%8A%E5%8D%881.46.14.png)

下表总结了或门和与非门的输出，并将其与我们想要的结果进行对比。

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2012%20A%20Binary%20Adding%20Machine/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202014-09-24%20%E4%B8%8A%E5%8D%881.47.16.png)

注意，我们想要的是1，那么这种情况只有在或门和与非门输出都为1时才会出现。这表明两个输出端可以通过一个与门连接到一起。

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2012%20A%20Binary%20Adding%20Machine/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202014-09-24%20%E4%B8%8A%E5%8D%881.47.34.png)

这就是我们想要的结果。

> 注意，在整个电路中仍然有两个输入和一个输出。两个输入同时作为或门和与非门的输入。或门和与非门的输出又分别作为一个与门的输入，最后得出了我们想要的结果。

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2012%20A%20Binary%20Adding%20Machine/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202014-09-24%20%E4%B8%8A%E5%8D%881.48.40.png)

实际上，这个电路有一个专门的名称，叫做异或门，简写为XOR。之所以叫异或门，因为不同于或门的地方在于当两端输入都为1时，其输出为0.为了不把或门、与非门和与门都画出来，我们可以用一个电气工程师都用的特定电气符合表示异或门。

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2012%20A%20Binary%20Adding%20Machine/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202014-09-24%20%E4%B8%8A%E5%8D%881.48.47.png)

异或门在输入端比或门多出了一条曲线，除此之外它看上去和或门非常相像。异或门的特征如下表所示。

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2012%20A%20Binary%20Adding%20Machine/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202014-09-24%20%E4%B8%8A%E5%8D%881.48.51.png)

还有一个逻辑门的输出和异或门相反，我们叫它同或门。只有当两个输入相同时，输出才为1.符号和异或门相似，只不过在输出端多了一个小圆圈。

### 半加器

让我们来回顾一下目前所了解到的内容。将两个二进制数相加将产生一个加法位和一个进位位。

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2012%20A%20Binary%20Adding%20Machine/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202014-09-24%20%E4%B8%8A%E5%8D%881.49.13.png)

可以利用下面两个逻辑门来实现这些结果。

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2012%20A%20Binary%20Adding%20Machine/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202014-09-24%20%E4%B8%8A%E5%8D%881.49.18.png)

两个二进制数相加的结果是由异或门和与门的输出给出的，而进位位是由与门的输出给出的。因此我们可以将异或门和与门连在一起来计算两个二进制数（即A和B）的和。

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2012%20A%20Binary%20Adding%20Machine/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202014-09-24%20%E4%B8%8A%E5%8D%881.49.21.png)

为了避免重复画与门和异或门，采用如下简单的表示方式。

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2012%20A%20Binary%20Adding%20Machine/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202014-09-24%20%E4%B8%8A%E5%8D%881.49.26.png)

这个符号被称为半加器（Half Adder）。之所以叫半加器是因为它只能应用于1位二进制的情况。

### 全加器

如果我们要计算的二进制位数大于1位。例如，下面两个二进制相加。

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2012%20A%20Binary%20Adding%20Machine/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202014-09-24%20%E4%B8%8A%E5%8D%881.49.45.png)

半加器用于最右列的相加。对于从右面算起的第二列，由于进位位的存在，实际上需要三个二进制相加，半加器没有做到的是将之前一次的加法可能产生的进位位纳入下一次运算。随后每一列二进制数相加都需要将进位位算进来。

为了对三个二进制进行加法运算，我们需要将两个半加器和一个或门做如下连接。

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2012%20A%20Binary%20Adding%20Machine/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202014-09-24%20%E4%B8%8A%E5%8D%881.49.49.png)

要理解它的工作原理，先从最左边的第一个半加器的A输入和B输入开始，其输出是一个和及相应的进位。这个和必须与前一列的进位输入相加，然后再把它们输入到第二个半加器中。第二个半加器的输出是最后的结果。两个半加器的进位输出又被输入到一个或门中。这里可以用一个半加器，但是我们考虑到所有的可能性，两个半加器的进位输出不会同时为1的。所以或门在这里足够可以取代半加器，因为或门除了在输入都为1的时候以外，其他情况下结果和异或门结果相同。

为了避免重复画上面这个图，我们用以下形式来替代上图中的一堆符号，它被称为全加器（Full Adder）。

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2012%20A%20Binary%20Adding%20Machine/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202014-09-24%20%E4%B8%8A%E5%8D%881.49.55.png)

### 控制面板

再看看之前提到的由灯泡和开关所组成的控制面板。现在我们可以将开关和灯泡连接到全加器了。

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2012%20A%20Binary%20Adding%20Machine/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202014-09-24%20%E4%B8%8A%E5%8D%881.50.20.png)

首先将最左端的两个开关和最右端的一个灯泡连接到一个全加器上。

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2012%20A%20Binary%20Adding%20Machine/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202014-09-24%20%E4%B8%8A%E5%8D%881.50.26.png)

当把两个二进制数字相加时，第1列的进位输入为0，所以将第一列的进位输入接地。

对于接下来的两个二进制位和灯泡，可以按照如下办法连接全加器。

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2012%20A%20Binary%20Adding%20Machine/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202014-09-24%20%E4%B8%8A%E5%8D%881.50.30.png)

第一列全加器的进位输出是第二列全加器的进位输入。随后的每列二进制数都以同样的方式连接。每一列进位输出都是下一列的进位输入。

最终，第8个灯泡和最后一对开关将已入夏方式连接全加法器上。

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2012%20A%20Binary%20Adding%20Machine/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202014-09-24%20%E4%B8%8A%E5%8D%881.50.35.png)

这里，最后一个进位输出将被连接到第9个灯泡上。

至此大功告成。

### 8位加法器

下图是8个全加器相连的示意图，每个全加器的进位输出都作为下一个全加器的进位输入。

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2012%20A%20Binary%20Adding%20Machine/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202014-09-24%20%E4%B8%8A%E5%8D%881.50.41.png)

下面是画成一个盒子的完整的8位二进制加法器，输入标记为A0～A7和B0～B7，输出标记为S0～S7.

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2012%20A%20Binary%20Adding%20Machine/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202014-09-24%20%E4%B8%8A%E5%8D%881.50.47.png)

这就是表示多位数字中各位数字的常用方法。A0，B0和S0是最低有效位，或者说，最右边的一位。A7、B7和S7是最高有效位，或者说是最左边的一位。

另一种8位加法器可用如下图表示。

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2012%20A%20Binary%20Adding%20Machine/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202014-09-24%20%E4%B8%8A%E5%8D%881.50.55.png)

一旦你搭建了8位二进制加法器，你就可以再搭建另外一个加法器。把它们集联起来就很容易扩展出一个16位加法器。

![](https://github.com/arcticlion/reading-lists/blob/master/Code/Chapter%2012%20A%20Binary%20Adding%20Machine/%E5%B1%8F%E5%B9%95%E5%BF%AB%E7%85%A7%202014-09-24%20%E4%B8%8A%E5%8D%881.51.00.png)

右边加法器的进位输出被连接到左边加法器的进位输入上。


