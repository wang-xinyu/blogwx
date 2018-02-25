Title: The Derivation of Generic Particle Filter for Robot
Date: 2017/6/26
Category: Robot
Tags: Math
Author: WangXinyu
# 机器人的普通粒子滤波推导

## 前言

我这里所说的普通粒子滤波也叫SIR（Sampling Importance Resampling）粒子滤波。因为时至今日已产生了很多粒子滤波的变种，例如：regularized particle filter，Rao-Blackwellized particle filter等等，这些都使用了某种方法对基本的粒子滤波进行了优化。

粒子滤波可以说是使用Monte Carlo方法的贝叶斯滤波的实现。这是因为SIS（Sequential Importance Sampling）是一种Monte Carlo方法，而粒子滤波、bootstap filter、condensation算法等，都可以基于SIS推导出来。

Kalman滤波只能处理高斯分布，且状态转移方程和测量方程都必须线性。EKF是通过Taylor级数展开，把状态转移方程和测量方程线性化，仍假设噪声是高斯分布。而粒子滤波就可以处理非线性、任意分布的问题了。

## 模型

与Kalman滤波不同，粒子滤波不是估计$t$时刻状态向量的置信分布$bel(x_t)$，而是$0:t$时刻所有状态向量序列的置信分布$bel(x_{0:t})$，可以看作是一个轨迹。

$$
bel(x_{0:t}) = p(x_{0:t}|z_{1:t},u_{1:t})
$$

我们可以用一堆粒子来表示这个分布，每个粒子都有一个状态$x_{0:t}^i$和一个权重$w_t^i$，权重是归一化的，因此，$bel(x_{0:t})$可由如下的方程近似表示，$\delta(x)$是Dirac delta函数：

$$
particle:\; \lbrace x_{0:t}^i, w_t^i \rbrace_{i=1}^N
$$

$$
\sum_{i=1}^N w_t^i = 1
$$

$$
bel(x_{0:t}) \approx \sum_{i=1}^N w_t^i \delta (x_{0:t} - x_{0:t}^i)
$$

## Importance Sampling

计算权重$w$，需要使用Importance Sampling。

一般情况下，计算一个目标函数$f(x)$的期望，我们需要知道$x$的概率分布$p(x)$，然后积分求得期望，但实际中可能求积分比较困难，所以可以根据$p(x)$采样，然后用Monte Carlo方法近似计算：

$$
E(f(x)) = \int f(x)p(x)dx \approx \frac 1 N \sum_{i=1}^N f(x_i)
$$

$$
x_{0:N} \sim p(x)
$$

但是，有时候我们不知道$p(x)$什么样，我们这里$p(x)$就是想求的target distribution是$bel(x_{0:t})$，所以没办法直接对它进行采样。这时就要用Importance Sampling了，引入一个新的分布$q(x)$，做如下变换，然后就可以根据$q(x)$采样，进而计算期望：

$$
E(f(x)) = \int {f(x)p(x) \over q(x)} q(x) dx \approx \frac 1 N \sum_{i=1}^N f(x_i) {p(x_i) \over q(x_i)}
$$

$$
x_{0:N} \sim q(x)
$$

$$
Importance \, Weights: \; w_i = {p(x_i) \over q(x_i)}
$$

那么回到我们的问题上，权重$w_t^i$如下，因为我们这里用权重表示概率，需要归一化，所以省略了归一化系数，用了一个正比符号。

$$
w_t^i \propto {p(x_{0:t}^i|z_{1:t}, u_{1:t}) \over q(x_{0:t}^i|z_{1:t}, u_{1:t})}
$$

## 权重的递推公式

\begin{split}
p(x_{0:t}|z_{1:t}, u_{1:t}) &＝ {p(z_t|x_{0:t}, z_{1:t-1}, u_{1:t})p(x_t|x_{0:t-1}, z_{1:t-1}, u_{1:t})p(x_{0:t-1}, z_{1:t-1}, u_{1:t}) \over p(z_{1:t}, u_{1:t})} \; Bayes\\\\
&= {p(z_t|x_{t})p(x_t|x_{t-1}, u_{t})p(x_{0:t-1}, z_{1:t-1}, u_{1:t}) \over p(z_{1:t}, u_{1:t})} \; Markov假设\\\\
&= {p(z_t|x_{t})p(x_t|x_{t-1}, u_{t})p(x_{0:t-1}|z_{1:t-1}, u_{1:t}) \over p(z_t|z_{1:t-1}, u_{1:t})} \; Bayes\\\\
&= {p(z_t|x_{t})p(x_t|x_{t-1}, u_{t})p(x_{0:t-1}|z_{1:t-1}, u_{1:t-1}) \over p(z_t|z_{1:t-1}, u_{1:t})} \; 忽略u_t \\\\
&\propto p(z_t|x_{t})p(x_t|x_{t-1}, u_{t})p(x_{0:t-1}|z_{1:t-1}, u_{1:t-1})
\end{split}

$$
q(x_{0:t}|z_{1:t}, u_{1:t}) ＝ q(x_t|x_{0:t-1}, z_{1:t}, u_{1:t})q(x_{0:t-1}|z_{1:t-1}, u_{1:t-1}) \; Bayes, 忽略z_t, u_t
$$

