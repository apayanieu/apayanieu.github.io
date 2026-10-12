---
title: "Crowd Synchrony and the 'Wobbly' Millennium Bridge: A Replication Study of the Strogatz Model"
excerpt: "Replicating the Strogatz explanation of the Millennium Bridge wobble using coupled oscillators, bifurcation analysis, and numerical simulation in Python."
collection: portfolio
tags:
  - Python
  - Dynamical Systems
  - Coupled Oscillators
  - Bifurcation Analysis
use_math: true
---

*Alfonso Payan Callejas (IE University, 2026)*

---

## Overview

On 10 June 2000, around **90,000 people** walked onto London's brand-new Millennium Bridge. Within three days the celebration became an engineering case study: the deck began swaying side to side, and the bridge had to be **closed for nearly two years**. It eventually reopened in February 2002 — after engineers installed viscous dampers — making it more of a "millennium +2 bridge" than a millennium one.

The cause was not a design flaw in the usual sense. Engineers discovered it was a problem of **synchronization**: once the bridge starts moving laterally, pedestrians subconsciously adjust their gait for balance, and that adjustment feeds energy back into the bridge's side-to-side vibration mode, amplifying the sway. Mathematician **Steven Strogatz** framed the problem as a single stability mechanism in which bridge wobbling and crowd synchrony emerge together, explained by the mathematics of **collective synchronization**.

This project replicates the Strogatz model in Python, implements numerical simulations of the coupled bridge–pedestrian system, identifies the **critical crowd size** at which the instability triggers, and explores how physical parameters shift the bifurcation threshold.

