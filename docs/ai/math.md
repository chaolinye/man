# 数学

## 排列组合

- 对于n个元素的全排列，所有可能的排列数量就是 `nx(n-1)x(n-2)x…x2x1`，也就是 `n!`；
- 对于n个元素里取出m(0≤n)个元素的不重复排列数量是 `nx(n-1)x(n-2)x…x(n - m + 1)`，也就是 `n!/(n-m)!`。
- n个元素里取出m个的组合，可能性数量就是n个里取m个的排列数量，除以m个全排列的数量，也就是 `(n! / (n-m)!) / m!`。
    > n个里取m个的排列，对于同样的 m 个元素，就是重复了 m 的全排列的次数，所以除以 !m 就可以得到组合的数量

## 概率统计

条件概率：`P(B|A) = P(AB) / P(A)`

全概率公式: `P(B) = P(AB) / P(A)`

$$\displaystyle\sum_{i=1}^n$$

## 线性代数

[线性代数的本质](https://www.bilibili.com/video/BV1ys411472E/?spm_id_from=333.1387.favlist.content.click)

### 向量

从计算机科学的角度，向量是一个数字的序列

从物理学的角度，向量是一个有方向有长度的量

通过数学的角度来概括，向量是一个起始点在坐标原点，有方向有长度的箭头；也可以用箭头终点的坐标来表示向量。

### 向量的线性组合、加法和数乘

向量相加就是将被加向量的起点平移到另一个向量的终点，这时被加向量的终点的坐标就是相加得到的向量，从几何意义上，就是按向量的方向和长度移动了两次。

计算：向量(a,b)可以看做是从原点往x轴移动a距离，然后往y轴移动b距离得到；如果加上(c,d)，那就是再往x轴移动c距离，往y轴移动d距离，因此一共往x轴移动了a+c距离，往y轴移动了b+d距离，最后到达的坐标是(a+c,b+d)，这就是向量加法的坐标计算。

> 向量一般习惯书写为列向量

向量的数乘，就是多个相同向量相加，由于方向相同，拼接后就是往同一个方向加长度，计算上也就是把x轴和y轴坐标加上多次，即 `3(a,b) = (3a,3b)`

> 数乘在几何上就是向量的缩放。

基向量：在一般的二维坐标系中，(0,1) 和 (1,0) 就是两个基向量

线性组合：两个向量的数乘相加叫做这两个向量的线性组合。所有二维向量都可以表达为两个基向量的线性组合.

每个二维向量的坐标，可以拆成两个标量，分别代表对两个基向量的数乘，数乘后组合在一起就是这个二维向量。

向量空间：基向量所有线性组合得到的向量组成向量空间。


### 线性变换

## 微积分

[你也能懂的微积分](https://zhuanlan.zhihu.com/p/94592123)

