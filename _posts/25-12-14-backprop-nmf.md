---
layout: post
title:  "Backpropagation through the NMF block"
subtitle: ""
excerpt_separator: "<!--more-->"
date:	2025-12-14
comments: true
categories: ["Annotated BI"]
---

### Motivation

We often decompose high-dimensional profiles into low-rank, sparse matrices using factorization algorithms. 
These algorithms are closely related to dimensionality reduction—the factors they produce naturally encode low-dimensional representations.

Before discussing factorization algorithms further, let's first understand dimensionality reduction as an instance of representation learning. 
This can be formalized through a universal framework called [I-Con](https://arxiv.org/abs/2504.16929). Representation learning aligns the transition probability (i.e., conditional distribution) from a supervisory signal to those of representations specified by a learnable function. 

The core I-Con loss function is:

$$\mathcal{L}(\theta, \phi) =\int_{i\in \mathcal{X}} D_{KL}(p_\theta(\cdot|i)||q_\phi(\cdot|i))=\int_{i\in\mathcal{X}}\int_{j\in\mathcal{X}}p_\theta(j|i)\log\frac{p_\theta(j|i)}{q_\phi(j|i)}$$

Here, $p_\theta$ is a supervisory distribution, while $q_\phi$ is learned by capturing the structure of the desired representation. 
We can optimize both $\theta,\phi$ (e.g., [X-Sample](https://arxiv.org/abs/2407.18134)), though most methods optimize only $\phi$. 
Many representation learning approaches—including t-SNE, SimCLR, and K-Means clustering—can be understood as instances of the I-Con framework.

Now let's circle back to factorization algorithms. 
These include PCA, K-Means clustering, and NMF, but here we focus on NMF as a representative factorization algorithm that learns latent factors as linear combinations of high-dimensional features and reconstructs the original features via linear projection. 
Note that NMF (nonnegative matrix factorization) is equivalent to K-means clustering on a bipartite graph (where nodes correspond to both samples and features), with relaxation that allows soft assignments to the K clusters([Ding et al.](https://www.sciencedirect.com/science/article/abs/pii/S0167947308000145)).

Since NMF provides a framework to interpret high-dimensional features as a coordinated set of factors, it is widely applied in statistical approaches to find modular structure in high-dimensional profiles and in mechanistic interpretability studies of neural activations. 

Interestingly, an NMF block can be seamlessly integrated into deep neural networks to enhance interpretability while still benefiting from the effectiveness of automatic backpropagation in neural networks. In this blog post, I take a closer look at the technical details underlying gradient calculation through the NMF block when incorporated into a neural network.

---

### Nonnegative matrix factorization

Previous blog post have explained the multiplicative update rule for NMF, which particularly solves the following NP-hard problem:

$$(U,W)=\arg\min_{U\geq0,W\geq0}||A-UW^T||_F^2$$

This problem is not convex w.r.t. the input pair $(U,W)$, but fixing the value of one of the two factors and optimizing the other—makes the NMF problem into a pair of convex NNLS problems. We call it an alternating NNLS problems, and its convexity ensures that alternating minimization eventually leads to a local minimum. Here, we first discuss the technical details in solving NMF optimization problem with alternating direction method of multipliers(ADMM).

The standard practice of ADMM in integrating nonnegativity constraints to optimization objective is introducing an auxiliary variable $\tilde{U}, \tilde{W}$ as follows:

$$\min_{U,\tilde{U},W,\tilde{W}}\frac{1}{2}||A-\tilde{U}\tilde{W}^T||^2_F+\delta(U)+\delta(W), \\ s.t. \tilde{U}=U,\tilde{W}=W, \delta{(H)}=0 \text{ if } H\geq 0, +\infty \text{ o.w. }$$

Introducing $\tilde{U}, \tilde{W}$ may seem redundant, but it separates the

(1) unconstrained optimization from the 

(2) constraints ($\delta(\cdot)$) applied to $U,W$.

Note that $\tilde{U},U$ and $\tilde{W},W$ differ during optimization but converge to equality at the limit. 
During optimization, dual variables $\bar{U},\bar{W}$ balance the objectives of (1) and (2). 

Following standard ADMM practice, we create an *augmented Lagrangian* incorporating these constraints:

$$\mathcal{L}(A,U,W,\tilde{U},\tilde{W},\bar{U},\bar{W})=$$

$$\frac{1}{2}||A-\tilde{U}\tilde{W}^T||_F^2 + \delta(U)+ \delta(W)$$

$$+\bar{U}^T(\tilde{U}-U)+\bar{W}^T(\tilde{W}-W)$$

$$+\frac{\rho}{2}(||\tilde{U}-U||_2^2+||\tilde{W}-W||_2^2)$$

We solve this Lagrangian by decomposing it into a sequence of convex problems. 

ADMM iterates over the $(U, \tilde{U}, \bar{U}), (W,\tilde{W},\bar{W})$ triplets as follows:

$$U_{t+1} = \arg\min_{U=\tilde{U}} \frac{1}{2}||A-\tilde{U}W^T_t||^2_F+\delta(U)+\frac{\rho}{2}||\tilde{U}-U||_2^2$$

$$W_{t+1} = \arg\min_{W=\tilde{W}} \frac{1}{2}||A-U_t\tilde{W}^T||^2_F+\delta(W)+\frac{\rho}{2}||\tilde{W}-W||_2^2$$

Each problem decomposes into three sub-problems solved by ADMM. Their simplicity and efficiency are detailed in [Fel et al., Appendix C.2](https://arxiv.org/pdf/2211.10154).
