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
In this blog post, I will explain details and practical aspects underlying Evo2. 
At a glance, we notice that Evo2 provides a template for deep consideration of tokenization, architecture design, training objective, and dataset distribution. 
We therefore aim to introduce key concepts in biological sequence modelling, with Evo2 as an example.

### Tokenization
We can intuitively think that the optimal tokenization strategy of biological sequences should somehow differ from those of natural languages.
Despite conventional tokenizers in NLP field or classical sequence models that include BPE(byte pair encoding) and k-mer tokenization are promising approaches, there is a room for improvement in developing biologically informed tokenizers.

One remarkable fact is that BPE tokenization are relatively more flexible that k-mer tokenizers due to the choice of vocabulary size and the choice of training corpus.
We can modify BPE tokenizer to perform tokenization in different granularity(by modifying vocabulary size) and encourage it to explicitly capture biologically informative chunk of nucleotides ([Zhou et al.](https://arxiv.org/pdf/2512.17126)).

Benchmark result from [Zhou et al.](https://arxiv.org/pdf/2512.17126) demonstrates that no single tokenizer consistently outperformed the others. This suggests that we can find an optimized tokenizer in a data-driven approach, which is recently realized by [Qiao et al.](https://arxiv.org/abs/2412.13716) and [Anonymous authors](https://openreview.net/pdf/0af9dfc7c2c6f83b726b55f01c5f0fb02aedb335.pdf).
