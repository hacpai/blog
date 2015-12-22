---
layout: post
title: 例题：斐波那契数列
cateory: 摘录
tags: snippet
description: 
---

本节讨论一个著名的数列，斐波那契数列——

> 0, 1, 1, 2, 3, 5, 8, 13, 21, 34,...                         (2.5)

数列用一个递推式和两个初始条件定义：

> 当n>1时，F(n)=F(n-1)+F(n-2)                                 (2.6)
>          F(0)=0,F(1)=1                                      (2.7)

在计算机科学领域，斐波那契有许多令人感兴趣的应用。例如，欧几里德算法的最差输入恰巧是斐波那契数列中的连续元素。这里我们只讨论，找到一个求第n个斐波那契数的精确公式和简要地讨论一下计算斐波那契数的算法。

### 第n个斐波那契数的精确公式

反向替换法无法解递推式(2.6),我们改用另外一个定理，求解**带常系数的齐次二阶线性递推式**

递推式(2.6)改写成

> F(n)-F(n-1)-F(n-2)=0                                        (2.10)

它的特征方程是

> r^2-r-1=0

该特征方程的根是

> r1,2=(1±√5)/2

该特征方程有两个不相等的实根,故

> F(n)=α((1+√5)/2)^n+β((1-√5)/2)^n

接着利用初始条件来确定参数α和β的确定值。我们把初始条件中的n值——0和1，分别代换到最后的那个等式，并且让这两个等式的值分别等于0和1（根据2.7）得到的F(0)和F(1)的值）

经过一系列标准的代数化简，得到一个由两个线性方程构成的联立方程组，该方程组包含两个未知数，α和β。

解这个方程组，我们得到这两个未知数的值，α＝1/√5，β＝-1/√5.因此

![](https://github.com/arcticlion/reading-lists/blob/master/Introduction%20to%20the%20Design%20and%20Analysis%20of%20Algorithms/02%20Fundamentals%20of%20the%20Analysis%20of%20Algorithm%20Efficiency/屏幕截图%202014-11-29%2002.18.28.png)

其中，Φ＝(1+√5)/2 ≈ 1.61803，而Φ'=(1-√5)/2≈-0.61803.常数Φ被认为是**黄金分割率**。从美学的角度来看，长方形的两条边符合这个比率是最完美的。

观察方程2.9，F(n)呈指数级增长，也就是说，F(n)∈Θ(Φ^n).

### 计算斐波那契数的算法

首先，我们通过递推式(2.6)和初始条件(2.7)得到一个显而易见的算法，来计算F(n).

```
算法 F(n)
    //根据定义，递归计算第n个斐波那契数
    //输入：一个非负整数n
    //输出：第n个斐波那契数
    if n≦1 return n
    else return F(n-1)+F(n-2)
```

对算法做正式分析之前，怎么知道它是不是一个高效的算法呢？我们需要做一个正式的分析。该算法的基本操作是加法，我们把A(n)定义为这个算法在计算F(n)的过程中所做的加法次数。因而，计算F(n-1)和F(n-2),该算法还需要做一次加法来计算它们的和。因此，对于A(n)我们有下面的递推式：

> 当n>1时，A(n)=A(n-1)+A(n-2)+1                              (2.13)  
> A(0)=0, A(1)=1

递推式A(n)-A(n-1)-A(n-2)=1等号右边不等于0，这种递推式被称为**非齐次递推式**.解这个特殊的递推式，有一个快速解题的捷径，我们可以把这个非齐次递推式简化为它的齐次形式，把它改写为

> [A(n)+1]-[A(n-1)+1]-[A(n-2)+1]=0

再做一个B(n)=A(n)+1代换：

> B(n)-B(n-1)-B(n-2)=0
> B(0)=1, B(1)=1

我们发现，B(n)和F(n)实际上是同一个递推式，唯一的区别是它以两个1作为开始，因此它总是比F(n)领先一步。所以B(n)=F(n+1),并且

![](https://github.com/arcticlion/reading-lists/blob/master/Introduction%20to%20the%20Design%20and%20Analysis%20of%20Algorithms/02%20Fundamentals%20of%20the%20Analysis%20of%20Algorithm%20Efficiency/屏幕截图%202014-11-29%2013.42.23.png)

因此，A(n)∈(φ^n)。

从递推式(2.13)中，可以预计到该算法的效率并不高。它包含两个递归调用，通过观察该算法的递归调用树，如图2.6

![](https://github.com/arcticlion/reading-lists/blob/master/Introduction%20to%20the%20Design%20and%20Analysis%20of%20Algorithms/02%20Fundamentals%20of%20the%20Analysis%20of%20Algorithm%20Efficiency/屏幕截图%202014-11-29%2013.45.06.png)

发现相同的函数值被一遍又一遍地重复计算，这很明显是一种效率低下的做法。

通过简单地对斐波那契数列的连续元素进行迭代计算，我们得到了一个快得多得算法。

```
算法 Fib(n)
    //根据定义，迭代计算第n个斐波那契数
    //输入：一个非负整数n
    F[0] <- 0, F[1] <- 1
    for i <- 2 to n do
        F[i] <- F[i-1] + F[i-2]
    return F(n)
```

很明显，这个算法要做n-1次加法运算。所以，它和n一样都是线性函数。没有必要特意使用一个数组来存储斐波那契数列中的前面元素：为了完成该任务，只需要存储两个元素就足够了。

最后，提出一种效率类型为Θ(log n)的算法，在计算第n个斐波那契数时只需要对整数进行操作。这个算法基于等式

![](https://github.com/arcticlion/reading-lists/blob/master/Introduction%20to%20the%20Design%20and%20Analysis%20of%20Algorithms/02%20Fundamentals%20of%20the%20Analysis%20of%20Algorithm%20Efficiency/屏幕截图%202014-11-29%2013.52.37.png)

效率取决于矩阵乘方的计算效率。
