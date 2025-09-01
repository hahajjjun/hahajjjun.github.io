---
layout: post
title:  "Why penalized regression with permutation test disturbs FDR control"
subtitle: ""
excerpt_separator: "<!--more-->"
date:	2025-09-01
comments: true
categories: ["Annotated BI"]
---
### Defining FDR in variable selection
---
Under linear model $y=X\beta+z, z \sim N(0, \sigma^2\mathbf{I})$, we consider controlling the FDR among all the selected variables. Formal definition of FDR, defined on a selected subset of variable $\hat{S}\subset \{1,...,p\}$ is:

$$FDR=\mathbb{E} [\frac{| \{j:\beta_j=0 \text{ and } j\in\hat{S}\} |}{max(|\{j:j\in\hat{S}\}|, 1)}]$$

Here, denominator indicates the size of the set comprising selected variables(avoiding zero) while nominator denotes the number of false discoveries among selected variables(i.e. coefficient is zero while selected). In hypothesis testing persepctive, we test for hypotheses $H_j: \beta_j=0$, therefore, rejecting $H_j$ mean that feature $j$ is selected via variable selection methods.
