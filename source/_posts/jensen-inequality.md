---
title: jensen_inequality
date: 2016-05-20 16:39:25
categories:
- Mathmatics
- PRML
tags:
- Inequality
- Metrics
- Expectation
---

## 凸函数与凹函数
凸函数`convex function`形象地说两头往上翘，中间往下沉。凹函数`concave function`恰好相反，两头往下沉，中间往上翘。下图是一个凸函数
![convex function](https://upload.wikimedia.org/wikipedia/commons/thumb/c/c7/ConvexFunction.svg/400px-ConvexFunction.svg.png)

凸函数的定义在某些国内教材中和Wiki中的含义不一，注意区分。
<!-- more -->

凸函数的性质
> $$\forall 0 \le t \le 1, tf(x_1) + (1-t)f(x_2) \ge f(tx_1 + (1-t)x_2)$$

这也是凸函数的定义。他是后面要提到的`Jensen's Inequality`的一个形式(*form*)。

## Jensen's Inequality
在概率论的语境下，若$\varphi$为凸函数，$X$是离散随机变量，*Jensen's Inequality*的形式变为
> $$ E[\varphi(X)] \ge \varphi(E[X]) $$

可以用上图直观解释。割线`secant line`上的点对应$E[\varphi(X)]$，图像`graph`上的点对应$\varphi(E[X])$。前者先应用$\varphi$后求期望（不在$\varphi$上），后者先求期望_$tx_1+(1-t)x_2$_再应用$\varphi$（还是在$\varphi$上）。

*Jensen's Inequality*的好处在于把$\varphi$提取出来，对于$E[\varphi(X)]$难以计算的情况行之有效。这里给出一个应用实例。

## KL-Divergence
*KL散度*`Kullback–Leibler divergence`是定义在两个随机变量$P$和$Q$上的距离`measurement of distance`，形式为
> $$ D_{KL}(P||Q) = \sum_i{P_i\log_2\frac{P_i}{Q_i}}$$

他不是一个度量`metrics`，因为它
- [ ] 自反， $D_{KL}(P||Q) = 0 \iff P=Q$
- [ ] 非负， $\forall P,Q, D_{KL}(P||Q) \ge 0$
- [x] 对称， $D\_{KL}(P||Q) = D\_{KL}(Q||P)$
- [x] 三角不等式， $D\_{KL}(P||Q) + D\_{KL}(Q||R) \ge D_{KL}(P||R)$

其中非负的性质证明需要用到*Jensen's Inequality*。

{% math %}
\begin{align*}
-\sum_i{P_i\log_2\frac{P_i}{Q_i}} =& \sum_i{P_i\log_2\frac{Q_i}{P_i}} \\
\le& \log_2\sum_i{P_i\frac{Q_i}{P_i}} \tag{Jensen's Inequality} \\
=& \log_2\sum_i{Q_i} \\
=& 0
\end{align*}
{% endmath %}



上面使用了*Jensen's Inequality*。因为$\varphi=\log_2(x)$是凹函数所以是小于而不是大于。



