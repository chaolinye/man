# 微积分

## 微积分的本质

微积分(calculus)基本定理：某个图像下方面积函数的导数能够还原出定义这个图像的函数

圆的周长函数2πr下的面积函数就是πr²

导数(derivatives)

```tex
面积A的导数= \frac{dA}{dx} = f(x)
```

衡量的是函数对取值的微小变化有多敏感

积分 integral 

某个图像下的面积函数的导数能够还原出定义这个图像的函数

积分和导数是对互逆的运算 

导数不是测量瞬时变化率，而是其变化率的最佳近似

用几何来求导

导数的本质是看某个量的微小变化以及它和它所导致的另一个量的微小变化有什么关系

!> 微小变化量才是导数的本质

## 函数组合的导数

函数组合的三种关系：加、乘、复合

求两个函数的组合的导数.

加法：
```tex
\frac{d}{dx}(g(x)+h(x))=\frac{dg}{dx}+\frac{dh}{dx}
```

如果你要处理两个东西的乘积，通过面积来理解会有好处

乘法：左乘右导 + 右乘左导
```tex
\frac{d}{dx}(g(x)\cdot h(x))=g(x)\frac{dh}{dx}+h(x)\frac{dg}{dx}
```

复合：几何上可以用多个数轴来表达
```tex
\frac{d}{dx}g(h(x))=\frac{dg}{dh}(h(x))\frac{dh}{dx}(x)
```
链式法则 递归    
Chain rule
```tex
\frac{d}{dx}g(h(x))=\frac{dg}{dh}\frac{dh}{dx}=\frac{dg}{dx}
```

## 指数函数的求导

```tex
\frac{d}{dt}(e^t) = e^t \\
2^t = e^{\ln 2^t} \\
2^t = (e^{\ln 2})^t = e^{\ln 2 \cdot t} \\
\frac{d}{dt}(e^{\ln 2 \cdot t}) = e^{\ln 2 \cdot t} \cdot \ln 2  = \ln 2 \cdot 2^t \\
\frac{d}{dt} a^t = \ln a \cdot a^t \\
\frac{dM}{dt} = (1 + r)M \to M(t) = e^{(1 + r)t} \\
可以通过变化率得到原函数
```

## 隐函数的求导 

![](../images/calculus1.png)

Related rates
相关变化率

x(t)² + y(t)² = 5²

```tex
\frac{d(x(t)^2 + y(t)^2)}{dt} = 0 \\ 
```

```tex
2x(t)\frac{dx}{dt} + 2y(t)\frac{dy}{dt} = 0 \\ 
```

```tex
\frac{dy}{dx} = -\frac{x}{y} \\ 
```

计算等式两边的微小变化量.

```tex
\sin(x)y^2 = x \\
\sin(x)(2ydy) + y^2(\cos(x)dx) = dx
```

多元微积分.

```tex
y = \ln(x) \\
e^y = x \\ 
e^y dy = dx \\ 
\frac{dy}{dx} = e^{-y} = e^{-\ln x} = x^{-1}
```

## References

- [微积分的本质](https://www.bilibili.com/video/BV1qW411N7FU)
- [你也能懂的微积分](https://zhuanlan.zhihu.com/p/94592123)
