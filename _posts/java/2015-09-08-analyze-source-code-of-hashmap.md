---
layout: post
title: HashMap 源码分析
category: Java
tags: java
description: 
---

## 背景

我们知道, hash 是一个函数，将任意长度的数据映射为固定长度数据。也就是说，无论输入的数据块 m 有多大，其输出值 h 都为固定长度。这是什么原理呢？可以将 m 分成固定长度 h，一次进行 hash 运算，然后用不同的方法迭代（比如前一块 hash 值与后一块 hash 值进行异或）。如果不够h位怎么办？用0或者1进行补全，这在算法中进行约定。

Hash 有两个特别的能力，抗碰撞能力和抗篡改能力。抗碰撞能力是，任意两个不同的数据块，其 hash 值相同的可能性极小；抗篡改能力是，对于一个数据块，哪怕改动其中一个比特位，其 hash 值也如天壤之别。在计算机科学中，数据结构和密码学都广泛用到了 hash，但是这两个领域的 hash 的含义并不相同，算法的设计侧重也不相同。

在用到 hash 进行管理的数据结构中，hash 存在的目的是为了加速键值对的查找，hash 的作用是为了将元素尽可能均匀的放在不同的桶里。因此数据结构中的 hash 算法对于抗篡改能力要求没那么高。但是算法的set性能与hash值的产生速度直接相关，所以这个时候的hash值的产生速度就尤为重要。

在在 Java 中，每个对象都有一个哈希码，这个值可以通过 hashCode() 获得。JDK 源码中，String.hashCode() 实现就是一个很简单的快速的乘法迭代：

```java
public int hashCode() 
    int h = hash;
    //hash default value : 0 
    if (h == 0 && value.length > 0) {
        //value : char storage
        char val[] = value;

        for (int i = 0; i < value.length; i++) {
            // 选择 31 这个乘数因子
            // 为了尽量使各个字符串映射的结果
            // 在 java 的整数域内均匀分布
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```

在不少的 hash 算法中，使用异或或者加法进行迭代，速度和hashCode的实现差不多。

而密码学中，hash 算法主要用于对消息的完整性进行检验。这就对 hash 的抗碰撞能力和抗篡改能力要求很高了，速度其次。在 hash 在密码学中的算法因为不是本文重点，在此点到即止。

## Hash Table

在编程的世界里，数据的基本组成形式可以说有三种：

* 对象（或者结构体）
* 数组
* 链表

其他任何的数据组织形式都可以看作是这三种数据组织形式的组合变体。

结构体（或对象）可以是基本数据类型或者其他结构体（或对象）的组合。结构体或对象一般用来描述一个复杂数据实体。

数组一般是一组同类型的变量的集合，在内存中表现为一片连续的空间，因为空间是连续的，且每一个数据单元占的空间的大小是相等的，所以根据地址的偏移对数据元素实现快速访问，但是当需要插入或者删除一个元素的时候，则需要对目标元素的之后的所有元素进行向前移动。

链表的单个节点一般是结构体或者对象，因为链表的单个节点除了需要保存数据之外还需要维护它的相邻节点的关系，如果想要获得链表中的某个节点的值，需要从链表的头节点开始遍历，直到找到需要的东西，而插入或删除某个节点的话，需要找到相应的节点，修改其相邻节点的指针的引用即可。

像其他数据类型，比如队列，栈，树，图都可以通过数组或者链表来组织，并实现相应的操作功能。

很多时候，我们需要对数据进行快速存取（比如缓存的实现），并用 key 来标记自己存取的数据。这是一个 key-value 的结构。

既然要“快速”，我们很自然地想到用数组，因为数组可以在 O(1) 的时间复杂度内完成指定位置元素的读写操作。Hash Table 就是基于数组实现的。这里，我们把每一个数组的单元叫做 bucket（桶）。

所以构造哈希表我们需要先构造哈希函数，哈希函数的作用是将 key 映射到一个存储地址。这样就可能出现两个不同的 key 哈希后对应的存储地址相同，这种情况就是哈希冲突。

### 哈希冲突

Hash Table 为了加快键值对的查找，哈希函数需要简单，运行速度快，往往会产生冲突。为了减少冲突，我们希望 hash 函数的结果均匀的分布在地址单元的空间里。这样可以有效的减少冲突。

这里需要提到一个装填因子 Load factor 的概念。Load factor(a) = 哈希表实际的元素数目(m)/哈希表的容量(n)，a 越大，哈希表冲突的概率越大，a 越接近 0，那么哈希表的空间就越浪费。一般情况下建议的 Load factor 的值为 0-0.7，Java 默认的 Load factor 的值为 0.75，当装载因子大于这个值的时候，HashMap 会对数组进行扩张至原来的两倍大。

冲突既然无可避免，那么我们就必须对冲突进行解决。解决冲突的方式分为两类，开放定址法和链接法。

