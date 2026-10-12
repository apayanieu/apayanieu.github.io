---
title: "Melting Routes: A GAM-Based Approach to Forecasting Commercial Traffic in the Pacific Arctic"
excerpt: "Modeling how Arctic shipping activity responds nonlinearly to sea ice, weather, and port distance — and showing that sea ice governs not just the level of traffic, but its predictability."
collection: portfolio
tags:
  - R
  - Statistics
  - GAM
  - Climate
  - Arctic Shipping
use_math: true
---

*Joint work with Pablo Pérez González and Ricardo Pérez Pérez-Piñeiro (IE University, 2025)*

---

## Overview

Arctic shipping routes could cut the voyage from Europe to Asia by **~40%** compared to the Suez Canal — saving roughly ten days per trip. Yet actual transit volumes remain low, averaging just **34 transits per year** between 2011 and 2018. This project addresses the gap between the rhetoric of an "opening Arctic" and the reality of a corridor that remains largely empty.

Rather than building an engineering cost model with hypothetical vessel assumptions, we take a **revealed-preference** approach: we study how actual vessel traffic responds to environmental conditions across the Pacific Arctic (northern Bering, Chukchi, and western Beaufort Seas) from 2015 to 2020.

**Key result:** Sea ice is the single most informative predictor of Arctic shipping activity, and its effect is nonlinear — traffic collapses above a critical ice concentration. But two other findings matter just as much:

1. **Distance to port creates a logistic ceiling** that ice retreat alone does not lift.
2. **Sea ice governs the variance of traffic, not just its mean** — the marginal ice zone is simultaneously the zone of greatest opportunity and greatest operational uncertainty.

![Shipping intensity across selected months](/images/melting-hero.png)
*Monthly vessel-days across the Pacific Arctic in January and July of 2018 and 2020. The seasonal ice retreat opens lanes that close again each winter — but the year-over-year pattern shows increasing utilization of the same corridors.*

---

## Research Question

Under what environmental conditions does Arctic shipping activity become operationally viable, and how strongly do those conditions shape observed traffic patterns?

Existing studies rely on engineering cost models that compare hypothetical voyage expenses under different ice scenarios. These are complex and fragile — their thresholds are sensitive to assumptions about icebreaker tariffs, insurance premia, and vessel speeds. We instead ask what the ships themselves reveal about when a route becomes viable.

---

## Data

We harmonized four heterogeneous products onto a common **hexagonal grid** (EPSG:3338) and built a balanced panel of **6,553 hexagons × 72 months** (2015–2020), for a total of **461,448 hexagon-months**.

| Layer | Source | Notes |
|---|---|---|
| **Vessel traffic** | AIS (Kapsar et al., 2022, 2023) | Cargo + "other" vessel classes (icebreakers, tugs, passenger). Fishing and tankers excluded. |
| **Sea-ice concentration** | NOAA/NSIDC passive microwave (Cavalieri et al., 1996) | Area-weighted mean per hexagon, preserving ice-edge gradients. |
| **Meteorology** | ERA5 monthly means (Hersbach et al., 2020) | 2 m air temperature and 10 m wind speed. |
| **Distance to port** | NGA World Port Index (2019) | Geodesic distance from hexagon centroid to nearest major port. |

### A Critical Data Issue: Collinearity Between Ice and Temperature

The dominant feature of the covariate correlation matrix is a **−0.78 correlation between ice concentration and air temperature**. Naively including both in a linear model would inflate standard errors and make the ice effect uninterpretable.

To address this, we use **ice-orthogonalized temperature** — the residual of a univariate smooth of temperature on ice. This isolates the component of temperature variation that ice concentration does not already explain. Concurvity between the two smooths drops from 0.815 (raw temperature) to 0.675 (orthogonalized).

### Two Structural Features of the Response

Two properties of the raw response shape every modeling decision downstream:

- **A massive spike at zero.** 43.3% of hexagon-months have exactly zero vessel-days. This is not a data gap — it is the dominant feature of the response, driven by cells that are structurally inaccessible.
- **A heavy right tail.** Conditional on positive traffic, vessel-days range across four orders of magnitude.

