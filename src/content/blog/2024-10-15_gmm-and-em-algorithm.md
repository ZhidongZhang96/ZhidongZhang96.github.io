---
title: "GMM and EM algorithm"
description: "Course notes about the GMM and EM algorithms."
publishDate: 2024-10-15
tags: ["course", "math"]
tldr:
  - "The EM algorithm is a general technique for maximum likelihood estimation in latent variable models, where the key difficulty is the sum-inside-log form of the marginal likelihood."
  - 'In the GMM, the latent variable $z_i$ is a one-hot cluster assignment, and the posterior $p(z_{ik}=1|\mathbf x_i)$ acts as the responsibility, enabling soft assignment.'
  - 'MLE for GMM yields fixed-point equations with no closed-form solution, so we iterate between an E-step that computes the responsibilities and an M-step that updates $\boldsymbol\mu_k, \boldsymbol\Sigma_k, \pi_k$.'
  - 'By Jensen''s inequality, the ELBO $\mathcal L(q,\theta)$ lower-bounds $\log p(X|\theta)$: the M-step maximizes it with respect to $\theta$, while the E-step tightens it by minimizing $D_{KL}(q\|p(Z|X,\theta))$.'
  - "When the exact posterior is intractable, we restrict $q$ to a tractable family, leading to Variational Inference."
---

The Expectation-maximization algorithm is a general technique for finding maximum likelihood solutions for latent variable models. Here I use the discrete latent-variable case, the Gaussian mixture model, to introduce the idea of EM.

## Gaussian Mixture Model

GMM is one of the discrete **latent variable models**, and also one of Mixture model that can be used for **clustering** and **density estimation**. It is similar to K-means, but uses Gaussian distributions and "responsibilities" to support more flexible modeling (from sphere to ellipse) and soft-assignment.

For a GMM, the latent variable $\mathbf z_i$ is defined as the assignment of a single data $\mathbf x_i$ to a cluster $k$ out of $K$ clusters, which normally uses _one-hot_ coding: $\mathbf z_i = (z_{i1},\cdots,z_{iK}) \in \{0,1\}^K$, where

$$
z_{ik}=\left\{
\begin{array}{lr}
1, & \text{if }\mathbf x_i \text{ belongs to cluster }k\\
0, & \text{otherwise}
\end{array}
\right.
$$

The relevant probabilities for a single data point $\mathbf x_i$ are shown below, with parameters $\theta = \{\pi, \mu,\Sigma\}$ to be learned.

- Prior: the probability of a random data belonging to cluster $k$

  $$
  P(z_{ik}=1) := \pi_k,\quad p(\mathbf z_i) = \prod_j^K \pi_j^{z_{ij}}
  $$

  with $\sum_k^K \pi_k=1$.

- Likelihood: the probability of the data given the cluster $k$

  $$
  p(\mathbf x_i | z_{ik}=1) := \mathcal N(\mathbf x_i;\boldsymbol\mu_k,\boldsymbol\Sigma_k)
  $$

- Joint likelihood:

  $$
  p(\mathbf x_i, z_{ik}=1) = \pi_k \mathcal N(\mathbf x_i;\boldsymbol\mu_k,\boldsymbol\Sigma_k) = \prod_j^K \pi_j^{z_{ij}}  \mathcal N(\mathbf x_i;\boldsymbol\mu_j,\boldsymbol\Sigma_j)^{z_{ij}}
  $$

- Marginal likelihood: marginal over all possible clusters:

  $$
  p(\mathbf x_i) = \sum_k^K p(\mathbf x_i, z_{ik}=1) = \sum_k^K \pi_k \mathcal N(\mathbf x_i;\boldsymbol\mu_k,\boldsymbol\Sigma_k)
  $$

- Posterior: the probability of the data belonging to cluster $k$

  $$
  \begin{aligned}
  p(z_{ik}=1|\mathbf x_i) &= \frac{p(\mathbf x_i, z_{ik}=1)}{\sum_{k'}p(\mathbf x_i, z_{ik'}=1)} = \frac{\pi_k \mathcal N(\mathbf x_i;\boldsymbol\mu_k,\boldsymbol\Sigma_k)}{\sum_{k'}\pi_{k'} \mathcal N(\mathbf x_i;\boldsymbol\mu_{k'},\boldsymbol\Sigma_{k'})}\\[0.5em] &:= r_{ik}\quad \text{(responsibility)}
  \end{aligned}
  $$

