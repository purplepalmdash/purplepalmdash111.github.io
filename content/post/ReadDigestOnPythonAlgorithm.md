+++
categories = ["Algorithm"]
date = "2016-09-26T16:28:31+08:00"
description = "AlgorithmForPython"
keywords = ["Python"]
title = "ReadDigestOnPythonAlgorithm"
+++
### 目标
本节目标是:

```
* 理解算法分析的重要性
* 能使用"Big-O"描述算法的执行时间
* 理解常用的Python数组和字典的"Big-O"执行时间
* 理解Python数据的实现是如何影响到算法分析的
* 理解如何对简单的Python程序进行性能基准测试
```

### 算法分析
典型问题是: 当两个程序解决了同一个问题，然而看起来有差别的时候，如何得知一种方案确实优
于另一种？    
理解：算法是用于解决针对某种输入的黑盒实现。某种算法可能针对多种不同程序，这取决于程序
的编写者及编程时使用的编程语言。    

#### 程序对比
两种累加器的实现：    

清晰、明了的实现:    

```
def sumOfN(n):
   theSum = 0
   for i in range(1,n+1):
       theSum = theSum + i

   return theSum

print(sumOfN(10))
```
糟糕的实现:    

```
def foo(tom):
    fred = 0
    for bill in range(1,tom+1):
       barney = bill
       fred = fred + barney

    return fred

print(foo(10))
```
为什么糟糕？ 考虑到程序的可读性。    

#### 算法分析关注点
关注点在于基于对计算资源的量化，以评价算法的好坏。比较两种不同算法，评价出其优劣。譬如
，如果一种算法在解决同一问题时使用了比另一种算法少得多的计算资源时，我们可以说该种算法
较为优秀。    

量化指标: A. 基于该算法在解决问题时所需要用到的内存大小来量化。B. 基于该算法所需的执行
时间来量化。（算法的执行时间/运行时间）.    

用于量化函数`sunOfN`的算法优劣，我们可以使用benchmark。该benchmark使用python的time模块
。用于计算该程序的执行时间.   

#### BenchMark
BenchMark求和程序:    

```
import time

def sumOfN2(n):
   start = time.time()

   theSum = 0
   for i in range(1,n+1):
      theSum = theSum + i

   end = time.time()

   return theSum,end-start

for i in range(5):
   print("Sum is %d required %10.7f seconds"%sumOfN2(10000))
```
运行结果:    

```
$ python2 bench.py
Sum is 50005000 required  0.0014350 seconds
Sum is 50005000 required  0.0013280 seconds
Sum is 50005000 required  0.0012429 seconds
Sum is 50005000 required  0.0012109 seconds
Sum is 50005000 required  0.0012770 seconds
```
更改累加数的大小(10,10000,1000000?)，可以看出程序执行时间的变化。    


优化过后的算法，可以使用公式: `sum = (n)*(n+1)/2`来快速求累加值，代码如下:     

```
import time

def sumOfN3(n):
   start = time.time()

   theSum = 0
   theSum = (n*(n+1))/2

   end = time.time()

   return theSum,end-start

print("Sum is %d required %10.7f seconds"%sumOfN3(10))
print("Sum is %d required %10.7f seconds"%sumOfN3(10000))
print("Sum is %d required %10.7f seconds"%sumOfN3(100000))
print("Sum is %d required %10.7f seconds"%sumOfN3(1000000))
print("Sum is %d required %10.7f seconds"%sumOfN3(10000000))
print("Sum is %d required %10.7f seconds"%sumOfN3(100000000))
```
运行结果为:    

```
$ python2 bench2.py
Sum is 55 required  0.0000019 seconds
Sum is 50005000 required  0.0000000 seconds
Sum is 5000050000 required  0.0000010 seconds
Sum is 500000500000 required  0.0000012 seconds
Sum is 50000005000000 required  0.0000010 seconds
Sum is 5000000050000000 required  0.0000000 seconds
```

