---
title: "From Dijkstra to RRT*: a Tour of Motion Planning Algorithms"
date: 2026-01-08
summary: "From Dijkstra on a grid to RRT* in continuous high-dimensional space — a ground-up tour of motion planning algorithms, with the maths behind asymptotic optimality."
tags: ["robotics", "planning", "algorithms"]
math: true
draft: true
---

Motion planning — finding a collision-free path from start to goal — is one of robotics'
oldest problems. The right algorithm depends on whether your configuration space is discrete
or continuous, the dimension of the space, and whether you need the optimal path or just a
feasible one fast. This post traces the line from graph search to sampling-based methods.

## Graph Search: Dijkstra and A\*

In a discrete graph $G = (V, E)$ with edge weights $w : E \to \mathbb{R}_{\geq 0}$,
Dijkstra's algorithm finds the shortest path from source $s$ to all other vertices in
$O((|V| + |E|)\log|V|)$ using a priority queue.

**A\*** improves on Dijkstra by adding a heuristic $h(v)$ — an estimate of the cost from $v$
to the goal. The priority of each node is:

$$
f(v) = g(v) + h(v)
$$

where $g(v)$ is the actual cost from $s$ to $v$. If $h$ is **admissible** ($h(v) \leq h^*(v)$
for all $v$, where $h^*$ is the true cost-to-go), A\* finds the optimal path.

A\* works well on grid maps but struggles in high-dimensional configuration spaces (a 7-DOF
arm has a 7D C-space) because the graph itself becomes intractable to enumerate.

## Probabilistic Roadmaps (PRM)

PRM sidesteps the curse of dimensionality by sampling random configurations:

1. Sample $q$ uniformly in $\mathcal{C}_{\text{free}}$
2. Connect $q$ to its $k$ nearest neighbours if the straight-line path is collision-free
3. Repeat until the roadmap connects start and goal
4. Query the roadmap with Dijkstra/A\*

PRMs are **multi-query**: build once, query many times. The tradeoff is that building a dense
roadmap is expensive and the roadmap may miss narrow passages.

## Rapidly-Exploring Random Trees (RRT)

RRT grows a tree from the start towards a random sample by extending the nearest tree node
in the direction of the sample by a step size $\eta$:

```
repeat:
  q_rand ← Sample()
  q_near ← Nearest(T, q_rand)
  q_new  ← Steer(q_near, q_rand, η)
  if CollisionFree(q_near, q_new):
    T.add(q_new)
```

RRT is **probabilistically complete**: as the number of samples $n \to \infty$, the
probability of finding a solution (if one exists) approaches 1. However, the paths it
produces are jagged and suboptimal.

## RRT\* — Asymptotic Optimality

RRT\* adds two operations that guarantee the path cost converges to the optimum as $n \to \infty$:

### 1. Choose Parent

Instead of connecting $q_{\text{new}}$ to $q_{\text{near}}$, choose the parent $q^*$ from
the neighbourhood $B(q_{\text{new}}, r_n)$ that minimises total cost:

$$
q^* = \arg\min_{q \in \text{Near}(q_{\text{new}}, r_n)} \left[ c(q) + d(q, q_{\text{new}}) \right]
$$

where $c(q)$ is the cost from start to $q$.

### 2. Rewire

After adding $q_{\text{new}}$, check whether routing existing neighbours through $q_{\text{new}}$ reduces their cost:

$$
\text{if}\quad c(q_{\text{new}}) + d(q_{\text{new}}, q') < c(q') \quad \Rightarrow \quad \text{rewire}(q', q_{\text{new}})
$$

With the neighbourhood radius $r_n = \gamma \left(\frac{\log n}{n}\right)^{1/d}$ (where $d$ is
dimension and $\gamma$ is a problem-dependent constant), RRT\* is **asymptotically optimal**.

## Comparison

| Algorithm | Complete | Optimal | C-space | Complexity |
|---|---|---|---|---|
| Dijkstra | ✓ (discrete) | ✓ | Discrete | $O(V \log V)$ |
| A\* | ✓ (discrete) | ✓ (admissible $h$) | Discrete | $O(V \log V)$ |
| PRM | Prob. complete | ✓ (dense) | Continuous | $O(n^2)$ build |
| RRT | Prob. complete | ✗ | Continuous | $O(n \log n)$ |
| RRT\* | Prob. complete | Asymp. ✓ | Continuous | $O(n \log n)$ |

## What's Next

**Informed RRT\*** focuses sampling inside an ellipsoid connecting start and goal once
a solution is found, dramatically accelerating convergence. **MPPI** (Model Predictive
Path Integral) takes a different angle entirely — it is a sampling-based *control* method
that plans and executes simultaneously. These will be the subject of a future post.
