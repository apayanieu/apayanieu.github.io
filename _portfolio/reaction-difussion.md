---
title: "Reaction-Diffusion, Turing Instability, and Pattern Formation"
excerpt: "A finite-difference study of Gierer-Meinhardt and Gray-Scott systems, from Turing's diffusion-driven instability to mode selection on 1D and 2D domains."
collection: portfolio
tags:
  - Mathematical Modeling
  - Partial Differential Equations
  - Numerical Methods
  - Pattern Formation
  - Python
use_math: true
---

## Overview

This project studies two classical reaction-diffusion systems — **Gierer-Meinhardt** and **Gray-Scott** — from three angles: analytical Turing instability analysis, mode selection on finite domains, and explicit finite-difference simulation. The goal is to connect the linear algebra that decides whether a homogeneous equilibrium destabilizes under diffusion with the spatial patterns that actually emerge on 1D and 2D grids.

The central object of study is Turing's counterintuitive result: a reaction system that is stable in isolation can destabilize when diffusion is added, provided one species diffuses much faster than the other. The Gierer-Meinhardt system, a short-range activation / long-range inhibition model, makes this structure explicit. The Gray-Scott system, a cubic autocatalytic model, produces a richer catalog of spot, stripe, and self-replicating patterns depending on its two parameters.

The full notebook, including all helper functions, parameter sweeps, animations, and diagnostics, is available on GitHub (link at the end).

## Mathematical Model

### Gierer-Meinhardt

The Gierer-Meinhardt system in its standard activator-inhibitor form is

$$
\begin{aligned}
\frac{\partial u}{\partial t} &= D_1 \nabla^2 u + \gamma \left( a - b u + \frac{u^2}{v} \right), \\[4pt]
\frac{\partial v}{\partial t} &= D_2 \nabla^2 v + \gamma \left( u^2 - v \right),
\end{aligned}
$$

where $u$ is the activator, $v$ is the inhibitor, $D_1$ and $D_2$ are diffusion coefficients, and $\gamma$ scales the reaction rate. Throughout this study we set $D_1 = 1$ and use $d = D_2 / D_1$ as the bifurcation parameter.

The homogeneous equilibrium is

$$
u^* = \frac{a + 1}{b}, \qquad v^* = (u^*)^2.
$$

### Gray-Scott

The Gray-Scott system is

$$
\begin{aligned}
\frac{\partial u}{\partial t} &= D_1 \nabla^2 u - u v^2 + F(1 - u), \\[4pt]
\frac{\partial v}{\partial t} &= D_2 \nabla^2 v + u v^2 - (F + k) v,
\end{aligned}
$$

with diffusion coefficients $D_1 = 0.1$ and $D_2 = 0.05$ throughout. The two parameters $F$ (feed rate) and $k$ (kill rate) control the qualitative regime, and the Pearson catalog labels types $A$ through $E$ covering spots, stripes, and self-replicating patterns.

## Methodology

### Turing instability analysis

The analysis proceeds in two steps, treated separately because they answer different questions.

The **reaction-level test** asks whether the Jacobian of the reaction terms at $(u^*, v^*)$ satisfies the classical Turing conditions:

$$
\text{tr}(J) < 0, \qquad \det(J) > 0, \qquad d \, f_u + g_v > 2\sqrt{d \, \det(J)}.
$$

The first two conditions guarantee stability without diffusion; the third is the diffusion-driven instability requirement.

The **finite-domain mode test** asks which spatial modes actually destabilize on a domain of length $L$. On a 1D interval with Neumann boundary conditions the Laplacian eigenvalues are

$$
\lambda_n = -\left(\frac{n\pi}{L}\right)^2, \qquad n = 0, 1, 2, \dots
$$

and the temporal growth rate for mode $n$ is the maximum real part of the eigenvalues of

$$
A_n = \begin{pmatrix} f_u + D_1 \lambda_n & f_v \\ g_u & g_v + D_2 \lambda_n \end{pmatrix}.
$$

A mode is unstable when $\max \operatorname{Re} \sigma(A_n) > 0$. On a rectangle $(0, L_1) \times (0, L_2)$ the same construction uses $\lambda_{mn} = -(m\pi/L_1)^2 - (n\pi/L_2)^2$.

### Numerical scheme

Both systems are integrated with an explicit Euler scheme in time and a five-point finite-difference Laplacian in space. Boundary conditions are Neumann for Gierer-Meinhardt and periodic for Gray-Scott. On a $N = 250$ grid the Gray-Scott baseline $dx = 1$, $dt = 2$ is stable; increasing $dt$ to $6$ or refining $dx$ to $0.5$ without adjusting $dt$ produces blow-up within a handful of steps, confirming the expected CFL-type restriction.

