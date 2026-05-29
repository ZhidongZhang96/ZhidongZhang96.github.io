---
title: "From Eigenvector to Eigenfunction"
date: 2026-01-14
# 自定义 URL 示例：
# slug: "from-eigenvector-to-eigenfunction"  # -> 生成 /blogs/demo-post/
# url: "/hello"        -> 生成 /hello/ (完全自定义)
description: "Personal notes about the connection between eigenvectors and eigenfunctions in linear algebra and calculus."
tags: ["Math"]
---

## 1 Function as a vector
In linear algebra, a matrix $A$ is a **linear transformation** acting on a vector space. The geometric effect of it can be seen as a linear composition of the scaling effect with a coefficient $\lambda_i$ in the direction of each eigenvector $\vec v_i$.

In calculus, we can extend this intuition to functional spaces. Functions can be viewed as "infinite-dimensional vectors," and operations like differentiation and integration act as linear transformations (or **operators**) upon them.

Let's start with a discrete function $f(x)$ with $x \in \mathbb{Z}$. It can be represented as a vector $\vec{f}$ where the index corresponds to the input $x$ (domain) and the element value corresponds to $f(x)$:$$f(x) \longrightarrow \vec{f} = \begin{pmatrix} \vdots \\ f(0) \\ f(1) \\ \vdots \end{pmatrix}$$, with the define domain being the index, and the value domain being the elements.

> While we use a discrete function for illustration, the concept generalizes to **continuous** functions by taking the limit as the interval $\Delta x \to 0$. In this limit, the vector becomes infinitely dense

## 2 Operators as Matrices
Once functions are treated as vectors, linear operators can also be visualized as matrices.
1. **Derivative as a Difference Matrix** 
   The derivative operator $\frac{d}{dx}$ corresponds to the finite difference operation $\Delta f(x) = f(x+1) - f(x)$. In matrix form, this looks like a band matrix with $-1$ on the main diagonal and $1$ on the super-diagonal:$$\Delta \ \text{or}\ \frac{d}{dx}\longrightarrow A_{\text{diff}}= \begin{pmatrix}
-1&1& \cdots &0&0 \\
0&-1&\cdots&0 &0 \\
\vdots &\vdots&\ddots &\vdots&\vdots \\
0&0&\cdots&-1&1 \\
0&0&\cdots&0&-1
\end{pmatrix}$$
2. **Integral as a Summation Matrix** 
   The integral operator $\int$ corresponds to the cumulative sum (prefix sum). This is represented by a lower triangular matrix of ones: $$\sum_x \ \text{or}\ \int_x\ \longrightarrow A_{\text{int}} = \begin{pmatrix}
1&0&\cdots&0&0\\
1&1&\cdots&0&0\\
\vdots&\vdots&\ddots&\vdots&\vdots\\
1&1&\cdots&1&0\\
1&1&\cdots&1&1
\end{pmatrix}$$

>$A_{\text{diff}}$ and $A_{\text{int}}$ are roughly inverses of each other, mirroring the Fundamental Theorem of Calculus.
## 3 From Eigenvector to eigenfunction

We have successfully mapped functions to vectors and operators to matrices. Now, let's apply the core concept of linear algebra: the **eigen-problem**.

For a square matrix $A$, an eigenvector is a vector that doesn't change its direction after the transformation. Similarly, for a linear operator $\hat{L}$, an **eigenfunction** $\psi(x)$ is a function that doesn't change its "shape" after the operation, only its amplitude (scaled by $\lambda$).

Mathematically, we look for a function $\psi(x)$ that satisfies:
$$\hat{L} \psi(x) = \lambda \psi(x)$$

### 3.1 Eigenfunctions of derivative and integration

Let's apply this to the **derivative operator** $\frac{d}{dx}$. We are looking for a function whose derivative is simply the function itself multiplied by a constant $\lambda$:
$$\frac{d}{dx} \psi(x) = \lambda \psi(x)$$

