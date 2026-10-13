---
title: "Credibility-Weighted Misinformation Diffusion on Adaptive Networks"
excerpt: "A coevolving-network model of false-claim diffusion where transmission is weighted by source credibility, edges rewire away from low-credibility spreaders, and a SIR+E extension captures irreversible entrenchment."
collection: portfolio
tags:
  - Mathematical Modeling
  - Network Science
  - Agent-Based Simulation
  - Epidemiology of Information
  - Python
use_math: true
---

*Joint work with Pablo Pérez and Ricardo Pérez (IE University, March 2026)*

---

## Overview

Misinformation spreads like a contagion, but the analogy breaks down in two important ways. First, whether you adopt a claim depends heavily on **who** is spreading it — a claim forwarded by a trusted account is far more contagious than the same claim from an unknown one. Second, the network is not fixed: users unfollow, mute, and curate their feeds in response to what they see, which means the topology **coevolves** with the states propagating through it.

This project builds a discrete-time simulator for a single false claim on a coevolving network. It combines three mechanisms that are each well studied in isolation but rarely modeled together:

- **Credibility-weighted transmission**, where each node carries a heavy-tailed credibility score and the per-contact adoption probability scales as a power of source credibility.
- **Credibility-sensitive rewiring**, where a susceptible node is more likely to disconnect from a low-credibility infectious neighbor than from a high-credibility one.
- **Adaptive credibility feedback**, where a node that later recovers from the claim penalizes the credibility of the neighbor who infected it — a minimal model of debunking or reputation erosion.

The core methodological contribution is the **SIR+E extension**: a modified SIR model with an absorbing **entrenched** state $E$ that remains infectious indefinitely. This is not "SIR but slower." It is a stronger behavioral claim: some fraction of exposed users become effectively irreversible spreaders, even after others have been corrected.

![Network misinformation diffusion hero](/images/sire_hero.jpg)

---

## Mathematical Model

### Network

Let $G_t = (V, E_t)$ be an undirected graph on $N$ nodes evolving over discrete time $t = 0, 1, 2, \ldots$. The edge set changes through rewiring. Three graph families are supported: **Erdős–Rényi (ER)**, **Watts–Strogatz (WS)**, and **Barabási–Albert (BA)**.

### Credibility distribution

Each node $i$ has credibility $c_i(t) \in (0, 1)$. Initial credibility is drawn from a heavy-tailed Lomax (Pareto-II) distribution and squashed into the unit interval:

$$X_i \sim \mathrm{Lomax}(\xi), \qquad c_i(0) = \frac{X_i}{1 + X_i}.$$

Smaller $\xi$ gives a heavier tail — more nodes with credibility near $1$. This captures the empirical fact that perceived trust and influence are highly heterogeneous on social platforms.

### States

The baseline SIR model uses three states:

$$X_i(t) \in \{S, I, R\},$$

where $S$ is susceptible (has not adopted the claim), $I$ is infected (believes and shares), and $R$ is recovered or corrected (immune to this specific claim within the simulation horizon).

The **SIR+E extension** adds a fourth state:

$$X_i(t) \in \{S, I, R, E\},$$

where $E$ is **entrenched** — an absorbing, permanently infectious state. Entrenched nodes cannot be corrected and continue to spread the claim indefinitely.

### Credibility-weighted transmission

Let $\mathcal{N}_t(j)$ be the neighbors of $j$ in $G_t$. The per-contact adoption probability from an infectious neighbor $i$ to a susceptible node $j$ is

$$p_{i \to j}(t) = \beta \, [c_i^{\mathrm{eff}}(t)]^{\alpha},$$

where $\beta \in [0, 1]$ is baseline transmission intensity and $\alpha \ge 0$ is the credibility amplification exponent. The effective credibility $c_i^{\mathrm{eff}}$ may be lowered by an optional downranking intervention.

Assuming independent exposures within one step, the probability that $j$ adopts at $t+1$ is

$$\Pr[X_j(t+1) = I \mid X_j(t) = S] = 1 - \prod_{i \in \mathcal{N}_t(j):\, i \text{ infectious}} \bigl(1 - \beta\,[c_i^{\mathrm{eff}}(t)]^{\alpha}\bigr).$$

"Infectious" means $X_i = I$ in SIR and $X_i \in \{I, E\}$ in SIR+E.

