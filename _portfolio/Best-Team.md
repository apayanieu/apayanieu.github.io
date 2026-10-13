---
title: "Best Team: Data-Driven Selection of a Spanish National XI"
excerpt: "A statistics-driven pipeline that pulls player data from the API-Football service, computes position-specific performance scores, and selects an optimal starting eleven — with a Random Forest analysis of which match features actually predict winning."
collection: portfolio
tags:
  - Sports Analytics
  - Data Engineering
  - Machine Learning
  - Python
  - API Integration
use_math: true
---

## Overview

National team selection is traditionally a matter of expert judgement: scouts, coaches, and pundits weigh form, tactical fit, and intangibles that resist quantification. This project asks a narrower but more tractable question: **given a large pool of players and a well-defined set of per-90 statistics, what starting eleven maximizes an aggregate performance score under positional constraints?**

The pipeline has three stages. First, a data-collection layer queries the [API-Football](https://www.api-football.com/) service for every player in Europe's top five leagues plus a curated set of additional clubs, caching every response locally so that repeated runs are free. Second, a rating model converts each player's raw statistics into a position-specific score that weights the metrics that matter most for that role — shooting for attackers, interceptions and duels for defenders, key passes and accuracy for midfielders. Third, a selection routine filters by nationality, sorts by rating within each position, and assembles a balanced squad.

As a natural follow-up, the same match-level data is used to train a Random Forest classifier that predicts match outcomes from team statistics. The feature-importance ranking gives an empirical answer to a question the position weights only assumed: **which on-pitch statistics actually correlate with winning?**

## Data

### Sources

All player- and match-level data come from the API-Football v3 service. The API is rate-limited to 300 requests per minute on the free tier, so the pipeline is engineered to make every request count.

Two data streams are collected:

- **Player statistics** — per-season aggregates for every player on every team in scope, including appearances, minutes, goals, assists, shots, passes, tackles, interceptions, duels, cards, and dribbles.
- **Match statistics** — per-fixture team aggregates, including possession, shots on/off target, passes and pass accuracy, fouls, corners, offsides, cards, goalkeeper saves, and expected goals.

### Scope

The player pool is built from two sources. The main leagues — Premier League, La Liga, Bundesliga, Serie A, Ligue 1, and the Primeira Liga — are pulled programmatically by league ID. Because the API's league endpoint was unreliable for some La Liga clubs during development, a manually curated list of 33 team IDs was added as a fallback, covering all twenty La Liga sides plus selected clubs from Germany, France, Italy, and Portugal.

For the match-level study, the analysis focuses on **Spain's national team** across seasons 2022 and 2023, drawing fixtures from four competition IDs (Nations League, friendlies, Euro qualifiers, and World Cup qualifiers).

### Caching

Every API response is hashed by endpoint and query parameters, then written to a local `api_cache/` directory. Repeated calls for the same data hit the cache instead of the network. This is not just a performance optimization — it makes the pipeline reproducible offline and keeps the free-tier quota from being consumed by development iterations.

## Methodology

### Position-specific rating model

Each player receives a scalar rating computed as a weighted sum of five categorical sub-scores:

$$
\text{Rating} = w_p \cdot S_{\text{pass}} + w_s \cdot S_{\text{shoot}} + w_c \cdot S_{\text{creativity}} + w_d \cdot S_{\text{defense}} + w_\phi \cdot S_{\text{discipline}}
$$

The base weights are shared across all players, but the position multipliers $(w_p, w_s, w_c, w_d, w_\phi)$ differ for each role to reflect the different demands of the position.

The individual sub-scores are:

$$
\begin{aligned}
S_{\text{pass}} &= 0.0020 \cdot \text{passes} + 0.030 \cdot \text{key\_passes} + 0.025 \cdot \text{accuracy}, \\[2pt]
S_{\text{shoot}} &= 0.0025 \cdot \text{shots} + 0.035 \cdot \text{shots\_on} + 0.40 \cdot \text{goals}, \\[2pt]
S_{\text{creativity}} &= 0.40 \cdot \text{assists} + 0.0030 \cdot \text{dribbles} + 0.010 \cdot \text{fouls\_drawn}, \\[2pt]
S_{\text{defense}} &= 0.0040 \cdot \text{tackles} + 0.0030 \cdot \text{interceptions} + 0.0030 \cdot \text{duels\_won}, \\[2pt]
S_{\text{discipline}} &= -0.015 \cdot \text{fouls\_committed} - 0.10 \cdot \text{cards}.
\end{aligned}
$$

The weight structure is **explicitly adversarial in the disciplinary dimension**: the negative sign on fouls and cards penalises players who concede possession cheaply. This makes the score a genuine trade-off rather than a pure production metric.

### Position multipliers

| Position | $w_p$ | $w_s$ | $w_c$ | $w_d$ | $w_\phi$ | Rationale |
|---|---|---|---|---|---|---|
| **Goalkeeper** | — | 0.035 | 0.50 | $-0.02$ (conceded) | — | Custom formula; save volume + penalty saves − goals conceded + pass accuracy |
| **Defender** | 1.3 | 1.0 | 1.0 | **1.5** | 1.0 | Defensive work is primary; passing accuracy matters for build-up |
| **Midfielder** | **1.8** | 1.0 | **1.5** | 1.0 | 1.0 | Passing and creativity dominate; defends less than a centre-back |
| **Attacker** | 1.2 | **1.5** | **1.3** | 1.0 | 1.0 | Shooting weighted highest; creativity rewarded for assisting |

The multipliers encode a footballing philosophy: a midfielder's value comes primarily from distribution, an attacker's from finishing, and a defender's from stopping attacks. The magnitudes were tuned by inspection against known top performers in each role.

### Squad selection

Given the rated player pool, selection is a filtering problem:

1. **Nationality filter** — restrict to players matching a target country (Spain for the current run).
2. **Positional sort** — within each position category, sort descending by rating.
3. **Quota fill** — take the top $k$ per position, where $k$ reflects a standard squad shape:

$$
(k_{\text{GK}}, k_{\text{DEF}}, k_{\text{MID}}, k_{\text{ATT}}) = (3, 8, 6, 6).
$$

The resulting 23-player squad is the input to any starting-eleven format the user prefers — a 4-3-3, a 4-2-3-1, or any other shape.

### Match-outcome model

On the match-level dataset, a Random Forest classifier is trained to predict whether Spain won a fixture, using the 17 in-game statistics listed above plus a binary `is_home` flag. The pipeline is:

- **Imputation** — missing numeric values filled with the column mean via `SimpleImputer`.
- **Scaling** — features standardised with `StandardScaler`.
- **Class balancing** — SMOTE oversampling on the training set, with $k_{\text{neighbors}} = \min(5, n_{\text{minority}} - 1)$ chosen dynamically to avoid degenerate resampling.
- **Classification** — `RandomForestClassifier` with default hyperparameters and fixed seed.

Feature importances are read off the fitted forest and ranked.

## Results

### The selected eleven

Applying the pipeline to the Spanish player pool for the 2023–24 season produces a squad whose starting eleven is visualised below, in a **4-3-3** shape.

![The selected Spanish starting XI, visualised in a 4-3-3 formation.](/images/best_team_xi.png)

*The optimised starting eleven. Each player card shows the position-specific rating computed by the pipeline, along with the underlying attribute scores. The lineup balances attacking output (Aspas, Moreno, Williams) with creative distribution (Rodri, Baena, Kirian) and defensive solidity (Pau Torres, Marsà, Grimaldo, Porro).*

The selection is interesting for what it reveals about the rating model. **Rodri's midfield score of 89** places him well clear of the next-best central midfielder, driven primarily by the passing and creative multipliers ($w_p = 1.8$, $w_c = 1.5$). **Grimaldo's defender score of 84** reflects his unusually high attacking contribution for a full-back — the defensive multiplier dominates, but his passing and creativity sub-scores still push the total higher than a traditional centre-back's. Up front, **Iago Aspas at 85** and **Gerard Moreno at 83** are the highest-rated attackers, both heavily weighted by goals and shots on target.

### What the model got right

The eleven includes several players who are, in hindsight, uncontroversial picks: Rodri, Grimaldo, Pau Torres, and Nico Williams were all regular internationals during the period. The model's top-rated attacker (Aspas) was not, which is a legitimate disagreement with the manager's selections rather than a model error — Aspas retired from international football in 2023, a fact the statistics alone cannot encode.

### Feature importances for winning

The Random Forest analysis on Spain's match statistics produces a clear ranking of which in-game features are most predictive of a Spanish victory.

The top features — **expected goals, shots on target, and pass accuracy** — are the three that most directly capture the mechanism of winning a football match: creating high-quality chances, hitting the target with those chances, and controlling possession well enough to sustain attacks. Features that are commonly cited in punditry but that carry less predictive weight in this dataset — **possession percentage, corners, and offsides** — rank lower, consistent with a now well-established finding in football analytics: possession without shot quality does not win matches.

The `is_home` flag contributes meaningfully, confirming the home-advantage effect that is expected in international friendlies and qualifiers.

## Discussion

Three observations are worth highlighting.

**First, position-specific weighting produces a materially different ranking from a single global score.** A naïve model that weighted every statistic equally would systematically over-rate attacking full-backs and under-rate defensive midfielders. The multipliers correct for this, and the resulting squad shape is balanced in a way that a flat rating would not produce.

**Second, the discipline penalty has measurable downstream effects.** The negative weight on fouls and cards shifts several high-volume tacklers down the defender ranking. Whether this is desirable is a modelling choice: for a possession-dominant team like Spain, minimising fouls conceded is valuable; for a counter-attacking side, it might not be. The parameterisation is a lever, not a truth.

**Third, the match-outcome model partially validates the rating model's feature selection.** The features that drive the RF's importance ranking (shots on target, pass accuracy, expected goals) are exactly the ones the rating model already emphasises through its passing and shooting sub-scores. The RF analysis is not independent confirmation — both use the same underlying data — but it is a consistency check that the intuitive weights were not arbitrary.

## Limitations

- **No positional availability constraints.** The pipeline selects the top $k$ players per position without regard to whether a chosen eleven is formable in a real 4-3-3 or 4-2-3-1. A dedicated optimisation step (e.g. an integer program with formation constraints) would remove the manual adjustment currently required.
- **No opponent modelling.** The ratings are computed against the player's full-season record, not against the specific opponents they would face in a match. A matchup-aware model would weight, for instance, defensive statistics differently against a possession-heavy side than against a low block.
- **No fatigue or availability information.** The API provides season aggregates; it does not expose injuries, international retirements, or minutes-managed rest periods. The Aspas selection is the clearest example of where this gap shows up in practice.
- **Small sample for the match-outcome model.** Spain's fixture list across two seasons is on the order of a few dozen matches. The Random Forest's importances are directionally reliable but the confidence intervals around them are wide, and no cross-validation was performed across multiple seasons.
- **Rating weights were tuned by inspection.** The BASE_WEIGHTS and position multipliers were set by iterating on the output and adjusting until the top-ranked players matched expectation. A more principled approach would fit the weights to a labelled target such as transfer value or manager-selection probability.

---

The complete codebase — data collection, rating model, selection routine, and the match-outcome analysis — is available on GitHub: **[Best Team Project →](https://github.com/apayanieu/Best_Team)**
