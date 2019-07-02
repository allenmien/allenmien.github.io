---
title: BP算法
---
# BP算法

## 前向传播

对于k，k+1 层的前向传播为：

$$
z^{(k)} = w^{(k)}*n^{(k-1)} + b^{(k)}, n^{(k-1)} = f(z^{(k-1)})
$$

$$
z^{(k+1)} = w^{(k+1)}*n^{(k)} + b^{(k+1)}, n^{(k)} = f(z^{(k)})
$$

## 反向传播

### 损失函数

$$
L（y,\hat{y}）
$$

Loss对k层$w^{(k)}, b^{(k)}$偏导数为：

$$
\frac{\partial{L(y, \hat{y})}}{\partial{w^{(k)}}} = \frac{\partial{L(y, \hat{y})}}{\partial{z^{(k)}}} \cdot \frac{\partial{z^{(k)}}}{\partial{w^{(k)}}}
$$

$$
\frac{\partial{L(y, \hat{y})}}{\partial{b^{(k)}}} = \frac{\partial{L(y, \hat{y})}}{\partial{z^{(k)}}} \cdot \frac{\partial{z^{(k)}}}{\partial{b^{(k)}}}
$$

假设：$\delta^{(k)} = \frac{\partial{L(y, \hat{y})}}{\partial{z^{(k)}}}$

### 计算$z^{(k)}$的导数

$$
\delta^{(k)} \\ = \frac{\partial{L(y, \hat{y})}}{\partial{z^{(k)}}} \\ = \frac{\partial{L(y, \hat{y})}}{\partial{z^{(k+1)}}} \cdot \frac{\partial{z^{(k+1)}}}{\partial{n^{(k)}}} \cdot \frac{\partial{n^{(k)}}}{\partial{z^{(k)}}} \\= \delta^{(k+1)} \cdot w^{(k+1)} \cdot f_k'(z^{(k)})
$$

### 计算$\frac{\partial{z^{(k)}}}{\partial{w^{(k)}}}$

$$
\frac{\partial{z^{(k)}}}{\partial{w^{(k)}}} = n^{(k-1)}
$$

### 计算$\frac{\partial{z^{(k)}}}{\partial{b^{(k)}}}$

$$
\frac{\partial{z^{(k)}}}{\partial{b^{(k)}}} = 1
$$

## 计算

反向传播

$$
\frac{\partial{L(y,\hat{y})}}{\partial{w^{(k)}}} = \delta^{(k)} \cdot \frac{\partial{z^{(k)}}}{\partial{w^{(k)}}} = \delta^{(k)} \cdot n^{(k-1)} 
$$

$$
\frac{\partial{L(y,\hat{y})}}{\partial{b^{(k)}}} = \delta^{(k)} \cdot \frac{\partial{z^{(k)}}}{\partial{b^{(k)}}} = \delta^{(k)}
$$
