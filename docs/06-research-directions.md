# Part 6: Research Directions -- Detailed Analysis of Paper-Worthy Angles

> **Goal**: Help you choose the most promising direction for a publishable paper.

---

## Table of Contents

1. [Direction Evaluation Framework](#1-direction-evaluation-framework)
2. [Direction A: Explainable AI (XAI) for Transit Classification](#2-direction-a-explainable-ai-xai-for-transit-classification)
3. [Direction B: Physics-Informed Neural Networks](#3-direction-b-physics-informed-neural-networks)
4. [Direction C: Multi-Mission Transfer Learning](#4-direction-c-multi-mission-transfer-learning)
5. [Direction D: Anomaly Detection for Novel Phenomena](#5-direction-d-anomaly-detection-for-novel-phenomena)
6. [Direction E: Atmospheric Prediction from Photometry](#6-direction-e-atmospheric-prediction-from-photometry)

---

## 1. Direction Evaluation Framework

For each direction, we'll assess:

| Criterion | What It Means |
|-----------|--------------|
| **Novelty** | Has this been done before? How different is it from published work? |
| **Feasibility** | Can we do this with our current skills, and resources? |
| **Impact** | Would astronomers and ML researchers care? |
| **Data availability** | Is the required data publicly accessible? |
| **Publication target** | Which journals/conferences would accept this? |

---

## 2. Direction A: Explainable AI (XAI) for Transit Classification

### The Pitch

"We don't just tell you IF a signal is a planet -- we show you WHY the model thinks so."

### Why It Matters

Astronomers are scientists. They need to understand the reasoning behind a classification to trust it. A black-box model that says "95% planet" is useless if the astronomer can't verify the reasoning.

Currently, most transit classifiers provide a single probability score.

### What We can Do

#### 1. Grad-CAM for the CNN

Gradient-weighted Class Activation Mapping shows which parts of the input the CNN focuses on.

```
Local View Input:  [1.0, 1.0, 0.99, 0.98, 0.97, 0.98, 0.99, 1.0, 1.0]

Grad-CAM Heatmap:  [0.0, 0.1, 0.6,  0.9,  1.0,  0.9,  0.6,  0.1, 0.0]
                                ↑           ↑           ↑
                          ingress      bottom       egress
                     (model focuses on transit shape)
```

For a planet: the model should focus on the U-shaped transit region.
For an EB: the model should focus on the V-shape and potentially the secondary eclipse.

#### 2. LSTM Attention Visualization

```
Global View:     [..., 1.0, 1.0, 0.98, 0.97, 0.98, 1.0, ..., 1.0, 0.98, ...]
                                  ↑ transit 1                    ↑ transit 2

Attention Wts:   [..., 0.01, 0.01, 0.15, 0.20, 0.15, 0.01, ..., 0.01, 0.15, ...]
                                    ↑ HIGH attention              ↑ HIGH attention
```

The attention should peak at transit locations and be low during flat out-of-transit regions.


#### 3. SHAP Values for Physics Features

SHAP (SHapley Additive exPlanations) shows how each physics feature contributes to the prediction.

```
Physics input: [Period=5.2d, Duration=2.1h, Depth=1200ppm, SNR=15.3, R_star=0.4]

SHAP output:
  Period=5.2d    ──────────→ +0.05  (slightly increases confidence)
  Duration=2.1h  ───→ +0.02         (slightly increases)
  Depth=1200ppm  ──────────────→ +0.12 (STRONGLY increases -- deep transit)
  SNR=15.3       ────────────→ +0.10   (high SNR = real signal)
  R_star=0.4     ──→ +0.03            (small star = deeper relative transit)
  
  Base value: 0.50 → Final: 0.82
```

#### 4. Failure Case Analysis

Run the model on known tricky cases (grazing transits, diluted EBs, stellar variability) and show where/why it fails using the above techniques.

### Assessment

| Criterion | Rating | Notes |
|-----------|--------|-------|
| Novelty | HIGH | No paper has done comprehensive XAI for a hybrid CNN+LSTM transit classifier |
| Feasibility | HIGH | With component scores and attention. Just need visualization |
| Impact | HIGH | Directly addresses astronomers' #1 complaint about ML |
| Data availability | HIGH | Use existing TESS/Kepler data |
| Publication target | AJ, MNRAS, ML4Astro workshop | Good fit for astronomy journals |

---

## 3. Direction B: Physics-Informed Neural Networks

### The Pitch

"We embed astrophysical knowledge directly into the neural network architecture and training, creating models that can't violate physical laws."

### What we can Do

#### 1. Physics-Informed Loss Function

Add penalty terms to the loss that enforce physical constraints:

```
    # Standard classification loss
    # Physics constraint 1: Duration can't exceed half the period
    # Physics constraint 2: Planet radius can't be negative or > 2 Jupiter radii
    # Physics constraint 3: Transit depth should match expected planet/star ratio
    # Combined loss
```

#### 2. Differentiable Transit Model Layer

Instead of using a simple dense network for physics features, embed the actual transit model (Mandel & Agol 2002) as a differentiable PyTorch layer.

#### 3. Physics-Constrained Inference

During inference, reject predictions that violate physical laws even if the neural network says "planet".

### Assessment

| Criterion | Rating | Notes |
|-----------|--------|-------|
| Novelty | VERY HIGH | True PINNs haven't been applied to transit classification |
| Feasibility | MEDIUM | Requires careful mathematical formulation and debugging |
| Impact | HIGH | Bridges the gap between ML and physics communities |
| Publication target | NeurIPS workshop, AJ, MNRAS | Appealing to both ML and astro |

---

## 4. Direction C: Multi-Mission Transfer Learning

### The Pitch

"We develop a systematic framework for transferring exoplanet detection models across space missions with different instruments, cadences, and noise characteristics."

### What we can Do

#### 1. Domain Adaptation Study

Systematically measure how well our model transfers between missions:

```
Train on Kepler → Test on Kepler    (baseline)
Train on Kepler → Test on TESS     (cross-mission)
Train on TESS   → Test on TESS    (baseline)
Fine-tune Kepler model on TESS     (transfer learning)
```

#### 2. Domain-Adversarial Training

Train the model to extract features that are mission-invariant:

```
                          ┌──────────────────┐
Light Curve ────────────► │  Feature Extractor │
                          └────────┬─────────┘
                                   │
                    ┌──────────────┤──────────────┐
                    ▼              │              ▼
            ┌──────────────┐      │       ┌──────────────┐
            │  Planet/FP    │      │       │ Mission      │
            │  Classifier   │      │       │ Discriminator│
            │  (maximize)   │      │       │ (minimize)   │
            └──────────────┘      │       └──────────────┘
                                  │
                  The feature extractor learns to CONFUSE
                  the mission discriminator → mission-invariant
```

#### 3. PLATO Readiness Assessment

Simulate PLATO-like data (longer baselines, different cadence) from existing Kepler/TESS data and test transfer.

### Assessment

| Criterion | Rating | Notes |
|-----------|--------|-------|
| Novelty | MEDIUM-HIGH | ExoMiner++ did basic transfer; systematic study is new |
| Feasibility | HIGH | Uses existing public data from both missions |
| Impact | HIGH | Directly useful for the community (PLATO is launching soon) |
| Data availability | HIGH | Kepler and TESS data are public |
| Publication target | AJ, MNRAS, ICML workshop | Both communities interested |

---

## 5. Direction D: Anomaly Detection for Novel Phenomena

### The Pitch

"We use unsupervised learning to find the weirdest light curves in TESS data -- potential new types of astronomical objects that don't fit any known category."

### What we can Do

#### 1. Variational Autoencoder (VAE) for Light Curves

Train a VAE to reconstruct "normal" light curves. Anything it can't reconstruct well is anomalous.

```
Normal star:     Input → [Encoder] → Latent → [Decoder] → Reconstruction
                 Reconstruction error: LOW (learned pattern)

Anomalous star:  Input → [Encoder] → Latent → [Decoder] → Reconstruction
                 Reconstruction error: HIGH (never seen this before!)
```

#### 2. Types of Anomalies to Search For

| Anomaly | What It Looks Like | Significance |
|---------|-------------------|-------------|
| **Exomoons** | TTVs (transit timing variations) + additional shallow dips | No confirmed exomoon yet! |
| **Exocomets** | Asymmetric, variable-depth dips | Only a few candidates known |
| **Disintegrating planets** | Asymmetric transits with variable depth and tail | Very rare (e.g., K2-22b) |
| **Trojan planets** | Two transit signals at same period, 60 degrees apart | Never observed |
| **Circumplanetary rings** | Extended ingress/egress, specific ring geometry | One candidate (J1407b) |
| **Star-planet interactions** | Phase-dependent variability correlated with planet orbit | Barely studied |

#### 3. Potential Discovery Impact

Finding ANY of the above would be a high-impact discovery paper. Exomoons in particular are the "holy grail" -- no confirmed detection exists despite decades of searching.

### Assessment

| Criterion | Rating | Notes |
|-----------|--------|-------|
| Novelty | VERY HIGH | Almost no work on unsupervised anomaly detection in TESS |
| Feasibility | MEDIUM | Requires processing large amounts of FFI data |
| Impact | POTENTIALLY VERY HIGH | Actual discovery = instant high-impact paper |
| Data availability | HIGH | TESS FFIs are public (but large) |
| Publication target | Nature Astronomy (if discovery), AJ, ApJ | Discovery → top journal |

**Risk**: High variance. Could find amazing things, or could find nothing interesting.

---

## 6. Direction E: Atmospheric Prediction from Photometry

### The Pitch

"Can we predict what JWST will see in a planet's atmosphere, using only the transit light curve data that's already available?"

### What we can Do

#### 1. Build a Prediction Model

```
Input: Transit features (depth, duration, period, stellar params)
  │
  ▼
ML Model (Regression, not classification)
  │
  ▼
Output: Predicted atmospheric properties
  - Scale height
  - Cloud coverage probability
  - Expected molecular detections (H2O, CO2, CH4)
  - Predicted JWST observation time needed
```

#### 2. Training Data

Use the ~100 exoplanets where JWST has already measured atmospheric spectra as training labels. Map transit photometric features → atmospheric properties.

#### 3. Validation

For planets where JWST data exists but wasn't used in training: predict the atmosphere from photometry alone, then compare with actual JWST results.

### Assessment

| Criterion | Rating | Notes |
|-----------|--------|-------|
| Novelty | VERY HIGH | Almost no work on this specific prediction task |
| Feasibility | LOW-MEDIUM | Very small training set (~100 planets with JWST spectra) |
| Impact | VERY HIGH | Would save massive amounts of JWST observing time |
| Data availability | MEDIUM | JWST spectra are public but processing is complex |
| Publication target | Nature Astronomy, ApJ Letters | High-profile if it works |

**Risk**: Very small training set. May need creative approaches (few-shot learning, physics priors).

---

## What's Next?

**Part 7: Paper Writing Guide** -- How to structure an ML+astronomy research paper, from title to references.