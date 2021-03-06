---
title: 基数排序(Radix Sort)、计数排序(Counting Sort)与前缀和(Prefix sum)
permalink: radix-sort-and-counting-sort/
date: 2017-03-16 20:09:21
tags:
- 算法
- 排序
- Legacy
categories:
- 技术笔记
author: Kaku
---
最近我在扩展英语的课上读了一篇与在多核GPU上优化排序算法相关的文章。由于本人对排序并没有过多研究，对于排序算法的了解恐怕只停留在数据结构课程涉及的那些，因此读文章时发现对一些排序算法的原理也是知之甚少。因此特地做一些笔记，以备后用。
<!--more-->

## 1.基于比较的排序与非基于比较的排序
在谈排序算法之前，有两个概念需要了解：**基于比较的排序** *(Comparison-based sorting)* 和 **非基于比较的排序** *(Non-Comparison-based sorting)* 。如字面解释，基于比较的排序就是通过比较集合中的元素，通过比较两个元素的键值来决定位置（如数字的大小顺序，字母的字母表顺序），最后达到排序的目的。比如最简单的冒泡排序、选择排序都是典型的基于比较的排序。之所以说它们简单，一方面实现很简单，相信很多工科专业的同学在接触C时，都会做过实现冒泡排序的练习题；另一方面，从逻辑上来看， **比较是最符合人类思维的排序方式。** 无论是挑水果，还是挑女朋友，相信每个人都会从比较开始得出三六九等的。这些行为抽象出来的事物就是排序。因此我们接触排序，大多是从基于比较的排序开始的。

那么非基于比较的排序又是什么呢？显然，既然基于比较的排序是通过比较键值来决定位置，那么非基于比较的排序的排序就是通过其他方式决定位置。事实上，非基于比较的排序是根据键值本身的定位特征来决定位置的，因此就不需要根据其键值的比较来决定其前后位置关系。后面提到的计数排序、基数排序都属于非基于比较的排序的范畴之内的。相比于基于比较的排序反复迭代的运算过程，**非基于比较的排序的时间复杂度通常要比前者低**。即便是快速排序，其在时间复杂度上的表现也不及本文提到的两个非基于比较的排序算法(O(nlgn)>O(n))。但是，这通常是一种以空间换时间的后果（下文将详细说明）。

## 2.基数排序(Radix Sort)
基数排序是非基于比较的排序算法，算法的时间复杂度是O(n+k)。相比于快速排序的O(nlgn)，从表面上看具有不小的优势。但事实上可能有些出入，因为基数排序的n可能具有比较大的基数k。因此在具体的应用中，应首先对这个排序函数的效率进行评估。

基数排序的主要思路是，将所有待比较数值（**注意，必须是正整数**）统一为同样的数位长度，数位较短的数前面补零。然后，从最低位开始，依次进行一次**稳定排序**。这样从最低位排序一直到最高位排序完成以后，数列就变成一个有序序列。**因为整个过程并不是直接比较数值大小，而是通过对数值的各个数位排序来获得最终顺序，因此它是一个非基于比较的排序算法。**

比如我们要对以下数列从小到大排序：621，497，961，16。以下是排序过程（粗体是排序位）。

排序前：

- 6 2 1
- 4 9 7
- 9 6 1
- 0 1 6

第一次排序（个位）：

- 6 2 **1**
- 9 6 **1**
- 0 1 **6**
- 4 9 **7**

第二次排序（十位）：

- 0 **1** 6
- 6 **2** 1
- 9 **6** 1
- 4 **9** 7

第三次排序（百位）：

- **0** 1 6
- **4** 9 7
- **6** 2 1
- **9** 6 1

排序结果即为：16，497，621，961。

两个问题:
> <font color="black">“为什么要从低位到高位排序？”

如果要从高位排序，那么次高位的排序会影响高位已经排好的大小关系。在数学中，数位越高，数位值对数的大小的影响就越大。从低位开始排序，就是对这种影响的排序。**数位按照影响力从低到高的顺序排序，数位影响力相同则比较数位值**。

> <font color="black">“为什么要保证稳定排序？”

稳定排序的意思是指，待排序**相同**元素之间的相对前后关系，在各次排序中不会改变。比如如果我们要排序的数列中有这样两个数字：128和164。当十位排序之后，128的位置会在164前面。当进行百位排序时，为了保证128和164两个数字的正确顺序，我们需要保证百位相同的这两个数能够保留十位的排序。因此我们需要使用稳定排序。

## 3.计数排序(Counting Sort)与前缀和(Prefix sum)
在刚刚谈到的基数排序中，我跳过了一个细节，即特定数位的排序的算法。当然，四个十进制数字在我们看来排起序来很简单，你可以用任何一种熟悉而又简单的稳定排序算法。**目前比较流行的一种算法就是计数排序**，与基数排序一样，这也是一种非基于比较的排序。

计数排序是一个非基于比较的排序算法，该算法于1954年由 Harold H. Seward 提出。它的优势在于在对一定范围内的整数排序时，它的复杂度为Ο(n+k)（其中k是整数的范围，即基数），快于任何比较排序算法。当然这是一种牺牲空间换取时间的做法，而且当O(n+k)>O(nlgn)的时候其效率反而不如基于比较的排序。

### 3.1 基数的概念
在说明计数排序的具体原理之前，先说明一个基本概念：**基数** *(radix)*。相信熟悉排序算法的同学对这个概念不会陌生。这其实集合系统中的概念。Wiki对基数这个概念给出了如下解释：