The 2D Gierer-Meinhardt runs use $N_x = 20$, $N_y = 50$ on the rectangle $[0,20] \times [0,50]$ with $dx = dy = 1$ and $dt = 0.005$. Gray-Scott runs use a $250 \times 250$ grid with a $20 \times 20$ perturbed patch at the center and a fixed random seed ($\text{seed} = 123$) for reproducibility.

## Results

### Turing space in the $(a, d)$-plane

For the Gierer-Meinhardt system with $b = 1$, the Turing space occupies a well-defined region in the $(a, d)$-plane. On a $500 \times 500$ grid spanning $a \in [0.01, 1.2]$ and $d \in [0, 120]$, **41.24%** of points lie inside the Turing region.

![Turing space for Gierer-Meinhardt in the $(a, d)$-plane with $b = 1$.](/images/gm_turing_space_b1.png)

*The blue region marks parameters satisfying the reaction-level Turing conditions. Case A ($a = 0.4$, $d = 30$) lies inside; Case B ($a = 0.4$, $d = 20$) lies below the diffusion threshold and is not in the Turing space.*

### Mode selection on a finite 1D domain

The reaction-level test alone does not guarantee pattern formation. On a domain of length $L = 40$ the admissible modes are quantized, and only modes whose growth rate is positive will amplify.

| Case | $a$ | $b$ | $d$ | Unstable modes | Leading $n$ | Wavelength | $\max \operatorname{Re} \sigma$ |
|---|---|---|---|---|---|---|---|
| **Case A** | 0.4 | 1.0 | 30.0 | $\{5, 6\}$ | 5 | 16.00 | $+0.0214$ |
| **Case B** | 0.4 | 1.0 | 20.0 | $\varnothing$ | 6 | 13.33 | $-0.0589$ |

Case A is Turing-unstable at the reaction level *and* possesses unstable modes on the domain. Case B satisfies the reaction stability conditions but fails the diffusion threshold: **no admissible mode is unstable**, and the simulation decays to the homogeneous equilibrium.

The 1D simulations confirm this. Starting from small noise about $(u^*, v^*)$ and running for $15{,}000$ steps ($t = 150$):

- **Case A**: the $u$-RMS amplitude grows from $1.15 \times 10^{-2}$ to $3.77 \times 10^{-2}$, a factor of $3.29$. The amplitude of the predicted mode $n = 5$ grows by a factor of $19.24$. Fitting the late-time profile to $\cos(5\pi x/L)$ gives a relative fit of **0.967** — the simulation is dominated by the analytically predicted mode.
- **Case B**: the $u$-RMS amplitude collapses to $1.02 \times 10^{-7}$ and the mode-$5$ amplitude to $2.41 \times 10^{-8}$, confirming decay to the homogeneous state.

![Case A: 1D Gierer-Meinhardt pattern formation, mode 5 emerging from noise.](/images/gm_1d_case_a.gif)

*Case A: the leading unstable mode $n = 5$ emerges from noise and saturates into a stable periodic pattern.*

![Case B: 1D Gierer-Meinhardt decay to the homogeneous equilibrium.](/images/gm_1d_case_b.gif)

*Case B: no admissible mode is unstable, and the perturbation decays to zero.*

### 2D Gierer-Meinhardt on a rectangle

Extending to a $20 \times 50$ rectangle with Neumann boundary conditions, the mode structure becomes two-dimensional. The leading unstable pairs are:

| Rank | $(m, n)$ | $\lambda_{mn}$ | Wavelength $x$ | Wavelength $y$ | $\max \operatorname{Re} \sigma$ |
|---|---|---|---|---|---|
| 1 | $(0, 7)$ | $-0.1934$ | $\infty$ | 14.29 | $+0.0259$ |
| 2 | $(2, 5)$ | $-0.1974$ | 20.00 | 20.00 | $+0.0256$ |
| 3 | $(1, 6)$ | $-0.1668$ | 40.00 | 16.67 | $+0.0248$ |

Case A admits **14 unstable mode pairs** in total; Case B admits **none**. In the 2D simulation, Case A's inhibitor variance grows from an initial $\text{std}(v - v^*) = 0.0202$ to $0.0067$ over $20{,}000$ steps ($t = 100$) after an initial transient — a modest but clearly structured spatial pattern emerges. Case B decays to $\text{std}(v - v^*) \approx 2 \times 10^{-6}$, effectively the homogeneous state.

![2D Gierer-Meinhardt inhibitor snapshots on the rectangle.](/images/gm_2d_inhibitor_snapshots.png)

