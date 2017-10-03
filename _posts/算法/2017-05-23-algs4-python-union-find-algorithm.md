---
title: Union-Find in Python
date: 2017-09-30 21:05:13 +0800
layout: post
categories:
  - Python
  - 算法
tags:
  - Python
  - 算法
---

# Union-Find in Python

Union-Find 算法（中文称并查集算法）是解决动态连通性（Dynamic Conectivity）问题的一种算法，作者以此为实例，讲述了如何分析和改进算法，本节涉及三个算法实现，分别是Quick Find, Quick Union 和 Weighted Quick Union。

## 动态连通性(Dynamic Connectivity）

动态连通性是计算机图论中的一种数据结构，动态维护图结构中相连接的组信息。
简单的说就是，图中各个点之间是否相连、连接后组成了多少个组等信息。我们称连接在一起就像形成了一个圈子似的，成为一个组(Component)，每个组有其自己的一些特征，比如组内所有成员都有同一个标记等。

提到圈子，大家比较好理解，我们在社交网络中，彼此熟悉的人之间组成自己的圈子，"熟悉"用计算机中的语言来表示就是“Connected 连通的。圈子是会变化的，今天你又新认识了某人，明天你跟某人友尽了，这种变化是动态的，所以有了动态连通性这种数据结构和问题。

比较常见的应用有，社交网络中比如LinkedIn, 判断某个用户与其它用户是否熟悉：如果你与用户A熟悉，用户A与用户B熟悉，则认为你与用户B也是连接的，你可以看到用户B的信息。在计算机网络中，也存在类似的情况，判断网络中某两个节点是否相连。

## Dynamic Connectivity 的计算机语言表述

给定一个整数对(p,q)，如果p 和 q尚未连通，则使二者相连通。p 和 q 相连后，我们称 p 和 q 在同一个组内。

当p 和 q 连通时，以下关系则成立：

- 自反性：p和p自身是相连的
- 对称性：如果p和q相连，那么q和p也相连
- 传递性：如果p和q相连，q和r相连，那么p和r也相连

在一个网络中，会存在很多类似的整数对(p,q)，假设网络容量是 N，我们可以定义一个从 0 到 N-1的整数数组，p,q是其中的值，我们可能需要的操作有：

