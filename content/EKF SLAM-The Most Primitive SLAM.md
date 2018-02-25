Title: EKF SLAM-The Most Primitive SLAM
Date: 2017/8/17
Category: Robot
Tags: Math
Author: WangXinyu
# EKF SLAM-The Most Primitive SLAM

## Introduction

In SLAM, the robot is given measurements $z_{1:t}$ and controls $u_{1:t}$, and acquires a map of its environment while simultaneously localizing itself relative to this map. From a probabilistic perspective, the online SLAM involves estimating the posterior over the current pose along with he map:

$$
p(x_{t}, m | z_{1:t}, u_{1:t})
$$

## Assumptions

- The maps in EKF SLAM are feature-based, they are composed of point landmarks.

- With known correspondences or data associations between observations and landmarks.

- All the noises are Gaussian.

## State Vector

$$
y_t = (x,y,\theta,m_{1,x},m_{1,y},...,m_{n,x},m_{n,y})^T
$$

The state vector includes robot pose and landmark positions. The dimension of this state vector is $2N+3$, where $N$ denotes the number of landmarks in the map. In robot localization problem, the state vector only contains robot pose, but for this EKF SLAM problem, we need to update the position of landmarks also. So now the posterior can be denoted as:

$$
p(y_{t}| z_{1:t}, u_{1:t})
$$

## Algorithm

——————————————

1. Algorithm EKF_SLAM_known_correspondences($\mu_{t-1}$, $\Sigma_{t-1}$, $u_{t}$, $z_{t}$)

2. $\bar \mu_t = g(u_t, \mu_{t-1})$
3. $\bar \Sigma_t = G_t \Sigma_{t-1} G_t^T+ R_t$

4. $K_t = \bar \Sigma_t H_t^T(H_t \bar \Sigma_t H_t^T + Q_t)^{-1}$
5. $\mu_t = \bar \mu_t + K_t(z_t - \hat z_t)$
6. $\Sigma_t = (I - K_t H_t) \bar \Sigma_t$

7. return $\mu_t$, $\Sigma_t$

——————————————

Basically, we are using EKF to estimate the belief of robot pose and landmark positions. The filter cycle is:

- State prediction, line 2 and 3, to predict the next state based on a motion model.
- Measurement prediction, for each observed landmark, we calculate its predicted measurement using the predicted robot pose and this landmark's position. We record the landmark's position the first time we see it. With this value, we can calculate $H_t$, $K_t$, $\hat z_t$ and $\Sigma_t$
- Measurement, after incorporating the measurement, we can calculate $\mu_t$
- Data association, we can skip this step, because the correspondences are known, but we need this step, if they are unknown.
- Update, update the belief.

## State Prediction

As robot moves, the state changes according to the Velocity Motion Model. Because the motion only affects the robot pose and all landmarks remain where they are, only the first three elements in the update are non-zero.

$
F_x = \begin{pmatrix} 1 & 0 & 0 & 0 & \cdots & 0 \\\\ 0 & 1 & 0 & 0 & \cdots & 0 \\\\ 0 & 0 & 1 & 0 &\cdots & 0 \end{pmatrix}_{3 \times (2N+3)}
$

$
y_t = \underbrace {
  y_{t-1} + F_x^T \begin{pmatrix}
    - \frac {v_t}{\omega_t}sin\mu_{t-1, \theta} + \frac {v_t}{\omega_t}sin(\mu_{t-1,\theta + \omega_t \Delta t}) \\\\
    \frac {v_t}{\omega_t}cos\mu_{t-1, \theta} - \frac {v_t}{\omega_t}cos(\mu_{t-1,\theta + \omega_t \Delta t}) \\\\
    \omega_t \Delta t \end{pmatrix}
}_{g}
 + \mathcal{N} \left(0, F_x^T R_t F_x \right)
$

The motion function $g$ is approximated using a first degree Taylor expansion.

$
g(y_{t-1}, u_t) \approx g(\mu_{t-1}, u_t) + G_t(y_{t-1} - \mu_{t-1})
$

Jacobian $G_t$ is the derivative of $g$ with respect to $y_{t-1}$ at $u_t$ and $\mu_{t-1}$.

$
G_t = \frac {\partial g} {\partial (x, y, \theta, m_{1,x}, m_{1,y}, ... , m_{n,x}, m_{n,y})} = \begin{pmatrix} 1 & 0 & -\frac {v_t}{\omega_t}cos\mu_{t-1, \theta} + \frac {v_t}{\omega_t}cos(\mu_{t-1,\theta + \omega_t \Delta t}) &  &  &  \\\\ 0 & 1 & \frac {v_t}{\omega_t}sin\mu_{t-1, \theta} - \frac {v_t}{\omega_t}sin(\mu_{t-1,\theta + \omega_t \Delta t}) &  & 0 &  \\\\ 0 & 0 & 1 &  &  & \\\\  & 0 & & & I & \end{pmatrix}_{(2N+3) \times (2N+3)}
$

