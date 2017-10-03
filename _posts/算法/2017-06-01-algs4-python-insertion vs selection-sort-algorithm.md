---
title: 插入排序算法和选择排序算法比较
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

- **选择排序算法**：详见 [《算法4》2.1 - 选择排序算法(Selection Sort), Python实现](http://www.jianshu.com/p/93aba5441cc2)

- **插入排序算法(Insertion Sort)**：详见[《算法4》2.1 - 插入排序算法(Insertion Sort), Python实现](http://www.jianshu.com/p/8c2ef0a86ab8)

# 插入排序算法和选择排序算法的复杂度分析:

插入排序和选择排序都有两层循环，外循环遍历整个数组，内循环稍有区别：

- 选择排序的内循环是遍历一组未排过序的数组，
- 插入排序的内循环是遍历一组已排过序的数组，

在此基础上，进行比较或交换。看起来已经排序过的数组中进行插入会感觉性能要好一点，实际未必，这要看数组的具体情况，比如最坏情况下所有数组元素都得过一遍。

插入排序在插入的时候可以做交换操作，也可以不做交换。

改进插入排序算法可以使用二分法等，这里只探讨普通的插入排序。

**算法复杂度**

算法 | 最好情况| 最坏情况
--------| ---------| -----
选择排序|交换0次，比较n(n-1)/2次 |交换N次
插入排序|交换0次，比较N-1次|交换n(n-1)/2次，比较n(n-1)/2次

看过一些教材，普遍说插入排序算法比选择排序要快，实际上从上面的分析可以看出，其实二者的复杂度差不多，都是O(N平方)。后面的代码实现测试中也证实了这一点。

# 插入排序和选择排序算法的比较：

我们的python测试程序，考虑了不同大小的数组排序:

- 1000
- 10000
- 20000

每种又考虑了三种情况：

- 随机生成数
- 最好情况：原数组已从小到大排好
- 最坏情况：原数组已从大到小排好

并与python自带的sort方法作了比较。

完整代码如下：

```
sizes = [
        1000,
        5000,
        10000
        ]

for size in sizes:
    # random generation of items to be sorted
    items = range
    print "-"*10 + "sorting numbers" + "-"*10
    items = []
    for i in range(0,size):
        items.append(random.randint(2,999))
    #print "original items: %r" % items
    # the worse case
    items_worse = range (size-1,-1,-1)
    # the best case
    items_best = range(0,size)

    to_be_sorted = [
            ("random case",items),
            ("worse case",items_worse),
            ("best case",items_best)
            ]

    def duration(sort_method):    
        # calculate execution time for our selection sort method
        start = time.clock()
        sort_method.sort()
        end = time.clock()
        duration = end - start
        return duration

    for item in to_be_sorted:
        temp = copy.deepcopy(item) # for reversing use after a certain sort
        print "-"*10 + item[0] + "-"*10
        # calculate duration for insertion sort
        insertion_sort = InsertionSort(item[1])
        dinsertion = duration(insertion_sort)
        item = temp
        # calculate duration for selection sort    
        selection_sort = SelectionSort(item[1])
        dselection = duration(selection_sort)
        item = temp
        # calculate duration for python builtin sort
        dpython = duration(item[1])
        print "%s: %ds" % ("insertion sort",dinsertion)
        print "%s: %ds" % ("selection sort",dselection)
        print "%s: %ds" % ("python built-in",dpython)
```

运行结果：

size = 1000：挺不错，都是毫秒级，但是看不出区别
```
----------random case----------
item len: 1000
insertion sort: 0s
selection sort: 0s
python built-in: 0s
----------worse case----------
item len: 1000
insertion sort: 0s
selection sort: 0s
python built-in: 0s
----------best case----------
item len: 1000
insertion sort: 0s
selection sort: 0s
python built-in: 0s
```

size=10000: 有区别了，但是很少，1s差别。不过可以明显看出，最好情况下选择排序却用了6s多，最坏情况下，插入排序比选择排序慢了。

```
----------random case----------
item len: 10000
insertion sort: 6s
selection sort: 7s
python built-in: 0s
----------worse case----------
item len: 10000
insertion sort: 8s
selection sort: 7s
python built-in: 0s
----------best case----------
item len: 10000
insertion sort: 0s
selection sort: 6s
python built-in: 0s
```

size=20000: 两种排序算法的耗时都明显提高了，但差别除了最好情况，差别仍然不大，基本说明二者的复杂度是差不多的。python自带的sort方法仍是毫秒级，过段时间等其他排序算法学了后研究下源码。

```
----------random case----------
item len: 20000
insertion sort: 30s
selection sort: 33s
python built-in: 0s
----------worse case----------
item len: 20000
insertion sort: 39s
selection sort: 33s
python built-in: 0s
----------best case----------
item len: 20000
insertion sort: 0s
selection sort: 32s
python built-in: 0s
```