> In mathematical numeral systems, the radix or base is the number of unique digits, including zero, used to represent numbers in a positional numeral system. For example, for the decimal system (the most common system in use today) the radix is ten, because it uses the ten digits from 0 through 9.

简单的说，就是某个数位上的可能出现的数字个数。如十进制下的基数就是10，因为十进制下的一个数位可能出现的数字是0~9共10个数字。所以我们可以说十进制是基10（radix-10）的。同理，基2、基4也是同理。~~不要和我提八进制！~~

### 3.2 前缀和的概念
在计数排序中，我们还会使用到一个概念：**前缀和** *(prefix sum)*。

Wiki上的前缀和：
> In computer science, the prefix sum, cumulative sum, inclusive scan, or simply scan of a sequence of numbers x0, x1, x2, ... is a second sequence of numbers y0, y1, y2, ..., the sums of prefixes (running totals) of the input sequence:
y0 = x0
y1 = x0 + x1
y2 = x0 + x1+ x2
...

比如自然数的前缀和如下：

|input numbers|1|2|3|4|5|6|...|
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
|prefix sum|1 |3 |6 |10|15|21|...|

我们主要用到的是数组前缀和。即：

```c++
    s[1]=a[1]
    s[2]=a[1]+a[2]
    s[3]=a[1]+a[2]+a[3]
    ...
```

### 3.3 开始计数排序
了解了基数和前缀和的概念，我们就可以开始考虑计数排序的算法了。之前提到，计数排序是一种以空间换时间的算法，原因就在相较于基于比较的排序算法，它所需要的内存空间更多一些（如计数数组）。下面通过一个例子来阐述计数排序的原理。

比如，我们要对6，2，8，6，2，4，9这个十进制数字数列进行从小到大的计数排序。

首先我们要申请一个长度为基数的数组，在这个例子我们要申请的就是长度为10的数组`r[10]`。这个数组是对数列中出现的数字进行计数。比如数列出现了一个1，我们让`r[1]`自加1。

之后我们要对待排数列进行一次扫描（pass），扫描过程中我们同时也对数组`r[10]`进行修改。这个例子中的`r[10]`在扫描过后的值如下所示。

|`n`|0|1|2|3|4|5|6|7|8|9|
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
|`r[n]`|0|0|2|0|1|0|2|0|1|1|

这样我们就获得了数列中每个数字的出现次数。

接下来我们就要使用`r[10]`的前缀和来确定待排数列元素的位置。

前缀和数组`s[n]`：

|`n`|0|1|2|3|4|5|6|7|8|9|
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
|`s[n]`|0|0|2|2|3|3|5|5|6|7|

根据前缀和数组，我们可以获知的信息是小于某个大于0的数字`n`的数字个数`s[n-1]`（如果是0，小于0的数字个数自然是0，以下的`n`均大于0）。比如，我们想要知道待排数列中小于6的数字有几个，我们就可以查看`s[5]`，即3个。

如此一来，知道了前面有几个小于`n`的数字个数(`s[n-1]`)，自然就知道了值为`n`应该在排序序列的第几个（从第几个开始，有几个放几个）。

排序过程如下：

（扫描0、1后，根据`r[0]`和`r[1]`获知原数组没有0和1，新数组均无变化，扫描2以后，`r[2]==2`，将两个2放入新数组）

|`n`|0|1|2|3|4|5|6|
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
|`a[n]`|2|2|-|-|-|-|-|

（同理，扫描3、4以后）

|`n`|0|1|2|3|4|5|6|
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
|`a[n]`|2|2|4|-|-|-|-|

（扫描5、6）

|`n`|0|1|2|3|4|5|6|
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
|`a[n]`|2|2|4|6|6|-|-|

...

最后结果：

|`n`|0|1|2|3|4|5|6|
|:---:|:---:|:---:|:---:|:---:|:---:|:---:|:---:|
|`a[n]`|2|2|4|6|6|8|9|

**	**

看到这里想必很多人都明白为什么基数排序中的数位排序多使用计数排序了。基数排序的每一数位的排序都是针对某一进制下的单位数字进行排序，这与计数排序的应用范围十分吻合。再加上计数排序良好的时间复杂度，使得计数排序成为了基数排序最佳损友~~简称基友~~之一。当然也有很多其他排序也可以应用到基数排序中，如桶排序等。

当然，也有一些不适用计数排序的场合。计数排序的时间复杂度是O(n+k)，其中n是待排数列长度，而k则是对应进制的基数。如果这个基数足够大，则很有可能导致时间复杂度很大，甚至大于一些基于比较的排序时，那么显然计数排序就不是最好的选择了。

好了，以上就是基数排序与计数排序的基本概念与原理。关于基数排序的并行化，以及更多深入的性能分析，本文均未涉及。不过日后可能还会继续研究吧，毕竟也是手头这篇文章的研究方向。

什么？你说代码实现呢？我还是先搞定扩展英语的weekly assignment再说吧……~~其实就是懒~~

---
**Reference Source:**
1. [算法总结系列之五: 基数排序(Radix Sort)](http://www.cnblogs.com/sun/archive/2008/06/26/1230095.html "http://www.cnblogs.com/sun/archive/2008/06/26/1230095.html")
2. [Radix - Wikipedia](https://en.wikipedia.org/wiki/Radix "https://en.wikipedia.org/wiki/Radix")
3. [Prefix sum - Wikipedia](https://en.wikipedia.org/wiki/Prefix_sum "https://en.wikipedia.org/wiki/Prefix_sum")
4. [Counting sort - Wikipedia](https://en.wikipedia.org/wiki/Counting_sort "https://en.wikipedia.org/wiki/Counting_sort")
