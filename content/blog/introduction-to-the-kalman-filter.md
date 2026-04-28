---
title: "Introduction to the Kalman Filter"
date: 2025-10-15
summary: "The Kalman filter is the workhorse of state estimation in robotics. I walk through the predict and update steps from first principles and show why it is the optimal linear estimator under Gaussian noise."
tags: ["estimation", "robotics", "math"]
math: true
draft: false
---

The Kalman filter is the cornerstone of state estimation in robotics. It gives us an optimal
linear estimator for a system corrupted by Gaussian noise — and understanding it from first
principles pays dividends across perception, control, and SLAM.

## The State-Space Model

We model the system as a discrete-time linear dynamical system:

$$
\mathbf{x}_k = \mathbf{F} \mathbf{x}_{k-1} + \mathbf{B} \mathbf{u}_k + \mathbf{w}_k
$$

$$
\mathbf{z}_k = \mathbf{H} \mathbf{x}_k + \mathbf{v}_k
$$

where:
- $\mathbf{x}_k \in \mathbb{R}^n$ is the hidden state at time $k$
- $\mathbf{z}_k \in \mathbb{R}^m$ is the measurement
- $\mathbf{w}_k \sim \mathcal{N}(\mathbf{0}, \mathbf{Q})$ is process noise
- $\mathbf{v}_k \sim \mathcal{N}(\mathbf{0}, \mathbf{R})$ is measurement noise

## Predict Step

Before seeing the new measurement we propagate our belief forward:

$$
\hat{\mathbf{x}}_{k|k-1} = \mathbf{F} \hat{\mathbf{x}}_{k-1|k-1} + \mathbf{B} \mathbf{u}_k
$$

$$
\mathbf{P}_{k|k-1} = \mathbf{F} \mathbf{P}_{k-1|k-1} \mathbf{F}^\top + \mathbf{Q}
$$

$\mathbf{P}$ is the **error covariance matrix** — it tracks our uncertainty about the state.

## Update Step

Once measurement $\mathbf{z}_k$ arrives, we compute the **Kalman gain**:

$$
\mathbf{K}_k = \mathbf{P}_{k|k-1} \mathbf{H}^\top \left( \mathbf{H} \mathbf{P}_{k|k-1} \mathbf{H}^\top + \mathbf{R} \right)^{-1}
$$

Intuitively, $\mathbf{K}_k$ balances trust between the prediction (via $\mathbf{P}$) and the sensor
(via $\mathbf{R}$). A noisy sensor ($\mathbf{R}$ large) gives a small gain — we rely more on the model.

Then we correct the state estimate:

$$
\hat{\mathbf{x}}_{k|k} = \hat{\mathbf{x}}_{k|k-1} + \mathbf{K}_k \left( \mathbf{z}_k - \mathbf{H} \hat{\mathbf{x}}_{k|k-1} \right)
$$

The term $\mathbf{z}_k - \mathbf{H} \hat{\mathbf{x}}_{k|k-1}$ is called the **innovation** — the surprise
carried by the new measurement. Finally, we update the covariance:

$$
\mathbf{P}_{k|k} = \left( \mathbf{I} - \mathbf{K}_k \mathbf{H} \right) \mathbf{P}_{k|k-1}
$$

## Why Is This Optimal?

The filter minimises the mean squared error $\mathbb{E}\!\left[\|\mathbf{x}_k - \hat{\mathbf{x}}_{k|k}\|^2\right]$
over all linear estimators. Under the Gaussian noise assumption it is also the **maximum a posteriori**
(MAP) estimator — a fact that connects Kalman filtering directly to Bayesian inference.

## Next Steps

In practice, robots rarely live in a linear world. The **Extended Kalman Filter** (EKF) linearises
nonlinear dynamics via a first-order Taylor expansion, while the **Unscented Kalman Filter** (UKF)
propagates a set of *sigma points* through the full nonlinear function — often giving better accuracy
at similar computational cost. A future post will cover both.