### Exit dynamics: recovery versus entrenchment

In the baseline SIR model, infected nodes recover with probability $\gamma$ per step. With the optional stubbornness mechanism enabled, recovery becomes credibility-dependent:

$$\gamma_i(t) = \gamma\,(1 - s\,c_i(t)),$$

where $s \in [0, 1]$ is the stubbornness strength. Higher-credibility nodes recover more slowly, so they stay infectious longer.

In SIR+E, entrenchment is applied before recovery:

$$\Pr[I \to E] = \rho, \qquad \Pr[I \to R] = (1 - \rho)\,\gamma_i(t).$$

The entrenched state is absorbing: $E \to E$. Entrenchment hazard $\rho$ is a hardening or radicalization parameter; stubbornness $s$ is a separate mechanism that makes recovery slower for high-credibility nodes but does not by itself create entrenchment.

### Credibility-sensitive rewiring

For an edge $(i, j) \in E_t$ where $i$ is infectious and $j$ is susceptible, the susceptible endpoint may disconnect with probability

$$w(i) = w_0\,(1 - c_i(t))^{\kappa},$$

where $w_0 \in [0, 1]$ is baseline rewiring strength and $\kappa \ge 0$ is credibility sensitivity. Higher $\kappa$ means susceptible nodes preferentially avoid low-credibility spreaders.

When rewiring triggers, the edge is removed and the susceptible endpoint reconnects to a new node $k$ (not already connected, not itself) chosen with probability

$$\omega_k \propto \mathbf{1}[k \text{ allowed}]\,c_k(t)^{\eta},$$

where $\eta \ge 0$ controls the preference for reconnecting to high-credibility nodes. If the optional *avoid infectious* flag is set, candidates currently in $I$ or $E$ are excluded, which lets the network actively segregate by state.

### Adaptive credibility feedback

Static credibility makes influence an exogenous trait. The adaptive feedback rule turns it into an **endogenous** variable shaped by the diffusion process itself. When susceptible node $j$ becomes infected at time $t$, we attribute the infection to a specific infector $\pi(j) \in \mathcal{N}_t(j)$ chosen probabilistically and weighted by the exposure strength $p_{i \to j}$. When $j$ later recovers ($I \to R$), the credibility of its infector is penalized:

$$c_{\pi(j)}(t+1) = \mathrm{clip}_{[0,1]}\bigl(c_{\pi(j)}(t) - \delta\bigr),$$

where $\delta$ is a small constant decrement. The result is a coupled feedback loop: recoveries silently reshape future transmission and rewiring by eroding the credibility of repeatedly debunked spreaders.

The full per-step dynamics are laid out in the source code, which runs a synchronous update over four phases: **infection**, **exit**, **rewiring**, and **credibility penalty**.

---

## Implementation

### The interactive simulator

The project ships with a **Matplotlib-based interactive simulator** designed for exploration rather than maximum performance. The window is split into two panels: a network view on the left and a time-series plot on the right.

![SIR+E simulator UI](/images/sire_simulator_ui.png)
*The simulator running in SIR+E mode. Sliders on the bottom adjust transmission, recovery, entrenchment, and rewiring parameters live. The avoid-infectious and stubbornness toggles control behavioral assumptions; the Model button swaps between SIR and SIR+E.*

The screenshot above is a **full-entrenchment regime**: after 69 steps, $S = 0.02$, $I = 0.00$, $E = 0.94$, $R = 0.04$. Nearly the entire network has hardened into the entrenched state, and because $E$ is absorbing and infectious, the infection pressure never vanishes even though $I$ has collapsed to zero. This is precisely the qualitative behavior **SIR cannot produce**.

Live-controllable parameters include baseline transmission $\beta$, credibility amplification $\alpha$, recovery $\gamma$, entrenchment hazard $\rho$, rewiring strength $w_0$, credibility sensitivity $\kappa$, reconnection preference $\eta$, stubbornness strength $s$, credibility penalty $\delta$, and network size $N$. Checkboxes toggle *avoid infectious* and *stubbornness*. Keyboard shortcuts: space pauses, `r` resets, `g` generates a new graph, `m` toggles the model.

### Batch mode

The same dynamics are exposed through a command-line interface for sweeps and reproducibility. Running

