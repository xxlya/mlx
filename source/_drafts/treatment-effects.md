---
title: Treatment Effects
tags:
  - healthcare
subtitle: Learnings from a few papers
author: Pranav Rajpurkar
---

Will a drug have a positive effect on a patient? For a patient $i$, the **treatment effect** (${τ_i}$) is the difference in potential outcomes if the patient receives a drug $(Y_i^{(1)})$ vs. if they don't $(Y_i^{(0)})$.

$$τ_i = Y_i^{(1)} - Y_i^{(0)}$$

We look at work that deals with binary outcomes, where 0 corresponds to a bad outcome (example: death by Cardiovascular Disease), and 1 corresponds to a good outcome. One task is to estimate the population average treatment effect, given by:

$$τ\,^p = \mathbb{E}[Y_i^{(1)} - Y_i^{(0)}]$$

However, we might be more interested whether the treatment would be effective not for the whole population, but for a particular individual, or a particular subpopulation $x$; this is refered to as heterogeneous treatment effects:

$$τ\left(x\right) = \mathbb{E}[Y_i^{(1)} - Y_i^{(0)} | X_i = x]$$

We seek to learn $τ$, but note the difficulty that we can only ever observe one of $(Y_i^{(0)})$ and $(Y_i^{(1)})$ in practice, because we can't treat and not treat the same patient simultaneously: this is called the *fundamental problem of causal inference*. It's therefore challenging to directly train machine learning algorithms on differences of the form $Y_i^{(1)} - Y_i^{(0)}$.

So how do we estimate such treatment effects? In this post, we look at a few papers to get some insights:

The first paper we look at is [Estimation and inference of heterogeneous treatment effects using random forests (2017)](http://www.tandfonline.com/doi/abs/10.1080/01621459.2017.1319839).

This paper extends the random forest algorithm to a causal forest for estimating heterogenous treatment effects:
- One assumption made is that of *unconfoundedness*: that the assignment of whether or not a patient gets treated $W_i \in \lbrace  0, 1 \rbrace$ is independent of the potential outcomes $(Y_i^{(0)}, Y_i^{(1)})$ when conditioned on $X_i$. The implication of this assumption is that nearby observations in the x-space can be treated as "having come from a randomized experiment."
- Trees can be thought of as nearest neighbor methods with an adaptive neighborhood metric, with the closest points to $x$ being in the same leaf that it is. Let's assume that we can building a classification tree by some method by observing independent samples $(X_i, Y_i)$, then a new $x$ can be classified by:
    - Identify the leaf containing $x$
    - In that leaf, take the mean of the $Y_i$s in that leaf.
- In a causal forest, we make use of the assignment labels $W_i$ in the examples. To make the prediction $τ\left(x\right)$:
    - Identify the leaf containing $x$
    - In that leaf, compute the mean of the $Y_i$s where $W_i = 0$, and subtract that from the mean of the $Y_i$s where $W_i = 1$.
- Given a procedure for generating one tree, a causal forest can generate an ensemble of such trees, and then take the mean of the resulting $τ\left(x\right)$s.
- The causal forest is compared to non-adaptive nearest neighbors method (k-NN matching baseline) which estimates treatment effect for any $x$:
    - Compute the mean of the $Y_i$s of the $k$ closest examples (L2 distance to $X_i$s) where $W_i = 0$, and subtract that from the mean of $Y_i$s of the $k$ closest examples where $W_i = 1$.
- Causal forests outperform the k-NN matching baseline, performing at order of magnitude better in mean-squared-error.
- The authors identify one weakness of nearest neighbor approaches and random forests as "they can fill the valleys and flatten the peaks of the true $τ\left(x\right)$ function, especially near the edge of feature space."

The next paper we'll look at is [Machine Learning Methods for Estimating Heterogeneous Causal Effects (2015)](https://pdfs.semanticscholar.org/86ce/004214845a1683d59b64c4363a067d342cac.pdf); this work inspires the causal tree approach we saw in the first paper. 

- The paper first goes over two baseline methods for estimating causal 