---
layout:     post
title:      深入分析排序算法-外部排序(external sorting)
subtitle:   深入分析排序算法-外部排序(external sorting)
date:       2018-06-13
author:     carson
header-img: img/post-bg-2015.jpg
catalog: true
tags:
    - 数据结构
---

    排序算法的情况应该分为两种情况，分别是*内部排序*和*外部排序*。这里的内部和外部指的是内存和外部存储设备，存在一些情况会因为数据量太大或者主存太小而装不进主存，这时候就需要用外部排序的方法进行排序。
    一般被广泛研究学习使用的，都是内部排序算法，或者说前提是数据可以全部装进主存内。

### 内部排序

    对于一般的排序算法，比如快速排序、插入排序等，都是只使用**比较**操作来排序的算法。理论上，内部排序在某些特定情况下，某种程度上可以达到线性时间复杂度，比如一个数组A只由小于M的数组成，可以创建一个大小为M的数组，称为Count，里面各项被初始化为0。然后遍历A，当读到A[i]时，Count[A[i]]增加1。A被遍历完后扫描数组Count，打印出来的数就是排序后的A。这个排序算法叫*桶式排序*。它的算法复杂度为O(M+N)。

因为内部排序相关在网上已经有很多资料学习，这里就不再叙述，下面列出一些排序算法的性质和网上搜索的关键词：

关键词：`决策树`、`信息-理论下界`
性质：
* N个元素排序的可能的结果有N!种，因为决策树中的树叶是N!个。
* 由上面性质可推：深度为d的二叉树最多有2ᵈ个树叶
* 由上面性质可反推：L片树叶的二叉树深度至少是⌈logL⌉
* 由前面定理可知：至少需要log(N!)次比较，只使用**比较**操作来排序的算法都需要进行Ω(NlogN)次比较。

下面是一些内部排序算法的参考数据：

