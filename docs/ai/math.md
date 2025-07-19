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

从数学的角度，要抽象概括上面两种角度，定义了向量要支持有意义的加法和数乘。

通过坐标系建立多种观点之间的联系： 向量是一个起始点在坐标原点，有方向有长度的箭头；也可以用箭头终点的坐标来表示向量。

向量可以分解为在每个 axis 上的移动的组合

向量的基本运算：加法和数乘

向量的加法就是向量移动的组合

向量的数乘可以理解为向量的缩放

线性代数的魅力在于空间箭头与数字序列的转换，例如数据的可视化；空间的变化可以通过转换成数列用计算机来计算。

向量相加就是将被加向量的起点平移到另一个向量的终点，这时被加向量的终点的坐标就是相加得到的向量，从几何意义上，就是按向量的方向和长度移动了两次。

计算：向量(a,b)可以看做是从原点往x轴移动a距离，然后往y轴移动b距离得到；如果加上(c,d)，那就是再往x轴移动c距离，往y轴移动d距离，因此一共往x轴移动了a+c距离，往y轴移动了b+d距离，最后到达的坐标是(a+c,b+d)，这就是向量加法的坐标计算。

> 向量一般习惯书写为列向量

向量的数乘，就是多个相同向量相加，由于方向相同，拼接后就是往同一个方向加长度，计算上也就是把x轴和y轴坐标加上多次，即 `3(a,b) = (3a,3b)`

> 数乘在几何上就是向量的缩放。

### 线性组合、张成的空间与基

核心概念：

单位向量（unit vector）

坐标系（coordinate system）

基向量（basic vector）：在一般的二维坐标系中，(0,1) 和 (1,0) 就是两个基向量

坐标的数字称为标量（scalar），坐标可以看作是标量对 basic vector 进行缩放和组合。

通过 basic vector 可以构建整个坐标系，选择不同的 basic vector 可以得到不同的坐标系

当用数字描述向量时，依赖于正在使用的基。

两个向量的线性组合（Linear combination）：两个向量的数乘相加

> 线性的理解：固定其中一个 scalar，变化另一个，得到一条直线

```tex
\vec{v} 与 \vec{w} 全部线性组合构成的向量集合称为张成的空间（span）
```

vector 与 point，当箭头表达在页面过于繁琐，可以用点表示

线性相关（Linear dependent）：一个向量可以表达为其它向量的线性组合。反之就是线性无关。

空间的基：张成该空间的一个线性无关向量的集合。

### 矩阵与线性变换（Linear transformation）

transformation 本质上就是个 function，input 是一个 vector，output 一个新的 vector

> 使用 transformation 这个词是因为更好地表达出一个 vector move 到新的 vector

线性变换：保持网格线平行且等距分布，并且保持原点不动

线性变换是操纵空间的一种手段。

空间中的向量可以看作是基向量的线性组合，经过线性变换后，仍然是相同的线性组合，因此可以通过变换后基向量的坐标得到所有其它向量的新坐标。

变换后的基向量可以写成矩阵的形式

```tex
\begin{bmatrix} a & c \\ b & d \end{bmatrix} \begin{bmatrix} x \\ y \end{bmatrix}
= x \begin{bmatrix} a \\ b \end{bmatrix} + y \begin{bmatrix} c \\ d \end{bmatrix}
= \begin{bmatrix} ax+cy \\ bx+dy \end{bmatrix}
```

线性变换可以用一组数字表达，这组数字每列就是变换后的基向量。

矩阵向量乘法就是计算线性变换作用于给定向量的一种途径。

看到一个矩阵，都可以解读为对空间的一种特定变换。

### 矩阵乘法与线性变换复合

复合的线性变换：

```tex
\begin{bmatrix} 0 & 2 \\ 1 & 0 \end{bmatrix} ( \begin{bmatrix} 1 & -2 \\ -2 & 0 \end{bmatrix} \begin{bmatrix} x \\ y \end{bmatrix} )
```

等同于单一最终的基向量变换

```tex
\begin{bmatrix} 2 & 0 \\ 1 & -2 \end{bmatrix} \begin{bmatrix} x \\ y \end{bmatrix}
```

也就是

```tex
\begin{bmatrix} 0 & 2 \\ 1 & 0 \end{bmatrix} \begin{bmatrix} 0 & -2 \\ 1 & 0 \end{bmatrix} = \begin{bmatrix} 2 & 0 \\ 1 & -2 \end{bmatrix}
```

可以理解从右到左的基向量的线性变换

通过空间的想象可以知道矩阵乘法不支持交换律，但是支持结合律

矩阵乘法 = 线性变换的复合

### 行列式（determinant）

行列式就是对空间的缩放比例

计算基向量变换后的面积/体积的缩放比例，即可得到转换矩阵的行列式

```tex
det(\begin{bmatrix} 2 & 0 \\ 0 & 3 \end{bmatrix} = 6
```
<br/>

```tex
det(\begin{bmatrix} 1 & 1 \\ 0 & 1 \end{bmatrix} = 1
```
<br/>

```tex
det(\begin{bmatrix} 4 & 2 \\ 2 & 1 \end{bmatrix} = 0   ==> 降维
```

行列式为负数，就是把空间反转了，是否反转可以用基向量的相对方向来判断（右手法则）

行列式为0，代表降维，也表明矩阵的列是线性相关的。

矩阵乘法的行列式：

```tex
det(\mathbf{M_1} \mathbf{M_1}) = det(\mathbf{M_1}) det(\mathbf{M_2})
```

### 逆矩阵、列空间与零空间

线性方程组（Linear system of equations）：在每一个方程中，所有未知量只具有常系数，这些未知量之间只进行加法

```
2x + 5y + 3z = -3   
4x + 0y + 8z = 0  
1x + 3y + 0z = 2
```

可以换成矩阵和向量的写法

```tex
\begin{bmatrix} 2 & 5 & 3 \\ 4 & 0 & 8 \\ 1 & 3 & 0 \end{bmatrix} \begin{bmatrix} x \\ y \\ z \end{bmatrix} = \begin{bmatrix} -3 & 0 & 2 \end{bmatrix}
```

这个矩阵叫做系数矩阵

```tex
\mathbf{A} \vec{x} = \vec{v}
```
<br/>

```tex
求解 \vec{x} 经过\mathbf{A} 变换后和 \vec{v} 重叠
```

```tex
需要考虑 det(\mathbf{A}) 是否等于 0，等于 0 意味着降维，会有无数的解或者无解（\vec{v}不在降维后的空间中）; det(\mathbf{A}) != 0，有且只有一个解
```
<br/>

```tex
从 \vec{v} 反转换为 \vec{x} 的转换被称为 \mathbf{A} 的逆记为\mathbf{A^{-1}}
```

逆矩阵明显有以下特性:

```tex
\mathbf{A^{-1}} \mathbf{A} = \begin{bmatrix} 1 & 0 \\ 0 & 1 \end{bmatrix}
```

<br/>

```tex
\vec{x} = \mathbf{A^{-1}}\vec{v}
```

秩（rank）：线性变换会将空间压缩到几维，这个变换的秩就是几。

由于 det(**A**) = 0，无法区别不同程度的降维，所以引入了秩，表达变换后空间的维数。

矩阵的列空间就是矩阵的列所张成的空间，所以秩就是列空间的维数。

如果秩等于矩阵的列数，称之为满秩。

零空间就是经过变换落在原点的原向量组成的空间，det(**A**) != 0 时只有零向量会落在原点。

## 微积分

[你也能懂的微积分](https://zhuanlan.zhihu.com/p/94592123)

