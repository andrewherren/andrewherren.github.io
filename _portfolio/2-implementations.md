---
title: "Implementations"
excerpt: "Minimal didactic implementations of common algorithms in statistics and applied math"
collection: portfolio
---

[Implementations](https://github.com/andrewherren/implementations) is a Github repository with simple implementations of common statistical and mathematical algorithms. The goal of the repo is to enable easy exploration of the challenges and tradeoffs inherent in statistics and machine learning methods, not to provide robust implementations for use in research or applications.

Some examples:

* [A (relatively) simple base R implementation of Bayesian CART](https://github.com/andrewherren/implementations/blob/main/papers/cart_mcmc.R) (Chipman, George, McCulloch 1998)
* [Simulation-based implementation and demonstration of the mathematical insights of Hirano et al (2003)](https://github.com/andrewherren/implementations/blob/main/papers/hirano_imbens_ridder_2003.R) that weighting estimators of the average treatment which use a fitted propensity score are asymptotically efficient, relative to estimators that use the true propensity function (if known). The paper is interesting because it packages several important insights which are not easily disentangled in the asymptotic analysis. A finite sample elaboration, assuming discrete covariates, is offered in section 4.2 of [my recent paper with Richard Hahn](https://arxiv.org/abs/2209.11400).
* [Simple implementation of a Gibbs sampler for the horseshoe prior in Bayesian linear regression](https://github.com/andrewherren/implementations/blob/main/statistics/horseshoe.R) (original paper defining the prior is Carvalho et al (2010) but a Gibbs sampler is articulated in Makalic and Schmidt (2015))
