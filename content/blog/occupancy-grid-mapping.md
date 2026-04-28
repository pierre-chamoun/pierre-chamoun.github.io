---
title: "Occupancy Grid Mapping with Log-Odds Updates"
date: 2025-12-10
summary: "Occupancy grids let a robot maintain a probabilistic map of its environment cell by cell. The log-odds update rule makes each Bayesian update a simple addition — fast, numerically stable, and easy to implement."
tags: ["robotics", "estimation", "perception"]
math: true
draft: false
---

Occupancy grids are arguably the most practical mapping representation for mobile robots.
The idea is elegant: discretise the environment into a grid of cells, and maintain for each
cell a probability that it is occupied. The log-odds update rule makes this computationally
cheap while remaining firmly grounded in Bayesian probability.

## The Setup

Let $m_i \in \{0,1\}$ be the binary occupancy state of cell $i$ ($1$ = occupied).
Given a sequence of sensor measurements $z_{1:t}$ and robot poses $x_{1:t}$, we want:

$$
p(m_i \mid z_{1:t},\, x_{1:t})
$$

Maintaining a full joint distribution over all cells is intractable, so the standard
assumption is **conditional independence** between cells:

$$
p(m \mid z_{1:t}, x_{1:t}) = \prod_i p(m_i \mid z_{1:t}, x_{1:t})
$$

Each cell is then updated independently.

## Bayes Filter for a Single Cell

Applying Bayes' rule recursively to cell $i$:

$$
p(m_i \mid z_{1:t}) \propto p(z_t \mid m_i,\, x_t)\; p(m_i \mid z_{1:t-1})
$$

where $p(z_t \mid m_i, x_t)$ is the **inverse sensor model** — the probability of measuring $z_t$ given that cell $i$ has state $m_i$.

## The Log-Odds Representation

Define the log-odds for cell $i$ at time $t$:

$$
l_{t,i} = \log\frac{p(m_i = 1 \mid z_{1:t})}{p(m_i = 0 \mid z_{1:t})}
$$

The Bayes update becomes a simple **additive increment**:

$$
l_{t,i} = l_{t-1,i} + \underbrace{\log\frac{p(m_i=1 \mid z_t, x_t)}{p(m_i=0 \mid z_t, x_t)}}_{\text{inverse sensor model}} - \underbrace{\log\frac{p(m_i=1)}{p(m_i=0)}}_{\text{log prior}}
$$

For a uniform prior ($p = 0.5$) the log prior is zero and the update is simply:

$$
l_{t,i} = l_{t-1,i} + \ell_{\text{occ}} \cdot \mathbf{1}[\text{cell hit}] + \ell_{\text{free}} \cdot \mathbf{1}[\text{cell in beam}]
$$

Typical values: $\ell_{\text{occ}} = 0.85$, $\ell_{\text{free}} = -0.4$.
Recovering probability at any time: $p = 1 - 1/(1 + e^{l})$.

## Ray Casting with Bresenham

To find which cells lie along a sensor beam, Bresenham's line algorithm traces a raster line
from the robot's position to the measured endpoint in $O(n)$ cell evaluations — no floating-point
required for the trace itself.

Each traversed cell receives the free update; the endpoint cell (if within maximum range)
receives the occupied update. Cells beyond the endpoint are left unchanged.

## Practical Considerations

| Parameter | Typical value | Effect |
|---|---|---|
| Cell resolution | 0.05 m | Finer = more detail, more memory |
| $\ell_{\text{occ}}$ | 0.85 | Higher = quicker to mark occupied |
| $\ell_{\text{free}}$ | −0.40 | More negative = quicker to clear |
| Clamp $[l_{\min}, l_{\max}]$ | [−10, 10] | Prevents cells becoming impossible to update |

Clamping is important: without it a cell hit by thousands of rays becomes so certain that it
can never be updated, which is wrong in dynamic environments.

## Connection to the Kalman Filter

Both occupancy grids and Kalman filters are Bayesian estimators. The key difference is that
the KF maintains a Gaussian belief over a *continuous* state, while the occupancy grid maintains
independent Bernoulli beliefs over a discrete grid. When the state space is a continuous map,
Gaussian Process regression (GP-based mapping) provides a principled generalisation.