This is a simple differential equation. The solution is the exponential function:
$$\psi(x) = C e^{\lambda x}$$
This result reveals a profound connection: ==Exponential functions ($e^{\lambda x}$) are the eigenfunctions of the derivative operator.==

Just as eigenvectors define a coordinate system where a matrix transformation becomes simple diagonal scaling, ==exponential functions form a "basis" where complex calculus operations become simple algebra==:
* **Differentiation** becomes **multiplication**: $\frac{d}{dx} e^{\lambda x} \to \lambda \cdot e^{\lambda x}$
* **Integration** becomes **division**: $\int e^{\lambda x} dx \to \frac{1}{\lambda} \cdot e^{\lambda x}$

> It is also why the solution of ordinary differential equation (ODE) are always multinomial of $e^{f(x)}$ ($e^{\lambda x}$ for constant coefficient), and why we can use **feature equation** to solve. 
> (*roughly like this, ignore the weird circle of argument pls*)

For example, for a 2-order constant coefficient linear ODE: 
$$y'' + ay' + by = 0$$

, we can always assume the solution to be $$y(x) = e^{\lambda x}$$, then we get $$\lambda^2 e^{\lambda x} + a\lambda e^{\lambda x} + b e^{\lambda x} = 0\quad\Rightarrow\quad \lambda^2 + a\lambda + b = 0$$

### 3.2 Eigenfunctions of convolution 

The convolution operator can be mapped to **Toeplitz matrix** (not circulant if noncircular or linear convolution, **circulant** if circular convolution).

$$
\begin{pmatrix}
y_0 \\
y_1 \\
y_2 \\
y_3 \\
y_4 \\
y_5
\end{pmatrix} =
\underbrace{
\begin{pmatrix}
h_0 & 0 & 0 & 0 \\
h_1 & h_0 & 0 & 0 \\
h_2 & h_1 & h_0 & 0 \\
0 & h_2 & h_1 & h_0 \\
0 & 0 & h_2 & h_1 \\
0 & 0 & 0 & h_2
\end{pmatrix}
}_{\text{Toeplitz Matrix (not circulant)}}
\begin{pmatrix}
x_0 \\
x_1 \\
x_2 \\
x_3
\end{pmatrix}
$$

$$ \begin{pmatrix} y_0 \\ y_1 \\ y_2 \\ y_3 \end{pmatrix} = \underbrace{\begin{pmatrix} h_0 & h_3 & h_2 & h_1 \\ h_1 & h_0 & h_3 & h_2 \\ h_2 & h_1 & h_0 & h_3 \\ h_3 & h_2 & h_1 & h_0 \end{pmatrix}}_{\text{Circulant Toeplitz Matrix}} \begin{pmatrix} x_0 \\ x_1 \\ x_2 \\ x_3 \end{pmatrix} $$

For circulant Toeplitz matrix $C$, it can be represented as a polynomial of $P$ (**single-step cyclic permutation matrix**): $$C=\sum_{k=0}^{n-1}c_kP^k$$ with $c_k = C_{1k}$, $P^{k}$ means shift the elements down by $k$ step. (e.g., $P\times(x_0,x_1,x_2)^{\top}= (x_2,x_0,x_1)^{\top}$, and $P^0=I$).

$$
P = \begin{pmatrix}
0 & 0 & \cdots & 0 & 1 \\
1 & 0 & \cdots & 0 & 0 \\
0 & 1 & \cdots & 0 & 0 \\
\vdots & \vdots & \ddots & \vdots & \vdots \\
0 & 0 & \cdots & 1 & 0
\end{pmatrix}
$$

For example:$$C = \begin{pmatrix}
h_0 & h_3 & h_2 & h_1 \\
h_1 & h_0 & h_3 & h_2 \\
h_2 & h_1 & h_0 & h_3 \\
h_3 & h_2 & h_1 & h_0
\end{pmatrix} = h_0 I + h_1 P + h_2 P^2 + h_3 P^3$$

Because $CP=PC$, they *share the same eigenvectors (not eigenvalues!)*. Then the problem is to explore $P$.

