---
title: "Semi-supervised learning and the question of true versus estimated propensity scores"
collection: publications
permalink: /publication/2020-ssl-propensity
excerpt: "Whether (and when) to use unlabeled data in causal inference via the propensity score."
date: 2020-09-14
venue: "arXiv"
paperurl: https://arxiv.org/abs/2009.06183
citation: "Herren, A., & Hahn, P. R. (2020). Semi-supervised learning and the question of true versus estimated propensity scores. arXiv preprint arXiv:2009.06183."
---

Suppose we have data:

* $Y$: an outcome of interest
* $Z$: a treatment that may (or may not) causally impact the outcome
* $X$: a set of control variables that may be related to $Z$, $Y$, or both

and suppose we're willing to make all of the assumptions that would allow us to identify and estimate a causal effect of $Z$ on $Y$ after adjusting for $X$.

If we were given a large amount of data from the same distribution, but without $Y$ observed ("unlabeled data"), can we use that data in our estimate of the causal effect?

The answer, it turns out, is "yes!" but the explanation is somewhat more subtle than "more data is always better."
This paper explores the challenges and opportunities that come with doing causal inference on "unlabeled data." After revisiting some classic asymptotic results and studying their properties in finite samples, we run a detailed simulation study that demonstrate the ways in which unlabeled data may improve treatment effect estimates.

The paper is available [on Arxiv]() and simulation code for this paper is available [on Github](https://github.com/andrewherren/semi-supervised-propensity)
