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

Despite impressive advances, the field of GRN inference is fragmented. 
Each method—whether based on regression, probabilistic graphical models, or graph neural networks—proposes its own formulation. 
As researchers, we often face the dilemma of choosing between methods, without a clear framework for understanding their shared foundations.

Inspired by the recent review of [1], I'd like to share one perspective on understanding diverse GRN inference methods with external information under a unified theoretical framework. 

Gene regulatory network(GRN), is defined as a directed, weighted graph with each node represents genes that could be a regulator or target. Here, let’s describe a GRN with regulatory influence matrix $\Theta \in \mathbb{R}^{p \times p}$, where $p$ is the number of nodes(genes) composing the GRN. Some works limit transcription factor(TF) as active regulator among genome-wide candidates, so $\Theta \in \mathbb{R}^{q \times p}$ for this case where $q$ is the number of selected TFs to compose GRN.

Coarsely we can consider two categories of GRN inference algorithms; </br>
1) directly inferring $\Theta$ from given scRNA-seq data $X$ with regularization term to incorporate external knowledge $\Omega$ <\br>
2) understanding $\Theta$ as a part of generative process that supports the realization of scRNA-seq data $X$ and apply soft or hard constraints with external data $Y$.

For instance if the first category, we consider CellOracle[2] and NetREX-CF[3]. Their loss function can be represented in a unified framework;
$$\hat{\Theta}=argmin_\Theta[{\mathcal{L}_{data}(X,\Theta)+\mathcal{L}_{reg}(\Theta,P)}]$$

The first term stands for the data fit loss, and the remainder stands for the regularization term with respect to the prior information $P$.

---
**Proposition 1.** CellOracle regression model is an instance of unified framework.
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


**Proposition 2.** NetREX-CF regression model is an instance of unified framework.

---