在一台比较慢的机器上的测试数据，其中优化的快速排序指的是使用三数中值法选取枢纽元的快速排序算法；希尔排序使用Sedgewick增量序列：
![](https://github.com/qweasdzxcpkh/images/blob/master/DataStructureAndAlgorithm/external_sorting0.png?raw=true)

然而这些算法都需要把数据一次过放入内存。数据量太大的情况下内部排序简直束手无策。这是有外部排序就派上用场了。

### 外部排序

外部存储设备中，存在限制最大的设备类型是磁带，磁带上面的元素只能被顺序访问，这里就用磁带作为例子。

外部排序的基本思想是使用归并排序中的合并操作，将两个有序的数组合并成一个。

##### 简单算法

设有4盘磁带分别是Ta1、Ta2、Tb1、Tb2，并且内存很小，只能一次容纳和排序M个记录。假设数据一开始都在磁带Ta1上
![](https://github.com/qweasdzxcpkh/images/blob/master/DataStructureAndAlgorithm/external_sorting1.png?raw=true)

从Ta1上不停提取M个元素并排序，交替地把这些**顺串**写到Tb1和Tb2上。假设M为3，即计算机一次只能排序3个记录
![](https://github.com/qweasdzxcpkh/images/blob/master/DataStructureAndAlgorithm/external_sorting2.png?raw=true)

从Tb1和Tb2上分别提取一个顺串，并使用归并排序的合并，得到一个二倍长的顺串。重复这一步骤，并把这些二倍长的顺串交替地写到Ta1和Ta2上
![](https://github.com/qweasdzxcpkh/images/blob/master/DataStructureAndAlgorithm/external_sorting3.png?raw=true)

最后递归地使用上述所有步骤，直到只有一个顺串，这个最后的顺串就是排序好的结果。
![](https://github.com/qweasdzxcpkh/images/blob/master/DataStructureAndAlgorithm/external_sorting4.png?raw=true)

    这就是一个简单算法的执行过程。

两个顺串的合并操作通过从两个顺串的开头比较出较小者，然后写入输出磁带，相应的输入磁带向前推进读取下一个数字，然后继续比较和写入输出磁带。这样内存中只会存在两个元素的比较操作，而不用把两个顺串都读入内存从而违背了计算机内存只能容纳M个元素的原则。
这个算法需要⌈log(N/M)⌉趟操作，外加一趟构造初始顺串的操作。

##### 多路合并

在简单算法一节中，每一趟的操作都是从两个磁带中各提取一个元素，这个操作可以称为2-路合并。那么可以考虑把这个合并操作扩充为k-路合并。

k-路合并的工作方式跟2-路合并类似，唯一不同的地方在于k-路合并在k个元素中找出最小元稍微有点复杂，不过可以使用优先队列找出这个最小元。需要注意的是：多路合并中的`多`并不能大于M，即k<=M，不然内存就违背了只能容纳M个元素的原则了。

用前面的例子，使用3-路合并
构造初始顺串：
![](https://github.com/qweasdzxcpkh/images/blob/master/DataStructureAndAlgorithm/external_sorting5.png?raw=true)

第一趟：
![](https://github.com/qweasdzxcpkh/images/blob/master/DataStructureAndAlgorithm/external_sorting6.png?raw=true)

第二趟：
![](https://github.com/qweasdzxcpkh/images/blob/master/DataStructureAndAlgorithm/external_sorting7.png?raw=true)


例子中，这个例子从2-路合并改为3-路合并后，执行的趟数从3趟变为2趟。

k-路合并需要的趟数为⌈log𝑘(N/M)⌉。

##### 多相合并

容易看出，k-路合并中需要使用2k盘磁带，这有点浪费磁带。只使用k+1盘磁带也有可能完成排序工作。

多相合并跟多路合并并不冲突，因为第二趟的顺串是从第一趟的顺串构造出来的，多相合并的核心只是第一趟的顺串与第二甚至第三趟等的顺串进行多路合并，这就说明一开始不同磁带上面的顺串数是不同的。

作为例子，这里阐述如何用3盘磁带完成2-路合并，看看效果是什么：
    假设有三盘磁带T1，T2，T3，初始的数据在T1上，它产生了34个顺串，不使用多相合并的话一般会在T2和T3上分别放置17个顺串。但是现在只有一盘额外的磁盘空间，所以可以把34个顺串不平均的分成两份，假设把21个顺串放在T2上，13个顺串放在T3上，合并的时候T3上的顺串会首先用完，产生13个二倍长的顺串在T1上，和8个残留顺串在T2上。继续将T1和T2合并，新的顺串放置在T3上。不停地执行这一个过程，直到剩下一个顺串。
![](https://github.com/qweasdzxcpkh/images/blob/master/DataStructureAndAlgorithm/external_sorting8.png?raw=true)


事实上，最初的顺串分配跟这个合并的过程有很大关系，如果顺串的初始个数是一个斐波那契数Fn，最好把它分裂成Fn-1和Fn-2。如果初始个数不是一个斐波那契数，可以用一些*哑顺串*补足成一个斐波那契数。

上面的做法可以扩充到k-路合并，这可能需要用到k阶斐波那契数来分配顺串。
（想了一下午还是不知道3-路合并怎么支持多相合并。。。目前只知道一个用3阶斐波那契数分配顺串的2-路伪多相合并的方法，实际用起来可能并没有什么用。）

##### 替换选择

外部排序的步骤可以归为两大步骤：构造顺串 => 递归合并顺串

多路、多相合并都是在合并顺串这一步进行时间和空间的优化，那么理应构造顺串这一步也可以进行优化。

替换选择是一个产生顺串的算法。开始，M个记录被读入内存并放在一个优先队列中，执行一次DeleteMin把最小的记录写到输出磁带中，继续从磁带读取下一个记录，如果这个记录比刚刚写到输出磁带的元素大就放进优先队列里并再次执行DeleteMin和写入，否则，不能把它放入优先队列里，现在优先队列缺少一个元素，因此可以把这个新元素存入优先队列的死区，直到优先队列里面充满死区元素。此时完成一个顺串的构造，对现在优先哪咧里的死区进行一次构建优先队列，开始构建第二个顺串。

使用上面的例子和替换选择算法构造顺串的过程如下，其中M=3，死区元素用*表示：
![](https://github.com/qweasdzxcpkh/images/blob/master/DataStructureAndAlgorithm/external_sorting9.png?raw=true)


这个例子中替换选择只产生了3个顺串，进行3-路合并只需要一趟就完成了。可以证明替换选择平均产生的顺串长度为2M。由于外部排序花费的时间过多，节省的每一趟操作都对运行时间产生显著的影响。

替换选择可能并不比标准的算法好，不过在外部排序需要尽量少的顺串从而节省趟数的情况下，替换选择有特殊的价值。

