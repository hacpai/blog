---
layout: post
title: 变量
category: 摘录
tags: snippet
description: 
---
## 基本类型和引用类型的值

ECMAScript变量可能包含两种不同的数据类型的值：基本类型值和引用类型值

基本类型值源自5种基本数据类型：Undefined, Null, Boolean, Number和String。这五种基本类型的值在内存中占有固定大小的空间，它们的值保存在栈内存中。

ECMAScript的字符串不是对象，因此不是引用类型。

引用类型的值是对象，保存在堆内存中。引用类型的变量是一个指向该对象的指针。

### 复制变量值

复制基本类型值创建了这个值的副本。

复制引用类型的值，复制的是它的指针，两个变量指向同一个对象。

### 检测类型

`typeof`操作符是确定一个变量是字符串、数值、布尔值、还是undefined的最佳工具。

引用类型用`instanceof`操作符检测
