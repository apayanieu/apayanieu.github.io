---
title: "F.I.T. Proteins: Fast Inference of Therapeutic Protein–Ligand Interactions"
excerpt: "Predicting BRD4-binding molecules from DNA-encoded library data using fingerprint, SMILES, and graph neural network models under a block-aware OOD evaluation framework."
collection: portfolio
tags:
  - Python
  - Machine Learning
  - Drug Discovery
  - Graph Neural Networks
---

*Joint work with Ricardo Pérez and Pablo Pérez (IE University, December 2025)*

---

## Overview

Cancer cells depend on dysregulated molecular signals to sustain uncontrolled proliferation. **BRD4** — a member of the bromodomain and extraterminal (BET) family — acts as a transcriptional switchboard that keeps cancer cells dividing, making its selective inhibition a high-value target in oncology.

This project builds machine learning models to identify BRD4-binding molecules from **DNA-encoded library (DEL)** data. We work with the **BELKA** dataset from the NeurIPS 2024 Kaggle competition, using a stratified subset of **50,000 molecules** (out of 133 million) that preserves the original ~0.5% active rate.

The central challenge is not just predictive accuracy, but **out-of-distribution generalization**: because DEL molecules are assembled combinatorially from shared building blocks, standard random train/test splits leak structural information and inflate performance. Our solution is a **block-aware OOD split** that mirrors the real-world virtual screening setting, where novel chemistry is the norm.

![Small molecule bound to a protein binding pocket](/images/fit-proteins.jpg)
*A small-molecule ligand docked in a protein binding pocket. The challenge of BRD4 inhibitor discovery is finding the few molecules — out of millions — that fit this pocket with the right chemical complementarity.*

---

## Data

**Source:** BELKA (Big Encoded Library for Chemical Assessment), NeurIPS 2024 Kaggle competition.

**Subset used:** 50,000 molecules targeting BRD4, stratified to preserve the ~0.5% active rate.

**Splits** (constructed by partitioning DEL building blocks into disjoint train/val/test pools):

| Split | # Molecules | # Positives | Positive Rate |
|---|---|---|---|
| Train-in | 34,819 | 119 | 0.34% |
| Val-OOD | 7,268 | 30 | 0.41% |
| Test-OOD | 7,913 | 120 | 1.52% |

![Dataset size per split](/images/fit-splits.png)
*Molecule count per split. The block-aware design ensures OOD samples share no building blocks with training data.*

---

## Methodology

### Molecular Representations

We compared three representation families, each suited to a different modeling approach:

- **ECFP fingerprints** — radius-2 circular fingerprints, 2,048-bit vectors, used with tree-based models.
- **Tokenized SMILES** — canonical SMILES strings, vocabulary built on the training set only, padded and masked for sequence models.
- **Molecular graphs** — atoms as nodes, bonds as edges, with one-hot features for element, charge, aromaticity, bond order, and ring membership.

### Models

We evaluated four model families under identical OOD evaluation protocols:

1. **Random Forest** on ECFP — ensemble of 400–800 trees, `class_weight="balanced_subsample"`, tuned over feature subsampling and depth.
2. **XGBoost** on ECFP — histogram-based gradient boosting, with `scale_pos_weight` set to the negative/positive ratio to counteract imbalance.
3. **SMILES-CNN** — 1D convolutions with kernel sizes (3, 5, 7), 256 filters each, global max pooling, trained with focal loss (α=0.9, γ=2.0).
4. **Graph Neural Network** — 4-layer GINECON with a learned virtual node, trained with weighted binary cross-entropy.

### Evaluation Metric

Given the severe class imbalance, we report **Average Precision (AP)** as the primary metric and **ROC-AUC** as a secondary one. Accuracy is uninformative at ~0.5% positive rate.

---

## Key Findings

### Model Comparison

| Model | Val AP | Test AP | Test ROC-AUC |
|---|---|---|---|
| RF (ECFP) | 0.310 | 0.080 | 0.625 |
| XGB (ECFP) | 0.275 | **0.142** | 0.778 |
| **SMILES-CNN** | **0.394** | **0.329** | **0.818** |
| GNN (PyG) | 0.179 | 0.096 | 0.805 |

![Model comparison on BRD4 (AP)](/images/fit-ap-comparison.png)
*Validation and test Average Precision across the four model families.*

![Model comparison on BRD4 (ROC-AUC)](/images/fit-roc-comparison.png)
*ROC-AUC on the OOD test set.*

### What This Means

- **The SMILES-CNN is the strongest model** under our constraints, achieving **over 20× enrichment** over random ranking (random baseline AP ≈ 0.015).
- **XGBoost outperforms Random Forest** substantially (AP 0.142 vs 0.080), suggesting boosting exploits ECFP structure more effectively under limited data.
- **The GNN achieves high ROC-AUC (0.805) but low AP** — it separates classes well but doesn't concentrate probability mass on the very top-ranked candidates. This is a known trade-off in GNN virtual screening and likely reflects the small training set.
- **Tree-based models do not capture scaffold-level interactions** across building blocks, which is why they fall behind sequence and graph models under the block-aware split.

### Why the Block-Aware Split Matters

Under a random split, complex models would memorize DEL building-block structure rather than learning transferable chemical patterns, yielding misleadingly high validation scores. Our block-aware design exposes this failure mode and forces the models to generalize to genuinely novel chemistry.

![Tree models: validation vs test AP](/images/fit-val-test-scatter.png)
*Gap between validation and test AP for RF and XGB. The larger drop for Random Forest suggests more overfitting to training-block patterns.*

---

## Limitations

- **No ensembling.** Each model was evaluated in isolation. Ensembles of diverse models typically improve virtual screening performance.
- **Limited data.** Hardware constraints (CPU-only, Intel MacBooks) forced us to work with 50k molecules out of 133M. Neural networks — especially the GNN — would likely benefit enormously from more data.
- **Single OOD split.** All results are based on one particular building-block partition, which is more realistic than random splitting but still simplified relative to real screening campaigns.

---

## Future Work

Two extensions are within reach and would meaningfully strengthen the conclusions:

1. **Larger data and GPU training.** Re-running the same pipelines on larger fractions of BELKA, or on related DEL datasets targeting other bromodomains, would let neural networks exploit their full capacity.
2. **UMAP-based OOD splits.** Embedding molecules with ECFP, SMILES, or GNN representations, then clustering with UMAP (as in Guo et al., 2025), yields even more challenging and realistic virtual-screening benchmarks.

---

## Tools

**Languages:** Python  
**ML libraries:** scikit-learn, XGBoost, PyTorch, PyTorch Geometric  
**Chemistry:** RDKit  
**Data:** NumPy, pandas, SciPy (sparse matrices)  
**Visualization:** Matplotlib

---

## Links

- [GitHub Repository](https://github.com/apayanieu/FIT-Proteins)

---

## Publication & Attribution

This work was submitted as a final project for the **Machine Learning** course at IE University, December 2025. All pipeline design, model selection, and analysis were done by the authors. AI tools (ChatGPT, GitHub Copilot, DeepSeek) were used for code suggestions and understanding chemical concepts, with all outputs reviewed and validated by us.
