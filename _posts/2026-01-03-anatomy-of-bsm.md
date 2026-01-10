---
layout: post
title:  "Anatomy of Biological Sequence Modeling"
subtitle: ""
excerpt_separator: "<!--more-->"
date:	2026-01-03
comments: true
categories: ["Annotated BI"]
---

### Overview
The ambitious journey to capture the long evolutionary history of biological sequences is reaching important landmarks—ESM3, Evo2, NTv3, and others—by implicitly learning the constraints underlying contemporary biological sequences.

A previous blog post emphasized the importance of negative examples, which are often under-represented due to selective pressure. However, modern approaches struggle even with observable biological sequences.

Evo2, a genomic sequence model trained on 9.3 trillion DNA base pairs from prokaryotic and eukaryotic genomes, is one of the open-sourced landmarks of this journey. It provides a valuable template considering tokenization, architecture design, zero-shot evaluation, probing and mechanistic interpretability analysis.

We therefore aim to explain key ingredients in biological sequence modeling using Evo2 as an example.

### Tokenization

| Evo 2 is trained using next-token-prediction on the byte-tokenized OpenGenome2 dataset.

Beyond this simple statement, there might be detailed considerations in choosing a tokenization approach.
We should first note that conventional tokenizers in NLP field, or classical sequence models that include BPE(byte pair encoding) and k-mer tokenization, are promising approaches.

