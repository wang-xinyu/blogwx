Title: The Derivation of Kalman Filter from Bayes Filter
Date: 2017/6/17
Category: Robot
Tags: Math
Author: WangXinyu
# 从贝叶斯滤波到卡尔曼滤波

## 卡尔曼滤波的三个假设

**1.** 状态转移方程是线性的，噪声服从均值为$0$，协方差为$R_t$的多元高斯分布。因此，状态转移概率服从均值$A_t x_{t-1} + B_t u_t$，协方差为$R_t$的多元高斯分布。

$$
\epsilon_t \sim N(0, R_t)
$$

$$
x_t = A_t x_{t-1} + B_t u_t + \epsilon_t
$$

$$
p(x_t|u_t, x_{t-1}) \sim N(A_t x_{t-1} + B_t u_t, R_t)
$$

$$
p(x_t|u_t, x_{t-1}) = det(2 \pi R_t)^{- \frac 1 2}exp\lbrace {- \frac 1 2}(x_t-A_t x_{t-1} - B_t u_t)^T R_t^{-1}(x_t-A_t x_{t-1} - B_t u_t)\rbrace
$$

**2.** 测量方程是线性的，噪声服从均值为$0$，协方差为$Q_t$的多元高斯分布。因此，测量概率服从均值$C_t x_t$，协方差为$Q_t$的多元高斯分布。

$$
\delta_t \sim N(0, Q_t)
$$

$$
z_t = C_t x_t + \delta_t
$$

$$
p(z_t|x_t) \sim N(C_t x_t, Q_t)
$$

$$
p(z_t|x_t) = det(2 \pi Q_t)^{- \frac 1 2}exp\lbrace {- \frac 1 2}(z_t-C_t x_t)^T Q_t^{-1}(z_t-C_t x_t)\rbrace
$$

**3.** 初始置信$bel(x_0)$概率服从均值为$\mu_0$，协方差为$\Sigma_0$的多元高斯分布。

$$
bel(x_0) = p(x_0) = det(2 \pi \Sigma_0)^{- \frac 1 2}exp\lbrace {- \frac 1 2}(x_0- \mu_0)^T \Sigma_0^{-1}(x_0- \mu_0)\rbrace
$$

## 递推公式
只要满足以上三个假设，即可证明$bel(x_t)$也满足高斯分布。即：

$$
bel(x_t) \sim N(\mu_t, \Sigma_t)
$$
$$
\overline {bel}(x_t) \sim N(\overline \mu_t, \overline \Sigma_t)
$$

根据贝叶斯滤波的递推公式和以上的假设，即可推导出卡尔曼滤波的递推公式。因为满足高斯分布，所以我们只需递推均值和协方差。

\begin{split}
\overline \mu_t &= A_t \mu_{t-1} + B_t u_t\\\\
\overline \Sigma_t &= A_t \Sigma_{t-1} A_t^T+ R_t\\\\
K_t &= \overline \Sigma_t C_t^T(C_t \overline \Sigma_t C_t^T + Q_t)^{-1} \\\\
\mu_t &= \overline \mu_t + K_t(z_t - C_t \overline \mu_t) \\\\
\Sigma_t &= (I - K_t C_t) \overline \Sigma_t
\end{split}

- $K_t$称为Kalman增益

- $I$是单位矩阵


## 简略推导

由贝叶斯滤波器可知$\overline {bel}(x_t)$的公式如下，在卡尔曼滤波中，$p(x_t|x_{t-1}, u_t)$和$bel(x_{t-1})$都服从高斯分布。

$$
\overline {bel}(x_t) = \int p(x_t|x_{t-1}, u_t) \cdot bel(x_{t-1})dx_{t-1}
$$

$$
p(x_t|x_{t-1}, u_t) \sim N(x_t; A_t x_{t-1} + B_t u_t, R_t)
$$

$$
bel(x_{t-1}) \sim N(x_{t-1};\mu_{t-1}, \Sigma_{t-1})
$$

这里$\overline {bel}(x_t)$可以看作两个高斯分布的卷积。省略中间步骤，最终推导出$\overline {bel}(x_t)$也是高斯分布，均值和协方差如下：

\begin{split}
\overline \mu_t &= A_t \mu_{t-1} + B_t u_t\\\\
\overline \Sigma_t &= A_t \Sigma_{t-1} A_t^T+ R_t
\end{split}

由贝叶斯滤波器可知$bel(x_t)$的公式如下，在卡尔曼滤波中，$p(z_t|x_t)$也服从高斯分布，上面已证明$\overline {bel}(x_t)$也服从高斯分布。

$$
bel(x_t) = \eta \cdot p(z_t|x_{t}) \cdot \overline {bel}(x_t)
$$

$$
p(z_t|x_t) \sim N(z_t; C_t x_t, Q_t)
$$

$$
\overline {bel}(x_t) \sim N(x_t; \overline \mu_t, \overline \Sigma_t)
$$

因此$bel(x_t)$可以看作两个高斯分布的乘积，结果也是高斯分布。省略中间推导，最终得到的均值和协方差如下：

\begin{split}
K_t &= \overline \Sigma_t C_t^T(C_t \overline \Sigma_t C_t^T + Q_t)^{-1} \\\\
\mu_t &= \overline \mu_t + K_t(z_t - C_t \overline \mu_t) \\\\
\Sigma_t &= (I - K_t C_t) \overline \Sigma_t
\end{split}

### 参考文献

Sebastian THRUN, Wolfram BURGARD, Dieter FOX. PROBABILISTIC ROBOTICS. 2005.

概率机器人的贝叶斯滤波器. 我的博文，[Link](https://wang-xinyu.github.io/bayes-filter-for-robot.html).

#### 版权声明：本文为作者原创，转载请注明出处[https://wang-xinyu.github.io](https://wang-xinyu.github.io)