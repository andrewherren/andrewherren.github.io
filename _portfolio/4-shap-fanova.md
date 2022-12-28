---
title: "The Relationship between SHAP and Functional ANOVA"
excerpt: "Implementations and simulation studies that detail the links between SHAP and the functional ANOVA, including possible modifications to SHAP"
collection: portfolio
---

This repository demonstrates many of the ideas discussed in [Statistical Aspects of SHAP: Functional ANOVA for Model Interpretation](https://arxiv.org/abs/2208.09970) (Herren and Hahn 2022). Specifically, it:

1. Implements approximations of the functional ANOVA terms for a variety of reference distributions
2. Computes Shapley values as a weighted linear combination of functional ANOVA terms
3. Implements a breadth-first screening procedure (first introduced in Hooker (2004)) for identifying low-order functional ANOVA terms and speeding up SHAP convergence

### References

Giles Hooker. Discovering Additive Structure in Black Box Functions. In *Proceedings of the tenth ACM SIGKDD international conference on Knowledge discovery and data mining*, pages 575–580, 2004.
