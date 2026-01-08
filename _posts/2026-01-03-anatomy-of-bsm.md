---
layout: post
title:  "Anatomy of Biological Sequence Modelling"
subtitle: ""
excerpt_separator: "<!--more-->"
date:	2026-01-03
comments: true
categories: ["Annotated BI"]
---

### Overview
The ambitious journey to capture the long evolutionary history of biological sequences is reaching important landmarks; ESM3, Evo2, NTv3, and others; by implicitly learning the constraints underlying contemporary biological sequences.
A previous blog post emphasized the importance of negative examples, which are often under-represented due to selective pressure. However, modern approaches struggle even with observable biological sequences.
Evo2, a genomic sequence model trained on 9.3 trillion DNA base pairs from prokaryotic and eukaryotic genomes, is one of the open-sourced landmarks of this journey. It provides a valuable template considering tokenization, architecture design, zero-shot evaluation, probing and mechanistic interpretability analysis.
We therefore aim to explain key ingredients in biological sequence modeling using Evo2 as an example.

### Tokenization

| Evo 2 is trained using next-token-prediction on the byte-tokenized OpenGenome2 dataset.

Beyond this simple statement, there might be detailed considerations in choosing a tokenization approach.
We should first note that conventional tokenizers in NLP field or classical sequence models that include BPE(byte pair encoding) and k-mer tokenization are promising approaches.

However, we can question about the optimality of tokenizer and conjecture that the optimal tokenization strategy of biological sequences should somehow differ from those of natural languages.
One remarkable fact is that BPE(byte-pair encoding) tokenization is an information-theoretic approach which is relatively more flexible that k-mer tokenizer due to the choice of vocabulary size and the choice of training corpus.
Specifically, we can modify BPE tokenizer to perform tokenization in different granularity(by modifying vocabulary size) and encourage it to explicitly capture biologically informative chunk of nucleotides ([Zhou et al.](https://arxiv.org/pdf/2512.17126)).

Benchmark result from [Zhou et al.](https://arxiv.org/pdf/2512.17126) demonstrates that no single tokenizer consistently outperformed the others. This suggests that we can find an optimized version of tokenizer in a data-driven approach, which is recently realized by [Qiao et al.](https://arxiv.org/abs/2412.13716) and [Anonymous authors](https://openreview.net/pdf/0af9dfc7c2c6f83b726b55f01c5f0fb02aedb335.pdf) (DNAChunker).

We’d like to take a deeper look at learnable tokenizers exemplified in these appraoches.
The motivation of introducing these adaptive appraoches are usually reasonable; since conventional approaches feel that they rely on heuristics.
However, Evo2 has showed byte-tokenizer was sufficient to achieve long-range contextual reasoning (>1M base pairs) with proper considerations in sequence modelling (StripeHyena2).
No matter how learnable tokens are produced, scientists will now expect that these optimized tokens might possess some important biological properties and perform well in long-range reasoning tasks.
In this context, modifying BPE with proper biological considerations could be a lightweight alternative with enhanced transparency.

### Sequence modelling

| Evo 2 uses StripedHyena 2, the first multi-hybrid architecture based on input-dependent convolutions. We roughly demonstrate step-by-step correspondence of convolutions to SSM(state space model), and SSM to self-attention mechanism. 

**Correspondence of convolutions to SSM**

A canonical linear SSM is decribed by the first-order difference equation:

$$ x_{t+1} = Ax_t + Bu_t $$
$$ y_t = Cx_t + Du_t $$

Recurrently solving the system of equation yields a convolution representation of $y_t$:

$$ y_t = \sum_{n=0}^t (CA^{t-n}B+D\delta_{t-n})u_n $$
$$ (\text{Note } (f*g)[t] = \sum f[n]g[t-n]) \text{ for convolution operator } \* )$$

For this system, we often specify the filter $h$ with respect to the SSM as:

$$ h_t = CA^tB+D\delta_t (\text{ for } t \geq 0)$$

**Correspondence SSM to self-attention**

General self-attention is a map which performs:

$$ y_i = \frac{\phi(Q_i)^T\sum_{j=1}^i \phi(K_j)V_j^T}{\phi(Q_i)^T\sum_{j=1}^i \phi(K_j)} - (*)$$

where $Q_i, K_i, V_i$ are query, key, value projections of input $u$ and $\phi(\cdot)$ is the nonlinear function.

This representation is often framed as applying attention vector $A=\text{softmax}(QK^T)$ to the value $V$, but we emphasize the decomposed version since we can efficienty calculate the result without explicitly constructing the matrix $A$ by sequentially calculating matrix muliplication with $\phi(Q), \phi(K)$.

By substituting $\sum_{j=1}^i\phi(K_j)V_j$ to $S_i$ and choose $\phi(\cdot)$ s.t. $\phi(Q_i)^T\sum_{j=1}^i\phi(K_j)=1$, we can notice that (*) equation can be alternatively represented with the language of SSM:

$$ S_i = S_{i-1}+\phi(K_i)V_i^T $$
$$ y_i = \phi(Q_i)^TS_i $$

We therefore can represent self-attention map as a SSM, and SSM can be viewed as a convolution operation.
Since SSM can be computationally empowered by FlashConv algorithm, we might be able to approximate self-attention in a computationally efficient way.

Hyena is alternatively applying convolutions in the time and then the frequency domain (or alternatively applying element-wise products in the time and frequency domain).
One potential explanation for the effectiveness of this procedure is that the convolution in the time domain (elementwise multiplication in the frequency domain) increases the
memory length, allowing for a broader context to be taken into account. 

Computing the convolution can be performed by element-wise multiplication in the spectral domain.
