---
title: "Rigid Body Rotations: from Matrices to Quaternions to Lie Groups"
date: 2025-11-20
summary: "Rotation matrices, quaternions, and Lie groups each represent the same thing — orientation in 3D — but with very different trade-offs. This post builds each representation from scratch and explains why SO(3) as a Lie group is the right abstraction for robotics."
tags: ["math", "robotics", "geometry"]
math: true
draft: false
---

Representing orientation in 3D is deceptively tricky. Euler angles suffer from gimbal lock,
rotation matrices are redundant, and quaternions confuse everyone at first. This post builds
up each representation from scratch and explains why Lie groups are the right language for
the job in modern robotics.

## Rotation Matrices — $SO(3)$

A rotation matrix $R \in \mathbb{R}^{3 \times 3}$ satisfies two constraints:

$$
R^\top R = I, \qquad \det(R) = +1
$$

The set of all such matrices forms the **Special Orthogonal group** $SO(3)$. Composition of
rotations is just matrix multiplication, which is associative but *not* commutative:

$$
R_{AB} \neq R_{BA} \quad \text{in general}
$$

The catch: $SO(3)$ has 9 entries but only 3 degrees of freedom. That redundancy costs memory
and makes interpolation awkward.

## Euler Angles — Intuitive but Dangerous

Any rotation can be decomposed into three successive rotations about coordinate axes.
A common convention is ZYX (yaw $\psi$, pitch $\theta$, roll $\phi$):

$$
R = R_z(\psi)\, R_y(\theta)\, R_x(\phi)
$$

When $\theta = \pm 90°$, the first and third rotations become aligned and one degree of freedom
is lost — **gimbal lock**. This makes Euler angles unsuitable for arbitrary 3D rotation control.

## Quaternions

A unit quaternion $\mathbf{q} = (w, \mathbf{v}) \in \mathbb{H}$ with $\|\mathbf{q}\| = 1$ encodes a rotation by angle $\theta$ about unit axis $\hat{\mathbf{n}}$ as:

$$
\mathbf{q} = \cos\frac{\theta}{2} + \sin\frac{\theta}{2}\,\hat{\mathbf{n}}
$$

Rotating a vector $\mathbf{p}$ is then:

$$
\mathbf{p}' = \mathbf{q} \otimes \begin{pmatrix}0 \\ \mathbf{p}\end{pmatrix} \otimes \mathbf{q}^{-1}
$$

Note that $\mathbf{q}$ and $-\mathbf{q}$ represent the **same** rotation — the unit quaternions
form a double cover of $SO(3)$, i.e. $SU(2) \cong S^3$.

Quaternions are compact (4 numbers), interpolate smoothly via **SLERP**, and avoid gimbal lock.
The downside is that the multiplication rule is non-obvious and the double-cover causes sign
ambiguity in attitude estimation.

## The Lie Group Perspective

$SO(3)$ is a **Lie group**: a group that is also a smooth manifold. Its associated **Lie algebra**
$\mathfrak{so}(3)$ is the tangent space at the identity, consisting of $3 \times 3$ skew-symmetric matrices:

$$
[\boldsymbol{\omega}]_\times = \begin{pmatrix}0 & -\omega_3 & \omega_2 \\ \omega_3 & 0 & -\omega_1 \\ -\omega_2 & \omega_1 & 0\end{pmatrix}
$$

The **exponential map** connects algebra to group:

$$
R = \exp\!\left([\boldsymbol{\omega}]_\times\right) = I + \sin\theta\,[\hat{\boldsymbol{\omega}}]_\times + (1-\cos\theta)\,[\hat{\boldsymbol{\omega}}]_\times^2
$$

where $\theta = \|\boldsymbol{\omega}\|$ and $\hat{\boldsymbol{\omega}} = \boldsymbol{\omega}/\theta$.
This is **Rodrigues' rotation formula**. The inverse is the logarithmic map $\log : SO(3) \to \mathfrak{so}(3)$.

### Why This Matters for Robotics

Working on the Lie algebra lets us:
- **Perturb** rotations without leaving $SO(3)$ (useful for EKF on manifolds)
- **Integrate** angular velocity $\dot{R} = R\,[\boldsymbol{\omega}]_\times$ reliably
- **Optimise** over rotations without chart-switching (important in SLAM back-ends)

Modern libraries like Sophus (C++) and `spatialmath` (Python) expose exactly this interface.
Mastering it unlocks the door to rigorous 3D robotics.
