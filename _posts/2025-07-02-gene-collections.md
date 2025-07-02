---
layout: post
title: "Collection of genes"
subtitle: ""
excerpt_separator: "<!--more-->"
date:	2025-07-02
comments: true
categories: ["Annotated BI"]
---


### Motivation
---
Global view on the pairwise relationships across gene functions, gene expressions and their regulatory pattern has been an important question of functional genomics. 

At the early era of gene expression measurements, simultaneous quantification of gene expression(e.g. microarray) enabled the computation of pairwise relationships of gene expression. 

After the advent of single cell genomics, we now get rich single cell resolution measurements across genome-wide transcripome thus naturally compute pairwise relationships across gene expression levels. 

Why should we focus on pairwise relationships? Let's consider PCA and NMF as representative instances of gene module detection algorithms. 

Taking a closer look at the mathematical motivation of these algorithms, they naturally compute pariwise relationships: PCA and NMF are algorithms that construct gene-gene graph$^*$ and gene-cell bipartite graph$^\dagger$ from the data, respectively.

---

$*$: For given normalized data X, PCA involves the computation of $X^TX$; which represents gene-gene graph where edge weight corresponds to the inner product of two gene expression vectors.
$\dagger$:  Ding et al. proves that NMF is equivalent to K-means clustering on the bipartite graph with relaxation of soft assignments to the K clusters.

---

A modern understanding about gene expression is based on the modular structure; we can group genes into modules, a collection of genes that could be obtained from grouping genes from their expression profiles. 

Here, we review diverse approaches to define collection of genes as modules. For an advanced topic, we cover the uncertainty quantification for detected modules and module detection from case-control studies. 

### Definitions of gene modules