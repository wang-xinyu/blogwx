Title: Total Probability Applied to Conditional Probability
Date: 2017/6/12
Category: Math
Tags: Math
Author: WangXinyu
# 使用全概率公式计算条件概率

## 公式
先给出公式，然后证明。

$$
P(A|B) = \sum_{i=1}^n P(A|B,C_i) \cdot P(C_i|B)
$$

$$
p(x|y) = \int p(x|y,z) \cdot p(z|y)dz
$$
 
## 证明
这里以离散概率为例进行证明，连续型与之类似。

\begin{equation}\begin{split}
P(A,B) &= \sum_{i=1}^n P(A,B,C_i)\\\\
&= \sum_{i=1}^n P(A|B,C_i) \cdot P(B,C_i)\\\\
&= \sum_{i=1}^n P(A|B,C_i) \cdot P(C_i|B) \cdot P(B)
\end{split}\end{equation}


\begin{equation}\begin{split}
P(A|B)&=  {P(A,B) \over P(B)}\\\\
&= {{\sum_{i=1}^n P(A|B,C_i) \cdot P(C_i|B) \cdot P(B)} \over P(B)}\\\\
&= \sum_{i=1}^n P(A|B,C_i) \cdot P(C_i|B)
\end{split}\end{equation}