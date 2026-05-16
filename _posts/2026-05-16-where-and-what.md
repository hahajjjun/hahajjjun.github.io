---
layout: post
title:  "Where and What for Large Volume Medical Images"
subtitle: ""
excerpt_separator: "<!--more-->"
date:	2026-05-16
comments: true
categories: ["Research"]
---

This is an article introducing my [research](https://www.nature.com/articles/s41598-026-51023-x) on a problem I’ve been pondered for a long time.

When I was studying anatomy in college, I found the “땡시[ddaeng-si]” type of exam particularly difficult. 
These were asking the name of a human anatomical structures and landmarks after looking at it for a very short amount of time. 

Since all surrounding structures that could serve as clues were covered, it was often difficult to even identify the approximate location based on just a tiny portion of observable structure.
At the time, I thought it would have been easier to deduce the answer if I could have known the approximate location of the presented structure.

This intuition became the starting point for my research.
For high-resolution pathology images or 3D medical images, artificial neural networks are often trained using small patches—smaller than the original image—to ensure efficient learning. 
Similar to the previous episode, patch-based learning poses a constraint of observing a narrower field of view compared to the entire image.

In this environment, it is important to consider the location from which the patch was sampled.
Generally, the morphology and signal intensity of medical images serve as critical information to identify organ boundaries.
The additional consideration of "location" represents a unique context that emerges in patch-based learning.
For example, we know that the heart is highly unlikely to appear in a patch sampled near the leg.

Therefore, my focus shifted to “where to sample the next patch” and “how to understand the anatomical context of the current patch’s location.”
In short, this boils down to “Where” and “What”, and this research is the result of our efforts to answer these questions.
Additionally, the method proposed in this process naturally possesses explainable properties.

In the era of large models and data, principles that are reemerging as a reflection on the efficiency of the learning process are catching my attention.
Among these, using the human reasoning process as an analytical framework and applying it to machine learning is a classic yet still interseting approach.