A simple linear regression explains less than **2.5%** of variance ($R^2 < 0.025$), confirming that the response is governed by nonlinearity, spatial structure, and seasonality — not by any single linear effect.

---

## Methodology

### Generalized Additive Models

We model the conditional expectation of shipping activity as a sum of flexible, data-driven functions of the covariates:

$$
g\big(\mathbb{E}[Y_{gt}]\big) = \alpha + \sum_{j} f_j(x_{j,gt}) + \mathbf{z}_{gt}^\top \gamma
$$

Each $f_j$ is a **penalized cubic regression spline** (P-spline, Eilers and Marx 1996), with a second-order difference penalty. The amount of smoothness is estimated by penalized likelihood maximization. Seasonality enters as a **cyclic cubic spline** so that the December–January transition is continuous.

### Distributional Assumptions

Two response variables, each with a different distributional approach:

| Response | Distribution | Rationale |
|---|---|---|
| **Vessel-days** ($\text{nOpRd}^{CO}$) | Tweedie | Zero mass + continuous positive part; the natural compound Poisson–gamma for mixed discrete-continuous traffic intensity. |
| **Unique vessel count** ($\text{nMMSI}^{CO}$) | Zero-inflated Poisson (ZiP) | Separates the count process from the structural-zero (presence) process. |

The ZiP presence process is where we extract the **accessibility threshold** — the ice concentration at which the probability of a cell being structurally empty crosses 0.5 and 0.9.

### Location-Scale Extension

After fixing the mean predictor, we fit a **Tweedie location-scale GAM** (`twlss`) that lets the variance vary with ice:

$$
\log \phi_{i,t} = \gamma_0 + g(\text{ice}_{i,t})
$$

This directly answers whether Arctic traffic becomes **more or less predictable** as ice conditions change — a question the mean-only models cannot address.

---

## The Build-Up Ladder

We add covariates one at a time and retain them based on AIC, deviance explained, concurvity, and basis-adequacy diagnostics.

| Model | Term added | Tweedie p | Dev. expl. (%) | ΔAIC |
|---|---|---|---|---|
| **M0b** | Seasonal + trend + spatial te() | 1.504 | 61.37 | — |
| **M1** | + s(ice_conc) | 1.507 | 65.71 | **−49,803** |
| **M2b** | + s(air_temp_resid) | 1.505 | 66.21 | −6,161 |
| **M3** | + s(wind_speed) | 1.505 | 66.30 | −1,015 |
| **M4** | + s(port_dist) | 1.500 | **67.04** | −8,435 |

The single most informative move in the ladder is adding sea ice: **AIC falls by nearly 50,000 points.** If you could only ask one question about a hexagon's traffic, you would ask about the ice.

---

## Key Findings

### 1. The Ice Effect Is Nonlinear and Steep

![Smooth term effects](/images/melting-smooths.png)
*All smooth terms of the final mean predictor. The ice smooth (center row, left) spans roughly two orders of magnitude on the response scale between open water and consolidated ice, and its effective degrees of freedom (18.87 of 19.00) nearly saturate the basis — confirming that the relationship is not well represented by any simple linear term.*

The form of the relationship is what matters here. It is not simply "traffic drops linearly with ice." It is a smooth, quantified function estimated jointly with seasonality, trend, spatial position, temperature, wind, and logistics — and it changes gradually across much of the range before collapsing near the ice edge.

### 2. Distance to Port Is a Ceiling Ice Retreat Won't Lift

Adding distance to port produces the second-largest improvement (ΔAIC = −8,435). The interpretation is direct: **even where water is open, a cell far from any port still sees little traffic** — because fuel, repair, and refuge remain out of reach.

This connects our empirical finding to the Northern Sea Route viability literature, which argues that viability depends not just on ice but on supporting infrastructure. That the model recovers this relationship independently is a form of empirical consistency check.

### 3. The Temperature Effect Is Asymmetric

After orthogonalizing against ice, the temperature smooth reveals an asymmetry: **unexpected warm anomalies are associated with sharply elevated traffic, whereas unexpected cold anomalies depress it only weakly.** The system responds more to "go" than to "stop."

