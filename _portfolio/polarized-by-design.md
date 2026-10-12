---
title: "Polarized by Design: A Phase Diagram of Opinion Polarization on Adaptive Hebbian Networks"
excerpt: "Charting when and how sharply a society splits into two echo chambers when only the strength of social ties — not the topology — is allowed to adapt."
collection: portfolio
tags:
  - Python
  - Network Science
  - Computational Social Science
  - Complex Systems
use_math: true
---

*Joint work with Pablo Pérez and Ricardo Pérez (IE University, 2025)*

---

## Overview

Polarization splits a population into camps that increasingly disagree — and stop talking. Classical opinion models show that local imitation can give either consensus or division, and adaptive-network models let agents rewrite away from disagreement, forming echo chambers.

![Political polarization](/images/polarized-hero.jpg)
*Two camps, one society. Our model asks when a fixed social graph — with no rewiring at all — can still end up permanently divided by nothing more than the decay of trust across opinion lines.*

**Our twist:** the wiring never changes. No tie is cut or created. Only its **strength** adapts, in a Hebbian style. We ask not *whether* polarization appears, but **when**, **how sharply**, and **how it depends on network structure**.

This question matters because it isolates a mechanism that is often conflated with rewiring. If echo chambers can form on a **fixed** social graph — simply because trust between disagreeing peers decays — then platform interventions that focus only on who follows whom will miss a fundamental driver of division.

---

## The Model

### Agents and Opinions

N agents sit on a fixed graph (Erdős–Rényi, Watts–Strogatz, or Barabási–Albert). Each holds one of two opinions: **F** (Fake-believer) or **D** (Debunker). All nodes update synchronously.

### Stubbornness

Each node has a stubbornness $S_i \in [1, 100]$. At $S_i = 1$, an agent is easily swayed; at $S_i = 100$, it is practically immovable.

### Persuasion

When neighbour $j$ tries to persuade $i$, the attempt succeeds with probability

$$
p_{ij} = \frac{w_{ij}}{S_i}
$$

where $w_{ij} \in [0,1]$ is the current trust weight of the edge. With multiple disagreeing neighbours, the probability that $i$ flips is

$$
p_{\text{flip}}(i) = 1 - \prod_{\substack{j \in \mathcal{N}(i) \\ x_j \neq x_i}} \left(1 - \frac{w_{ij}}{S_i}\right)
$$

### Hebbian Weight Update

After opinions update, every edge adapts by a fixed step $a$, clipped to $[0,1]$:

$$
w_{ij} \leftarrow \Pi_{[0,1]}\left(w_{ij} + a \cdot \sigma_{ij}\right), \quad
\sigma_{ij} = \begin{cases} +1 & \text{if agree,} \\ -1 & \text{if disagree.} \end{cases}
$$

This is the Hebbian rule applied to social trust: **agreement reinforces, disagreement erodes**.

### The Dynamical Race

On every cross-cutting tie, **persuasion** and **decay** compete. If persuasion wins, the bridge converts to agreement before it weakens. If decay wins, the weight reaches $0$ and permanently insulates the pair — and because a dead tie rarely revives, that is the engine of **irreversibility**.

---

## Polarization, Made Visible

We summarise the system by the mean weight of cross-opinion (F–D) ties. The **polarization coefficient** is

$$
P = 1 - \bar{w}_X
$$

where $\bar{w}_X$ is the average weight of cross-opinion edges. $P = 0.5$ means "no dynamics" (all ties at baseline $w = 0.5$); $P = 1$ means every bridge has decayed to silence.

![Initial vs polarized network](/images/polarized-network.png)
*Left: initial 50/50 society (t = 0, P = 0.5). Right: the same graph at t = 157 (P = 1) — every cross-camp tie has decayed to a silent bundle. Edges are neither created nor destroyed; the graph did not break, its bridges went quiet.*

---

## Experimental Design

The fate of a society is a tug-of-war between two rates — how fast ties reshape ($a$) versus how hard opinions resist ($S$). Rather than guess a threshold, we **chart the entire plane**.

- **Grid:** log-spaced $25 \times 25$ over $a \in [10^{-3}, 0.5]$ and $S \in [1, 100]$.
- **Topologies:** Erdős–Rényi, Watts–Strogatz, Barabási–Albert.
- **Degrees:** $\langle k \rangle = 4, 6, 8, 10$.
- **Replications:** 20–35 random seeds per cell, across three phases (coarse sweep, long runs near the transition, and 3×3 jittered sub-grids at the boundary).
- **Total:** **462,780 simulations**, all in Python.