开放定址法(Open addressing)就是在计算一个 key 的哈希的时候，发现目标地址已经有值了，即发送冲突，这个时候通过相应的函数在此地址的后面的地址去找，知道没有冲突为止。这个方法常用的方法有线性探测，二次探测，再哈希。这种解决方法有一个不好的地方是，当冲突发生之后，会在之后的地址空间中放一个值进去，这样就有可能出现一个 key 的哈希出来的结果也是位于这个地址空间，非相同哈希值的两个 key 冲突的情况。

链接法(Separate chaining)链接法是通过数组和链表组合而成的。当发送冲突的时候只要将其加到对应的链表中即可。

和开放定址法相比，链接法有如下几个优点：

1. 链接法处理冲突简单，且无堆积现象，即非同义词决不会发生冲突，因此平均查找长度较短。
2. 链接法中各链表上的节点空间是动态申请，故它更适合于造表前无法确定哈希表大小的情况。
3. 开发定址法为减少冲突，要求装填因子 a 较小，因此当表规模较大时会浪费很多空间。而链接法中可取 a >= 1，当表规模较大时，链接法中增加的指针忽略不计，因此节省空间。
4. 用链接法构造的哈希表中，删除节点的操作易于实现。只要简单地删去链表上相应的节点即可。而开放定址法构造的哈希表删除节点时，不能简单地将被删节点的空间置为空，否则将截断同义词的查找路径。这是因为开放定址法中，空地址单元是查找失败的条件。因此在用开放定址法执行删除操作时，只能在被删节点上做删除标记，而不能真正删除节点。

当然链接法也有缺点，链接法的缺点是，指针需要额外空间，当节点规模较小时，开放定址法更为节省空间。对于相同较小规模的哈希表来说，如果将指针所花费的空间用来扩大散列表的规模，对于开放定址法来说，装载因子因此而减少，不仅减少了冲突，也提高了平均的查询速度。

### HashMap

在 Java 中，HashMap 可以看作是 Java 实现的 Hash Table。HashMap 中存放的是 key-value 对，对应的类型为 java.util.HashMap.Entry，所以在 HashMap 中数据都存放在一个 Entry 引用类型的数组 table 中。我们使用 [key 的 hashCode % table 的长度]来计算 key 的位置。

    transient Entry[] table

Entry 是实现键值对存储的关键，里面包含 key，value，hash 和 指向下一个 Entry 对象的引用。

```java
static final class HashEntry<K, V> {
        final int hash;
        final K key;
        volatile V value;
        volatile ConcurrentHashMap.HashEntry<K, V> next;
    }
```

put 方法

```java
public V put(K key, V value) {
        if (key == null)
            return putForNullKey(value);
        int hash = hash(key.hashCode());
        int i = indexFor(hash, table.length);
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }

        modCount++;
        addEntry(hash, key, value, i);
        return null;
    }
```

在 HashMap 中我们的 key 可以为 null，所以第一步就处理了 key 为 null 的情况。

当 key 为 非 null 的时候，代码又对 hashCode 再次做了一次哈希，这是为什么呢？这个还得先看 indexFor(hash, table.length) 方法，这个方法决定了 存放位置。

```java
static int indexFor(int h, int length) {
        return h & (length-1);
    }
```

由于 HashMap 中 table 的长度为 2n，所以 h&(length-1) 就等价与 h%length。这样的话，如果对原本的 hashCode 不做变换得到话，大于 length-1 的部分一定会冲突，于是就对 hashCode 再做一次哈希。

```java
static int hash(int h) {
        // This function ensures that hashCodes that differ only by
        // constant multiples at each bit position have a bounded
        // number of collisions (approximately 8 at default load factor).
        h ^= (h >>> 20) ^ (h >>> 12);
        return h ^ (h >>> 7) ^ (h >>> 4);
    }
```

当找到 key 所对应的位置的时候，对对应位置的 Entry 的链表进行遍历，如果以及存在 key 的话，就更新对应的 value，并返回老的 value。如果是新的 key 的话，就将其增加进去。modCount 是用来记录 hashmap 结构变化的次数的，这个在 hashmap 的fail-fast 机制中需要使用（当某一个线程获取了 map 的游标之后，另一个线程对 map 做了结构修改的操作，那么原先准备遍历的线程会抛出异常）。addEntry 的方法如下：

```
void addEntry(int hash, K key, V value, int bucketIndex) {
    Entry<K,V> e = table[bucketIndex];
        table[bucketIndex] = new Entry<K,V>(hash, key, value, e);
        if (size++ >= threshold)
            resize(2 * table.length);
    }
```

get 方法：

```java
public V get(Object key) {
        if (key == null)
            return getForNullKey();
        int hash = hash(key.hashCode());
        for (Entry<K,V> e = table[indexFor(hash, table.length)];
             e != null;
             e = e.next) {
            Object k;
            if (e.hash == hash && ((k = e.key) == key || key.equals(k)))
                return e.value;
        }
        return null;
    }
```

get 方法其实就是将 key 以 put 时相同的方法算出在 table 所在的位置，然后对所在位置的链表进行遍历，找到 hash 值和 key 都相等的 Entry 并将 value 返回。