```text
python cred_weighted_misinformation_sim.py --mode trial --model sire
```

produces a summary table of peak infected fraction, time-to-peak, final recovered fraction, final entrenched fraction, and final largest-connected-component fraction. The `--mode alpha_sweep` option runs a parameter sweep over $\alpha$ with rewiring disabled, isolating the effect of credibility amplification alone.

---

## Key Results and Interpretation

The project's central claims are **structural rather than numeric** — this is a modeling paper about what changes when a new mechanism is introduced, not a calibrated forecast for any specific platform.

- **Different absorbing sets.** SIR ends in an absorbing set with $I = 0$. SIR+E can end with $E > 0$, meaning a **persistent infectious reservoir** survives indefinitely. This is the qualitative break: the epidemic endpoint itself is different.

- **Persistence without external forcing.** Once $E$ exists and remains connected to susceptibles, infection pressure does not vanish. Persistence becomes possible without assuming an external broadcast source. In the simulator screenshot, the population converged to a state that SIR can never reach.

- **Rewiring can stabilize polarization.** Because susceptibles rewire away from infectious neighbors, the network can fragment into components. With $E$ absorbing, "infectious clusters" can become **stable echo chambers** that survive indefinitely rather than dying out.

- **No clean threshold.** In SIR on static networks, outbreak conditions are linked to the spectral radius of the transmission matrix, or equivalently to a reproduction number $R_0$. With adaptive rewiring and an absorbing infectious state, there is no single clean threshold: you can get regimes where diffusion is suppressed globally but entrenched pockets persist locally. A useful diagnostic proxy is the spectral radius of the credibility-weighted adjacency matrix,

$$W(t) = \mathrm{diag}\bigl((c^{\mathrm{eff}}(t))^{\alpha}\bigr)\,A(t),$$

though with adaptive $A(t)$ the quantity is time-varying and does not summarize outbreak risk the way it does in the static case.

- **Credibility becomes endogenous.** Under the adaptive feedback rule, high-credibility nodes start as powerful amplifiers but lose influence if many of their infected neighbors later recover. This can produce **non-monotone trajectories**: early fast spread followed by a reputation collapse of the central spreaders. The credibility penalty admits an interpretation as platform moderation, fact-checking visibility, or social learning that degrades trust in repeatedly debunked accounts.

### Methodology note: why SIR+E

The SIR+E extension is deliberately framed as a **behavioral assumption**, not a numerical tweak. SIR encodes the optimistic belief that everyone is ultimately correctable on similar time scales. SIR+E encodes the pessimistic belief that some fraction of the population becomes irreversibly committed to a claim, and continues to spread it after the broader conversation has moved on. Both are assumptions. Making the second one explicit is the point.

The consequence is that model behavior in SIR+E is qualitatively different from SIR at the same parameter values, and not only in the tail. The absorbing set is different, the long-run infectious mass is different, and the network fragmentation induced by rewiring interacts with entrenchment in a way that has no SIR analogue.

---

## Limitations

- **Infector attribution is a modeling choice.** The rule implemented here — pick one infector weighted by exposure strength — is simple and interpretable, but other attribution schemes (uniform, maximum-credibility, first-exposure) are possible and would change the feedback loop.
- **No empirical calibration.** Credibility is sampled from a Lomax distribution with a fixed shape parameter, and the transmission function is a power law. Neither is fit to platform data. The model is a mechanism-exploration tool, not a calibrated forecast.
- **Homogeneous forgetting.** Recovered nodes are modeled as permanently immune to the specific claim. In reality, users who have been corrected may re-adopt if exposed again, especially under repeated pressure from entrenched sources.
- **Rewiring is local.** Susceptible nodes rewire individual edges. Real platforms support bulk actions (unfollow-all, block-list import) that would fragment the network faster than the edge-at-a-time rule used here.
- **No content features.** The model treats the claim as a single unit. Real misinformation has varying plausibility, framing, and affective charge, all of which interact with source credibility.

---

## Tools

**Language:** Python  
**Simulation:** NumPy, NetworkX  
**Visualization:** Matplotlib (interactive UI with sliders, buttons, keyboard shortcuts)  
**CLI:** argparse (batch trial and parameter-sweep modes)

---

## Links

- [GitHub Repository](https://github.com/apayanieu/SIR_misinformation)

---
