---
layout: post
title:  "Unified perspective on GRN inference with prior knowledge"
subtitle: ""
excerpt_separator: "<!--more-->"
date:	2025-06-07
comments: true
categories: ["Annotated BI"]
---

Understanding how genes regulate each other is central to unraveling the complex circuitry of cellular life. 
Gene regulatory networks (GRNs) offer a blueprint of these intricate molecular conversations. 
With the advent of multiplexed, high-throughout, and multi-modal measurements, new computational models now seek to infer GRNs with unprecedented granularity.

### Motivation
Recently, several sporadic reports provide a clue about the shape of underlying topology of GRN; we can make theoretical hypotheses about the relationship between the amount of information about GRN which can be extracted from diverse modalities of observation. Which provides a more granular view of GRN? Gene expression patterns, mainly co-expression, from scRNA-seq data? Or the counterparts obtained from causal studies such as perturb-seq? Is there any underlying inequality that holds for the information from two types of observation?

What is currently understood; when gene expression was simulated via a stochastic process from GRNs, the amount of information from direct causal studies seemed to provide a higher resolution view of the underlying network structure than co-expression[1]. Additionally, another report show scRNA-seq atlas scale learning is sufficient to effectively extrapolate on genetic perturbation effects; especially using scGPT and GEARS embeddings[2]. These findings therefore suggest the feasibility of **pushing the limit** **approach** of analytical methods to infer GRN from scRNA-seq co-expression information.

Naturally, this raises the question of when we perform GRN inference based on scRNA-seq co-expression, whether perturb-seq can effectively aid inference(*very limited at this point, since genome-wide CRISPR KO single cell screens are known for one cell type*) compared to another type of priors(e.g. ATAC-seq or knowledge based databases) and whether it can be used as a universal prior despite limitations in cell type differences.

Unfortunately, the field of GRN inference is fragmented. Each method— based on regression, probabilistic graphical models, or graph neural networks—proposes its own formulation. 
As researchers, we often face the dilemma of choosing between methods, without a clear framework for understanding their shared foundations. As end users, we also need a holistic benchmark stratified by the types of external information to aid GRN inference; since some external modality could be critically powerful in GRN inference.

Inspired by the recent review of [3], I'd like to share one perspective on understanding diverse GRN inference methods with external information under a unified theoretical framework. 

---

### Unified perspective
Gene regulatory network(GRN), is defined as a directed, weighted graph with each node represents genes that could be a regulator or target. Here, let’s describe a GRN with regulatory influence matrix $\Theta \in \mathbb{R}^{p \times p}$, where $p$ is the number of nodes(genes) composing the GRN. Some works limit transcription factor(TF) as active regulator among genome-wide candidates, so $\Theta \in \mathbb{R}^{q \times p}$ for this case where $q$ is the number of selected TFs to compose GRN.

Coarsely we can consider two categories of GRN inference algorithms; <br/>
1) directly inferring $\Theta$ from given scRNA-seq data $X$ with regularization term to incorporate external knowledge $\Omega$ <br/>
2) understanding $\Theta$ as a part of generative process that supports the realization of scRNA-seq data $X$ and apply soft or hard constraints with external data $Y$. <br/>

For instance if the first category, we consider CellOracle[4] and NetREX-CF[5]. Their loss function can be represented in a unified framework;
$$\hat{\Theta}=argmin_\Theta[{\mathcal{L}_{data}(X,\Theta)+\mathcal{L}_{reg}(\Theta,P)}]$$

The first term stands for the data fit loss, and the remainder stands for the regularization term with respect to the prior information $P$.

---
**Proposition 1.** CellOracle regression model is an instance of unified framework. <br/>
*proof sketch.*
CellOracle regression model is based on LASSO regression, and candidate regulators are limited to the pre-defined set informed by the prior knowledge.

For gene $j$, let $S_j$ be the set of candidate regulators. For gene $i$ and $j$, introduce a prior-informed support mask $P_m=[p_{ij}]$ where $p_{ij}=1(i \in S_j) \text{ or } 0(i \notin S_j)$.

