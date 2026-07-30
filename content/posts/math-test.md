---
title: "Math Test"
date: 2026-07-29
draft: true
math: true
tags: ["test"]
categories: ["test"]
description: "Temporary post for testing KaTeX / LaTeX rendering. Safe to delete."
---

> **This is a throwaway test post.** Delete `content/posts/math-test.md` once you've confirmed the equations render.

## Inline math

Euler's identity, inline: $e^{i\pi} + 1 = 0$. And a quick one: the golden ratio $\varphi = \tfrac{1+\sqrt{5}}{2}$.

## The classics

Gaussian integral:

$$\int_{-\infty}^{\infty} e^{-x^2}\,dx = \sqrt{\pi}$$

Basel problem:

$$\sum_{n=1}^{\infty} \frac{1}{n^2} = \frac{\pi^2}{6}$$

## Getting nasty

Maxwell's equations in differential form:

$$
\begin{aligned}
\nabla \cdot \mathbf{E} &= \frac{\rho}{\varepsilon_0} &
\nabla \cdot \mathbf{B} &= 0 \\
\nabla \times \mathbf{E} &= -\frac{\partial \mathbf{B}}{\partial t} &
\nabla \times \mathbf{B} &= \mu_0\mathbf{J} + \mu_0\varepsilon_0\frac{\partial \mathbf{E}}{\partial t}
\end{aligned}
$$

The Schrödinger equation:

$$i\hbar\,\frac{\partial}{\partial t}\,\Psi(\mathbf{r},t) = \left[ -\frac{\hbar^2}{2m}\nabla^2 + V(\mathbf{r},t) \right]\Psi(\mathbf{r},t)$$

A continued fraction and a matrix, for good measure:

$$
\varphi = 1 + \cfrac{1}{1 + \cfrac{1}{1 + \cfrac{1}{1 + \cdots}}}
\qquad
A = \begin{pmatrix} a & b \\ c & d \end{pmatrix},\quad
A^{-1} = \frac{1}{ad-bc}\begin{pmatrix} d & -b \\ -c & a \end{pmatrix}
$$

The Riemann zeta functional equation, because why not:

$$\zeta(s) = 2^s \pi^{s-1} \sin\!\left(\frac{\pi s}{2}\right)\Gamma(1-s)\,\zeta(1-s)$$

If all of that rendered as real math instead of raw dollar signs and backslashes — it works. 🎉
