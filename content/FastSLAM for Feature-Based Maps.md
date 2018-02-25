Title: FastSLAM for Feature-Based Maps
Date: 2017/8/25
Category: Robot
Tags: Math
Author: WangXinyu
# FastSLAM for Feature-Based Maps

## Introduction

FastSLAM applys a version of particle filters to SLAM known as Rao-Blackwellized Particle Filter. Which uses particles to represent the posterior over some variables, along with Gaussians to represent all other vaviables. 

## Assumptions

- The maps in EKF SLAM are feature-based, they are composed of point landmarks.

- With known correspondences or data associations between observations and landmarks.

## Factoring the SLAM Posterior

$
p(x_{1:t}, m| z_{1:t}, u_{1:t}) = p( x_{1:t} | z_{1:t}, u_{1:t}) p(m|x_{1:t},z_{1:t}) = p( x_{1:t} | z_{1:t}, u_{1:t}) \prod_{i=1}^N p(m_i|x_{1:t},z_{1:t})
$

FastSLAM uses a particle filter to compute the posterior over robot paths, i.e. $p( x_{1:t} | z_{1:t}, u_{1:t})$. For each of these particles, the individual map errors are conditionally independent. Hence the mapping problem can be factored into separate problems $p(m_i|x_{1:t},z_{1:t})$, one for each feature in the map. FastSLAM estimates these map feature locations by using low-dimensional EKFs.

The feature estimators are conditioned on the robot path, which means we will have a separate copy of each feature estimator, one for each particle. With $M$ particles and $N$ feature, the total number of filters will be $1+MN$, one particle filter and $MN$ EKFs.

## Algorithm


——————————————

1. Algorithm FastSLAM_known_correspondences($z_{t}$, $c_{t}$, $u_{t}$, $\mathcal{X}_{t-1}$)

2. $for \ k = 1 \ to \ M \ do$
3. $\ \ \langle  x_{t-1}^{[k]}, \langle \mu_{1,t-1}^{[k]}, \Sigma_{1,t-1}^{[k]}, ... \rangle \rangle, \ Get \ particle \ k \ in \  \mathcal{X}_{t-1}$

4. $\ \ x_t^{[k]} \sim  p(x_t|x_{t-1}^{[k]}, u_t)$
5. $\ \ j = c_t, \ feature \ j \ is \ observed$
6. $\ \ if \ feature \ j \ never \ seen \ before$
7. $\ \ \ \ initialize \ \mu_{j,t}^{[k]}, H, \Sigma_{j,t}^{[k]} = H^{-1} Q_t (H^{-1})^T, w^{[k]} = p_0$
8. $\ \ else$
9. $\ \ \ \ \langle \mu_{j,t}^{[k]}, \Sigma_{j,t}^{[k]} \rangle = \text{EKF-Update()}$
10. $\ \ \ \ w^{[k]} = |2 \pi Q|^{- \frac 1 2} exp \lbrace - \frac 1 2 (z_t - \hat z ^{[k]})^T Q^{-1} (z_t - \hat z ^{[k]}) \rbrace $
11. $\ \ endif$
12. $endfor$

13. $\mathcal{X}_t = resample( particle \ 1, \ ..., \ M)$

14. return $\mathcal{X}_t$

——————————————

Basically, we are using Partilce Filter to estimate the robot path and using EKF to update landmark positions. The filter cycle is:

- Loop over all particles, for each particle, we sample the new pose, update the mean and covariance of landmark according to the observation, and finaly calculate the weight of this particle.
- Resampling.

## Importance Weight

\begin{split}
w^{[k]} &= {target(x^{[k]}) \over proposal(x^{[k]})} \\\\
&= {p(x_{1:t}^{[k]}|z_{1:t},u_{1:t}) \over p(x_{1:t}^{[k]}|z_{1:t-1},u_{1:t})} \\\\
&= {p(x_{1:t}^{[k]}|z_{1:t},u_{1:t}) \over p(x_{t}^{[k]}| x_{t-1}^{[k]}, u_t) p(x_{1:t-1}^{[k]}|z_{1:t-1},u_{1:t-1})} \\\\
&= {p(z_t|x_{1:t}^{[k]}, z_{1:t-1},u_{1:t}) p(x_{1:t}^{[k]}|z_{1:t-1},u_{1:t}) \over p(z_t|z_{1:t-1},u_{1:t})} {1 \over p(x_{t}^{[k]}| x_{t-1}^{[k]}, u_t) p(x_{1:t-1}^{[k]}|z_{1:t-1},u_{1:t-1})} \\\\
&= \eta \cdot p(z_t|x_{1:t}^{[k]}, z_{1:t-1},u_{1:t}) p(x_{1:t}^{[k]}|z_{1:t-1},u_{1:t}) {1 \over p(x_{t}^{[k]}| x_{t-1}^{[k]}, u_t) p(x_{1:t-1}^{[k]}|z_{1:t-1},u_{1:t-1})} \\\\
&= \eta \cdot p(z_t|x_{1:t}^{[k]}, z_{1:t-1})
\end{split}

Our proposal is $p(x_{1:t}^{[k]}|z_{1:t-1},u_{1:t})$, normally we sample from proposal, but since it can be factored into $ p(x_{t}^{[k]}| x_{t-1}^{[k]}, u_t) p(x_{1:t-1}^{[k]}|z_{1:t-1},u_{1:t-1})$, so we can sample $x_{t}^{[k]}$ from $p(x_{t}^{[k]}| x_{t-1}^{[k]}, u_t)$.

We can't calculate $p(z_t|x_{1:t}^{[k]}, z_{1:t-1})$ directly, it will be necessary to transform it further. The idea is integrating over the position of observed landmark.

\begin{split}
w^{[k]} &= \eta \cdot p(z_t|x_{1:t}^{[k]}, z_{1:t-1}) \\\\
&= \eta \cdot \int p(z_t|x_{1:t}^{[k]}, z_{1:t-1}, m_j) p(m_j|x_{1:t}^{[k]}, z_{1:t-1})dm_j \\\\
&= \eta \cdot \int p(z_t|x_{t}^{[k]}, m_j) p(m_j|x_{1:t-1}^{[k]}, z_{1:t-1})dm_j
\end{split}

$$
p(z_t|x_{t}^{[k]}, m_j) \sim \mathcal{N} \left( z_t; \hat z^{[k]}, Q_t \right)
$$

$$
p(m_j|x_{1:t-1}^{[k]}, z_{1:t-1}) \sim \mathcal{N} \left( m_j; \mu_{j,t-1}^{[k]}, \Sigma_{j,t-1}^{[k]} \right)
$$

Thus $w^{[k]}$ actually results from the convolution of two Gaussian.

$$
w^{[k]} \approx \eta \cdot |2 \pi Q|^{- \frac 1 2} exp \lbrace - \frac 1 2 (z_t - \hat z ^{[k]})^T Q^{-1} (z_t - \hat z ^{[k]}) \rbrace
$$

$$
Q = H \Sigma_{j,t-1}^{[k]} H^T + Q_t
$$


### References

Sebastian THRUN, Wolfram BURGARD, Dieter FOX. PROBABILISTIC ROBOTICS. 2005.

Cyrill Stachniss. FastSLAM, YouTube.com
