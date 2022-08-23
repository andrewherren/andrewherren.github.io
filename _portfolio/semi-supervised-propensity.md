---
title: "Semi-supervised Propensity Score Methods"
excerpt: "Simulation studies on the use of unlabeled data in causal inference"
collection: portfolio
---

[semi-supervised-propensity](https://github.com/andrewherren/semi-supervised-propensity) is a repo containing simulation studies underlying the paper "[Semi-supervised learning and the question of true versus estimated propensity scores](https://arxiv.org/abs/2009.06183)." Specifically, the code generates data from a complicated nonlinear model in which the probability of being assigned treatment is a function of the expected outcome. We then compare four methods that make use of the unlabeled treatment-covariate data:

1. IPW, with a misspecified parametric logistic propensity model
2. IPW, with a flexible, nonparametric BART propensity
3. TMLE
4. BCF

We also run each of the four methods above on simulated data with randomized treatment assignment.