- 判断 p 和 q 是否相连
- 如果未相连，则连接 p 和 q, 如果已相连，则可以不做啥
- 查找 p 或 q 属于哪个组中 (如圈子）

这里的一个关键是，如何确定 p 和 q 是在同一个组内。这意味着，每个组需要有一些特定的属性，我们在后面的算法中会有考虑。

## Union-Find 算法描述 Dynamic Connectivity

Union-Find 算法中，提供了对应的方法来实现我们前面提到的可能的操作：

- connected(): 判断 p 和 q 是否相连，这里要调用 find(p) 和 find(q)，如果二者属于同一个组，则认为是相连的，即isConnected()返回true.
- union(): 如果未相连，则连接 p 和 q, 如果已相连，则可以不做啥
- find(): 查找 p 或 q 属于哪个组中 (如圈子），这里返回值是整数，作为组的标识符(component identifier)。

- count(): 返回组的数量

算法4中的API:

```
class UF:
    def __init__(self,N):
    def union(self,p,q): # initialize N sites with integer names
    def find(self,p): #return component identifier for p
    def connected(self,p,q): #return true if p and q are in the same component
    def count(): #number of components
```

## Union-Find 算法及实现

根据我们前面的描述，如果确定每个组的标识符似乎比较关键，只要确定了，就可以判断是否相连。

那用什么来作为标识符，区分各个组呢？

最简单的一个办法是，所有的节点都赋予一个 ID，如果两个节点相连，则将这两个节点的 ID 设成一样的，这样，这两个节点便属于同一个组了。网络中每个组都有了一个唯一的 ID。只要节点 p 和 q 的 ID 相同，则认为节点 p 和 q 相连。我们用数组来放置节点 ID，find()方法可以快速返回 ID，所以我们的第一个算法就叫做 QuickFind。

### QuickFind 算法

QuickFind 算法中，find方法比较简单，union(p,q)方法需要考虑的一点是，要将与p相连的所有节点 id 都设为q当前的 id，使p所在的组和q所在的组结合成了一个同一组。（注：也可以把与q相连的所有节点id都设为p的id）

最开始的时候，所有节点都互不相连。我们假设所有的节点由id=0到N-1的整数表示。

![图片.png](http://upload-images.jianshu.io/upload_images/2220305-0977af423722723d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

代码：

```
# -*- coding: utf-8 -*-

class QuickFind(object):
    id=[]
    count=0

    def __init__(self,n):
        self.count = n
        i=0
        while i<n:
            self.id.append(i)
            i+=1

    def connected(self,p,q):
        return self.find(p) == self.find(q)

    def find(self,p):    
        return self.id[p]

    def union(self,p,q):
        idp = self.find(p)
        if not self.connected(p,q):
            for i in range(len(self.id)):
                if self.id[i]==idp: # 将p所在组内的所有节点的id都设为q的当前id
                    self.id[i] = self.id[q]
            self.count -= 1

```
我们的测试端代码如下：

```
# -*- coding: utf-8 -*-

import quickfind

qf = quickfind.QuickFind(10)

print "initial id list is %s" % (",").join(str(x) for x in qf.id)

list = [
        (4,3),
        (3,8),
        (6,5),
        (9,4),
        (2,1),
        (8,9),
        (5,0),
        (7,2),
        (6,1),
        (1,0),
        (6,7)
        ]

for k in list:
    p =  k[0]
    q =  k[1]
    qf.union(p,q)
    print "%d and %d is connected? %s" % (p,q,str(qf.connected(p,q)    ))

print "final id list is %s" % (",").join(str(x) for x in qf.id)
print "count of components is: %d" % qf.count
```
运行结果：
```
initial id list is 0,1,2,3,4,5,6,7,8,9
4 and 3 is connected? True
3 and 8 is connected? True
6 and 5 is connected? True
9 and 4 is connected? True
2 and 1 is connected? True
8 and 9 is connected? True
5 and 0 is connected? True
7 and 2 is connected? True
6 and 1 is connected? True
1 and 0 is connected? True
6 and 7 is connected? True
final id list is 1,1,1,8,8,1,1,1,8,8
count of components is: 2
```
下图是算法4中的图示，可供参考：


![图片.png](http://upload-images.jianshu.io/upload_images/2220305-e229d78f2b2185f0.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


QuickFind 算法分析：

find方法快速返回数组的值，但union方法最坏情况下，几乎需要遍历整个数组，如果数组很大（比如社交网络巨大） 、需要连接的节点对很多的时候，QuickFind算法的复杂度就相当大了。所以我们需要改进一下union方法。

### QuickUnion 算法

前面的QuickFind算法中，union的时候可能需要遍历整个数组，导致算法性能下降。有没有什么办法可以不用遍历整个数组，又可以保证同一个组内的所有节点都有一个共同属性呢？树结构。树的所有节点都有一个共同的根节点，每个树只有一个根节点，那每个树就可以代表一个组。union(p,q)的时候，只要把p所在的树附加到q所在的树的根节点，这样，p和q就在同一树中了。

改进后的算法即是QuickUnion算法。我们同样要用到 id 数组，只是这里的 id 放的是节点所在树的根节点。

find(p): 返回的是 p 所在树的根节点
union(p,q): 将 p 所在树的根节点的 id 设为 q 所在树的根节点

![图片.png](http://upload-images.jianshu.io/upload_images/2220305-87d37a8f514246f8.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

代码实现：

```
# -*- coding: utf-8 -*-

class QuickUnion(object):
    id=[]
    count=0

    def __init__(self,n):
        self.count = n
        i=0
        while i<n:
            self.id.append(i)
            i+=1

    def connected(self,p,q):
        if self.find(p) == self.find(q):
            return True
        else:            
            return False

    def find(self,p):   
        while (p != self.id[p]):
            p = self.id[p]
        return p

    def union(self,p,q):
        idq = self.find(q)
        idp = self.find(p)
        if not self.connected(p,q):
            self.id[idp]=idq
            self.count -=1            

```
类似的测试端代码：

```
# -*- coding: utf-8 -*-

import quickunion

qf = quickunion.QuickUnion(10)

print "initial id list is %s" % (",").join(str(x) for x in qf.id)

list = [
        (4,3),
        (3,8),
        (6,5),
        (9,4),
        (2,1),
        (8,9),
        (5,0),
        (7,2),
        (6,1),
        (1,0),
        (6,7)
        ]

for k in list:
    p =  k[0]
    q =  k[1]
    qf.union(p,q)
    print "%d and %d is connected? %s" % (p,q,str(qf.connected(p,q)    ))

print "final root list is %s" % (",").join(str(x) for x in qf.id)
print "count of components is: %d" % qf.count
```

运行结果：

```
initial id list is 0,1,2,3,4,5,6,7,8,9
4 and 3 is connected? True
3 and 8 is connected? True
6 and 5 is connected? True
9 and 4 is connected? True
2 and 1 is connected? True
8 and 9 is connected? True
5 and 0 is connected? True
7 and 2 is connected? True
6 and 1 is connected? True
1 and 0 is connected? True
6 and 7 is connected? True
final root list is 1,1,1,8,3,0,5,1,8,8
count of components is: 2
```
算法4中的图示供参考理解：

![图片.png](http://upload-images.jianshu.io/upload_images/2220305-bf94537765b75e35.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

QuickUnion 算法分析：

union方法已经很快速了现在，find方法比QuickFind慢了，其最坏的情况下，如下图，一次find需要访问1+..+N次数组，union方法中需要调用两次find方法，即复杂度变成2(1+...+N)=(N+1)N，接近N的平方了。

![图片.png](http://upload-images.jianshu.io/upload_images/2220305-ab1ff1d142f8817b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### Weighted Quick Union 算法

前面的QuickUnion算法中，union的时候只是简单的将两个树合并起来，并没有考虑两个树的大小，所以导致最坏情况的发生。改进的方法可以是，在union之前，先判断两个树的大小（节点数量），将小点的树附加到大点的树上，这样，合并后的树的深度不会变得非常大。

示例如下：

![图片.png](http://upload-images.jianshu.io/upload_images/2220305-48a8944da6ba3bdd.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

要判断树的大小，需要引进一个新的数组，size 数组，存放树的大小。初始化的时候 size 各元素都设为 1。

代码：

```
# -*- coding: utf-8 -*-

class WeightedQuickUnion(object):
    id=[]
    count=0
    sz=[]

    def __init__(self,n):
        self.count = n
        i=0
        while i<n:
            self.id.append(i)
            self.sz.append(1) # inital size of each tree is 1
            i+=1

    def connected(self,p,q):
        if self.find(p) == self.find(q):
            return True
        else:            
            return False

    def find(self,p):   
        while (p != self.id[p]):
            p = self.id[p]
        return p

    def union(self,p,q):
        idp = self.find(p)
        print "id of %d is: %d" % (p,idp)
        idq = self.find(q)
        print "id of %d is: %d" % (q,idq)
        if not self.connected(p,q):            
            print "Before Connected: tree size of %d's id is: %d" % (p,self.sz[idp])
            print "Before Connected: tree size of %d's id is: %d" % (q,self.sz[idq])
            if (self.sz[idp] < self.sz[idq]):
                print "tree size of %d's id is smaller than %d's id" %(p,q)
                print "id of %d's id (%d) is set to %d" % (p,idp,idq)
                self.id[idp] = idq

                print "tree size of %d's id is incremented by tree size of %d's id" %(q,p)
                self.sz[idq] += self.sz[idp]    
                print "After Connected: tree size of %d's id is: %d" % (p,self.sz[idp])
                print "After Connected: tree size of %d's id is: %d" % (q,self.sz[idq])         
            else:                  
                print "tree size of %d's id is larger than or equal with %d's id" %(p,q)
                print "id of %d's id (%d) is set to %d" % (q,idq,idp)
                self.id[idq] = idp
                print "tree size of %d's id is incremented by tree size of %d's id" %(p,q)
                self.sz[idp] += self.sz[idq]   
                print "After Connected: tree size of %d's id is: %d" % (p,self.sz[idp])
                print "After Connected: tree size of %d's id is: %d" % (q,self.sz[idq])         

            self.count -=1    
```

测试端代码：
```
# -*- coding: utf-8 -*-

import weightedquickunion

qf = weightedquickunion.WeightedQuickUnion(10)

print "initial id list is %s" % (",").join(str(x) for x in qf.id)

list = [
        (4,3),
        (3,8),
        (6,5),
        (9,4),
        (2,1),
        (8,9),
        (5,0),
        (7,2),
        (6,1),
        (1,0),
        (6,7)
        ]

for k in list:
    p =  k[0]
    q =  k[1]
    print "." * 10 + "unioning %d and %d"  % (p,q)  + "." * 10
    qf.union(p,q)
    print "%d and %d is connected? %s" % (p,q,str(qf.connected(p,q)    ))

print "final id list is %s" % (",").join(str(x) for x in qf.id)
print "count of components is: %d" % qf.count
```

代码运行结果：
```
initial id list is 0,1,2,3,4,5,6,7,8,9
..........unioning 4 and 3..........
id of 4 is: 4
id of 3 is: 3
Before Connected: tree size of 4's id is: 1
Before Connected: tree size of 3's id is: 1
tree size of 4's id is larger than or equal with 3's id
id of 3's id (3) is set to 4
tree size of 4's id is incremented by tree size of 3's id
After Connected: tree size of 4's id is: 2
After Connected: tree size of 3's id is: 1
4 and 3 is connected? True
..........unioning 3 and 8..........
id of 3 is: 4
id of 8 is: 8
Before Connected: tree size of 3's id is: 2
Before Connected: tree size of 8's id is: 1
tree size of 3's id is larger than or equal with 8's id
id of 8's id (8) is set to 4
tree size of 3's id is incremented by tree size of 8's id
After Connected: tree size of 3's id is: 3
After Connected: tree size of 8's id is: 1
3 and 8 is connected? True
..........unioning 6 and 5..........
id of 6 is: 6
id of 5 is: 5
Before Connected: tree size of 6's id is: 1
Before Connected: tree size of 5's id is: 1
tree size of 6's id is larger than or equal with 5's id
id of 5's id (5) is set to 6
tree size of 6's id is incremented by tree size of 5's id
After Connected: tree size of 6's id is: 2
After Connected: tree size of 5's id is: 1
6 and 5 is connected? True
..........unioning 9 and 4..........
id of 9 is: 9
id of 4 is: 4
Before Connected: tree size of 9's id is: 1
Before Connected: tree size of 4's id is: 3
tree size of 9's id is smaller than 4's id
id of 9's id (9) is set to 4
tree size of 4's id is incremented by tree size of 9's id
After Connected: tree size of 9's id is: 1
After Connected: tree size of 4's id is: 4
9 and 4 is connected? True
..........unioning 2 and 1..........
id of 2 is: 2
id of 1 is: 1
Before Connected: tree size of 2's id is: 1
Before Connected: tree size of 1's id is: 1
tree size of 2's id is larger than or equal with 1's id
id of 1's id (1) is set to 2
tree size of 2's id is incremented by tree size of 1's id
After Connected: tree size of 2's id is: 2
After Connected: tree size of 1's id is: 1
2 and 1 is connected? True
..........unioning 8 and 9..........
id of 8 is: 4
id of 9 is: 4
8 and 9 is connected? True
..........unioning 5 and 0..........
id of 5 is: 6
id of 0 is: 0
Before Connected: tree size of 5's id is: 2
Before Connected: tree size of 0's id is: 1
tree size of 5's id is larger than or equal with 0's id
id of 0's id (0) is set to 6
tree size of 5's id is incremented by tree size of 0's id
After Connected: tree size of 5's id is: 3
After Connected: tree size of 0's id is: 1
5 and 0 is connected? True
..........unioning 7 and 2..........
id of 7 is: 7
id of 2 is: 2
Before Connected: tree size of 7's id is: 1
Before Connected: tree size of 2's id is: 2
tree size of 7's id is smaller than 2's id
id of 7's id (7) is set to 2
tree size of 2's id is incremented by tree size of 7's id
After Connected: tree size of 7's id is: 1
After Connected: tree size of 2's id is: 3
7 and 2 is connected? True
..........unioning 6 and 1..........
id of 6 is: 6
id of 1 is: 2
Before Connected: tree size of 6's id is: 3
Before Connected: tree size of 1's id is: 3
tree size of 6's id is larger than or equal with 1's id
id of 1's id (2) is set to 6
tree size of 6's id is incremented by tree size of 1's id
After Connected: tree size of 6's id is: 6
After Connected: tree size of 1's id is: 3
6 and 1 is connected? True
..........unioning 1 and 0..........
id of 1 is: 6
id of 0 is: 6
1 and 0 is connected? True
..........unioning 6 and 7..........
id of 6 is: 6
id of 7 is: 6
6 and 7 is connected? True
final id list is 6,2,6,4,4,6,6,2,4,4
count of components is: 2
```
算法4中的图示：

![图片.png](http://upload-images.jianshu.io/upload_images/2220305-83bd72206365d44e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
