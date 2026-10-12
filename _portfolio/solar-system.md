---
title: "Solar System Simulation: Newtonian Gravity, Chaotic Experiments, and a Black Hole"
excerpt: "A physics-based Python simulation of planetary motion using Newtonian gravity and pygame, extended into six experiments exploring dwarf planets, hyperbolic orbits, black holes, and binary stars."
collection: portfolio
tags:
  - Python
  - Physics
  - Simulation
  - Object-Oriented Programming
use_math: true
---

*Joint work with Armando Trivellato and Patricia Nistor (IE University, October 2025)*

---

## Overview

This project simulates the motion of the Solar System using **Newtonian gravity** and visualizes planetary orbits interactively with **pygame**. What starts as a faithful reconstruction of the eight planets around the Sun becomes a sandbox for exploring orbital mechanics: what happens if we add dwarf planets, increase their masses beyond the Sun's, launch planets past their escape velocity, or drop a black hole into the system?

Each experiment is a separate script built on the same `Planet` class, so the physics is consistent and the variations are minimal — just a few changed numbers in the data dictionary.

![Solar System zoomed out](/images/solar-system-main.png)
*The main simulation, zoomed out. All eight planets orbit the Sun on elliptical paths, with real astronomical distances, masses, and initial velocities.*

---

## The Model

### Newton's Law of Gravitation

Every pair of bodies in the simulation exerts a force on each other:

$$
F = G \frac{m_1 m_2}{r^2}
$$

where $G = 6.674 \times 10^{-11}$ m³ kg⁻¹ s⁻². The force is decomposed into $x$ and $y$ components via

$$
F_x = F \cos\theta, \quad F_y = F \sin\theta, \quad \theta = \operatorname{atan2}(y_2 - y_1, x_2 - x_1)
$$

### Position Update

Velocities and positions update each timestep of one day:

$$
\vec{v} \leftarrow \vec{v} + \frac{\vec{F}}{m} \Delta t, \quad \vec{x} \leftarrow \vec{x} + \vec{v} \Delta t
$$

This is a forward Euler integrator — fast, simple, and good enough for visualization (though it accumulates small errors over long runs).

### The `Planet` Class

All bodies share a single class with four methods:

- `newton_law(other)` — returns the gravitational force from another body
- `total_forces(planets)` — sums the forces from every other body
- `update_position(planets)` — integrates velocity and position forward one timestep
- `draw(win, x_offset, y_offset)` — renders the planet, its orbit trail, and its distance to the Sun

The class ensures every body — the Sun, the planets, dwarf planets, even a black hole — follows the same physical rules while keeping its own unique attributes.

### Interactive Features

The simulation supports real-time interaction:

- **Zoom:** `+` and `-` keys dynamically change the pixel-per-meter scale
- **Pan:** Arrow keys translate the viewport
- **Live distance readout:** Each planet displays its current distance to the Sun in km

---

## Experiments

The value of this project is not just the base simulation but the **six variations** that probe the physics.

### 1. Dwarf Planets: Tiny and Insignificant?

We added **Pluto** and **Eris** to the system with their real masses (~$10^{22}$ kg). As expected, neither visibly perturbs the eight planets.

We then pushed them closer to Neptune — still no effect. Only when we scaled their masses up by **10 orders of magnitude** did the system destabilize.

![Dwarf planets with huge mass](/images/solar-system-dwarf.png)
*Pluto and Eris with masses greater than the Sun. The orbits of the outer planets are visibly warped — a small mass does almost nothing, but a massive one is catastrophic.*

**Lesson:** Mass ratio is everything in orbital dynamics. A body that is 0.0000002% of the Sun's mass is invisible; one ten times larger than the Sun rewrites the system.

### 2. Elliptical Orbits

With all interplanetary forces included, planets trace ellipses around the Sun — as they should. Kepler's first law emerges naturally from the $1/r^2$ force and off-center initial velocities.

![Elliptical orbits](/images/solar-system-elliptical.png)
*Earth's orbit, showing the perihelion-aphelion distance variation of roughly 5 million km — matching real astronomical data.*

### 3. Hyperbolic Orbits

By launching the inner planets with velocities **above their escape velocity**, we obtained hyperbolic trajectories — they fly past the Sun once and never return.

![Hyperbolic orbits](/images/solar-system-hyperbolic.png)
*Hyperbolic escape trajectories. When the initial velocity exceeds $\sqrt{2GM/r}$, the planet's total energy is positive and the orbit becomes open.*

**Lesson:** Orbital type is determined by energy, not by the force law. The same $1/r^2$ force produces ellipses, parabolas, or hyperbolas depending on whether $E < 0$, $E = 0$, or $E > 0$.

### 4. The Black Hole

We inserted a black hole of $4 \times 10^6$ solar masses (matching Sagittarius A*) at 10,000 AU from the Sun.

![Black hole effect](/images/solar-system-blackhole.png)
*The entire Solar System drifts toward the black hole on the left, while the black hole itself (right) remains stationary in view. The Sun and planets are pulled as a unit before any orbital disruption becomes visible.*

**Lesson:** Long-range gravity from a distant massive body can drag an entire system without disrupting its internal orbits — at least initially.

### 5. Binary Star System

We replaced the Sun with two stars of $1.5$ and $1.0$ solar masses orbiting their common center of mass.

![Binary star system](/images/solar-system-binary.png)
*Two stars in mutual orbit — a "celestial dance". Adding planets to this system proved numerically unstable with the current integrator.*

**Lesson:** Two-body systems with comparable masses orbit their **barycenter**, not one another. The larger star moves less; the smaller moves more.

### 6. Circular Orbits — A Failed Attempt

We tried to produce perfectly circular orbits by removing interplanetary forces and giving each planet a hand-tuned velocity. **This failed.** Numerical drift in the Euler integrator accumulated over time, and orbits spiraled outward or inward rather than closing.

**Lesson:** Circular orbits are a knife-edge in parameter space. Real numerical schemes need higher-order integrators (Verlet, Runge-Kutta) to maintain closed orbits over long horizons.

---

## Key Findings

- **Class-based design scales effortlessly.** Six experiments were implemented by editing the `planet_data` dictionary — the physics engine itself never changed.
- **Mass ratios dominate.** Dwarf planets are invisible unless their mass is scaled by ~$10^{10}$.
- **Escape velocity marks the boundary** between bound (elliptical) and unbound (hyperbolic) orbits.
- **The forward Euler integrator has limits.** It's fast and good enough for visualization, but it cannot maintain fine-tuned configurations like circular orbits over many periods.
- **A distant massive body can drag an entire system** without visibly disrupting its internal structure.

---

## Tools

**Language:** Python  
**Libraries:** `pygame` (visualization), `math` (trigonometry), `os` (file paths)  
**Editor:** Visual Studio Code  
**Data source:** [NASA Planetary Fact Sheet](https://nssdc.gsfc.nasa.gov/planetary/factsheet/)

---

## What I'd Improve

- **Replace the Euler integrator with Verlet or Runge-Kutta.** This would make circular orbits achievable and improve long-run accuracy for the black-hole experiment.
- **Add a barycenter for the binary star system.** Currently the two stars are placed symmetrically, but their center of mass should be tracked explicitly to allow planets.
- **Save orbit trails as images.** A simple screenshot or SVG export would let users share their configurations without running the code.

---

## Links

- [GitHub Repository](https://github.com/apayanieu/Solar_System)

---