![Phase diagram](/images/polarized-phase-diagram.png)
*Fraction of societies that polarize over the (a, S) plane (Erdős–Rényi, ⟨k⟩ = 8). A mixed corner, a polarized field, and a diagonal bistable band (the 0.5 contour) between them.*

---

## Three Regimes — and a Coin Flip

Run-to-run variance $\sigma_{\text{rms}}$ separates three distinct regimes:

| Regime | (a, S) | Median opinion changes | Fraction polarized | σ_rms |
|---|---|---|---|---|
| Deterministically polarized | (0.25, 100) | 0 | 1.0 | ~0 |
| Bistable | (0.009, 48) | 0 | 0.28 | **large** |
| Deterministically mixed | (0.017, 4.4) | 0.34 | 0.1 | ~0 |

**Microstructure.** The divided state is total: pooled, every cross tie is dead ($\bar{w} = 0$) and every in-group tie saturates ($\bar{w} = 1$). The mixed state is perpetual churn — the median agent changes its mind on roughly 2 of every 5 steps, indefinitely.

**Bistability.** In the band, two identical societies can diverge to opposite fates. The outcome is a coin flip, not a deterministic function of parameters.

---

## Connectivity and Architecture

![Boundary plots](/images/polarized-boundary.png)
*Polarization boundary in (a, S) for each topology and degree. Higher ⟨k⟩ lifts the boundary: more stubbornness is needed to divide a denser society.*

Connectivity is **protective**. The critical stubbornness $S^+$ rises with $\langle k \rangle$:

| Topology | k = 4 | k = 6 | k = 8 | k = 10 |
|---|---|---|---|---|
| Barabási–Albert | 14.6 | 20.5 | 27.2 | 33.8 |
| Erdős–Rényi | 11.2 | 16.4 | 21.6 | 27.8 |
| Watts–Strogatz | 7.5 | 12.8 | 18.5 | 23.8 |

**Architecture matters.** Small-world divides most (0.357), random sits in the middle (0.325), and scale-free resists most (0.289) — hubs anchor many cross-ties. Given enough stubbornness, division is inevitable for every architecture.

![Microstructure grid](/images/polarized-microstructure.png)
*Fraction polarized across all 12 topology × degree combinations. The boundary shifts systematically with both structure and connectivity.*

---

## Discussion

A society here meets one of three fates: deterministically united, deterministically divided, or **bistable** — poised between, its outcome a coin flip.

**Connectivity buys resilience.** Denser societies polarize less easily, all else equal: each extra cross-cutting tie is one more bridge that must be starved to zero before the camps can separate.

**Influence, not infrastructure.** Because the topology never moves, polarization here is purely a story about the **strength** of contact. Where a platform fixes who-follows-whom but interaction intensity is free to adapt, echo chambers can form with **no rewiring at all**.

**The bistable regime is the signature prediction.** Two societies indistinguishable in every parameter can diverge to opposite fates through nothing but the accumulated accidents of who changed their mind first.

---

## Limitations and Extensions

**Limits.** Binary opinions; homogeneous stubbornness; polarization read as cross-tie death. Weights adapt but topology does not. The baseline $P = 0.5$ is not "half polarized," and under bistability the mean describes no single run.

**Extensions — each unlocks behaviour the present model cannot show:**

- **Heterogeneity.** Draw $S$ from a heavy-tailed law: a few near-immovable zealots would seed and anchor each camp.
- **Structure evolution.** Let a weight threshold cut and create edges — the topology itself fragments into true echo chambers.
- **Asymmetric $a$.** A larger step for one camp — one side radicalizes faster, modelling a lopsided media ecosystem.
- **Continuous opinions.** Beliefs on $(-1, 1)$ with a pull to the median voter — graded moderation replaces all-or-nothing flips.
- **Also:** continuous-time (Gillespie) updating, pair / quenched mean-field, triadic closure, finite-size scaling of $S^+$.

---

## Tools

**Languages:** Python  
**Libraries:** NumPy, NetworkX, Matplotlib, concurrent.futures  
**Infrastructure:** Process-pool parallelism for ~463k simulations, CSV-backed resumable pipeline  
**Visualization:** Matplotlib (phase diagrams, boundary curves, network snapshots)

---

## Links

- [GitHub Repository](https://github.com/apayanieu/Polarized_by_Design)


