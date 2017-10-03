---
title: 选择排序算法(Selection Sort)
date: 2017-09-30 21:05:13 +0800
layout: post
categories:
  - Python
  - 算法
tags:
  - Python
  - 算法
---

选择排序算法(Selection Sort)是排序算法的一种初级算法。虽然比较简单，但是基础，理解了有助于后面学习更高深算法，勿以勿小而不为。

# 排序算法的语言描述：

给定一组物体，根据他们的某种可量化的属性，进行从大到小或从小到大排序。
比如，上体育课的时候，同学们按照身高排队。

排序看起来是一个简单的问题，但针对它的计算机算法有很多，性能各不一样。本文的选择算法即是其中一种。

# 选择排序算法的语言描述：

选择排序算法是，从一组未排序的物体中，根据某可量化的属性，先选出最小或最大的一个，放到第一个位置；然后再从剩下的物体中，选出最小（或最大）的，放到第二个位置，以此类推直到全部排完。

这里的重点是，存在两个选择问题，一个是每次要选出一个极值，另一个是选择极值是最大还是最小，所以叫选择排序算法，这是我为了便于记住这个算法名及其逻辑，这么理解的。

以同学们按身高排队为例：第一次，所有同学中最矮的那个拎出来，跟第一个同学换一下位置，这样第一个位置上的同学就是最矮的了，保持不动。接下来，比较第二个同学与他后面的同学挨个比较，找出最矮的和第二个同学换下位置，如果第二个同学是最矮的就保持不动。以此类推。

# 选择排序算法的计算机语言描述

从一个 N 个数的数组或列表中，按从大到小或从小到大排序，排序的方法是：

1 确定是按从大到小还是从小到大排。（这里我们选择从小到大排序）

2 从小到大排的话，数组的第一个值最后存放的是最小的值，最后一个值存放最大值

3 确定是从第一个值还是最后一个值开始。从第一个值开始的话，每次找出最小值，从前往后更新数组。从最后一个值开始的话，每次找出最大值，从后往前更新数组。（下文的算法实现选择的是后者）

4 假设我们每次找最大值，从后往前更新数组。第一轮将最后一个值与其他所有值挨个比较，将最大的那个与最后一个值相交换。如果最后一个值就是最大值，则无需作。这样，数组的最后一个值现在存放的就是整个数组的最大值了。

5 第二轮将倒数第二个值与它前面的所有值挨个比较，找出最大值与倒数第二个值交换。

6 以此类推，直到第二个和第一个值比较。

# 选择排序算法的实现

这里用的是Python，([Java 实现请参考Selection.java](http://algs4.cs.princeton.edu/21elementary/Selection.java.html))

选择的是每次寻找最大值，从后往前排。和经典选择排序算法有点不太一样（最小值法），是为了给自己增加点难度避免直接看答案。

算法实现代码(selection_sort.py)：

```
# -*- coding: utf-8 -*-

class SelectionSort(object):
    items=[]
    def __init__(self,items):
        self.items = items

    def sort(self):
        print "iten len: %d" % len(self.items)
        for i in range(len(self.items)-1,0,-1):
            maximum = i
            for j in range(0,i):            
                if (self.items[i] < self.items[j]):
                    maximum = j
            self.items[i],self.items[maximum]=self.swap(self.items[i],self.items[maximum])
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
import string
from timeit import default_timer as timer

from selection_sort import SelectionSort

print "-"*10 + "sorting numbers" + "_"*10
items = []
# generate random numbers and put in items
for i in range(0,10):
    items.append(random.randint(2,999))
print "original items: %r" % items

ssort = SelectionSort(items)

# calculate execution time for our selection sort method
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
print "Duration: our selection sort method - %ds, python builtin sort - %ds" % (duration1, duration2)

```

测试代码中，我们还用了python自带的sort方法，通过 "assert ssort.items == items" 一行语句是来验证我们的选择排序算法运行结果的正确性。并且加了timer,来比较我们的算法和python自带的sort方法的运行时间。

运行结果表明，排序的结果是一样的，但我们的算法在数组很大的时候，比如数组size 在4000左右，需要耗时1s多，而python自带的算法超快，毫秒级。当数据到10,000时我们的算法要运行5s多，但python自带的sort方法仍然不到1s。这说明选择排序算法简单，但是性能并不佳。

运行结果示例(数组size=10)：

```
----------sorting numbers__________
original items: [434, 706, 256, 95, 549, 380, 585, 535, 722, 689]
iten len: 10
sorted items: [95, 256, 380, 434, 535, 549, 585, 689, 706, 722]
Duration: our selection sort method - 0s, python builtin sort - 0s
```

关于字符串排序，也一并放上来测试代码和运行结果：

```
# -*- coding: utf-8 -*-
import random
import string
from timeit import default_timer as timer
from selection_sort import SelectionSort

print "-"*10 + "sorting alpha characters" + "_"*10
items=[]
for i in range(0,10):
    items.append(random.choice(string.ascii_letters))
print "original items: %r" % items
ssort = SelectionSort(items)
ssort.sort()
items.sort()
assert ssort.items == items
print "sorted items: %r" % ssort.items

print "-"*10 + "sorting strings" + "_"*10
items=[]
for i in range(0,10):
    items.append("".join(random.choice(string.ascii_letters+string.digits) for s in range(0,10) ))
print "original items: %r" % items
ssort = SelectionSort(items)
ssort.sort()
items.sort()
assert ssort.items == items
print "sorted items: %r" % ssort.items
```

运行代码：

```

----------sorting alpha characters__________
original items: ['m', 'q', 'a', 'c', 'n', 'w', 'V', 'f', 'a', 'p']
iten len: 10
sorted items: ['V', 'a', 'a', 'c', 'f', 'm', 'n', 'p', 'q', 'w']
----------sorting strings__________
original items: ['QSvTmfkUAX', 'uf2dEtEtlk', 'lzyYWD3w59', '3pCKM10RPK', 'ARDf403rrl', 'dZyirpxn2N', 'picoZ7yvhR', 'VU9aW1PSbt', 'YvwqwzO39r', 'ROxlks3zAl']
iten len: 10
sorted items: ['3pCKM10RPK', 'ARDf403rrl', 'QSvTmfkUAX', 'ROxlks3zAl', 'VU9aW1PSbt', 'YvwqwzO39r', 'dZyirpxn2N', 'lzyYWD3w59', 'picoZ7yvhR', 'uf2dEtEtlk']
```

# 选择排序算法分析

通过前面算法实现的例子，选择排序算法是有性能问题的。

我们试着通过在算法中用到的比较次数和值交换次数来分析一下：

找第一个最大值时，需要比较 N-1次，交换1次
找第二个最大值时，需要比较 N-2次，交换1次
找最后一个值时，需要比较1次，交换1次

所以，一共交换 1+2+...+(N-1) 次 = (N的平方)/2次，其复杂度已经不是线性的了。总交换次数是N次，相对还好点。

思考一下，如果要改进选择排序算法，有什么办法？
