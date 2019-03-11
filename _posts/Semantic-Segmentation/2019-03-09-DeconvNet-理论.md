---
layout: post
title: DeconvNet 理论
category: 语义分割
tags: deconvnet
description: deconvnet
---

<div>
$$
\mathbf{V}_1 \times \mathbf{V}_2 =
\begin{vmatrix}
  \mathbf{i} & \mathbf{j} & \mathbf{k} \\
  \frac{\partial X}{\partial u} & \frac{\partial Y}{\partial u} & 0 \\
  \frac{\partial X}{\partial v} & \frac{\partial Y}{\partial v} & 0
\end{vmatrix}
$$
</div>

<div>
$$
W\left(x\right)=
\begin{cases}
 \left(a+2\right)\left|x\right|^{3}-\left(a+3\right)\left|x\right |^{2}+1 & \text{ for } \left|x\right|\leqslant1 \\
 a\left|x\right|^{3}-5a\left|x\right|^{2}+8a\left|x\right|-4a & \text{ for } 1<\left|x\right|<2 \\
 0 & \text{ otherwise }
\end{cases}
$$
</div>

<div>
$$
\begin{align}
  \nabla\times\vec{\mathbf{B}}-\frac{1}{c}\frac{\partial\vec{\mathbf{E}}}{\partial t} &= \frac{4\pi}{c}\vec{\mathbf{j}} \\
  \nabla\cdot\vec{\mathbf{E}} &= 4\pi\rho \\
  \nabla\times\vec{\mathbf{E}}+\frac{1}{c}\frac{\partial\vec{\mathbf{B}}}{\partial t} &= \vec{\mathbf{0}} \\
  \nabla\cdot\vec{\mathbf{B}} &= 0
\end{align}
$$
</div>

<div>
$$
\begin{aligned}
  & \phi(x,y) = \phi \left(\sum_{i=1}^n x_ie_i, \sum_{j=1}^n y_je_j \right)
  = \sum_{i=1}^n \sum_{j=1}^n x_i y_j \phi(e_i, e_j) = \\
  & (x_1, \ldots, x_n) \left( \begin{array}{ccc}
      \phi(e_1, e_1) & \cdots & \phi(e_1, e_n) \\
      \vdots & \ddots & \vdots \\
      \phi(e_n, e_1) & \cdots & \phi(e_n, e_n)
    \end{array} \right)
  \left( \begin{array}{c}
      y_1 \\
      \vdots \\
      y_n
    \end{array} \right)
\end{aligned}
$$
</div>

```python
import tensorflow as tf
```

![](https://raw.githubusercontent.com/chiemon/chiemon.github.io/master/img/DeconvNet/1.png)