So the predicted mean $\bar \mu_t$ and covariance $\bar \Sigma_t$ is the mean and covariance of Gaussian distribution $y_t$.

$
\bar \mu_t = \mu_{t-1} + F_x^T \begin{pmatrix} - \frac {v_t}{\omega_t}sin\mu_{t-1, \theta} + \frac {v_t}{\omega_t}sin(\mu_{t-1,\theta + \omega_t \Delta t})\\\\ \frac {v_t}{\omega_t}cos\mu_{t-1, \theta} - \frac {v_t}{\omega_t}cos(\mu_{t-1,\theta + \omega_t \Delta t}) \\\\ \omega_t \Delta t \end{pmatrix}
$

$\bar \Sigma_t = G_t \Sigma_{t-1} G_t^T+ F_x^T R_t F_x$

## Measurement Prediction

We can predicte the measurement using the following measurement model.

$
z_t^i = \underbrace {
  \begin{pmatrix}
    \sqrt{(m_{j,x} - x)^2 + (m_{j,y} - y)^2} \\\\
    atan2(m_{j,y} - y, m_{j,x} - x) - \theta
  \end{pmatrix}
}_{h} + \mathcal{N} (0, Q_t)
$

$
Q_t = 
  \begin{pmatrix}
    \sigma_r & 0 \\\\
    0 & \sigma_\phi
  \end{pmatrix}
$

$i$ is the index of landmark observation in $z_t$, $j = c_t^i$ is the index of landmark at time $t$.

The measurement function $h$ is also approximated using a first degree Taylor expansion.

$
h(y_{t}, j) \approx h(\bar \mu_{t}, j) + H_t^i(y_{t} - \bar \mu_{t})
$

Jacobian $H_t^i$ is the derivative of $h$ with respect to the full state vector $y_{t}$ at $\bar \mu_{t}$ and $j$. And we use $\delta$ and $q$ to make the formulas simpler.

$
\delta = \begin{pmatrix} \delta_x \\\\ \delta_y \end{pmatrix} = \begin{pmatrix}  \bar \mu_{j,x} - \bar \mu_{t, x} \\\\ \bar \mu_{j,y} - \bar \mu_{t, y} \end{pmatrix}
$

$
q = \delta_x^2 + \delta_y^2
$

$
H_t^i = \frac {\partial h} {\partial (x, y, \theta, m_{1,x}, m_{1,y}, ... , m_{n,x}, m_{n,y})} = 
{\begin{pmatrix}
  - \frac {\delta_x} {\sqrt{q}} & - \frac {\delta_y} {\sqrt{q}} & 0 & 0_{1 \times (2j-2)} & \frac {\delta_x} {\sqrt{q}} & \frac {\delta_y} {\sqrt{q}} & 0_{1 \times (2N-2j)} \\\\
  \frac {\delta_y} {q} & - \frac {\delta_x} {q} & -1 & 0_{1 \times (2j-2)} & -\frac {\delta_y} {q} & \frac {\delta_x} {q} & 0_{1 \times (2N-2j)} \end{pmatrix}}_{2 \times (2N+3)}
$

The position of landmark $j$ is $\bar \mu_{j}$, which is cached the first time this landmark is observed.

$
\begin{pmatrix}
\bar \mu_{j,x} \\\\
\bar \mu_{j,y}
\end{pmatrix}
=
\begin{pmatrix}
\bar \mu_{t,x} \\\\
\bar \mu_{t,y}
\end{pmatrix}
+
\begin{pmatrix}
r_t^i cos(\phi_t^i + \bar \mu_{t,\theta}) \\\\
r_t^i sin(\phi_t^i + \bar \mu_{t,\theta})
\end{pmatrix}
$

So now we can calcute $\hat z_t^i$, $K_t^i$ and further $\mu_t$ and $\Sigma_t$.

$
\hat z_t^i = 
\begin{pmatrix}
\sqrt{q} \\\\
atan2(\delta_y, \delta_x) - \bar \mu_{t, \theta}
\end{pmatrix}
$

$K_t^i = \bar \Sigma_t H_t^{iT}(H_t^i \bar \Sigma_t H_t^{iT} + Q_t)^{-1}$

$\bar \mu_t = \bar \mu_t + K_t^i(z_t^i - \hat z_t^i)$

$\bar \Sigma_t = (I - K_t^i H_t^i) \bar \Sigma_t$

When processing the $i_{th}$ observation in the loop, we update the predicted mean and covariance using the values we got from $(i-1)_{th}$ observation. That is we update $\bar \mu_t$ and $\bar \Sigma_t$ iteratively. And after the for loop, the updated $\bar \mu_t$ and $\bar \Sigma_t$ are the final $\mu_t$ and $\Sigma_t$ respectively.

$\mu_t = \bar \mu_t$

$\Sigma_t = \bar \Sigma_t $

### References

Sebastian THRUN, Wolfram BURGARD, Dieter FOX. PROBABILISTIC ROBOTICS. 2005.

Cyrill Stachniss. EKF SLAM, YouTube.com