> The posterior is also called the **responsibility**, which performs a _soft-assignment_.

Similarly, the corresponding probability terms for the whole dataset $X$ are:

$$
p(Z)=\prod_i^N\prod_j^K\pi_j^{z_{ij}},\quad p(X|Z)=\prod_i^N\prod_j^K \mathcal N(\mathbf x_i;\boldsymbol\mu_j,\boldsymbol\Sigma_j)^{z_{ij}}
$$

$$
p(X,Z)=\prod_i^N\prod_j^K \pi_j^{z_{ij}} \mathcal N(\mathbf x_i;\boldsymbol\mu_j,\boldsymbol\Sigma_j)^{z_{ij}}
$$

$$
p(X)=\prod_i^N\sum_k^K \pi_k \mathcal N(\mathbf x_i;\boldsymbol\mu_k,\boldsymbol\Sigma_k)
$$

## Learning of GMM

For a GMM, the learning aims to maximize the marginal log-likelihood $\log p(X|\theta)$.

By setting the derivative w.r.t. each parameter to be zero, we can write down the equations:

$$
R^*_{k} := \sum_i^N r_{ik}
$$

$$
\begin{aligned}
\frac{\partial}{\partial \boldsymbol \mu_k} \log p(X|\theta) = 0\quad\Rightarrow & \quad \boldsymbol \mu_k^* = \frac{1}{R^*_{k}} \sum_i^N r_{ik} \mathbf x_i \\[0.5em]
\frac{\partial}{\partial \boldsymbol \Sigma_k} \log p(X|\theta) = 0\quad\Rightarrow & \quad \boldsymbol \Sigma_k^* = \frac{1}{R^*_{k}} \sum_i^N r_{ik} (\mathbf x_i - \boldsymbol\mu_k)(\mathbf x_i - \boldsymbol\mu_k)^\top \\[0.5em]
\frac{\partial}{\partial \pi_k} \left[\log p(X|\theta) + \lambda (1-\sum_k^K \pi_k)\right] = 0\quad\Rightarrow & \quad \pi^*_k = \frac{1}{N} R^*_k
\end{aligned}
$$

Note that the fixed-point equations above _do not_ provide a closed-form solution for the parameters, because the responsibilities $r_{ik}$ which appear in the right-hand side depend on those parameters.

However, it does suggest an iterative scheme to find a solution, which turns out to be an instance of the Expectation-maximization algorithm:

- E-step: Assign responsibilities by computing the posterior.

  $$
  r_{ik}:= p(\mathbf z_{ik}=1|\mathbf x_i)=\frac{\pi_k \mathcal N(\mathbf x_i;\boldsymbol\mu_k,\boldsymbol\Sigma_k)}{\sum_{k'}\pi_{k'} \mathcal N(\mathbf x_i;\boldsymbol\mu_{k'},\boldsymbol\Sigma_{k'})},\quad R_k = \sum_i^Nr_{ik}
  $$

- M-step: Update parameters with the computed $r_{ik}$.

  $$
  \boldsymbol\mu_k^{\text{new}}\leftarrow \frac{1}{R_{k}} \sum_i^N r_{ik} \mathbf x_i, \quad \pi_k^{\text{new}}\leftarrow \frac{1}{N} R_k
  $$

  $$
  \boldsymbol\Sigma_k^{\text{new}} \leftarrow \frac{1}{R_{k}} \sum_i^N r_{ik} (\mathbf x_i - \boldsymbol\mu_k)(\mathbf x_i - \boldsymbol\mu_k)^\top
  $$

- Check convergence by evaluating $\log p(X|\theta)$

The initialization matters for a good and fast convergence. In practice, the K-means algorithm is used to find a suitable initialization for it, which takes much less iterations than GMM to converge. But note that it does not guarantee a global maximum, as the log-likelihood $\log p(X|\theta)$ can be multimodal.

## EM-algorithm

Why the EM-algorithm for GMM introduced earlier actually works to increase $\log p(X|\theta)$ was poorly shown. Here let's dive into the algorithm deeper with abstract settings.