For $P$ with $N$ dimension, $P^N = I$ (*back to place*), then:

$$
\begin{aligned} P\ \vec v &= \lambda\ \vec v\\
P^N\ \vec v &= \lambda^N \vec v\\
\vec v &= \lambda^N\ \vec v\\
\end{aligned}
$$

As $\vec v \ne \vec 0$, we have $$\begin{aligned} &\lambda^N=1 = e^{i2\pi} \\ \Rightarrow\quad &\lambda_k = e^{i 2\pi k/N}, \quad k=0,1,\cdots,N-1 \end{aligned}$$
For a specific eigenvalue $\lambda_k = e^{i 2\pi k/N}$, given a vector $\vec v = (v_0,\cdots,v_{N-1})^{\top}$: 

$$\begin{aligned} P\vec v &= \lambda_k \vec v \\
\begin{pmatrix}
v_{N-1}\\
v_0\\
\vdots\\
v_{N-2}
\end{pmatrix}&= \lambda_k \begin{pmatrix}
v_0\\
v_1\\
\vdots\\
v_{N-1}
\end{pmatrix}
\end{aligned}$$, then we have $$v_j = \lambda_k^{-j}v_0$$
, and 
$$\vec v = v_0\begin{pmatrix}
1\\
\lambda_k^{-1}\\
\vdots\\
\lambda_k^{-(N-1)}
\end{pmatrix}\xrightarrow{v_0\equiv1} \begin{pmatrix}
1\\
\lambda_k^{-1}\\
\vdots\\
\lambda_k^{-(N-1)}
\end{pmatrix} = \begin{pmatrix}
1\\
e^{-i\omega}\\
\vdots\\
e^{-(N-1)i\omega}
\end{pmatrix}, \quad \omega= 2\pi k/N$$

Then the eigenvector for $\lambda_k$ contains all angles that one can rotate to, with step $\lambda_k=\omega^k$, which is one **fourier basis vector** in this frequency.

As a result, the eigenvectors of $P$, and also circulant Toeplitz matrix $C$, is **fourier basis vectors**.

The $n$-th element of $\vec v [n]= e^{i\frac{2\pi k}{N} n}$, then it can be extended to continuous function $u(n)=e^{i\omega n}$ or $u(t)=e^{i\omega t}$ if $N\rightarrow \infty$, which is the eigenfunction of convolution.

---
It can also be shown below:

Let $\mathcal{H}$ be a convolution operator defined by the impulse response $h(t)$. The output of our candidate eigenfunction $g(t) = e^{i\omega t}$ is: 
$$\begin{aligned} 
(h*g)(t) &= \int_{-\infty}^{\infty} h(\tau) g(t-\tau) d\tau \\
& = \int_{-\infty}^{\infty} h(\tau) e^{i\omega (t-\tau)} d\tau \\
&= \int_{-\infty}^{\infty} h(\tau) e^{i\omega t} e^{-i\omega \tau} d\tau\\
&= e^{i\omega t} \cdot \underbrace{\int_{-\infty}^{\infty} h(\tau) e^{-i\omega \tau} d\tau}_{\lambda_\omega}
\end{aligned}$$
The expression above is exactly in the form of an **eigen-problem**: $$\mathcal{H} \{ e^{i\omega t} \} = \lambda_\omega \cdot e^{i\omega t}$$ where the eigenvalue $\lambda_\omega$ is: $$\lambda_\omega = H(\omega) = \int_{-\infty}^{\infty} h(\tau) e^{-i\omega \tau} d\tau$$

>This integral is precisely the **fourier transform** of the **impulse response** $h(t)$. It acts by projecting the temporal response $h(t)$ onto the **fourier basis** $e^{i\omega t}$, yielding a continuous set of **eigenvalues** $H(ω)$. 
>In this context, $H(ω)$ represents the **scaling coefficient** (or frequency response) that dictates how the system amplifies or attenuates each specific 'eigen-frequency'.