\begin{split}
w_t^i &\propto {p(z_t|x_{t}^i)p(x_t^i|x_{t-1}^i, u_{t})p(x_{0:t-1}^i|z_{1:t-1}, u_{1:t-1}) \over q(x_t^i|x_{0:t-1}^i, z_{1:t}, u_{1:t})q(x_{0:t-1}^i|z_{1:t-1}, u_{1:t-1})}\\\\
&= w_{t-1}^i {p(z_t|x_{t}^i)p(x_t^i|x_{t-1}^i, u_{t}) \over q(x_t^i|x_{0:t-1}^i, z_{1:t}, u_{1:t})}
\end{split}

如果根据Markov假设进一步做如下简化，则$t$时刻的状态估计至于$t-1$时刻的状态和当前的测量向量和控制向量有关，与$0:t-1$时刻的状态路径以及历史测量值都无关。

$$
w_t^i \propto w_{t-1}^i {p(z_t|x_{t}^i)p(x_t^i|x_{t-1}^i, u_{t}) \over q(x_t^i|x_{t-1}^i, z_{t}, u_{t})}
$$

因此我们可以近似认为路径估计等于当前状态估计：

$$
bel(x_{0:t}) = p(x_{0:t}|z_{1:t},u_{1:t}) \approx p(x_{t}|z_{1:t},u_{1:t})
$$

即：

$$
bel(x_{t}) \approx \sum_{i=1}^N w_t^i \delta (x_{0:t} - x_{0:t}^i)
$$

## 选择Proposal Distribution

至此我们可以递归地计算权重了，但是权重的递推公式里面有一个proposal distribution－$q(x_t|x_{t-1}, z_{t}, u_{t})$还不知道是什么。

$q(x_t|x_{t-1}, z_{t}, u_{t})$的最优解是$p(x_t|x_{t-1}, z_{t}, u_{t})$，为什么是最优我还没搞清楚-_-!

我们通常会选择$p(x_t|x_{t-1}, u_{t})$作为$q(x_t|x_{t-1}, z_{t}, u_{t})$，如此一来，我们的权重递推公式可以进一步得到简化：

$$
w_t^i \propto w_{t-1}^i p(z_t|x_{t}^i)
$$

## Resampling

然后还有重要的一个步骤Resampling，Resampling是为了解决Degeneracy的问题。Degeneracy简单来说就是，经过若干次迭代后，权重都集中在了一个粒子的身上，其他粒子的权重都变得很小。

Resampling就是对下面的离散分布进行重采样，用新的粒子集$\lbrace x_t^{i*}\rbrace_{i=1}^N$替换旧的粒子集$\lbrace x_t^i\rbrace_{i=1}^N$。采样时，旧的粒子是否保留的概率与权重$w_t^i$成正比。

$$
bel(x_{t}) \approx \sum_{i=1}^N w_t^i \delta (x_{0:t} - x_{0:t}^i)
$$

Resampling后每个粒子的权重被置为$\frac 1 N$。如果每一次迭代都进行Resampling的话（SIR粒子滤波就是这样做的），那我们的权重可以进一步简化为：

$$
w_t^i \propto p(z_t|x_{t}^i)
$$

## 算法

——————————————

Algorithm SIR Particle Filter($x_{t-1}$, $u_{t}$, $z_{t}$)

for i = 1:N

- Sample $x_t^i$ from $p(x_t|x_{t-1}^i, u_{t})$

- $w_t^i = p(z_t|x_t^i)$

end for

Normalize $\lbrace w_t^i\rbrace_{i=1}^N$

Resampling:

- Sample from $x_t^i$ with probability $\propto w_t^i$

- $\lbrace w_t^i\rbrace_{i=1}^N = \frac 1 N$

end for

——————————————

## 扩展

粒子滤波是个很大的研究课题，基于SIS框架，可以利用很多优化方法得到粒子滤波的各种变种。例如：选择不同的proposal，使用不同的Resampling方法等等。

粒子滤波有两个主要问题，一个是Degeneracy，可以通过逼近最优的proposal，或者Resampling来缓解。二是Resampling造成的sampling impoverishment。通过不同的方法优化这些问题，可产生很多粒子滤波的变种。针对具体的问题和应用场景，在传统粒子滤波的基础上优化某个点，也许会产生更好的效果。

### 参考文献

Sebastian THRUN, Wolfram BURGARD, Dieter FOX. PROBABILISTIC ROBOTICS. 2005.

M\. Sanjeev Arulampalam, Simon Maskell, Neil Gordon, and Tim Clapp. A Tutorial on Particle Filters for Online Nonlinear/Non-Gaussian Bayesian Tracking. IEEE TRANSACTIONS ON SIGNAL PROCESSING, VOL. 50, NO. 2, FEBRUARY 2002.

徐亦达. Particle Filter Part1-8, YouTube.com


#### 版权声明：本文为作者原创，转载请注明出处[https://wang-xinyu.github.io](https://wang-xinyu.github.io)