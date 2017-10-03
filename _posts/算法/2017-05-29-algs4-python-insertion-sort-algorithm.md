---
title: 插入排序算法 Insertion Sort
date: 2017-09-30 21:05:13 +0800
layout: post
categories:
  - Python
  - 算法
tags:
  - Python
  - 算法
---

**排序算法列表电梯**：

- **选择排序算法：详见 [Selection Sort](http://www.jianshu.com/p/93aba5441cc2) **

- **插入排序算法(Insertion Sort)**：非常适用于小数组和部分排序好的数组，是应用比较多的算法。详见本文

# 插入排序算法的语言描述：

大家都打过牌吧，理牌的时候，每人手里一把牌，一般都会按由大到小顺序排好，每抓一个新牌（比如 5），都会找到4和6，把6往后挪一下，然后把5插到4和6之间。

插入排序算法的原理与理牌是一样的，在一组未排序或部分排序的物体中，将物体从左到右挨个比较，每比较一次，将物体从小到大排好，每次比较后，前面几个物体都是排好序了的，后面的物体插入到前面已排好的序列，以此类推直到全部排序完毕。

这里的关键是，在前面已经排好序的数组中**插入**后面的物体，所以叫做插入排序。

# 插入排序算法的计算机语言描述

从一个 N 个数的数组或列表中，按从大到小或从小到大排序，排序的方法是：

1 确定是按从大到小还是从小到大排。（这里我们选择从小到大排序）

2 从小到大排的话，将第二个与第一个比较，如果小于第一个，则与第一个交换位置。反之不变。

4 将第三个与其前两个分别比较，如果小于则插入。

6 以此类推，直到最后一个。

要注意的是，插入的时候，需要把数组中待插部位的值往后挪哦。

# 插入排序算法的实现

这里用的是Python。

刚开始的时候为了突出理解插入的概念，用了个笨办法，显性的加了一步将数组值后挪的步骤，以便插入。后面会给出更简洁的代码。

算法实现代码(insertion_sort.py)：

```
# -*- coding: utf-8 -*-

class InsertionSort(object):
    items = []
    def __init__(self,items):
        self.items = items
    def sort(self):
        for i in range(0,len(self.items)-1):
            temp = self.items[i]
            if (self.items[i] < self.items[i-1]):
                for j in range(0,i):
                    if (self.items[j] > temp):
                        for k in range (i,j,-1):
                            self.items[k] = self.items[k-1]
                        self.items[j] = temp
                        break;
```

上面的代码可以比较清晰的看到插入是如何工作的，但确实很傻，可以精简如下：

```
class InsertionSort_Refined(object):
    items = []
    def __init__(self,items):
        self.items = items
    def sort(self):
        for i in range(0,len(self.items)-1):
            j = i;
            while (j > 0 and self.items[j] < self.items[j-1]):
                self.items[j],self.items[j-1] = self.swap(self.items[j],self.items[j-1])
                j -=1
    def swap(self,i,j):
        temp = j
        j = i
        i = temp
        return i,j

```

测试代码：

```
# -*- coding: utf-8 -*-
import random
from timeit import default_timer as timer

from insertion_sort import InsertionSort

print "-"*10 + "sorting numbers" + "_"*10
items = []
for i in range(0,10):
    items.append(random.randint(2,999))
print "original items: %r" % items
ssort = InsertionSort(items)

# calculate execution time for our Insertion sort method
start = timer()
ssort.sort()
end = timer()
duration1 = end - start
# calculate execution time for python built-in sort method
start = timer()
items.sort()
end = timer()
duration2 = end - start

assert ssort.items == items
print "sorted items: %r" % ssort.items
print "Duration: our Insertion sort method - %ds, python builtin sort - %ds" % (duration1, duration2)

```

测试代码中，我们还用了python自带的sort方法，通过 "assert ssort.items == items" 一行语句是来验证我们的插入排序算法运行结果的正确性。并且加了timer,来比较我们的算法和python自带的sort方法的运行时间。

运行结果表明，排序的结果是一样的，和选择排序算法差不多，当数据量大的时候，运行性能比python自带的sort算法差很多。

运行结果示例(数组size=10)：

```
----------sorting numbers__________
original items: [420, 373, 678, 818, 264, 30, 150, 310, 101, 833]
sorted items: [30, 101, 150, 264, 310, 373, 420, 678, 818, 833]
Duration: our Insertion sort method - 0s, python builtin sort - 0s
```

# 插入排序算法分析

通过前面算法实现的例子，插入排序算法也是有性能问题的。

我们试着通过在算法中用到的比较次数和值交换次数来分析一下：

第二个与第一个比较时，需要比较 1 次，可能需要交换1次
第三个的时候，可能需要比较 2 次，可能需要交换2次
找最后一个值时，可能需要比较N-1次，可能需要交换N-1次

所以，最坏情况下，一共交换 1+2+...+(N-1) 次 = (N的平方)/2次，比较次数也是 1+2+...+(N-1) 次 = (N的平方)/2次。其复杂度已经不是线性的了。

但最好情况下，即数组已经是排序好了的，则只需要N-1次比较，无需任何交换。

所以，如果数组已经是部分或者全部排序过了，则使用插入排序算法无疑是不错的选择。