Consider a regression model
$$X_j=\sum_{i\neq j}\Theta_{ij}X_i+\epsilon_j, \text{ where } \epsilon_j \sim N(0,\sigma_j^2)$$
Then optimization objective for LASSO regression is
$$\mathcal{L_{data}} = \sum_{j=1}^p[{1\over 2\sigma_j^2}||X_j-\sum_{i\neq j}\Theta_{ij}X_i||_2^2+\kappa \sum_{i\neq j}|\Theta_{ij}|]$$
, and with prior regularization, 
Additionally, hard constraint on unavailable regulatory pairs on $\Theta$ yields
$$\mathcal{L}_{reg}=0\text{ if } \Theta_{ij}=0 \text{ for all } i,j \text{ with } m_{ij}=0, \text{ otherwise } \infty$$
so that $\Theta_{ij}=0$ is enforced outside $S_j$.
Thus, we get optimal $\Theta$ by solving $$\hat{\Theta}=argmin_\Theta[{\mathcal{L}_{data}(X,\Theta)+\mathcal{L}_{reg}(\Theta,P)}]$$


**Proposition 2.** NetREX-CF regression model is an instance of unified framework.<br/>
*proof sketch.*
Let $P^{(k)} \in \mathbb{R}^{q\times p}|_{k=1}^K$ be the prior, quantified knowledge of regulatory pair existence from $K$ sources(it can be diverse modalities).
The NetREX-CF model is based on NCA(network component analysis), which loss function can be described as:
$$\mathcal{L}_{data}=[||X-\Theta A||_2^2+\kappa||A||_2^2+\lambda||\Theta||_2^2+\eta||\Theta||]$$
For prior regularization perspective, the model jointly achieves prior regularization with the optimization of collaborative filtering(CF) objective by explicitly introducing a penalty matrix of CF $C=1+a•\sum_k P^{(k)}$ and prior mask $B=\mathbb{1}(\sum_kP^{(k)}>0)$. With respect to $\Theta$, these matrices are refined to $\Omega=\alpha_1C|\Theta|\odot B+\alpha_2|\Theta|\odot(1-B)+C$ and $M=|\Theta|+(1-|\Theta|)\odot B$.
Then prior regularization term is designed to optimize regulator latent matrix $U$ and gene latent matrix $V$, where loss function is:
$$\mathcal{L}_{reg}=\sum_{i,j}\Omega_{i,j}(M_{i,j}-u_i^Tv_j)^2$$

---
For the second category, here we consider probabilistic graphical model Symphony[4] and PMF-GRN[5] as an instance.
Under a bayesian perspective, given observation $X$ and prior $P$, Variational approach is used to approximate the distribution of latent GRN $\Theta$, thus enables estimation of statistics on the posterior distribution of $\Theta$. 

We can consider two types of external knowledge to provide soft and hard constraints; data itself($Y$, e.g. ATAC-seq) can be jointly incorporated to the generative process(soft), or existing knowledge(e.g. available TF-gene pairs) can be used as a parameter of prior distribution to guide the inference.

Despite differences in the generative process of scRNA-seq observation $X$, unified perspective of incorporating external data $Y$ or established knowledge as a prior $P$ can be described as:
$$P(X,Y|\Theta, P, \Omega_{x,pa}, \Omega_{x, ch}, \Omega_{y})$$
Here, we assume a hierarchical model with parameters $\Omega_{x}, \Omega_{y}$ to describe generative process of data $X, Y$, respectively. For simplicity, we consider a dependence relationship that satisfies $P(X,Y|\Theta, P, \Omega_{x}, \Omega_{y})=P(X|\Theta, P, \Omega_{x})•P(Y|\Theta,\Omega_y)$ and $\Theta$ is also dependent to $P, \Omega_{x}$.

Variational perspective yields ELBO as an objective function:
$$\mathcal{L}_{elbo}=\mathbb{E}_{q(\Theta,\Omega_x,P)}[\log p(X|\Theta,\Omega_x,P)]+\mathbb{E}_{q(\Theta,\Omega_y,P)}[\log p(Y|\Theta,\Omega_y,P)]+D_{KL}(q(\Theta,\Omega_x,\Omega_y,P)||p(\Theta,\Omega_x,\Omega_y,P))$$
Here, we can apply known regulatory relationship to design prior $p(P)$, for instance, directly incorporating knowledge to mean parameter of the prior distribution.

Two different algorithms, harmony and PMF-GRN differs in the data generative process and the choice of strategy for external knowledge injection but they both employ variational inference to optimize the objective function.

---

### The next step
We can consider a naive approach; regarding diverse types of prior information under unified objective function, and systematically assess the information gain obtained from diverse types of prior knowledge.
Some might help, while some might not. Another aspect is, how we could set up with a ground truth data via experiment. Currently, some methods are utilizing TF-gene interaction as a prior knowledge while some consider it for method evaluation.
How could we tackle such issues for fair benchmark? geneRNIB[8] shows a step forward and could be a solid foundation for improving GRN inference benchmark studies.