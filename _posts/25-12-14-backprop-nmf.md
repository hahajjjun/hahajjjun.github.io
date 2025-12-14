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
This can be formalized through a universal framework called I-Con[ref]. Representation learning aligns the transition probability (i.e., conditional distribution) from a supervisory signal to those of representations specified by a learnable function. 

The core I-Con loss function is:

$$\mathcal{L}(\theta, \phi) =\int_{i\in \mathcal{X}} D_{KL}(p_\theta(\cdot|i)||q_\phi(\cdot|i))=\int_{i\in\mathcal{X}}\int_{j\in\mathcal{X}}p_\theta(j|i)\log\frac{p_\theta(j|i)}{q_\phi(j|i)}$$

Here, $p_\theta$ is a supervisory distribution, while $q_\phi$ is learned by capturing the structure of the desired representation. 
We can optimize both $\theta,\phi$ (e.g., X-Sample[sobal]), though most methods optimize only $\phi$. 
Many representation learning approaches—including t-SNE, SimCLR, and K-Means clustering—can be understood as instances of the I-Con framework.

Now let's circle back to factorization algorithms. 
These include PCA, K-Means clustering, and NMF, but here we focus on NMF as a representative factorization algorithm that learns latent factors as linear combinations of high-dimensional features and reconstructs the original features via linear projection. 
Note that NMF (nonnegative matrix factorization) is equivalent to K-means clustering on a bipartite graph (where nodes correspond to both samples and features), with relaxation that allows soft assignments to the K clusters[ref].

Since NMF provides a framework to interpret high-dimensional features as a coordinated set of factors, it is widely applied in statistical approaches to find modular structure in high-dimensional profiles and in mechanistic interpretability studies of neural activations. 

Interestingly, an NMF block can be seamlessly integrated into deep neural networks to enhance interpretability while still benefiting from the effectiveness of automatic backpropagation in neural networks[ref; craft]. In this blog post, I take a closer look at the technical details underlying gradient calculation through the NMF block when incorporated into a neural network.

---

### Nonnegative matrix factorization

Previous blog post have explained the multiplicative update rule for NMF, which particularly solves the following NP-hard problem:

$$(U,W)=\arg\min_{U\geq0,W\geq0}||A-UW^T||_F^2$$

This problem is not convex w.r.t. the input pair $(U,W)$, but when we fix the value of one of the two factors and optimizing the other makes the NMF problem into a pair of convex NNLS problems. We call it an alternating NNLS problems, and its convexity ensures that alternating minimization eventually leads to a local minimum. Here, we first discuss the technical details in solving NMF optimization problem with alternating direction method of multipliers(ADMM).

The standard practice of ADMM in integrating nonnegativity constraints to optimization objective is introducing an auximilariy variable $\tilde{U}, \tilde{W}$ as follows:

$$\min_{U,\tilde{U},W,\tilde{W}}\frac{1}{2}||A-\tilde{U}\tilde{W}^T||^2_F+\delta(U)+\delta(W), \\ s.t. \tilde{U}=U,\tilde{W}=W, \delta{(H)}=0 \text{ if } H\geq 0, +\infty \text{ o.w. }$$

This problem also fulfills the KKT conditions… (C.2. implicit differentiation)
