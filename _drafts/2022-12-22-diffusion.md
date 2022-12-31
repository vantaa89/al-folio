---
layout: post
title: Neural Differential Equations and Diffusion Probabilistic Models
date: 2022-12-21 01:00:00 +0900
# description: 
tags: deep-learning
giscus_comments: true
---

# Neural ODE
먼저 $$L$$개 layer를 가진 residual network를 생각하자.

\begin{aligned}
    h_\theta(X) &= z_L \\\\\\
    z_L &= z_{L-1} + f(z_{L-1}, \theta, L-1) \\\\\\
    &\vdots\\\\\\
    z_2 &= z_1 + f(z_1,\theta, 1) \\\\\\
    z_1 &= z_0 + f(z_0, \theta, 0) \\\\\\
    z_0 &= X
\end{aligned}