![The Millennium Bridge at night, with St Paul's Cathedral in the background](/images/hero-millenium-bridge.jpg)
*London's Millennium Bridge — the suspension footbridge whose opening-day wobble became one of the most famous case studies in coupled-oscillator dynamics.*

---

## The Bridge as a Damped Oscillator

A bridge is not a rigid object. It stores elastic energy when it bends (stiffness), has inertia because it has mass, and loses energy because joints and materials are not perfectly lossless (damping). For small displacements the dominant restoring force is approximately proportional to displacement, so the relevant lateral vibration mode can be idealized as a **weakly damped harmonic oscillator**.

Engineers decompose the full structure into vibration modes. Near one resonant frequency a single mode dominates, reducing the bridge to one modal coordinate $X(t)$:

$$M\ddot{X}(t) + B\dot{X}(t) + KX(t) = (\text{external forcing})$$

where $M$, $B$, $K$ are the **modal mass, damping, and stiffness**.

Without forcing, the equilibrium $X = 0$ is stable — perturb it, and damping removes energy so the motion dies out. A crowd changes the story.

---

## The Instability Mechanism

Pedestrians can inject energy back into the lateral mode. If the injected power per cycle exceeds the dissipated power, the equilibrium $X = 0$ becomes **unstable** and the bridge amplitude grows from small perturbations.

With pedestrian–bridge feedback, the effective damping can be reduced — even becoming negative — causing the fixed point to lose stability. In mechanical systems this typically corresponds to the birth of sustained oscillations: a **limit cycle** in the $(X, \dot{X})$ phase plane. Strogatz *et al.* explicitly frame the onset as a **bifurcation** where wobbling and synchrony emerge together as dual aspects of one instability.

---

## The Kuramoto Framework

Kuramoto's model is the canonical framework for collective synchronization:

$$\dot{\theta}_i = \omega_i + \frac{K}{N} \sum_{j=1}^{N} \sin(\theta_j - \theta_i), \quad i = 1, \ldots, N$$

with order parameter:

$$R e^{i\Phi} = \frac{1}{N} \sum_{j=1}^{N} e^{i\theta_j}, \quad R \in [0, 1]$$

where $R \approx 0$ means incoherence and $R = \mathcal{O}(1)$ means partial synchrony.

The Millennium Bridge setup matches the Kuramoto worldview unusually well: pedestrians have gait phases, natural cadence varies across people, and the bridge provides a **shared, mean-field coupling channel** — everyone feels the same platform motion.

---

## The Strogatz Bridge Model

Strogatz *et al.* couple a bridge oscillator to $N$ pedestrian gait phases:

$$M\ddot{X} + B\dot{X} + KX = \sum_{i=1}^{N} G \sin \Theta_i(t)$$

$$\dot{\Theta}_i(t) = \Omega_i + C\, A(t) \sin\!\big(\Psi(t) - \Theta_i(t) + \alpha\big), \quad i = 1, \ldots, N$$

Bridge amplitude $A(t)$ and phase $\Psi(t)$ are defined at the bridge frequency $\Omega = \sqrt{K/M}$ by:

$$X(t) = A(t) \sin \Psi(t), \qquad \dot{X}(t) = A(t)\,\Omega \cos \Psi(t)$$

### Parameter Meanings

| Symbol | Meaning |
|---|---|
| $M, B, K$ | Modal mass, damping, stiffness; $\Omega = \sqrt{K/M}$ is the modal frequency |
| $G$ | Peak lateral forcing per pedestrian |
| $N$ | Number of pedestrians on the bridge |
| $\Omega_i$ | Natural stepping frequency of pedestrian $i$, drawn from density $P(\Omega)$ |
| $C$ | Gait sensitivity to bridge motion amplitude |
| $\alpha$ | Phase lag between bridge motion and pedestrian response |

**Instability mechanism in one sentence:** small wobble → phase pulling → more coherence → larger net forcing → larger wobble.

---

## Critical Crowd Size

Under the special case $\alpha = \pi/2$ and a symmetric unimodal frequency density, averaging yields a closed-form expression for the **critical crowd size**:

$$N_c = \frac{2B\Omega}{\pi\, G\, C\, P(\Omega)}$$

**Interpretation:** larger damping $B$ raises the threshold; larger forcing $G$, sensitivity $C$, or density near resonance $P(\Omega)$ lowers it.

### How Parameters Move the Bifurcation

- **Bridge parameters:** increasing $B$ increases $N_c$; changing $M$ or $K$ shifts $\Omega$ and therefore $P(\Omega)$.
- **Crowd/coupling:** increasing $G$ or $C$ lowers $N_c$.
- **Frequency spread:** broader cadence distributions make coherence harder, pushing thresholds up.

---

## Numerical Replication

### Simulation Parameters

| Parameter | Value | Meaning |
|---|---|---|
| $M$ | 1.0 | Modal mass |
| $f_0$ | 1.0 Hz | Bridge natural frequency ($\Omega = 2\pi$ rad/s) |
| $\zeta$ | 0.02 | Damping ratio |
| $G$ | 0.015 | Force per pedestrian |
| $C$ | 0.80 | Coupling sensitivity |
| $\alpha$ | $\pi/2$ | Phase lag (Strogatz "simple case") |
| $\sigma$ | 0.60 rad/s | Cadence spread |

The simulation uses a **vectorized RK4 integrator** with substep $dt = 0.01$ s, coupling the bridge ODE to $N$ phase oscillators simultaneously.

### Experiment 1: Crowd Ramp

The core diagnostic is a **staircase crowd ramp**: starting with 20 pedestrians and adding 10 every 6 seconds up to 240, then continuing for 15 s. The theoretical critical crowd size is:

$$N_c \approx 126.00, \qquad t_c \approx 66.10 \text{ s}$$

Before $t_c$, the wobble amplitude $A(t)$ remains near zero and synchrony $R(t)$ stays small (incoherent fluctuations). After $t_c$, **both $A(t)$ and $R(t)$ rise together**, consistent with the predicted feedback-driven onset.

![Crowd ramp experiment showing N(t), A(t), and R(t) over time](/images/mb-ramp-experiment.png)
*Crowd ramp $N(t)$ (top), wobble amplitude $A(t)$ (middle), and synchrony $R(t)$ (bottom). The vertical dashed line marks the moment $t = t_c$ where $N(t)$ first reaches the theoretical $N_c$.*

### Experiment 2: Onset vs Crowd Size

Using tail-step averaging — computing mean amplitude and synchrony over the last 1.5 s of each crowd step — shows the onset more directly: **mean tail $A$ and mean tail $R$ jump sharply near $N_c$**.

![Mean tail amplitude and synchrony vs crowd size](/images/mb-wobble-vs-crowd.png)
*Mean tail amplitude (left) and mean tail synchrony (right) per crowd step vs $N$. The dashed line marks $N_c$.*

### Experiment 3: Coupling Sweep

The critical crowd formula predicts $N_c \propto 1/C$. Sweeping over three coupling values ($C = 0.60, 0.80, 1.10$) confirms this: **larger $C$ shifts onset leftward** to a smaller threshold crowd size.

| $C$ | Predicted $N_c$ |
|---|---|
| 0.60 | 168.0 |
| **0.80** | **126.0** |
| 1.10 | 91.6 |

![Effect of coupling C on wobble onset](/images/mb-coupling-sweep.png)
*Effect of coupling sensitivity $C$ on wobble onset. Each curve shows mean tail amplitude vs $N$, with dashed lines marking each run's theoretical $N_c$.*

### Experiment 4: Bifurcation Diagram

A bifurcation-style visualization plots the **steady-state (tail RMS) wobble amplitude** across 100 crowd sizes around threshold. The cloud of points stays near zero below threshold and becomes clearly nonzero above it, with scatter due to randomness in phase/frequency realizations — characteristic of a **supercritical bifurcation**.

![Bifurcation diagram: bridge amplitude vs crowd size](/images/mb-bifurcation.png)
*Empirical bifurcation diagram of steady wobble amplitude vs $N$ (100 values). The dashed marker indicates the theoretical threshold $N_c \approx 126$.*

### Experiment 5: Critical Coupling and $N_c(C)$

Rearranging the threshold formula yields a critical coupling for a fixed crowd:

$$C_{\text{crit}}(N) = \frac{2B\Omega}{\pi\, G\, N\, P(\Omega)}$$

For $N = 200$: $C_{\text{crit}} \approx 0.5040$.

The numerical evaluation of $P(\Omega)$ via trapezoidal integration matches the exact Gaussian to **relative error $\approx -4.63 \times 10^{-7}$**, confirming the numerical pipeline.

![Nc as a function of coupling strength C](/images/mb-nc-vs-c.png)
*$N_c(C)$ computed from theory, with the reference point $(C_{\text{crit}}, N = 200)$ highlighted. The hyperbolic relationship $N_c \propto 1/C$ means that even small increases in coupling sensitivity dramatically reduce the crowd needed to trigger wobble.*

---

## Key Findings

1. **The Strogatz model reproduces the instability.** The numerical replication confirms that wobble and synchrony emerge together at a sharp, predictable crowd threshold — consistent with a **supercritical bifurcation**.

2. **$N_c \approx 126$ pedestrians** trigger the instability under the baseline parameters, matching the theoretical prediction exactly. The critical time $t_c \approx 66$ s.

3. **Coupling sensitivity $C$ is the most leverageable parameter.** Doubling $C$ roughly halves the required crowd. This explains why the real bridge was so sensitive — even mild pedestrian responsiveness to platform motion can dramatically lower the threshold.

4. **The bifurcation diagram shows the classic supercritical shape:** near-zero amplitude below threshold, then a smooth, monotonically growing branch above it, with finite-$N$ stochastic scatter.

5. **Numerical integration validates the analytical pipeline.** The numerical $P(\Omega)$ agrees with the exact Gaussian to $\sim 10^{-7}$ relative error, confirming that no discretization artifacts contaminate the threshold predictions.

---

## The Whole Story in One Sentence

The Millennium Bridge can be modeled as a lightly damped lateral oscillator driven by many pedestrian phase oscillators; as crowd size passes a critical threshold, the motionless/desynchronized equilibrium loses stability and a stable wobbling state appears — consistent with a supercritical bifurcation in reduced slow dynamics and a Hopf-type "equilibrium → oscillation" transition in the physical $(X, \dot{X})$ dynamics.

---

## Assumptions and Limitations

- **Single lateral mode.** The model captures one dominant degree of freedom, which does not account for vertical motion or mode coupling.
- **Phase-only pedestrians.** Real pedestrians have mass, stride length, and can stop walking — none of which is modeled.
- **Hypothetical coupling law.** The sinusoidal coupling in the phase equation is plausible and testable, but not directly measured from biomechanical data.
- **Weak damping / near resonance.** The averaging analysis that yields the closed-form $N_c$ requires these assumptions, though the full numerical simulation does not.
- **No spatial effects.** All pedestrians feel the same bridge motion; in reality, different bridge locations have different mode amplitudes.

---

## Future Work

1. **Spatial mode shapes.** Incorporating the first few lateral mode shapes, so pedestrians at different positions contribute differently to the forcing, would make the model more physically realistic.
2. **Heterogeneous coupling.** Allowing $C$ and $\alpha$ to vary across pedestrians (some people adapt more to platform motion than others) would test the robustness of the threshold prediction.
3. **Comparison with Belykh *et al.*'s alternative paradigm.** A newer model proposes a different instability mechanism; replicating both and comparing predictions against the Arup test data would be a natural extension.

---

## Tools

**Language:** Python  
**Numerical methods:** NumPy (vectorized RK4 integration)  
**Visualization:** Matplotlib (static plots + interactive real-time dashboard with sliders, buttons, and animation)  
**Architecture:** Modular — `millenium_bridge.py` (model + integrator) + Jupyter notebook (experiments + figures)

---

## Links

- [GitHub Repository](https://github.com/apayanieu/crowdsynch)

