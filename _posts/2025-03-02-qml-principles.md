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

Inspired by the recent review of [ref], I'd like to share my perspective on understanding diverse GRN inference methods under a unified theoretical framework.