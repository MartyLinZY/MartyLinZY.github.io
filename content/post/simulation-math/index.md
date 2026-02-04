+++
title = '仿真模拟-蒙特卡洛'
date = 2026-02-04T20:00:00+08:00
description = '上课用'
tags = ['数学', '仿真', '数值计算', '作业']
categories = ['lesson']
image = 'anime-simulation-math-header.jpg'
draft = false
+++

# Monte Carlo Simulations 课程笔记（L1–L3）

> 本笔记配合三讲 PPT 使用，目标是帮助读者从“概率的基本概念”逐步理解  
> Monte Carlo 模拟中 **随机变量、分布、变量变换与抽样方法** 的数学基础。

---

## 第一讲（L1）：Monte Carlo 的基本思想与均匀分布

### 1. Monte Carlo 的核心思想

Monte Carlo 方法的本质是：

> **用随机抽样，把难以解析计算的期望或概率，转化为样本平均。**

其理论基础是 **强大数定律（Strong Law of Large Numbers）**。

---

### 2. 强大数定律（SLLN）

设随机变量 \(X\) 的期望存在：
\[
\mathbb{E}(X) = \mu
\]

取 \(X_1, X_2, \dots, X_m\) 为 \(X\) 的独立同分布样本，则：
\[
\mathbb{P}\!\left(
\lim_{m\to\infty} \frac{1}{m}\sum_{j=1}^m X_j = \mu
\right) = 1
\]

因此，Monte Carlo 的基本估计量为：
\[
\hat{\mu}_m = \frac{1}{m}\sum_{j=1}^m X_j
\]

---

### 3. 均匀分布的定义与直觉

#### 定义

若随机变量 \(X\) 在区间 \([a,b]\) 上满足：
\[
f_X(x) =
\begin{cases}
\dfrac{1}{b-a}, & a < x < b \\
0, & \text{otherwise}
\end{cases}
\]
则称 \(X \sim U[a,b]\)。

#### 直觉理解

- 区间内 **等长子区间 → 概率相等**
- 概率只与“长度”有关，与位置无关

---

### 4. 标准均匀分布

\[
Z \sim U[0,1]
\]

这是计算机中随机数生成器（`rand()`）的数学模型。

#### 区间变换

若 \(R \sim U[0,1]\)，则：
\[
X = a + (b-a)R \sim U[a,b]
\]

---

### 5. 示例：随机四边形面积的期望

在 \(L \times W\) 的矩形四条边上各取一个随机点，构成四边形，其面积为：
\[
A = \frac{1}{2}LW - \frac{1}{2}(x_1 - x_2)(y_1 - y_2)
\]

由于 \(x\) 与 \(y\) 方向独立，且
\[
\mathbb{E}(x_1 - x_2) = 0,\quad \mathbb{E}(y_1 - y_2) = 0
\]

得到：
\[
\boxed{\mathbb{E}(A) = \frac{1}{2}LW}
\]

---

### 6. 指示变量（Indicator Variable）

对事件 \(A\)，定义：
\[
I_A =
\begin{cases}
1, & A \text{发生} \\
0, & A \text{不发生}
\end{cases}
\]

则：
\[
\mathbb{E}(I_A) = \mathbb{P}(A)
\]

这是 Monte Carlo 估计概率的标准工具。

---

## 第二讲（L2）：随机变量函数的分布（CDF 与 PDF）

### 1. 问题背景

设：
\[
X, Z \sim U[0,1], \quad A = \frac{1}{2}XZ
\]

目标：求随机变量 \(A\) 的 **分布函数（CDF）和密度函数（PDF）**。

---

### 2. 取值范围

\[
0 \le A \le \frac{1}{2}
\]

---

### 3. 从 CDF 入手

定义：
\[
F_A(a) = \mathbb{P}(A \le a)
= \mathbb{P}\!\left(\frac{1}{2}XZ \le a\right)
\]

使用条件概率与全概率公式：
\[
F_A(a) = \int \mathbb{P}\!\left(Z \le \frac{2a}{x}\right) f_X(x)\,dx
\]

---

### 4. 几何积分计算结果

对 \(0 < a < \frac{1}{2}\)，得到：
\[
\boxed{
F_A(a) = 2a\big(1 - \ln(2a)\big)
}
\]

---

### 5. 得到 PDF

\[
f_A(a) = \frac{d}{da}F_A(a)
= -2\ln(2a), \quad 0 < a < \frac{1}{2}
\]

完整形式：
\[
f_A(a) =
\begin{cases}
-2\ln(2a), & 0 < a < \frac{1}{2} \\
0, & \text{otherwise}
\end{cases}
\]

---

### 6. 合法性检查

- 非负性：\(\ln(2a) < 0\)
- 归一化：
\[
\int_0^{1/2} f_A(a)\,da = 1
\]

---

### 7. 从 PDF 反算期望

\[
\mathbb{E}(A) = \int_0^{1/2} a f_A(a)\,da = \frac{1}{8}
\]

与直接计算一致。

---

## 第三讲（L3）：变量变换与反变换抽样法

### 1. 单调变量变换（One-to-One）

设：
\[
Y = g(X)
\]
且 \(g(x)\) 为严格单调函数。

则：
\[
f_Y(y) = \frac{f_X(x)}{|g'(x)|},\quad x = g^{-1}(y)
\]

---

### 2. 示例：指数变换

\[
X \sim U[0,1],\quad Y = e^X
\]

反函数：
\[
x = \ln y
\]

得到：
\[
f_Y(y) =
\begin{cases}
\dfrac{1}{y}, & 1 < y < e \\
0, & \text{otherwise}
\end{cases}
\]

---

### 3. 非单调变换：必须回到 CDF

当 \(g(x)\) 非单调时：

- 反函数不唯一
- Jacobian 公式失效

正确做法：
\[
F_Y(y) = \mathbb{P}(g(X) \le y)
\]
再对 \(y\) 求导得到 PDF。

---

### 4. 反变换抽样法（Inverse Transform Method）

#### 定理

设随机变量 \(X\) 的 CDF 为 \(F_X(x)\)，定义：
\[
Y = F_X(X)
\]
则：
\[
\boxed{Y \sim U[0,1]}
\]

---

### 5. 抽样构造

若：
\[
U \sim U[0,1]
\]
则：
\[
X = F_X^{-1}(U)
\]
服从目标分布 \(F_X\)。

---

### 6. 混合分布的反变换

当 CDF 含有：

- 连续段
- 跳跃点（离散质量）

需将 \([0,1]\) 分段，对应反解 \(F_X^{-1}(u)\)。

这是 Monte Carlo 中生成复杂分布样本的通用方法。

---

## 总结

1. **概率是最基本对象**
2. **CDF 永远存在**
3. **PDF 是 CDF 的导数（在可导时）**
4. **变量变换与反变换是 Monte Carlo 的核心工具**

---
