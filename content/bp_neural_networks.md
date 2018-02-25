Title: BP Neural Networks
Date: 2018/2/25
Category: Math
Tags: Math
Author: WangXinyu

# BP Neural Networks

---

## Feed Forward

![nn](/images/neural_networks.png)

$$
z_1^{(2)} = w_{11}^{(2)}x_1 + w_{12}^{(2)}x_2 + w_{13}^{(2)}x_3 + b_1^{(2)}
$$

$$
a_1^{(2)} = f(z_1^{(2)})
$$

## Back Propagation

We got N pairs of training data, and the dimension of output vector is k.

$$
\{ (X^{(1)}, Y^{(1)}), (X^{(2)}, Y^{(2)}), ... , (X^{(N)}, Y^{(N)}) \}
$$

$$
Y^{(i)} = \{ y_1^{(i)}, ... , y_k^{(i)} \}
$$

The loss function is:

$$
E^{(i)} = \frac 1 2 \sum_{j=1}^k (y_j^{(i)} - a_j^{(i)})^2
$$

We use Gradient Descent to update weights and biases.

$$
w_{ij}^{(l)} = w_{ij}^{(l)} - \mu \frac {\partial E} {\partial w_{ij}}
$$

$$
b_{i}^{(l)} = b_{i}^{(l)} - \mu \frac {\partial E} {\partial b_{i}}
$$

Calculate the derivative:

\begin{split}
\frac {\partial E} {\partial w_{11}^{(3)}} &= \frac 1 2 \cdot -2 (y_1 - a_1^{(3)}) \cdot \frac {\partial a_{1}^{(3)}} {\partial w_{11}^{(3)}} \\\\
&= -(y_1 -a_1^{(3)}) \cdot  f'(z_1^{(3)}) \cdot  \frac {\partial z_{1}^{(3)}} {\partial w_{11}^{(3)}}  \\\\
&= -(y_1 -a_1^{(3)}) \cdot  f'(z_1^{(3)}) \cdot  a_1^{(2)}  \\\\
\end{split}

Introduce $\delta$ to simplify the formula:

\begin{split}
\delta_i^{(l)} &= \frac {\partial E} {\partial a_{i}^{(l)}} \cdot \frac {\partial a_{i}^{(l)}} {\partial z_{i}^{(l)}} \\\\
\frac {\partial E} {\partial w_{ij}^{(l)}} &= \delta_i^{(l)} \cdot  \frac {\partial z_{i}^{(l)}} {\partial w_{ij}^{(l)}} =  \delta_i^{(l)} \cdot a_{j}^{(l-1)} \\\\
\end{split}

Calculate $\delta$ recursively:

\begin{split}
\delta_i^{(l)} &= \frac {\partial E}  {\partial z_{i}^{(l)}} \\\\
&= \sum_{j=1}^{n_{l+1}} \frac {\partial E}  {\partial z_{j}^{(l+1)}} \frac {\partial z_{j}^{(l+1)}}  {\partial z_{i}^{(l)}} \\\\
&= \sum_{j=1}^{n_{l+1}} \delta_j^{(l+1)} \frac {\partial z_{j}^{(l+1)}}  {\partial z_{i}^{(l)}} \\\\
&= \sum_{j=1}^{n_{l+1}} \delta_j^{(l+1)} w_{ji}^{(l+1)} f'(z_i^{(l)}) \\\\
\end{split}