For an arbitrary latent variables model, the marginal log-likelihood has to be obtained by summing over all possible values of the latent variables $Z$, which results in a sum-inside-log form:[^1]

$$
\log p(X|\theta) = \log\left[ \sum_Z p(X,Z|\theta)\right]
$$

The sum-inside-log form makes the optimization intractable. It "prevents the log from acting directly on the joint distribution, resulting in complicated expressions for the maximum likelihood".[^2]

How can we deal with it? Since the logarithm function $\log(\cdot)$ is _concave_, then **Jensen's inequality** apply:

$$
\log(\mathbb E[X]) \ge \mathbb E[\log(X)]
$$

That is, the log of the mean is bigger than the mean of the logs.

Moreover, we can turn the summations into an expectation with a proposal distribution $q(Z)$:

$$
\sum_Z p(X,Z|\theta) = \sum_Z q(Z) \frac{p(X,Z|\theta)}{q(Z)}  = \mathbb E_{q(Z)}\left[\frac{p(X,Z|\theta)}{q(Z)}\right]
$$

Therefore we can write down the lower bound of the marginal log-likelihood:

$$
\begin{aligned}
\log p(X|\theta) = \log\left[ \sum_Z p(X,Z|\theta)\right]  &= \log \left[ \sum_Z q(Z)  \frac{p(X,Z|\theta)}{q(Z)} \right] \\[0.5em]
&\ge  \sum_Z q(Z)\log \frac{p(X,Z|\theta)}{q(Z)}  := \mathcal L(q,\theta)
\end{aligned}
$$

The lower bound $\mathcal L(q,\theta)$ gets rid of the annoying sum-inside-log form, and thus is much easier for optimization.

With $\mathcal L(q,\theta)$, also called **evidence lower bound** (ELBO) or negative free energy, one may naturally think of maximizing it with respect to $\theta$ to increase the targeted $\log p(X|\theta)$. That is exactly the underlying idea of the general EM-algorithm! The only with additional step is to also update the ELBO so that it can 'catch up with' $\log p(X|\theta)$. These two steps actually correspond to the M-step and E-step, respectively. Next let's see how it works.

For the M-step, given a fixed proposal $q(Z)$, we can maximize the ELBO with respect to the parameters:

$$
\begin{aligned}\theta_{\text {new}} \leftarrow \arg\max_{\theta} \mathcal L(q,\theta) &= \arg\max_{\theta}\left\{ \sum_Z q(Z)\log \frac{p(X,Z|\theta)}{q(Z)}
\right\} \\[0.5em]
&= \arg\max_{\theta}\left\{ \sum_Z q(Z)\log p(X,Z|\theta) \underbrace{- \sum_Z q(Z)\log q(Z)}_{\text{Entropy: }H(Z)}
\right\} \\[0.5em]
&= \arg\max_{\theta} \sum_Z q(Z)\log p(X,Z|\theta) \\[0.5em]
&= \arg\max_{\theta} \mathbb E_{q(Z)}\left[ \log p(X,Z|\theta)\right]
\end{aligned}
$$

> The optimization objective is then changed to maximize $\mathbb E_{q(Z)}\left[ \log p(X,Z|\theta)\right]$, which is the **expected complete-data** log-likelihood.
> It is called 'complete-data' in the sense that it contains _both the observed and latent variables_. Because we cannot collect the complete dataset $\{X,Z\}$ but only the incomplete one $\{X\}$, we consider the expected values over $q(Z)$, or the posterior $p(Z|X,\theta)$ ideally.

For the E-step, we want to update the ELBO with $\theta_{\text{new}}$, so that it is as close to $\log p(X|\theta)$ as possible. To do so, we need to formalize the 'distance' between them, which turns out to be the **KL-divergence** between the proposal $q(Z)$ and the posterior $p(Z|X,\theta)$:

$$
\begin{aligned}
\mathcal L(q,\theta) &= \sum_Z q(Z)\log \frac{p(X,Z|\theta)}{q(Z)} \\[0.5em]
&= \sum_Z q(Z)\log \frac{p(X|\theta)p(Z|X,\theta)}{q(Z)} \\[0.5em]
&= \sum_Z q(Z)\log p(X|\theta) + \sum_Z q(Z)\log \frac{p(Z|X,\theta)}{q(Z)} \\[0.5em]
&= \log p(X|\theta) - D_{KL}{\large[} q(Z) \| p(Z|X,\theta) {\large]}
\end{aligned}
$$