A warm window opens opportunities that are exploited aggressively. A cold spell mostly makes already-committed voyages more cautious rather than cancelling them.

### 4. Sea Ice Also Governs the Variance of Traffic

![Tweedie dispersion vs ice concentration](/images/melting-dispersion.png)
*The location-scale Tweedie model shows that dispersion is highest at low ice concentrations and decreases monotonically as ice thickens. This is the study's primary contribution: sea ice does not just determine how much traffic a region sees — it determines how predictable that traffic is.*

This result has direct practical implications. **The marginal ice zone — where ice concentration is low but nonzero — is simultaneously the zone of greatest opportunity and the zone of greatest operational uncertainty.** Insurers, route planners, and policymakers should not read "ice retreat" as "predictable shipping."

### 5. Gaussian Fails, Tweedie and Negative Binomial Are Close

| Model | Family | Dev. expl. (%) | AIC | ΔAIC |
|---|---|---|---|---|
| **D_tw** (M4) | Tweedie | **67.04** | 2,189,424 | — |
| **D_nb** | Negative binomial | 65.87 | **2,048,217** | −141,207 |
| **D_gaussian** | Gaussian | 39.80 | 4,318,174 | +2,128,750 |

Gaussian is eliminated decisively: it explains 27 fewer percentage points of deviance and costs over 2 million AIC points.

![Gaussian residual diagnostics](/images/melting-residuals.png)
*Gaussian-family QQ plot and residuals vs. linear predictor. The symmetric error model cannot reproduce a response that is 43.3% exact zeros with a heavy right tail.*

Negative binomial beats Tweedie on raw AIC by 141,207 points — but Tweedie explains 1.17 pp **more** deviance and, crucially, is the only family that supports the location-scale extension on which the study's primary contribution rests. We adopt Tweedie as the model of record for that reason.

---

## Why This Matters

Three contributions distinguish this study from prior work:

**1. A correlational result becomes a quantified function.** Kapsar et al. (2023) documented a negative association between sea ice and Arctic traffic. We estimate *how that relationship behaves* across the full range of ice conditions, jointly with seasonality, trend, space, weather, and logistics.

**2. The threshold is empirical, not assumed.** The NSR viability literature often discusses ice thresholds around the 25% range. Our GAM framework lets the data determine where the response changes most rapidly, rather than imposing a threshold a priori.

**3. The question shifts from "how much" to "how reliable."** Prior work focuses on accessibility, traffic growth, or incident risk. Our location-scale model shows that the regions of greatest opportunity are also those of greatest uncertainty — a distinction that matters for insurance pricing, route planning, and infrastructure investment.

---

## Limitations

- **All results are associational, not causal.** Ships avoid ice; ice and traffic share drivers such as season and climate. The model states that less ice coincides with more traffic, not that removing ice produces it.
- **The study covers 2015–2020.** Extrapolating to a more ice-free future assumes the fitted relationships persist beyond the observed range.
- **The temperature effect is orthogonalized, not clean.** Concurvity between ice and temperature was reduced from 0.815 to 0.675 but not eliminated.
- **Port distance is partly endogenous.** Ports were built where traffic already concentrated, so its smooth cannot be read as purely causal.
- **The spatial tensor-product basis is detectably tight** (k-index ≈ 0.66, p ≈ 0) in every fit — the most defensible target for future refinement.

---

## Tools

**Language:** R  
**Core libraries:** `mgcv` (GAMs, Tweedie, ZiP, location-scale), `bam` with fREML for large-scale fitting, `ecmwfr` (ERA5 retrieval)  
**Diagnostics:** DHARMa (simulated residuals), `k.check()`, Moran's I, ACF  
**Data processing:** sf, terra, dplyr, tidyr  
**Visualization:** ggplot2  
**Reproducibility:** A single annotated R script with a fixed configuration block drives the entire pipeline, including intermediate products and a row ledger of dropped hexagons.

---

## Links

- [GitHub Repository](https://github.com/pabloperezglez/new_stats_modeling.git)
