---
layout: post
title:  "Intrinsic-hoc methods"
subtitle: ""
excerpt_separator: "<!--more-->"
date:	2026-08-24
comments: true
categories: ["Research"]
---

### Intrinsic-hoc
The rapid advancement of deep learning has changed the practice of computational biology.
Large-scale sequencing, imaging, and perturbation experiments still matter, while increasingly expressive models can learn complex relationships from these measurements.
In many applications, one typical strategy is to provide a model with sufficiently large datasets and allow it to learn a mapping between i/o with relatively few assumptions about the underlying mechanism.

This data-driven approach is useful when the relevant structure is unknown or difficult to formulate explicitly.
However, biological modeling differs from deep learning models because biological systems are governed by physical laws, biochemical mechanisms, and experimentally established relationships.
These forms of prior knowledge are not merely useful for interpreting a model after training. 
They can determine how a model should be constructed in the first place.

The concept of _intrinsic-hoc_ approaches, discussed in [recent correspondence](https://www.nature.com/articles/s41587-025-02852-0), provides a useful framework for considering this distinction.
The essential idea is that domain knowledge could be incorporated intrinsically into the modeling procedure rather than being applied only before or after prediction.
In this sense, intrinsic-hoc is not a particular architecture. It is a way of organizing the relationship between prior knowledge, statistical learning, and prediction.

The distinction among _ad hoc, post hoc, and intrinsic-hoc_ approaches concerns where problem-specific knowledge enters a computational method.
An ad hoc method is constructed specifically for a particular problem and may rely on assumptions or procedures tailored to that application. A post hoc approach separates prediction from subsequent interpretation or analysis.
For example, a neural network may first be trained to predict a biological phenotype, after which an interpretation method is applied to determine which sequence features appear to influence the prediction.

An intrinsic-hoc approach differs in that relevant domain knowledge is incorporated into the model itself. The model is therefore not simply a generic predictor followed by a biological interpretation. Instead, the known structure of the biological problem constrains the relationship that the model learns.

This distinction is important because a predictive model can reproduce an empirical relationship without representing the mechanism responsible for that relationship.
If biological knowledge can specify part of the mechanism, there is a reason to incorporate that knowledge before asking a statistical model to learn the remaining unknown components.

### Anecdote of predicting prime-editing outcomes
The recent work on prime-editing efficiency prediction provides a useful example ([OptiPrime](https://www.nature.com/articles/s41587-026-03261-7)).

A sequence-based predictor can treat the relationship between pegRNA sequence and editing efficiency as a statistical mapping. 
Given sufficiently large training datasets, a neural network can learn sequence representations and associate them with measured editing outcomes (e.g., [DeepPrime](https://www.cell.com/cell/fulltext/S0092-8674(23)00331-8)).

The OptiPrime approach takes a different modeling perspective.
Rather than representing prime editing only as a direct mapping from sequence to final efficiency, it incorporates a mechanistic description of the editing process.
Machine-learning components estimate quantities associated with individual steps, while these quantities are subsequently combined according to the proposed kinetic structure of the editing process.
The significance of this approach is not simply that the resulting model is more interpretable.

More importantly, the mechanistic assumptions constrain the form of the prediction problem. 
The model is not required to infer the entire relationship between sequence and editing efficiency independently from observations.
Instead, it learns parameters within a representation motivated by the known biology of prime editing.

This illustrates an important characteristic of intrinsic-hoc.
Prior knowledge changes the learning problem itself.

The relevant comparison is therefore not necessarily between a mechanistic model and a neural network. 
A more useful comparison is between a model in which the neural network is asked to learn the entire input-output relationship and one in which the neural network is used to estimate components of a biologically structured model.
The latter can be advantageous when the proposed biological structure is sufficiently accurate and when the corresponding parameters can be estimated reliably from available data.

### Sequence modeling and physical constraints
[SPARXS](https://www.science.org/doi/10.1126/science.adn5968) provides a different example of a similar principle.
In this work, sequence-dependent molecular behavior is investigated through large-scale measurements together with thermodynamic and kinetic modeling.
Relationships based on physical principles, including Arrhenius-type kinetics, provide a framework for describing how sequence changes influence measurable molecular behavior.

This example is important because it demonstrates that a high-dimensional input space does not necessarily imply that the appropriate predictive model must be highly complex.
A DNA sequence can contain a very large number of possible configurations.

However, if sequence variation influences the measured phenotype primarily through a smaller set of physically meaningful quantities, the effective modeling problem may be considerably lower-dimensional than the original sequence space suggests.
In such a situation, a relatively simple statistical model can be sufficient once the relevant physical representation has been identified.

This should not be interpreted as evidence that linear models generally replace deep learning for sequence modeling. Nor does it imply that sequence complexity is unimportant. 
Rather, it illustrates a more specific point: the complexity of the observations and the complexity of the underlying relationship are not necessarily the same.

A deep neural network can learn a complex mapping directly from raw observations.
A mechanistically informed model instead attempts to identify the intermediate quantities that make the mapping simpler.

### Intrinsic-hoc as an inductive bias
From a machine-learning perspective, intrinsic-hoc can be viewed as a form of domain-specific inductive bias.
Every predictive model has an inductive bias. Neural-network architecture, regularization, training data, loss functions, and optimization procedures all constrain the set of functions that can be learned effectively.
Intrinsic-hoc makes an additional source of inductive bias explicit: knowledge about the biological system itself.

This can be useful when the knowledge is sufficiently well established. 
A physical conservation law, a kinetic relationship, or a known sequence-to-function mechanism can exclude classes of models that would otherwise fit the observed data but contradict established principles.

However, this also introduces a limitation. 
A mechanistic assumption that is incomplete or incorrect can constrain the model in the wrong direction.
The advantage of intrinsic-hoc therefore depends on the quality of the prior knowledge being incorporated.

This point is important because intrinsic-hoc should not be regarded as automatically superior to data-driven modeling.
It represents a different allocation of assumptions between the model and the data.

When biological knowledge is weak, uncertain, or incomplete, a flexible data-driven model may be preferable.
When the relevant biological structure is well established, explicitly incorporating it may reduce the burden placed on statistical learning.