Therefore, with a new parameter $\theta_{\text{new}}$, the E-step aims to find a new $p(Z)$ that minimizes the KL-divergence, and an ideal one is to just use the posterior, so that $D_{KL}{\large[} q(Z) \| p(Z|X,\theta) {\large]} = 0$, and the ELBO can 'tight' the marginal log-likelihood.

:::note[The General EM-algorithm]
Given a joint distribution $p(X,Z|\theta)$ over observed variables $X$, latent variables $Z$ and the parameters $\theta$, the goal is to maximize the marginal likelihood $p(X|\theta)$ with respect to $\theta$.
:::

1. Initialize the parameters $\theta = \theta_0$
2. Iteration
   1. **E-step** Evaluate the posterior $p(Z|X,\theta_{\text{old}})$, use it as the proposal $q(Z) = p(Z|X,\theta_{\text{old}})$ (_find the lower bound_)
   2. **M-step** Update the parameter (_tight the lower bound_):

      $$
      \theta_{\text{new}} = \arg\max_{\theta} \mathcal L(\theta,\theta_{\text{old}}) = \arg\max_{\theta} \sum_Z p(Z|X,\theta_{\text{old}}) \log p(X,Z|\theta)
      $$

   3. Check for convergence, and apply $\theta_{\text{old}} \leftarrow \theta_{\text{new}}$ if not satisfied.

> In many cases, it is pretty hard to actually find the posterior to make $D_{KL}{\large[} q(Z) \| p(Z|X,\theta) {\large]}=0$ perfectly. And we have to use $q(Z)$ in a convenient family to make life easier, for example, Gaussian distributions or exponential family. The optimization over function $q(Z)$ towards this objective leads to calculus of variations, and the idea of **Variational Inference**.

## GMM revisited

Now let's turn back to the GMM.

The E-step remains the same: evaluate the posterior as the responsibility $r_{ik} := p(z_{ik}=1|\mathbf x_i, \theta_{\text{old}})$.

For the M-step we need to recompute the update rules:

$$
\begin{aligned}
\log p(X,Z|\theta) &= \log\prod_i^N \prod_j^K \pi_j^{z_{ij}} \mathcal N(\mathbf x_i;\boldsymbol\mu_j,\boldsymbol\Sigma_j)^{z_{ij}} \\[0.5em]
&= \sum_i^N \sum_j^K z_{ij} {\large(}\log \pi_j^{} + \log\mathcal N(\mathbf x_i;\boldsymbol\mu_j,\boldsymbol\Sigma_j){\large)} \\[0.5em]
\mathbb E_{p(Z|X,\theta_{old})}[\log p(X,Z|\theta)] &= \sum_i^N \sum_j^K E_{p(Z|X,\theta_{old})}[z_{ij}] \cdot{\large(}\log \pi_j^{} + \log\mathcal N(\mathbf x_i;\boldsymbol\mu_j,\boldsymbol\Sigma_j){\large)} \\[0.5em]
&= \sum_i^N \sum_j^K r_{ij} {\large(}\log \pi_j^{} + \log\mathcal N(\mathbf x_i;\boldsymbol\mu_j,\boldsymbol\Sigma_j){\large)} \\[0.5em]
\end{aligned}
$$

Then

$$
\theta_{\text{new}} = \arg\max_{\theta} \mathbb E_{p(Z|X,\theta_{old})}[\log p(X,Z|\theta)]
$$

By setting $\nabla_{\theta} \mathbb E_{p(Z|X,\theta_{old})}[\log p(X,Z|\theta)] = 0$, one can easily get the exact process shown in [Learning of GMM](#learning-of-gmm).

[^1]: The following discussion apply equally well to continuous latent variables, by replacing the sum with an integral

[^2]: Bishop, Christopher M., and Nasser M. Nasrabadi. _Pattern recognition and machine learning_. Vol. 4. No. 4. New York: springer, 2006.
