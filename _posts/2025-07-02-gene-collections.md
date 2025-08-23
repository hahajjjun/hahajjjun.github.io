---
layout: post
title: "Collection of genes: A sc-linker case study"
subtitle: ""
excerpt_separator: "<!--more-->"
date:	2025-07-02
comments: true
categories: ["Annotated BI"]
---

---
### Introduction
---

With the advent of **single cell genomics**, we now have access to high-resolution, genome-wide transcriptomic measurements at the resolution of single cells. This unprecedented level of detail allows us to explore how genes are expressed across diverse **cell types, states, and molecular contexts**. 

One key objective in single-cell RNA sequencing (scRNA-seq) analysis is to discover a collection of genes that are jointly expressed across diverse cell types and molecular contexts. We often call it as **gene modules**, sets of genes that exhibit coordianted expression patterns.

To assess the biological relevance of these gene modules, researchers commonly perform **ontology**$\dagger$ **enrichment analysis**, to support the functional coherence of collected genes. 

$\dagger$ Notable advancement in gene ontology; from GO terms, molecular signatures to large-scale model-based embeddings - I'll discuss these topics later.

---
### Gene Modules from Pairwise Gene Expression
---

We naturally expect functional relevance across gene collections inferred from gene expression patterns, and one key principle shared by diverse module detection algorithms is the measurement of **pairwise relationships**. Common approaches in gene module detection, principal component analysis(PCA) and nonnegative matrix factorization(NMF), both leverage this pricniple, interpreting gene module as a linear combination of individual gene instances. 
- In PCA, For normalized gene-feature matrix $X$, we compute $X^TX$; which represents gene-gene graph where edge weight corresponds to the inner product of two feature vectors.
- NMF(nonnegative matrix factorization) has equivalence with K-means clustering on the bipartite graph(which nodes correspond to both cells and genes), applying relaxation by allowing soft assignments to the K clusters[1].
- Other non-linear algorithm, Hotspot[2], explicitly computes local autocorrelation on the defined neighborhood graph for all selected feature pairs.

Despite differences in methodology, these approaches share a conceptual foundation: **modeling gene-gene relationships** as a graph structure. This observation naturally leads to a question: **how useful are the detected gene modules in a biological context?** An elegant framework addressing this question is `sc-linker`[3]. This method identifies gene modules **contrastively**—focusing on cell-type-specific patterns—and then links these modules to **complex trait associations** using **GWAS summary** statistics. By doing so, `sc-linker` not only discovers expression-driven modules but also prioritizes those that are enriched for trait heritability, effectively nominating them as functional categories relevant to disease biology.

---
### A case of `sc-linker`
---
`sc-linker` define gene module $M$ as a linear combination of gene $x_i$:
$$P=\sum_{i=1}^N w_ix_i$$
where $x_i$ is the expression of gene $i$ and $w_i$ is the corresponding weight. Definition of gene module is roughly categorized into three types: 
1. $M_{cell}$(cell-type specific): $$M_{cell}=\sum w_ix_i \text{ where } w_i=\sigma(\chi_i) \text{ for } \chi_i=-2log(P_i)~\chi_2^2$$. Here, $P_i$ is a p-value after DE test comparing specific cell type $C$ to all others, and $\sigma(\cdot)$ is a min-max scaling function.
  
2. $M_{dis}$(disease dependent): Defined similarly to $M_{cell}$, but the p-values are derivd from disease vs. healthy comparisons.

3. $M_{proc}$(cellular process): Obatined using contrastive NMF, which lears shared and condition-specific components from healthy and diseased scRNA-seq data.

---
**Contrastive NMF for Module identification**:

Given two scRNA-seq feature matrices:
- $H_{P\times N_1}$ for healthy samples
- $D_{P\times N_2}$ for diseased samples
  
Conventional NMF decompose each matrix into:

$$H_{P\times N_1}\approx [L^{CH}L^{UH}]F^H, D_{P\times N_2}\approx [L^{CD}L^{UD}]F^D$$

,where $L^H=[L^{CH} L^{UH}], L^D=[L^{CD}L^{UD}]$ contain **shared**($L^{CH}, L^{CD}$) component and **unique** components ($$L^{UH}, L^{UD}$$).

