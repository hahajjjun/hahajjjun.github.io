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

After the advent of single cell genomics, we now get rich single cell resolution measurements across genome-wide transcripome. From scRNA-seq data, we usually expect to discover a collection of genes that are jointly expressed across diverse cell types and molecular contexts. Validity of detected modules are usually supported by the functional coherence of collected genes: which we usually do it by GO(gene ontology) enrichment analysis.

We naturally expect functional relevance across gene collections inferred from gene expression patterns, and one key principle shared by diverse module detection algorithms is the measurement of "pairwise relationships". In simplified statements, we can observe:

1. PCA(principled component analysis) involves the computation of $X^TX$; which represents gene-gene graph where edge weight corresponds to the inner product of two gene expression vectors.
2. NMF(nonnegative matrix factorization) has equivalence with K-means clustering on the bipartite graph(which nodes correspond to both cells and genes) with relaxation of allowing soft assignments to the K clusters(Ding et al.).
3. Hotspot algorithm explicitly computes local autocorrelation on the defined neighborhood graph for all selected feature pairs.

Based on this concept, we can make quantitative hypotheses about the relationships between pairwise gene expression patterns and their corresponding functions. For instance, we might be able to quantify how much extent gene co-expression might inform gene co-functionality. 

Historically, landscape view on the pairwise relationships across gene functions, gene expressions and their regulatory pattern has been an important question of functional genomics. At the early era of gene expression measurements, simultaneous quantification of gene expression(e.g. microarray) enabled the computation of pairwise relationships of gene expression. Co-regulation of gene pair was often quantified by the amount of shared transcription factor binding motifs(or transcription factor itself), and co-function of gene pair was probed by the knowledge databse: mainly represented by the GO term similarity.

### Modern quantitative studies on pairwise gene relationships
---
Intersetingly, we can understand several important results revealed from genome-wide CRISPR perturbation paired with single cell measurements, through the lens of conventional studies about gene co-expression, co-regulation, and co-function. First of all, here we demonstrate the key results identified from important benchmark studies on perturbation response prediction.

1. scGPT embedding linearly extrapolates perturbation response
2. knowledge base embedding informs perturbation response
3. diverse biological prior knowledge on pairwise relationship could be used to predict perturbation response: while orthogonal scCRISPR screen highly benefits prediction.

### Looking ahead
---