However, we can question the optimality of tokenizer and conjecture that the optimal tokenization strategy of biological sequences should somehow differ from those of natural languages.
One remarkable fact is that BPE(byte-pair encoding) tokenization is an information-theoretic approach which is relatively more flexible that k-mer tokenization due to the choice of vocabulary size and the choice of training corpus.
Specifically, we can modify BPE tokenizer to perform tokenization in different granularity(by modifying vocabulary size) and encourage it to explicitly capture biologically informative chunks of nucleotides ([Zhou et al.](https://arxiv.org/pdf/2512.17126)).

Benchmark results from [Zhou et al.](https://arxiv.org/pdf/2512.17126) demonstrate that no single tokenizer consistently outperformed the others. This suggests that we can find an optimized version of tokenizer in a data-driven approach, which is recently realized by [Qiao et al.](https://arxiv.org/abs/2412.13716) and [Anonymous authors](https://openreview.net/pdf/0af9dfc7c2c6f83b726b55f01c5f0fb02aedb335.pdf) (DNAChunker).

The motivation for introducing these adaptive approaches is usually reasonable; since conventional approaches feel that they rely on heuristics.
However, Evo2 has shown that a byte-tokenizer is sufficient to achieve long-range contextual reasoning (>1M base pairs) with proper considerations in sequence modeling (StripeHyena2).
No matter how learnable tokens are produced, scientists will now expect that these optimized tokens might possess some important biological properties and perform well in long-range reasoning tasks.
In this context, modifying BPE with proper biological considerations could be a lightweight alternative with enhanced transparency.

### Sequence modeling

| Evo 2 uses StripedHyena 2, the first multi-hybrid architecture based on input-dependent convolutions. 

We roughly demonstrate step-by-step correspondence of convolutions to SSM(state space model), and SSM to self-attention mechanism. 

**Correspondence of convolutions to SSM**

A canonical linear SSM is described by the first-order difference equations:

$$ x_{t+1} = Ax_t + Bu_t $$
$$ y_t = Cx_t + Du_t $$

Recurrently solving the systems of equation yields a convolution representation of $y_t$:

$$ y_t = \sum_{n=0}^t (CA^{t-n}B+D\delta_{t-n})u_n $$
$$ (\text{Note } (f*g)[t] = \sum f[n]g[t-n]) \text{ for convolution operator } * )$$

For this system, we often specify the filter $h$ with respect to the SSM as:

$$ h_t = CA^tB+D\delta_t (\text{ for } t \geq 0)$$

**Correspondence of SSM to self-attention**

General self-attention is a map which performs:

$$ y_i = \frac{\phi(Q_i)^T\sum_{j=1}^i \phi(K_j)V_j^T}{\phi(Q_i)^T\sum_{j=1}^i \phi(K_j)} - (*)$$

where $Q_i, K_i, V_i$ are query, key, value projections of input $u$ and $\phi(\cdot)$ is the nonlinear function.

This representation is often framed as applying attention vector $A=\text{softmax}(QK^T)$ to the value $V$, but we emphasize the decomposed version since we can efficiently calculate the result without explicitly constructing the matrix $A$, by sequentially calculating matrix muliplication with $\phi(Q), \phi(K)$.

By substituting $\sum_{j=1}^i\phi(K_j)V_j$ with $S_i$ and choosing $\phi(\cdot)$ s.t. $\phi(Q_i)^T\sum_{j=1}^i\phi(K_j)=1$, we can notice that $(*)$ equation can be alternatively represented with the language of SSM:

$$ S_i = S_{i-1}+\phi(K_i)V_i^T $$
$$ y_i = \phi(Q_i)^TS_i $$

We therefore represent self-attention map as a SSM, and SSM is viewed as a convolution operation.
Since SSM can be computationally empowered by FlashConv algorithm, we might be able to approximate self-attention in a computationally efficient way.

Recall the fact that convolution can be performed by element-wise multiplication in the spectral domain, then Hyena can be viewed as an alternating application of convolutions in the time and frequency domain.
[Poli et al.](https://arxiv.org/abs/2302.10866) abstractly explains the effectiveness of this procedure as:
- **Convolution** in the **time** domain increases the **memory length**, allowing for a broader context to be taken into account
- **Element-wise multiplication** in the **time** domain allows for more fine-grained **selection** of specific frequency components of the signal

Now we can define Hyena operator as an extension of H3 mechanism:

$$ \text{H3}(q,k,v) = A(q,k)v = D_q{S_\psi}D_k{S_\phi}$$
$$ \text{ where } S_\psi, S_\phi \text{ are Toeplitz matrices of SSM with filters } \psi, \phi$$
$$ \text{Hyena}(q,k,v) = A(q,k)v = D_x^N{S_x^N}...D_x^1{S_x^1}v$$
$$ :\text{ extension of H3 with arbitrary no. of projections }$$

StripedHyena model was applied in Evo([Nguyen et al.](https://www.science.org/doi/10.1126/science.ado9336)), which is composed of Hyena operator and rotary attention.
Extending StripedHyena, StripedHyena2 model applies multiple Hyena operators with inner filters designed with different principles: SE(short explicit), MR(medium regularized), and LI(long implicit).
7B scale ablation experiments demonstrated that SE-MR-LI architecture surpassed alternative choices of design, where Evo2 utilizes this architecture for training model with 40B parameters.

### Evaluation and Benchmarks
| Perplexity, Long-context recall, Mutational effects, Variant pathogenicity

We have previously discussed what properties a biological sequence model might possess in an earlier [blog post](https://hahajjjun.github.io/annotated%20bi/2025/11/22/nlp.html).
We were able to distinguish objectives that are achievable through fine-tuning and zero-shot inference(e.g. probing).


In addition to these examples, Evo2 was also evaluated with novel benchmarks. 

Briefly, the perplexity metric traditionally used in the NLP field was used for determining model architecture and demonstrating the benefit of model scale/context length.
Additionally, Evo2 utilizes a needle-in-a-haystack benchmark by synthetically inserting 100bp needles in a long sequence of haystack($2^9 - 2^{20}$ base pairs) to evaluate the retrieval strength. Formally, we can use the Euclidean magnitude of the categorical jacobian([Zhang et al.](https://www.pnas.org/doi/10.1073/pnas.2406285121#eqn5)), which measures the effects on model predictions for the query when we mutate the needle sequence.

For inferring variant effects, we can explicitly mutate the sequence and observe the difference of likelihood by using Evo2 as an oracle. Calculated likelihood difference itself can be used for inference or to train a simple classifier in a supervised fashion, aiming at **variant fitness prediction, exon classification, gene/lncRNA essentiality prediction, and variant pathogenicity prediction**.

### Mechanistic Interpretability

| Batch-TopK SAE was trained on Evo 2’s representations at layer 26

Over the past two decades, annotation and cataloging of genomic sequences have continuously advanced, with these results primarily obtained through observational studies. Generally speaking, if a comprehensive biological sequence model exists, we can leverage the catalog to annotate features and then utilize the model's rich extrapolatability to perform massive in silico evaluation and prioritization across the entire sequence.

SAE(sparse autoencoder) is a dictionary learning approach to decompose polysemantic neurons into features that activate with high specificity and sensitivity for the hypothesized context.
We aspire that such dictionary learning approaches have a sufficient capacity to decompose intermingled concepts into monosemantic ones at the level of human cognition, so we apply a sparsity regularization to make SAE features sparse(note that without sparsity regularization, if latent dimension is larger than the input, the identity mapping will be the trivial solution under reconstruction error term as an objective).

Compared to vanilla SAE, TopK-SAE (which applies L0 regularization instead of L1) is superior in terms of reducing dead latents([Gao et al.](https://arxiv.org/abs/2406.04093)), so Evo2 chose BatchTopK-SAE. Another noticable choice is that Evo2 selected layer 26(probably we could choose another layer, e.g. penultimate) to apply SAE and inspect the features, which is also reasonable considering that late layers(penultimate was optimal in this case)([Fel et al.](https://arxiv.org/abs/2306.07304v2)) tend to demonstrate higher concept importance evaluated with diverse ablation studies.

Evo2 mapped annotation to features by collecting and comparing feature activations for annotated sequence elements associated with a specific biological question of interest.
As expected, we can observe a hierarchy of features: a lot of feature variation account for high-level categories including species(semantic) to fine-grained associations such as protein secondary structure(structure), and organization of genome(ORF, intergenic). Furthermore, applying approaches used in the mechanistic interpretability field to design sequences that maximally activate specific features, discovering circuits, and automating these processes is also a promising direction.

### Generative Design

| Generation of genome across the domain of life and guiding sequence towards target epigenomic signal

The key idea of generative design is using existing genome sequences as a prompt. Evo2 demonstrates its generative ability across mitochondrial, prokaryotic, and eukaryotic genome. One remarkable result is that the generation of these sequences is achieved in a autoregressive way, since I've expected that there might be an external guidance to achieve such results. 

Evo2 also demonstrates the use of model by plugging in reward signals to guide generated sequence towards reward signal. It resembles a traditional approach introduced in NLP fields: beam search. For generated sequences with Evo2, [Borzoi](https://www.nature.com/articles/s41588-024-02053-6) and [Enformer](https://www.nature.com/articles/s41592-021-01252-x) are used to evaluate generated sequences' epigenomic features and beam search was applied to accept or reject the generated sequence. Accepted sequences are then appended to the prompt sequence for generation, and iteratively applying this procedure can generate sequences that possess desired epigenomic signal. 

We've already discussed some ambitious directions in applying biological selection systems as an endogenous oracle of providing reward signal, so I recommend to see the previouis [blog post](https://hahajjjun.github.io/annotated%20bi/2025/09/06/fm-sb.html).

### Conclusion

Biological sequence modeling is transitioning from heuristic-driven representations toward principled, scalable, and mechanistically interpretable systems. Through Evo2, we observe that advances in architecture design, long-context modeling, evaluation methodology, and interpretability can collectively unlock biological reasoning without relying on massive fine-tuning.

While adaptive tokenization and biological informed representations remain an active area of exploration, Evo2 demonstrates that carefully designed sequence modeling alone can recover long-range dependencies across entire genomes.

As these models become increasingly interpretable and generative, we can consider a new paradigm: one in which biological hypotheses can be generated, evaluated, and refined in silico at unprecedended scale. By this anecdotal example of Evo2, we were able to look how mechanistic interpretability, and biologically grounded evaluation can transform sequence modeling into a predictive and generative engine for biology.