In the **contrastive setting**, we enforce similarity between shared components by introducing a penalty term $\Vert L^{CH}-L^{CD}\Vert $. This term is added to conventional NMF obejctive thus it leads to the minimzation of objective $Q$:

$$Q=\frac{1}{2} \Vert{H-L^HF^H}\Vert _F^2+\frac{1}{2}\Vert{D-L^DF^D}\Vert _F^2+\frac{\mu}{2}(\Vert{L^H} \Vert _F^2+\Vert{L^D}\Vert _F^2)+\frac{\gamma}{2}(\Vert{L^{CH}-L^{CD}}\Vert _F^2)$$

Compuptation of gradient $\nabla Q(L^H), \nabla Q(L^D), \nabla Q(F^H), \nabla Q(F^D)$ yields a multiplicative update rule, derived from splitting the gradient into positive and negative components[4]:

$$\nabla Q(L)=Q_+ - Q_-, L \leftarrow L \circ \frac{Q_-}{Q_+}$$

This encourages shared components to capture common structure while allowing disease-specific modules to emerge.

---
**Linking Gene Modules to GWAS via s-LDSC**
Once modules are defined, `sc-linker` treats each module as a functional category and uses **stratified LDSC** (s-LDSC)[5] to quantify its contribution to trait heritability.

It is a long journey to introduce **s-LDSC** from scratch, but some key concepts include:
- regressing GWAS summary statistics(chi-square statistic) with LD score(sum of LD $r^2$ values between that SNP and all others in the region) yields a estimate of hearitability and confounding factors such as population structure.
- s-LDSC extends LDSC into the functional category of SNPs
- in `sc-linker` study, genes comprising the module are mapped to SNPs via enhancer-gene mapping(Roadmap-ABC)[6-10] strategy thus translates gene functional categories to a collection of mapped SNPs.

These consecutive steps in identifying gene programs and linking with GWAS summary statistics, provide a **foundational framework** in **identifying and applying a collection of genes to interpret complex, context-dependent hierarchy across variant-enhancer-gene-trait**.

### Reference
[1] Ding, C., Li, T., & Peng, W. (2008). On the equivalence between non-negative matrix factorization and probabilistic latent semantic indexing. Computational Statistics & Data Analysis, 52(8), 3913-3927.

[2] DeTomaso, D., & Yosef, N. (2021). Hotspot identifies informative gene modules across modalities of single-cell genomics. Cell systems, 12(5), 446-456.

[3] Jagadeesh, K. A., Dey, K. K., Montoro, D. T., Mohan, R., Gazal, S., Engreitz, J. M., ... & Regev, A. (2022). Identifying disease-critical cell types and cellular processes by integrating single-cell RNA-sequencing and human genetics. Nature genetics, 54(10), 1479-1492.

[4] Lee, Daniel, and H. Sebastian Seung. "Algorithms for non-negative matrix factorization." Advances in neural information processing systems 13 (2000).

[5] Finucane, Hilary K., et al. "Partitioning heritability by functional annotation using genome-wide association summary statistics." Nature genetics 47.11 (2015): 1228-1235.

[6] Ernst, J., Kheradpour, P., Mikkelsen, T. S., Shoresh, N., Ward, L. D., Epstein, C. B., ... & Bernstein, B. E. (2011). Mapping and analysis of chromatin state dynamics in nine human cell types. Nature, 473(7345), 43-49.

[7] Kundaje, A., Meuleman, W., Ernst, J., Bilenky, M., Yen, A., Kheradpour, P., ... & Roadmap Epigenomics Consortium. (2015). Integrative analysis of 111 reference human epigenomes. Nature, 518(7539), 317.

[8] Liu, Y., Sarkar, A., Kheradpour, P., Ernst, J., & Kellis, M. (2017). Evidence of reduced recombination rate in human regulatory domains. Genome biology, 18(1), 193.

[9] Fulco, C. P., Nasser, J., Jones, T. R., Munson, G., Bergman, D. T., Subramanian, V., ... & Engreitz, J. M. (2019). Activity-by-contact model of enhancer–promoter regulation from thousands of CRISPR perturbations. Nature genetics, 51(12), 1664-1669.

[10] Nasser, Joseph, et al. "Genome-wide enhancer maps link risk variants to disease genes." Nature 593.7858 (2021): 238-243.
