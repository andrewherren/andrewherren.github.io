---
title: "Feature selection in stratification estimators of causal effects: lessons from potential outcomes, causal diagrams, and structural equations"
collection: publications
permalink: /publication/2022-causal
excerpt: "Insights on Feature Selection for Treatment Effect Estimation."
date: 2022-09-23
venue: "arXiv"
paperurl: https://arxiv.org/abs/2209.11400
citation: "Hahn, P. R. & Herren, A. (2022). Feature selection in stratification estimators of causal effects: lessons from potential outcomes, causal diagrams, and structural equations. arXiv preprint arXiv:2209.11400."
---

What role (if any) can modern, machine-learning-based feature selection techniques play in average treatment effect (ATE) estimation in causal inference? 
This work addresses the question under several assumptions:

1. Discrete covariates
2. No post-treatment covariates
3. No unmeasured confounding

In this case, a stratification estimator that estimates and re-aggregates the treatment-control contrasts ($\bar{Y}_{Z=1,X=x} - \bar{Y}_{Z=0,X=x}$) for each unique value $x$ of the covariate space will identify the ATE.

If you've read this far, you may be thinking "has Drew ever worked with a real dataset before?? Even if we have discrete covariates and are willing to make the assumptions above, splitting on every unique covariate level is going to give us a massive variance!"

Well, rest assured, I have worked with real data before, and I do feel your pain! That's what inspired my advisor Richard and I to think about how we might try to reduce the number of covariates in our adjustment set.

There are many challenges in doing this work with only the assumptions laid out above. Just to call out a few:

1. We don't have the causal DAG and we are reluctant to assume that we have enough data to estimate the graph from the data. 
    * This means we have no idea if an individual covariate is a collider, instrument, confounder, etc...
2. Available dimension reduction methods either cannot remove instrumental variables (i.e. propensity score) or impose assumptions on the data generating process (i.e. latent deconfounding variables as in Causal-VAE, constant treatment effects as in prognostic scores)
    * This means that without further assumptions, we will trade off minimality with bias in our search for adjustment sets. 

The paper fuses three frameworks for doing causal inference (Causal DAGs, Potential Outcomes, and Structural Equations) and uses important concepts from each framework. We establish some theory on minimality and optimality of adjustment sets and then illustrate the problems and pitfalls of feature selection in a series of examples.

If you've read this far and wondered whether you can use this paper in the real world, I have two responses:

1. The examples presented in Section 4 package many general insights about treatment effect estimation that aren't commonly presented in causal inference textbooks. You can treat those examples as a supplement to what you already know about doing causal inference in practice!
2. This work lays the **theoretical groundwork** for ongoing research on machine learning based feature selection for ATE estimation. Stay tuned for R and Python code that implements estimators addressing the tradeoffs presented above! 
