# 概率统计

## 贝叶斯定理（Bayes theorem)

```tex
P(H|E) = \frac{P(H) \cdot P(E|H)}{P(E)}
```

!> 贝叶斯定理最根本的结论：新证据不能直接凭空的决定你的看法，而是应该更新你的先验看法（之前的经验）

![](../images/probability1.png)

任何事件都对应概率空间的一个子集，并且事件发生的概率就是子集的面积

后验概率(posterior)
```tex
P(H|E) = \frac{area1}{area1 + area2}
= \frac{P(H) \cdot P(E|H)}{P(H) \cdot P(E|H) + P(\neg H)P(E|\neg H)}
= \frac{P(H) \cdot P(E|H)}{P(E)} 
```

> P(E|H) 称为似然概率(likelihoods)

把贝叶斯定理看作是一个关于比例的表述

```tex
× P(A \text{ and } B) = P(A)P(B) “独立事件才成立”）\\
\\
√ P(A \text{ and } B) = P(A)P(B|A) 
```

贝叶斯定理只有在非独立事件下才有意义，精准衡量一个变量多大程度依赖另一个变量