*Inhibitor deviation $v - v^*$ for Case A (top) and Case B (bottom) at four time points. Case A develops spatial structure; Case B remains flat.*

### Gray-Scott across the Pearson catalog

The Gray-Scott simulations use a $250 \times 250$ grid, $dx = 1$, $dt = 2$, and $5{,}000$ steps ($t = 10{,}000$) starting from a $20 \times 20$ perturbed patch. Five parameter types were tested:

| Type | $F$ | $k$ | Final $\text{std}(u)$ | Final $\text{std}(v)$ | Qualitative behavior |
|---|---|---|---|---|---|
| **A** | 0.040 | 0.060 | 0.1556 | 0.1077 | Pattern saturates into spots |
| **B** | 0.014 | 0.047 | 0.2634 | 0.0987 | Broad stripes, slow coarsening |
| **C** | 0.062 | 0.065 | 0.0270 | 0.0161 | Patch decays; no sustained pattern |
| **D** | 0.078 | 0.061 | 0.0442 | 0.0274 | Patch decays; no sustained pattern |
| **E** | 0.082 | 0.059 | 0.2073 | 0.1223 | Slower growth, complex structures |

The variance trajectories show that types A, B, and E sustain pattern growth across the full $10{,}000$ time units, while types C and D stabilize quickly and never develop sustained structures — consistent with their location relative to the saddle-node bifurcation in the Gray-Scott parameter plane.

### Numerical stability

A short stability study on the Gray-Scott system with type A parameters confirms the practical CFL constraint:

| Configuration | $dx$ | $dt$ | Status | Steps to failure |
|---|---|---|---|---|
| Baseline | 1.0 | 2.0 | Stable | — |
| Large $dt$ | 1.0 | 6.0 | Unstable | 4 |
| Fine $dx$ | 0.5 | 2.0 | Unstable | 5 |

The fine-grid case blows up despite the same $dt$, showing that the diffusion term $D \, dt / dx^2$ — not $dt$ alone — controls the stability bound.

## Discussion

Three findings stand out.

**First, the reaction-level Turing test is necessary but not sufficient.** Case B is a clean demonstration: it satisfies the algebraic conditions on the Jacobian and yet produces no pattern, because on a finite domain no admissible Laplacian eigenvalue falls into the unstable band. The mode-selection step is what distinguishes an academic Turing condition from an observable pattern.

**Second, the finite-difference simulations reproduce the linear prediction quantitatively.** The late-time 1D Case A profile correlates with the analytically predicted $\cos(5\pi x / L)$ mode at **0.967**. This is the strongest evidence in the study that the numerical scheme is faithfully integrating the underlying PDE and that mode selection is the correct explanation for the observed pattern.

**Third, the Gray-Scott parameter sweep reveals how sharp the boundary is between "pattern" and "no pattern."** Types C and D stabilize within a few hundred time units, while types A, B, and E continue to grow. The gap between these regimes is small enough that a coarse parameter grid would miss it.

## Limitations

- **Explicit Euler only.** The scheme is straightforward but requires small $dt$ for stability. An implicit or semi-implicit scheme (Crank-Nicolson, IMEX) would allow larger time steps and reduce total computation.
- **No adaptive mesh.** A uniform grid was used everywhere. For the 2D Gray-Scott runs at $N = 250$ this is acceptable, but finer resolution near pattern boundaries would improve accuracy.
- **No quantitative comparison of the Gray-Scott types to the Pearson phase diagram.** The five types were chosen from the standard catalog but not checked against the published bifurcation boundaries.
- **Bounded parameter sweep.** The Turing-space grid covered $a \in [0.01, 1.2]$ and $d \in [0, 120]$. Parameter regions outside this range, or different $b$, could produce additional structure.

## References

1. Gierer, A., & Meinhardt, H. (1972). A theory of biological pattern formation. *Kybernetik*, **12**(1), 30–39.
2. Pearson, J. E. (1993). Complex patterns in a simple system. *Science*, **261**(5118), 189–192.
3. Turing, A. M. (1952). The chemical basis of morphogenesis. *Philosophical Transactions of the Royal Society B*, **237**(641), 37–72.
4. Murray, J. D. (2003). *Mathematical Biology II: Spatial Models and Biomedical Applications* (3rd ed.). Springer.

---

The complete notebook, including all helper functions, parameter sweeps, and animation generation, is available on GitHub: **[Reaction-Diffusion Project →](https://github.com/apayanieu/reaction_difussion)**

This project was developed as part of the Mathematical Modeling course at IE University (2nd Assignment), April 2026.
