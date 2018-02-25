Title: Bayes Filter for Robot
Date: 2017/6/13
Category: Robot
Tags: Math
Author: WangXinyu
# 概率机器人的贝叶斯滤波器

## 模型
- $x_t$表示$t$时刻机器人的状态向量
- $z_t$表示$t$时刻的测量向量
- $u_t$表示$t$时刻的控制向量
- $bel(x_t)$表示状态向量的置信分布（Belief Distributions），根据它我们就可以确定$t$时刻机器人的状态，贝叶斯滤波器最终的输出结果就是它。它的定义如下，即在已知1到$t$时刻的测量向量和控制向量的条件下，机器人处于状态$x_t$的概率。

$$
bel(x_t) = p(x_t|z_{1:t},u_{1:t})
$$

- $\overline {bel}(x_t)$是计算$bel(x_t)$的一个中间值，称为预测（Prediction）。它的定义如下，即在得到$t$时刻的测量向量$z_t$之前获取的置信概率，因此称为预测。而得到$z_t$后，由$\overline {bel}(x_t)$计算$bel(x_t)$的过程称为纠正或更新。

$$
\overline {bel}(x_t) = p(x_t|z_{1:t-1},u_{1:t})
$$


## 递推公式
先给出贝叶斯滤波器的递推公式，然后证明。

$$
\overline {bel}(x_t) = \int p(x_t|u_{t},x_{t-1}) \cdot bel(x_{t-1})dx_{t-1}
$$

$$
bel(x_t) = \eta \cdot p(z_t|x_{t}) \cdot \overline {bel}(x_t)
$$

- $p(x_t|u_{t},x_{t-1})$是机器人的状态转移概率

- $p(z_t|x_{t})$是测量概率

- $\eta$是概率归一化常量

根据这个公式我们可以看出来，只要知道状态转移概率分布，测量概率以及前一时刻的置信概率，就可以递归求得当前机器人状态的置信概率。

## 证明


\begin{split}
bel(x_t) &= p(x_t|z_{1:t},u_{1:t})\\\\
&= {p(x_t,z_{1:t},u_{1:t}) \over p(z_{1:t},u_{1:t})}条件概率\\\\
&= {p(z_t|x_t, z_{1:t-1},u_{1:t}) \cdot p(x_t, z_{1:t-1},u_{1:t}) \over p(z_{1:t},u_{1:t})}条件概率\\\\
&= {p(z_t|x_t, z_{1:t-1},u_{1:t}) \cdot p(x_t|z_{1:t-1},u_{1:t}) \cdot p(z_{1:t-1},u_{1:t}) \over p(z_t|z_{1:t-1},u_{1:t}) \cdot p(z_{1:t-1},u_{1:t})} 条件概率\\\\
&= {p(z_t|x_t, z_{1:t-1},u_{1:t}) \cdot p(x_t|z_{1:t-1},u_{1:t}) \over p(z_t|z_{1:t-1},u_{1:t})} 约分\\\\
&= {p(z_t|x_t) \cdot p(x_t|z_{1:t-1},u_{1:t}) \over p(z_t|z_{1:t-1},u_{1:t})} Markov假设\\\\
&= \eta \cdot p(z_t|x_t) \cdot \overline {bel}(x_t) 替换
\end{split}

\begin{split}
\overline {bel}(x_t) &= p(x_t|z_{1:t-1},u_{1:t})\\\\
&= \int p(x_t|x_{t-1}, z_{1:t-1},u_{1:t}) \cdot p(x_{t-1}|z_{1:t-1},u_{1:t})dx_{t-1} 用全概率计算条件概率\\\\
&= \int p(x_t|x_{t-1}, u_{t}) \cdot p(x_{t-1}|z_{1:t-1},u_{1:t-1})dx_{t-1} Markov假设并去掉u_t\\\\
&= \int p(x_t|x_{t-1}, u_{t}) \cdot bel(x_{t-1})dx_{t-1} 替换
\end{split}

这里的$\eta = {p(z_t|z_{1:t-1},u_{1:t})}^{-1}$，可以看作概率的归一化因子。因为它等于分子中两个PDF的乘积的积分：

\begin{split}
p(z_t|z_{1:t-1},u_{1:t}) &＝ \int p(z_t|x_t, z_{1:t-1},u_{1:t}) \cdot p(x_t|z_{1:t-1},u_{1:t})dx_t用全概率计算条件概率\\\\
&= \int p(z_t|x_t) \cdot p(x_t|z_{1:t-1},u_{1:t})dx_t \; Markov假设
\end{split}

Markov假设当前状态是完整的，若已知当前状态$x_t$，则$t$时刻及其之前的测量$z_{1:t}$和控制向量$u_{1:t}$对未来状态没有影响。

### 参考文献

Sebastian THRUN, Wolfram BURGARD, Dieter FOX. PROBABILISTIC ROBOTICS. 2005.

M\. Sanjeev Arulampalam, Simon Maskell, Neil Gordon, and Tim Clapp. A Tutorial on Particle Filters for Online Nonlinear/Non-Gaussian Bayesian Tracking. IEEE TRANSACTIONS ON SIGNAL PROCESSING, VOL. 50, NO. 2, FEBRUARY 2002.

使用全概率公式计算条件概率. 我的博文，[Link](https://wang-xinyu.github.io/total-probability-applied-to-conditional-probability.html).

#### 版权声明：本文为作者原创，转载请注明出处[https://wang-xinyu.github.io](https://wang-xinyu.github.io)