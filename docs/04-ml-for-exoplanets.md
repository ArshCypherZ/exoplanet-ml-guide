# Part 4: Machine Learning for Exoplanet Detection -- The Complete Guide

> **Prerequisites**: [Parts 1-3](01-astronomy-basics.md). Basic Python and ML familiarity helpful but not required.  
> **What you'll learn**: Some ML techniques used in exoplanet detection, from traditional algorithms to deep learning, with code intuition.

---

## Table of Contents

1. [Why ML for Exoplanets?](#1-why-ml-for-exoplanets)
2. [The Classification Problem](#2-the-classification-problem)
3. [Classical ML Approaches](#3-classical-ml-approaches)
4. [Convolutional Neural Networks (CNNs)](#4-convolutional-neural-networks-cnns)
5. [Recurrent Neural Networks (RNNs/LSTMs)](#5-recurrent-neural-networks-rnnslstms)
6. [Transformers](#6-transformers)
7. [The Class Imbalance Problem](#7-the-class-imbalance-problem)
8. [Training Data Sources](#8-training-data-sources)
9. [Evaluation Metrics](#9-evaluation-metrics)
10. [Loss Functions](#10-loss-functions)
11. [Data Augmentation for Light Curves](#11-data-augmentation-for-light-curves)
12. [Model Deployment (ONNX)](#12-model-deployment-onnx)
13. [Putting It All Together](#13-putting-it-all-together)

---

## 1. Why ML for Exoplanets?

### The Scale Problem

- TESS observes ~200 million stars
- Each star produces a time series with thousands of data points
- The SPOC pipeline flags ~thousands of TCEs per sector
- Manual vetting of each candidate takes an astronomer 10-30 minutes
- **At scale, manual vetting is impossible**

### The Pattern Problem

Distinguishing real planets from false positives requires recognizing subtle patterns:
- Transit shape (U vs V)
- Depth consistency (odd/even transits)
- Secondary eclipses
- Centroid shifts
- Stellar variability patterns

**Humans are good at this -- but slow. ML is fast AND can learn patterns humans might miss.**

### The History

| Year | Milestone |
|------|-----------|
| 2014 | First ML applied to Kepler data (Random Forest) |
| 2018 | AstroNet CNN (Shallue & Vanderburg) -- deep learning breakthrough |
| 2021 | ExoMiner (Valizadegan et al.) -- NASA's official ML vetter |
| 2023 | Transformers applied to TESS FFIs (Salinas et al.) |
| 2024 | ExoMiner++ -- transfer learning Kepler to TESS |

---

## 2. The Classification Problem

At its core, transit vetting is a **binary classification** task:

```
Input: Light curve + metadata
  │
  ▼
┌─────────────────────┐
│    ML CLASSIFIER     │
└──────────┬──────────┘
           │
    ┌──────┴──────┐
    ▼              ▼
 PLANET         FALSE POSITIVE
 (real transit)  (EB, BEB, noise,
                  stellar variability)
```

### What the Model Sees

Depending on the approach:

**Approach 1 -- Feature-based (Classical ML)**:
```
Input: [period=5.2d, depth=1200ppm, duration=2.1h, snr=15.3, 
        odd_even_ratio=1.02, secondary_depth=0ppm, ...]
```

**Approach 2 -- Image-based (Deep Learning)**:
```
Input: Phase-folded light curve as a 1D "image"
       [1.0, 1.0, 0.999, 0.998, ..., 0.988, ..., 0.998, 1.0, 1.0]
        ↑ out-of-transit    ↑ in-transit       ↑ out-of-transit
```

---

## 3. Classical ML Approaches

Before deep learning, astronomers used traditional ML with hand-crafted features.

### 3.1 Feature Engineering

The astronomer extracts meaningful features from the light curve:

| Feature | What It Measures | Formula/Method |
|---------|-----------------|----------------|
| Period | Orbital period | BLS periodogram peak |
| Depth | Planet size | Max(dip) in phase-folded curve |
| Duration | Transit length | Width of BLS box |
| SNR | Signal confidence | depth / (noise / sqrt(N)) |
| Odd-even ratio | EB check | depth_odd / depth_even |
| Secondary depth | EB check | Depth of signal at phase 0.5 |
| Shape metric | U vs V | Kurtosis of transit dip |
| Ingress slope | Transit type | d(flux)/d(time) at contact 1 |

### 3.2 Random Forest

An ensemble of decision trees, each trained on random subsets of features and data.

```
Tree 1:                    Tree 2:                    Tree 3:
  depth > 500ppm?          SNR > 10?                  odd_even < 1.05?
  ├── Yes: duration>1h?    ├── Yes: secondary<50?     ├── Yes: PLANET
  │   ├── Yes: PLANET      │   ├── Yes: PLANET        └── No: EB
  │   └── No: FP           │   └── No: EB
  └── No: FP               └── No: FP

Final: MAJORITY VOTE → PLANET (2/3 agree)
```

**Pros**: Interpretable (we can see which features matter), robust to noise, fast to train.  
**Cons**: Limited by the quality of hand-crafted features. Can't learn new patterns.

### 3.3 XGBoost / Gradient Boosted Trees

Similar to Random Forest but trees are built sequentially, each correcting the errors of the previous one. Often achieves better accuracy.

### 3.4 Why Classical ML Falls Short

- Features are designed by humans -- we might miss important patterns
- Features don't capture the full shape/temporal information
- Different stars have different noise properties -- hand-crafted features may not generalize
- **Deep learning can learn features directly from raw data**

---

## 4. Convolutional Neural Networks (CNNs)

CNNs are the **workhorse** of deep learning for transit classification. They learn to recognize spatial patterns in 1D light curves, just like they recognize edges and shapes in 2D images.

### 4.1 How 1D CNNs Work

A 1D convolution slides a small filter across the light curve, detecting local patterns:

```
Input light curve:   [1.0, 1.0, 0.99, 0.98, 0.98, 0.99, 1.0, 1.0]

Filter (kernel):     [+1, -2, +1]  (edge detector)

Convolution:
  Position 1: 1.0*(+1) + 1.0*(-2) + 0.99*(+1) = -0.01
  Position 2: 1.0*(+1) + 0.99*(-2) + 0.98*(+1) = 0.00
  Position 3: 0.99*(+1) + 0.98*(-2) + 0.98*(+1) = +0.01
  ...

Output: [     -0.01,      0.00,      0.01,     0.00,     0.01,     0.00   ]
         ↑ ingress detected                    ↑ flat bottom    ↑ egress
```

### 4.2 CNN Architecture for Transits

A typical transit classification CNN:

```
Input: Phase-folded light curve (1 x 200 points)
  │
  ▼
┌─────────────────────────────────┐
│ Conv1D(1→32, kernel=7)          │  Detects coarse features
│ BatchNorm + ReLU + MaxPool(2)   │  (200 → 100)
├─────────────────────────────────┤
│ Conv1D(32→64, kernel=5)         │  Detects medium features
│ BatchNorm + ReLU + MaxPool(2)   │  (100 → 50)
├─────────────────────────────────┤
│ Conv1D(64→64, kernel=3)         │  Detects fine features (transit shape)
│ BatchNorm + ReLU + AvgPool(→1)  │  (50 → 1)
├─────────────────────────────────┤
│ Fully Connected (64→64)         │  Feature vector
│ ReLU                            │
├─────────────────────────────────┤
│ Fully Connected (64→1)          │  Planet probability
│ Sigmoid                         │  (0.0 to 1.0)
└─────────────────────────────────┘
```

This is essentially used for finding shapes.

### 4.3 What CNNs Learn

Layer by layer, the CNN learns increasingly abstract features:

```
Layer 1 (Conv 7): Edge detection
  - Finds ingress/egress slopes
  - Detects sharp vs gradual transitions

Layer 2 (Conv 5): Shape patterns
  - Recognizes U-shape vs V-shape
  - Detects flat bottoms vs pointed bottoms

Layer 3 (Conv 3): High-level features
  - Combines edges and shapes into "planet transit" vs "EB eclipse" patterns
  - Detects symmetry of the transit
```

### 4.4 Key Concepts

**BatchNorm**: Normalizes the output of each layer to have mean=0, std=1. This stabilizes training and allows higher learning rates.

**ReLU (Rectified Linear Unit)**: Activation function that outputs max(0, x). Introduces non-linearity (without it, the entire network would be a linear function).

**MaxPool**: Downsamples by taking the maximum value in each window. Reduces dimensionality and makes the model somewhat invariant to small shifts.

**AdaptiveAvgPool**: Reduces the output to a fixed size regardless of input size.

### 4.5 Notable CNN Models

**AstroNet** (Shallue & Vanderburg, 2018):
- First major deep learning paper for transit classification
- Applied to Kepler data
- Validated 2 new planets (Kepler-90i and Kepler-80g)
- Architecture: Two separate CNNs (global + local view) merged with dense layers

**ExoMiner** (Valizadegan et al., 2021):
- NASA's official ML-based vetter
- Validated 301 new planets from Kepler
- Multi-input CNN (uses diagnostic tests as additional inputs)

**ExoMiner++** (2024-2025):
- Transfer learning from Kepler to TESS
- First large-scale ML vetting catalog for TESS 2-minute cadence data

---

## 5. Recurrent Neural Networks (RNNs/LSTMs)

While CNNs detect spatial patterns, RNNs are designed for **sequential/temporal** data -- they process the light curve one point at a time and maintain a "memory" of what they've seen.

### 5.1 Basic RNN

```
Time step:    t=1      t=2      t=3      t=4      ...
Input:        x_1      x_2      x_3      x_4
               ↓        ↓        ↓        ↓
Hidden:       h_1  →   h_2  →   h_3  →   h_4  →  ...
               ↑        ↑        ↑        ↑
              (memory passes from step to step)
```

At each time step, the hidden state is updated:
```
h_t = f(W_h * h_{t-1} + W_x * x_t + b)
```

**Problem**: Vanilla RNNs suffer from the **vanishing gradient problem** -- they can't remember things from many time steps ago. For a light curve with 1000 points, early transits would be "forgotten."

### 5.2 LSTM (Long Short-Term Memory)

LSTMs solve the vanishing gradient problem with a **gating mechanism**:

```
                  ┌──────────────────────────┐
                  │         LSTM Cell          │
                  │                            │
  x_t ──────────►│  forget gate: What to     │
                  │    forget from memory      │
  h_{t-1} ──────►│                            │──► h_t
                  │  input gate: What new     │
  c_{t-1} ──────►│    info to store           │──► c_t
                  │                            │
                  │  output gate: What to     │
                  │    output                  │
                  └──────────────────────────┘
```

Three gates control information flow:
- **Forget gate**: Decides what to throw away from the cell state
- **Input gate**: Decides what new information to store
- **Output gate**: Decides what to output based on the cell state

### 5.3 Bidirectional LSTM

It processes the sequence in BOTH directions:

```
Forward LSTM:   → → → → → → → → → → →
                x_1  x_2  x_3  ...  x_1000

Backward LSTM:  ← ← ← ← ← ← ← ← ← ← ←
                x_1  x_2  x_3  ...  x_1000

Output at each step: [forward_h_t ; backward_h_t]
```

**Why bidirectional?** Transits can be better understood by looking at what comes before AND after. The forward LSTM sees the approach to the transit; the backward LSTM sees the recovery.

### 5.4 Attention Mechanism

**Temporal attention** on top of the LSTM:

```
LSTM outputs: h_1, h_2, h_3, ..., h_1000
                ↓    ↓    ↓         ↓
Attention:    α_1, α_2, α_3, ..., α_1000   (weights, sum to 1)
                ↓    ↓    ↓         ↓
Output:       Σ (α_i * h_i)  =  single attended vector
```

The attention mechanism learns which time steps are most important. For transit detection, it should learn to attend to the transit regions and ignore the flat out-of-transit data.

### 5.5 Why LSTM for Light Curves?

- Light curves are **time series** -- order matters
- LSTMs can detect **periodicity** (repeated transits at regular intervals)
- LSTMs can detect **Transit Timing Variations (TTVs)** -- slight changes in transit timing caused by gravitational interactions between planets
- LSTMs can detect **secondary eclipses** at phase 0.5
- Combined with attention, they can focus on the most informative parts of the curve

---

## 6. Transformers

Transformers are the newest architecture being applied to transit classification. They use **self-attention** to model relationships between ALL pairs of time steps simultaneously.

### 6.1 Self-Attention

Instead of processing sequentially (like RNNs), transformers compare every point to every other point:

```
For each point x_i:
  1. Compute Query (Q_i), Key (K_i), Value (V_i)
  2. Compare Q_i with ALL keys: score_ij = Q_i . K_j
  3. Softmax the scores to get attention weights
  4. Output_i = Σ (weight_ij * V_j)

Result: Each point's output is a weighted combination of ALL other points
```

### 6.2 Why Transformers for Transits?

- **Long-range dependencies**: Can relate the transit at one phase to the out-of-transit behavior at another phase
- **Parallelizable**: Unlike RNNs, all time steps are processed simultaneously (faster training on GPUs)
- **No sequence length limit**: RNNs struggle with very long sequences; transformers handle them natively
- **Position encoding**: Transformers add positional information, which maps naturally to orbital phase

### 6.3 Current State

Transformers for transit classification are still relatively new:
- **Salinas et al. (2023, 2025)**: Applied to TESS FFIs, showed competitive performance with CNNs
- **Research opportunity**: Transformers are underexplored in this domain compared to NLP/vision

### 6.4 Transformer vs CNN vs LSTM

| Feature | CNN | LSTM | Transformer |
|---------|-----|------|-------------|
| Local patterns | Excellent | Good | Good |
| Long-range patterns | Poor (limited by receptive field) | Good (but sequential) | Excellent |
| Training speed | Fast | Slow (sequential) | Fast (parallel) |
| Data efficiency | Good (inductive bias) | Moderate | Poor (needs lots of data) |
| Interpretability | Moderate (filters) | Low | Moderate (attention maps) |
| Current use in exo | Dominant | Growing | Emerging |

---

## 7. The Class Imbalance Problem

This is the **single biggest challenge** in ML for exoplanets.

### The Reality

```
In a typical TESS sector:
- Stars observed:     ~200,000
- Stars with TCEs:    ~2,000   (1%)
- Real planets:       ~20      (0.01%)
- False positives:    ~1,980   (0.99%)

Even among TCEs:
- Planets:            ~1%
- Eclipsing binaries: ~40%
- BG eclipsing binary: ~20%
- Stellar variability: ~20%
- Instrumental:       ~19%
```

### Why It's a Problem

If you train a model that ALWAYS predicts "not a planet," it gets 99% accuracy! But it finds zero planets -- completely useless.

```
Naive model: "Everything is NOT a planet"
- Accuracy: 99%
- Recall (planets found): 0%  ← USELESS
```

### Possible Solutions

**1. Weighted Loss Function (Focal Loss)**

**Focal Loss**, down-weights easy examples (correctly classified non-planets) and up-weights hard examples (misclassified planets).


**2. Weighted Random Sampling**

Ensures each training batch has a balanced number of planets and non-planets, even though the dataset is imbalanced.

**3. Synthetic Data Augmentation**

Can generate fake planet transits injected into real TESS noise backgrounds. This increases the number of positive examples.

**4. Threshold Tuning**

Instead of using 0.5 as the classification threshold, we can optimize it for the desired precision-recall tradeoff.

---

## 8. Training Data Sources

### 8.1 Kepler Labelled Time Series

Kaggle CSV file containing Kepler light curves with confirmed labels:
- **Label 2**: Confirmed planet (positive class)
- **Label 1**: Non-planet (negative class)

This is the most widely used training dataset in the field.

### 8.2 Synthetic Injection

Can create additional training examples:


This is important because:
- Increases positive examples (helps with class imbalance)
- We control the planet parameters (can generate hard-to-detect cases)
- Uses real TESS noise (realistic instrumental/stellar noise patterns)

### 8.3 Example Data Preprocessing for Training

Before feeding data to the model:

```
Raw light curve
     │
     ▼
1. Normalize
     │
     ▼
2. Phase fold at known/detected period
     │
     ▼
3. Generate dual views:
   - Local: 200 points around transit (CNN)
   - Global: 1000 points full orbit (LSTM)
     │
     ▼
4. Compute physics vector:
   - [Period, Duration, Depth, SNR, StarRadius]
   - Normalize: Period/12, Duration*24/5, Depth*50, SNR/30
     │
     ▼
5. Data augmentation:
   - Random jitter (translational invariance)
   - Gaussian noise injection
   - Noisy physics vector (prevent data leakage)
```

---

## 9. Evaluation Metrics

Accuracy alone is meaningless for imbalanced data. Here's what actually matters:

### Confusion Matrix

```
                    Predicted
                 Planet | Not Planet
Actual Planet  |  TP   |    FN     |
Not Planet     |  FP   |    TN     |

TP = True Positive  (real planet, correctly identified)
FP = False Positive (not a planet, incorrectly flagged)
FN = False Negative (real planet, missed!)
TN = True Negative  (not a planet, correctly rejected)
```

### Key Metrics

| Metric | Formula | What It Means | Target |
|--------|---------|---------------|--------|
| **Precision** | TP / (TP + FP) | Of all flagged candidates, how many are real? | >80% |
| **Recall** | TP / (TP + FN) | Of all real planets, how many did we find? | >95% |
| **F1 Score** | 2 * (Precision * Recall) / (P + R) | Harmonic mean of precision and recall | >85% |
| **AUC-ROC** | Area under ROC curve | Overall discrimination ability | >0.95 |
| **False Positive Rate** | FP / (FP + TN) | What fraction of non-planets are flagged? | <5% |

### Why Recall Matters Most

In astronomy, **missing a real planet is worse than flagging a false positive**. A false positive just wastes some follow-up time. A missed planet is a lost discovery.

### ROC Curve

```
True Positive Rate (Recall)
1.0 ─────────────────────────── * (perfect)
    │                        *
    │                     *
    │                  *
    │               *
    │            *
0.5 │         *
    │      *
    │   * ← Random classifier (diagonal)
    │ *
0.0 *──────────────────────────
    0.0        0.5        1.0
          False Positive Rate
```

---

## 10. Loss Functions

### Binary Cross-Entropy (BCE)

The standard loss for binary classification:

```
BCE = -[y * log(p) + (1-y) * log(1-p)]

Where:
  y = true label (0 or 1)
  p = predicted probability
```

**Problem**: Treats all examples equally. With 99% negatives, the model just learns to predict "not planet."

### BCE with Logits

This is numerically more stable than applying sigmoid then BCE separately:

```
BCEWithLogits = -[y * log(sigmoid(z)) + (1-y) * log(1 - sigmoid(z))]

Where z = raw model output (logit), before sigmoid
```

### Focal Loss

Adds a modulating factor to down-weight easy examples:

```
FocalLoss = -alpha * (1-p)^gamma * log(p)       when y=1
          = -(1-alpha) * p^gamma * log(1-p)      when y=0

```
---

## 11. Data Augmentation for Light Curves

Unlike image augmentation (rotation, flipping), light curve augmentation must respect physics:

### Valid Augmentations

| Augmentation | What It Does | Why It Helps |
|-------------|-------------|-------------|
| **Random phase jitter** | Shift the transit slightly in phase | Translational invariance -- model shouldn't depend on exact transit position |
| **Gaussian noise injection** | Add random noise to flux values | Robustness to different noise levels |
| **Transit depth scaling** | Slightly vary the transit depth | Generalize to different planet sizes |
| **Baseline variation** | Add slow sinusoidal trends | Robustness to stellar variability |
| **Noisy physics injection** | Add noise to physics vector values | Prevents the model from "leaking" answers from physics features |

### Invalid Augmentations (Don't Do These!)

| Bad Augmentation | Why It's Wrong |
|-----------------|----------------|
| Flip the light curve vertically | Transits are always dips, never bumps |
| Random time shuffling | Destroys the temporal structure |
| Extreme stretching | Changes the physical transit shape |

---

## 12. Model Deployment (ONNX)

### What is ONNX?

**ONNX** (Open Neural Network Exchange) is a standardized format for ML models. You train in PyTorch, export to ONNX, and run inference anywhere (without needing PyTorch installed).

```
Training (PyTorch)          Deployment (ONNX Runtime)
┌──────────────────┐        ┌──────────────────────┐
│ model.py          │        │ inference.py           │
│ train.py          │ ──►    │ onnxruntime            │
│ GPU, big deps     │ .onnx  │ CPU only, lightweight  │
│ Research          │  file  │ Production             │
└──────────────────┘        └──────────────────────┘
```

### Why ONNX?

1. **Lightweight**: ONNX Runtime is much smaller than PyTorch (~50MB vs ~2GB)
2. **No GPU needed**: Runs on CPU (important for free-tier hosting)
3. **Deterministic**: Same output every time (no training-mode randomness)
4. **Language agnostic**: Could run in C++, C#, Java, not just Python

---

## 13. Putting It All Together

Here's the complete ML pipeline of an example system:

```
┌─────────────────────────────────────────────────────────────┐
│                     TRAINING PHASE                           │
│                                                              │
│  Kepler CSV + Synthetic Injections                          │
│       │                                                      │
│       ▼                                                      │
│  Preprocessing: Normalize → Phase Fold → Dual Views         │
│       │                                                      │
│       ▼                                                      │
│  Augmentation: Jitter + Noise + Physics noise               │
│       │                                                      │
│       ▼                                                      │
│  Training: (PyTorch)                                        │
│    - Focal Loss (gamma=2, alpha=0.75)                       │
│    - AdamW optimizer                                         │
│    - CosineAnnealing LR scheduler                           │
│    - WeightedRandomSampler for balance                      │
│    - 50 epochs, batch size 32                               │
│       │                                                      │
│       ▼                                                      │
│  Export to ONNX                                              │
└──────────────────────────┬──────────────────────────────────┘
                           │
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                    INFERENCE PHASE                            │
│                                                              │
│  New TESS light curve                                        │
│       │                                                      │
│       ▼                                                      │
│  BLS → Period, Depth, Duration, SNR, Dual Views             │
│       │                                                      │
│       ▼                                                      │
│  Preprocessing:                                              │
│    Local view: 200 pts → (1, 1, 200) tensor                │
│    Global view: 1000 pts → (1, 1000) tensor                 │
│    Physics: [P, Dur, Dep, SNR, R] → normalize → (1, 5)     │
│       │                                                      │
│       ▼                                                      │
│  ONNX Runtime:                                               │
│    ┌─────────┐ ┌──────────┐ ┌───────────┐                  │
│    │          │           │ │           │                  │
│    │  (CNN)  │ │ (BiLSTM) │ │  (Dense)  │                  │
│    └────┬────┘ └────┬─────┘ └─────┬─────┘                  │
│         └───────────┼─────────────┘                          │
│                     ▼                                        │
│              ┌──────────┐                                    │
│              │  Fusion   │                                   │
│              │   Head    │                                   │
│              └─────┬────┘                                    │
│                    ▼                                         │
│         confidence: 0.95                                     │
│         shape_score: 0.98                                    │
│         temporal_score: 0.89                                 │
│                    │                                         │
│                    ▼                                         │
│         Verdict: POSITIVE / NEGATIVE / REVIEW               │
└─────────────────────────────────────────────────────────────┘
```

---

## Key Takeaways

1. **CNNs** detect local transit shapes (U vs V)
2. **Bi-LSTMs with attention** capture temporal patterns
3. **Physics injection** adds domain knowledge
4. **Class imbalance** is the #1 challenge -- can be possibly solved via focal loss, weighted sampling, synthetic data
5. **ONNX** enables lightweight deployment without PyTorch
6. **Evaluation**: Recall matters more than accuracy. Missing a real planet is worse than flagging a false positive.

---

## What's Next?

**Part 5: Research Landscape** -- What papers have been published, what's the state of the art, and where are the gaps?
