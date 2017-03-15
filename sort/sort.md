时间复杂度：分析关键字的比较次数和记录的移动次数。  
它定量描述了该算法的运行时间。这是一个代表算法输入值的字符串的长度的函数。时间复杂度用大O符号表述，不包括这个函数的低阶项
和首项系数。使用这种方式时，时间复杂度可被称为是渐近的，亦即考察输入值大小趋近无穷时的情况。  

为了计算时间复杂度，我们通常会估计算法的操作单元数量，每个单元运行的时间都是相同的。因此，总运行时间和算法的操作单元数量最多
相差一个常量系数。

相同大小的不同输入值仍可能造成算法的运行时间不同，因此我们通常使用算法最坏情况复杂度，记为`T(n)`，定义为任何大小的输入n所需的
最大运行时间。另一种较少使用的方法是平均情况复杂度，通常有特别指定才会使用。时间复杂度可以用函数`T(n)`的自然特性加以分类。
例如：T(n) = O(n)的算法被称作*线性时间算法*；而 T(n) = O(Mn) 和 Mn= O(T(n)) ，其中 M ≥ n > 1 的算法被称作指数时间算法。

[时间复杂度](https://zh.wikipedia.org/wiki/时间复杂度)
|名称|复杂度类|运行时间(T(n))|运行时间举例|算法举例|
| ---- | ---- | ---- | ---- | ---- |
|常数时间|      |O(1)|10|判断一个二进制数的奇偶|
|迭代对数时间|      |O(log*n)|    |      |
|对数时间|      |O(logn)|logn, logn^2|二分搜索|
|线性时间|      |O(n)|n|无序数组的搜索|
|线性对数时间|      |O(nlogn)|nlogn, logn!|最快的比较排序|
|二次时间||O(n^2)|n^2|冒泡排序、插入排序|
|三次时间||O(n^3)|n^3|矩阵乘法的基本实现，计算部分相关性|
-------
空间复杂度：分析排序算法中需要多少辅助内存
记做S(n)=O(f(n))。比如直接插入排序空间复杂度O(1)。而一般递归算法就要有O(n)的空间复杂度，
因为每次递归都要存储返回信息。

-----------
稳定性：若两个记录A和B的关键字相等，但是排序后AB的先后次序保持不变（稳定）

# 1.插入排序
## 1.1 性能分析
时间复杂度`O(n^2)`,空间复杂度`O(1)`
排序时间与输入相关：输入的元素个数；元素已排序的程度。
最佳情况，输入数组是已经排好序的数组，运行时间是`n`的线性函数；最坏情况，输入数组是逆序，运行时间是`n`的二次函数

## 1.2 核心代码