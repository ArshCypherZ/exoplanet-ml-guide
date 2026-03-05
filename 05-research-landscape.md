# Part 6: Research Landscape -- What's Been Done and Where Are the Gaps

> **Prerequisites**: [Parts 1-4](01-astronomy-basics.md).  
> **What you'll learn**: Key published papers, the current state of the art, what has NOT been done yet.

---

## Table of Contents

1. [Timeline of Key Papers](#1-timeline-of-key-papers)
2. [Foundational Papers](#2-foundational-papers)
3. [State of the Art (2024-2025)](#3-state-of-the-art-2024-2025)
4. [What Has Been Done](#4-what-has-been-done)
5. [The Gaps -- What's Missing](#5-the-gaps----whats-missing)
6. [Reading List (Priority Order)](#6-reading-list-priority-order)

---

## 1. Timeline of Key Papers

```
2014  ──  First ML applied to Kepler (McCauliff et al.) - Random Forest
          │
2017  ──  Autovetter (Catanzarite 2017) - Automated transit vetting
          │
2018  ──  AstroNet (Shallue & Vanderburg) - CNN, validated 2 new planets  ◄── LANDMARK
          │
2018  ──  TESS launches
          │
2019  ──  Ansdell et al. - CNN on Kepler with global+local view
          │
2020  ──  Osborn et al. - Random Forest for TESS (NGTS follow-up)
          │
2021  ──  ExoMiner (Valizadegan et al.) - NASA's ML vetter         ◄── LANDMARK
          │    [Validated 301 new Kepler planets]
          │
2022  ──  ExoMiner v2 - Multi-task learning, diagnostic features
          │
2023  ──  Salinas et al. - Transformers for TESS FFIs               ◄── EMERGING
          │
2024  ──  ExoMiner++ - Transfer learning Kepler→TESS                ◄── LATEST
          │    [First large-scale ML catalog for TESS 2-min cadence]
          │
2024  ──  Gorchakova & Creaner - Synthetic augmentation for transits
          │
2025  ──  CNN+RNN hybrids for Kepler DR25
          │
2025  ──  Trehan et al. - Bayesian ML for atmospheric prediction
```

---

## 2. Foundational Papers

### Paper 1: AstroNet (Shallue & Vanderburg, 2018)

**Title**: "Identifying Exoplanets with Deep Learning: A Five Planet Resonant Chain around Kepler-80 and an Eighth Planet around Kepler-90"

**Why it matters**: This was the paper that proved deep learning could find planets that humans missed. Google Brain + Harvard collaboration.

**Architecture**:
```
Global View (2001 pts) → CNN → Feature Vector ─┐
                                                 ├── Dense Layers → Planet Probability
Local View (201 pts) → CNN → Feature Vector  ───┘
```

**Key contributions**:
- Dual-view approach (global + local)
- Validated Kepler-90i (8th planet in a system!) and Kepler-80g
- Showed CNNs outperform hand-crafted features

**Limitations**:
- No physics injection (purely data-driven)
- No temporal modeling (CNN only, no LSTM)
- No attention mechanism
- Trained and tested only on Kepler

### Paper 2: ExoMiner (Valizadegan et al., 2021)

**Title**: "ExoMiner: A Highly Accurate and Explainable Deep Learning Classifier That Validates 301 New Exoplanets"

**Why it matters**: NASA's official ML-based planet vetter. Used to validate more planets than any other ML system.

**Architecture**:
```
Transit View → CNN ──────────────────────┐
Odd/Even View → CNN ─────────────────────┤
Secondary Eclipse View → CNN ────────────┤
                                          ├── Dense → Planet Prob
Diagnostic Features → Dense ─────────────┤
  (centroid shift, crowding, etc.)       │
Stellar Parameters → Dense ──────────────┘
```

**Key contributions**:
- Multi-input design with multiple diagnostic views
- Used centroid analysis (spatial information) -- detects background EBs
- Explainability through per-view scores
- Validated 301 new confirmed planets

**Limitations**:
- Kepler-specific (different noise than TESS)
- No temporal (LSTM) component
- Complex pipeline with many specialized inputs

### Paper 3: ExoMiner++ (2024)

**Title**: Transfer learning from Kepler to TESS

**Key contribution**: Showed that models trained on Kepler can be adapted to TESS through fine-tuning, producing the first large-scale ML vetting catalog for TESS 2-minute cadence data.

**Limitation**: Transfer wasn't perfect -- TESS's shorter baselines and different systematics cause performance degradation.

---

## 3. State of the Art (2024-2025)

### What the Best Systems Look Like

| System | Architecture | Data | Precision | Recall | Unique Feature |
|--------|-------------|------|-----------|--------|----------------|
| ExoMiner++ | Multi-CNN | Kepler→TESS | ~95% | ~93% | Transfer learning |
| Salinas 2023 | Transformer | TESS FFI | ~90% | ~88% | Self-attention |
| CNN+RNN 2025 | CNN + BiLSTM | Kepler DR25 | ~92% | ~95% | Hybrid temporal |

### Current Benchmarks

On Kepler DR25 test set:
- **ExoMiner**: AUC-ROC = 0.98, Precision = 95%, Recall = 93%
- **AstroNet**: AUC-ROC = 0.96, Precision = 90%, Recall = 88%
- **Random Forest (baseline)**: AUC-ROC = 0.92, Precision = 85%, Recall = 80%

On TESS data (less standardized benchmarks):
- **ExoMiner++**: Precision ~93%, Recall ~90% (fine-tuned)
- Most other models haven't been rigorously benchmarked on TESS

---

## 4. What Has Been Done

### Well-Explored Areas

| Area | Status | What's been done |
|------|--------|-----------------|
| CNN for transit classification | Mature | AstroNet, ExoMiner, dozens of papers |
| Random Forest for transit vetting | Mature | Multiple papers, well-understood |
| BLS for signal detection | Very mature | Standard algorithm, widely used |
| Kepler data classification | Saturated | Little room for improvement |
| Hot Jupiter detection | Saturated | Easy targets, all found |
| Synthetic transit generation | Growing | Several papers on physics-informed generation |
| Transfer learning (Kepler→TESS) | Active | ExoMiner++ leading, still room for improvement |

### What This Means for us

Basic exoplanet detection with ML is well-trodden ground. A paper that just says "we used a CNN to classify TESS transits" will struggle to get published because dozens of groups have done similar work.

**We need a unique angle.**

---

## 5. The Gaps -- What's Missing

These are the areas where our research could make a genuine contribution:

### Gap 1: Explainable AI (XAI) for Transit Classification

**The problem**: Deep learning models are black boxes. Astronomers don't trust classifications they can't understand. A model says "planet with 95% confidence" -- but WHY?

**Current state**: ExoMiner has per-view scores (some explainability). But no one has done deep XAI analysis (Grad-CAM, SHAP, attention visualization) specifically for transit classifiers.

**Opportunity**: Can make model which has component scores (shape_score, temporal_score). and could extend this with:
- Grad-CAM on the CNN to visualize which parts of the transit shape the model focuses on
- Attention weight visualization from the LSTM to show which time steps matter
- SHAP values for the physics features to show how BLS parameters influence the decision

**Novelty level**: HIGH. No paper has done comprehensive XAI for a hybrid transit classifier.

### Gap 2: Physics-Informed Neural Networks (PINNs) for Transit Detection

**The problem**: Most ML models are purely data-driven -- they learn from labeled examples but don't "know" physics. When they encounter unusual situations (rare planet types, unusual stars), they fail silently.

**Opportunity**: 
- Physics-informed loss functions (penalize predictions that violate Kepler's laws)
- Embedding transit models (Mandel & Agol 2002) as differentiable layers
- Using physics constraints during inference (reject candidates that violate physical laws, even if the CNN says "planet")

**Novelty level**: HIGH. True PINNs haven't been applied to transit classification.

### Gap 3: Multi-Mission Transfer Learning

**The problem**: Models trained on Kepler don't work well on TESS (different noise, cadence, baseline). Future missions (PLATO, Roman) will have yet different characteristics.

**Current state**: ExoMiner++ showed basic transfer learning works. But the methodology is ad-hoc.

**Opportunity**: Systematic study of domain adaptation techniques:
- Feature alignment between missions
- Adversarial domain adaptation
- Multi-task learning across missions simultaneously
- Zero-shot transfer to PLATO (before it launches)

**Novelty level**: MEDIUM-HIGH.

### Gap 4: Anomaly Detection for Novel Phenomena

**The problem**: All current classifiers are trained on KNOWN categories (planet vs. EB). But what about phenomena we haven't categorized yet?

**Current state**: Almost no work on unsupervised/semi-supervised anomaly detection in TESS light curves.

**Opportunity**:
- Autoencoders trained on "normal" light curves -- flag anything unusual
- Potential discoveries: exomoons, exocomets, disintegrating planets, Boyajian's-Star-like dippers

**Novelty level**: HIGH. Very few papers on this for TESS.

### Gap 5: Atmospheric Prediction from Photometry

**The problem**: Can we predict what a planet's atmosphere looks like BEFORE JWST observes it, using just transit photometric data?

**Current state**: Trehan et al. (2025) started this with a Bayesian ML framework, but it's very early.

**Opportunity**: Can use transit light curve features (depth, duration, wavelength-dependent depth if available) to predict atmospheric properties (composition, cloud coverage, scale height).

**Novelty level**: VERY HIGH. Extremely new area.

### Gap 6: Robust Evaluation and Benchmarking

**The problem**: Every paper uses different datasets, different preprocessing, different train/test splits, and different metrics. It's nearly impossible to fairly compare methods.

**Current state**: No standardized benchmark exists for TESS transit classification.

**Opportunity**: Create a standardized benchmark dataset and evaluation protocol for TESS. This would be a valuable community resource paper.

**Novelty level**: MEDIUM (but HIGH impact).

---

## 6. Reading List (Priority Order)

### Must-Read (Start Here)

1. **Shallue & Vanderburg (2018)** - "Identifying Exoplanets with Deep Learning"
   - The paper that started it all.
   - arXiv: 1712.05044

2. **Valizadegan et al. (2022)** - "ExoMiner: A Highly Accurate and Explainable Deep Learning Classifier"
   - NASA's state of the art.
   - arXiv: 2111.10009

3. **Pearson et al. (2018)** - "Searching for Exoplanets using Artificial Intelligence"
   - Good overview of ML approaches for transit detection.
   - arXiv: 1706.04319

### Important Context

4. **Kovacs et al. (2002)** - "A box-fitting algorithm in the search for periodic transits"
   - The BLS algorithm paper.

5. **Kempton et al. (2018)** - "A Framework for Prioritizing the TESS Planetary Candidates Most Amenable to Atmospheric Characterization"
   - TSM formula.

6. **Kopparapu et al. (2013)** - "Habitable Zones Around Main-Sequence Stars"
   - Habitable zone calculation.

### Cutting Edge (For Our Research Angle)

7. **ExoMiner++ (2024)** - Transfer learning Kepler→TESS
   - arXiv: 2601.14877

8. **Salinas et al. (2023/2025)** - "Exoplanet Transit Candidate Identification via a Transformer-Based Algorithm"
   - Transformers for TESS.

9. **Trehan et al. (2025)** - Bayesian ML for atmospheric prediction
   - New frontier.

10. **Gorchakova & Creaner (2024)** - Synthetic augmentation for transit detection

### For XAI Direction

11. **Selvaraju et al. (2017)** - "Grad-CAM: Visual Explanations from Deep Networks"
    - The Grad-CAM technique.

12. **Lundberg & Lee (2017)** - "A Unified Approach to Interpreting Model Predictions" (SHAP)
    - SHAP values for feature importance.

### How to Find Papers

- **arXiv** (arxiv.org): Preprint server. All astronomy and ML papers appear here first. Search for "exoplanet machine learning" or "transit classification deep learning."
- **NASA ADS** (ui.adsabs.harvard.edu): The astronomy-specific paper search engine. More focused than Google Scholar.
- **Google Scholar**: Good for finding citation counts and related papers.
- **Semantic Scholar**: Good for finding the most influential papers in a topic.

---

## Key Takeaways

1. **CNN-based transit classification is mature** -- we can't publish "just another CNN" paper
2. **The biggest gaps** are in XAI, physics-informed ML, anomaly detection, and multi-mission transfer
3. **We need a clear unique angle** and rigorous evaluation to publish
4. **Read the AstroNet and ExoMiner papers** first -- they're our direct predecessors

---

## What's Next?

**Part 6: Research Directions** -- Detailed analysis of each potential research direction, with feasibility assessment and implementation roadmap.