# Part 7: Paper Writing Guide -- From Research to Publication

> **Goal**: A practical guide on how to write and publish your ML + astronomy research paper.  
> **Audience**: First-time paper writers. No prior publishing experience assumed.

---

## Table of Contents

1. [How Academic Publishing Works](#1-how-academic-publishing-works)
2. [Choosing a Venue](#2-choosing-a-venue)
3. [Paper Structure](#3-paper-structure)
4. [Writing Each Section](#4-writing-each-section)
5. [Figures and Visualizations](#5-figures-and-visualizations)
6. [Common Mistakes to Avoid](#6-common-mistakes-to-avoid)
7. [The Review Process](#7-the-review-process)
8. [Tools and Resources](#8-tools-and-resources)

---

## 1. How Academic Publishing Works

### The Pipeline

```
Your Research
     │
     ▼
Write Paper (LaTeX + Figures)
     │
     ▼
Post to arXiv (preprint -- establishes priority)
     │
     ▼
Submit to Journal/Conference
     │
     ▼
Peer Review (1-3 anonymous reviewers)
     │
     ├── Accept → Published!
     ├── Minor Revision → Fix small issues → Accept
     ├── Major Revision → Significant changes → Re-review
     └── Reject → Try another venue
```

### Key Terms

| Term | Meaning |
|------|---------|
| **arXiv** | Free preprint server. Most astronomy/ML papers go here FIRST, before journal submission |
| **Peer review** | Anonymous experts evaluate your paper for correctness, novelty, and significance |
| **Open access** | Paper is freely available to everyone (most astronomy journals are OA) |
| **Impact factor** | A metric of how often papers in a journal are cited. Higher = more prestigious |
| **DOI** | Digital Object Identifier -- permanent URL for your published paper |
| **BibTeX** | Citation format used in LaTeX |

### The arXiv Strategy

In astronomy and ML, posting to arXiv is standard practice:
1. Write your paper
2. Post to arXiv (this establishes your priority -- proves you did it first)
3. Submit to a journal simultaneously
4. The journal reviews it while it's already publicly available on arXiv
5. Most astronomers read arXiv daily and may cite your paper even before it's formally published

---

## 2. Choosing a Venue

### Astronomy Journals

| Journal | Abbreviation | Impact Factor | Review Time | Best For |
|---------|-------------|---------------|-------------|----------|
| **The Astronomical Journal** | AJ | ~5.5 | 2-4 months | Methods papers, catalogs |
| **The Astrophysical Journal** | ApJ | ~4.8 | 2-4 months | Results-heavy papers |
| **Monthly Notices of RAS** | MNRAS | ~4.8 | 1-3 months | Broad astronomy topics |
| **Astronomy & Astrophysics** | A&A | ~5.4 | 2-4 months | European-leaning |
| **Astronomy & Computing** | A&C | ~1.9 | 1-3 months | ML/computational methods |
| **Publications of the ASP** | PASP | ~3.3 | 2-3 months | Instrumentation + methods |

### ML Conferences/Workshops

| Venue | Type | Deadline | Best For |
|-------|------|----------|----------|
| **NeurIPS ML4PS** | Workshop | September | ML + Physical Sciences |
| **ICML ML4Astro** | Workshop | March | ML for Astrophysics |
| **ICLR** | Conference | October | If your ML contribution is strong |
| **AAAI** | Conference | August | Applied AI |

---

## 3. Paper Structure

The standard structure for an ML + astronomy paper:

```
1. Title
2. Abstract (~200 words)
3. Introduction (1.5-2 pages)
4. Related Work (1-1.5 pages)
5. Data (1-1.5 pages)
6. Methods (2-3 pages)
7. Experiments & Results (2-3 pages)
8. Discussion (1-1.5 pages)
9. Conclusion (0.5-1 page)
10. Acknowledgments
11. References
12. Appendix (optional)
```

Total: 12-18 pages (typical for AJ/MNRAS)

---

## 4. Writing Each Section (Example)

### 4.1 Title

**Formula**: [Method Name]: [What it does] for [Domain Application]

**Examples**:
- "XAI-Transit: An Explainable Physics-Informed Neural Network for TESS Exoplanet Classification"
- "PhysNet: Physics-Constrained Deep Learning for Transit Signal Vetting in TESS Photometry"
- "Explaining the Machine: Interpretable Exoplanet Detection with Multi-Head Neural Networks"

**Tips**:
- Include key differentiators (explainable, physics-informed, multi-head)
- Mention the mission (TESS) for discoverability
- Keep it under 15 words if possible

### 4.2 Abstract

**Structure** (one sentence each):

1. **Problem**: "Transit-based exoplanet detection suffers from high false positive rates and opaque ML classifiers."
2. **Gap**: "Existing deep learning approaches lack physical grounding and interpretability."
3. **Method**: "We present [name], a hybrid CNN + Bi-LSTM + physics-prior network with physics-informed loss functions and multi-level explainability."
4. **Results**: "Our model achieves X% precision and Y% recall on [dataset], competitive with ExoMiner, while providing Grad-CAM and attention-based explanations."
5. **Impact**: "Explainability analysis reveals the model uses physically meaningful transit features, building trust for deployment in large-scale surveys."

**Length**: 150-250 words.

### 4.3 Introduction

**Structure** (4-5 paragraphs):

1. **Context**: Exoplanets are everywhere. TESS is producing massive data. We need automated classification.
2. **Problem**: Current methods are either (a) traditional algorithms with limited accuracy or (b) black-box deep learning with limited trust.
3. **What exists**: AstroNet, ExoMiner, etc. Brief overview of prior work.
4. **The gap**: No existing approach combines (a) hybrid architecture, (b) physics-informed training, and (c) comprehensive explainability.
5. **Our contribution**: We present [model name]. Our key contributions are: (1) ..., (2) ..., (3) .... We release our code and trained model.

**Tips**:
- End the Introduction with a clear list of contributions (bullet points)
- Don't oversell -- be precise about what's new
- Cite relevant prior work even in the Introduction

### 4.4 Related Work

**Organize by category**:

1. **Traditional transit detection**: BLS (Kovacs 2002), TLS (Hippke & Heller 2019)
2. **Classical ML for vetting**: Random Forest (McCauliff 2014), feature engineering approaches
3. **Deep learning for vetting**: AstroNet (2018), ExoMiner (2021), ExoMiner++ (2024)
4. **Transformers**: Salinas et al. (2023, 2025)
5. **Explainable AI in astronomy**: (sparse literature -- note the gap!)
6. **Physics-informed neural networks**: PINNs in other domains, limited application in exoplanets

**End with**: "To our knowledge, no prior work has combined a multi-head hybrid architecture with physics-informed training and comprehensive explainability analysis for transit classification."

### 4.5 Data

**Must include**:
- What dataset you used (Kepler DR25, TESS TOI catalog, etc.)
- How you preprocessed it (normalization, phase folding, view generation)
- Train/validation/test split (with exact numbers)
- Class distribution (N planets, N false positives)
- Any data augmentation (synthetic injection, jitter)

**Example table**:

```
| Split     | Planets | False Positives | Total |
|-----------|---------|----------------|-------|
| Train     | 2,500   | 25,000         | 27,500|
| Validation| 500     | 5,000          | 5,500 |
| Test      | 500     | 5,000          | 5,500 |
```

### 4.6 Methods

**Structure by component**:

1. **Architecture overview** (diagram + description)
2. **ShapeExpert (CNN)**: Input, layers, output, what it detects
3. **SequenceExpert (Bi-LSTM)**: Input, layers, attention, output
4. **PhysicsPrior**: Input features, normalization, output
5. **Fusion Head**: Concatenation, dense layers, output heads
6. **Physics-informed loss**: Standard loss + constraint terms, lambda values
7. **Training details**: Optimizer, LR schedule, epochs, batch size, augmentation
8. **Explainability methods**: Grad-CAM, attention extraction, SHAP

**Include an architecture diagram** (most important figure in the paper).

### 4.7 Experiments & Results

**Must include**:

1. **Comparison with baselines**:

```
| Model            | Precision | Recall | F1    | AUC-ROC |
|-----------------|-----------|--------|-------|---------|
| Random Forest    | 85.2%     | 80.1%  | 82.6% | 0.92    |
| AstroNet (CNN)   | 90.3%     | 88.5%  | 89.4% | 0.96    |
| Ours (no physics)| 91.7%     | 90.2%  | 90.9% | 0.97    |
| Ours (full)      | 93.1%     | 92.4%  | 92.7% | 0.98    |
```

2. **Ablation study** (remove one component at a time):

```
| Variant                        | AUC-ROC | Delta |
|-------------------------------|---------|-------|
| Full model                     | 0.98    | --    |
| Without LSTM                   | 0.96    | -0.02 |
| Without Physics Prior          | 0.97    | -0.01 |
| Without Physics-informed Loss  | 0.975   | -0.005|
| CNN only (AstroNet-like)       | 0.96    | -0.02 |
```

3. **XAI analysis**:
   - Grad-CAM figures showing what the CNN focuses on
   - Attention maps showing which time steps the LSTM prioritizes
   - SHAP plots showing physics feature importance
   - Examples for correctly and incorrectly classified cases

4. **Application to real TESS targets**:
   - Run on your cached targets (WASP-18b, TOI-700, Pi Mensae, EB)
   - Show the pipeline correctly identifies known planets and rejects the EB

### 4.8 Discussion

**Cover**:

1. **What the model learns**: XAI analysis shows the model focuses on physically meaningful features (ingress/egress shape, periodicity, depth-radius consistency)
2. **Edge cases**: Where the model struggles (grazing transits, blended systems, very shallow signals)
3. **Physics-informed benefits**: How the constraint loss improves performance on physically ambiguous cases
4. **Limitations**: 
   - Single-sector data limitations (27-day baseline)
   - No centroid analysis (can't detect background EBs from photometry alone)
   - Small training set for TESS-specific features
5. **Future work**: Transfer learning, anomaly detection, atmospheric prediction

### 4.9 Conclusion

**3-4 sentences**:
1. We presented [model name], a [brief description]
2. Key findings: [1-2 most important results]
3. The XAI analysis demonstrates [key insight about what the model learns]
4. Our code and model are publicly available at [GitHub URL]

---

## 5. Figures and Visualizations

### Must-Have Figures

| Figure | What It Shows | Section |
|--------|-------------|---------|
| **Architecture diagram** | Full model with 3 heads + fusion | Methods |
| **Training curves** | Loss and accuracy vs. epochs | Results |
| **ROC curve** | Your model vs. baselines | Results |
| **Confusion matrix** | TP, FP, FN, TN breakdown | Results |
| **Grad-CAM examples** | CNN attention on transit shapes | Results/XAI |
| **Attention maps** | LSTM attention over time series | Results/XAI |
| **SHAP summary** | Physics feature importance | Results/XAI |
| **TESS target examples** | Pipeline output for known planets/EBs | Results |

### Figure Design Tips

- Use **matplotlib** or **plotly** for consistency
- Use a consistent color scheme throughout
- Always include axis labels with units
- Use colorblind-friendly palettes (viridis, etc.)
- Keep figures clean -- no gridlines unless necessary
- Resolution: 300 DPI minimum for print

### Architecture Diagram

Create this using draw.io, tikz (LaTeX), or matplotlib:

```
┌─────────────────────────────────────────────────────────┐
│                   XAI-Transit Architecture                │
│                                                          │
│  Local View          Global View        Physics Vector   │
│  (1×200)             (1×1000)            (5)            │
│     │                    │                  │            │
│     ▼                    ▼                  ▼            │
│ ┌─────────┐      ┌────────────┐     ┌──────────┐       │
│ │  Shape   │      │  Sequence  │     │ Physics  │       │
│ │  Expert  │      │  Expert    │     │  Prior   │       │
│ │  (CNN)   │      │ (Bi-LSTM)  │     │ (Dense)  │       │
│ └────┬────┘      └─────┬──────┘     └────┬─────┘       │
│      │(64)              │(256)            │(32)          │
│      └─────────────┬────┘────────────────┘              │
│                    │                                     │
│              ┌─────▼─────┐                              │
│              │  Fusion    │                              │
│              │   Head     │                              │
│              └─────┬─────┘                              │
│                    │                                     │
│         ┌─────────┼─────────┐                           │
│         ▼         ▼         ▼                           │
│    confidence  shape_s  temporal_s                       │
│    (0.0-1.0)   (0-1)    (0-1)                          │
│                                                          │
│              Physics-Informed Loss:                       │
│    L = FocalLoss + λ₁·duration_constraint               │
│                  + λ₂·radius_constraint                  │
│                  + λ₃·depth_consistency                  │
└─────────────────────────────────────────────────────────┘
```

---

## 6. Common Mistakes to Avoid

### Technical Mistakes

| Mistake | Why It's Bad | How to Fix |
|---------|-------------|-----------|
| Data leakage | Test set performance is artificially inflated | Ensure strict train/test separation. No data augmentation on test set. |
| No baseline comparison | Reviewers can't assess if your method is actually better | Always compare with at least 2 published baselines |
| No ablation study | Can't tell which components actually help | Remove one component at a time and measure |
| Reporting only accuracy | Meaningless for imbalanced data | Report Precision, Recall, F1, AUC-ROC |
| Small test set | Results aren't statistically significant | Use 5-fold cross-validation or bootstrap confidence intervals |

### Writing Mistakes

| Mistake | How to Fix |
|---------|-----------|
| Overclaiming | Use hedging language: "suggests," "indicates," not "proves" |
| Not citing enough | Cite EVERY related paper you're aware of. Underciting annoys reviewers |
| Burying the contribution | State your contributions explicitly in bullet points in the Introduction |
| Too much jargon | Define every term the first time you use it |
| No code release | Reviewers value reproducibility. Release code on GitHub |

### Presentation Mistakes

| Mistake | How to Fix |
|---------|-----------|
| Low-resolution figures | Use vector graphics (PDF) or 300+ DPI |
| Inconsistent notation | Define notation once and use consistently |
| Wall of text | Use tables, figures, and equations to break up text |
| Missing error bars | Report standard deviation or confidence intervals |

---

## 7. The Review Process

### What Reviewers Look For

1. **Novelty**: Is this genuinely new? Or just a minor tweak of existing work?
2. **Correctness**: Is the methodology sound? Any errors in the math/code?
3. **Completeness**: Are baselines included? Ablation study? Error analysis?
4. **Significance**: Does this matter to the community?
5. **Clarity**: Is it well-written and easy to follow?
6. **Reproducibility**: Is there enough detail to replicate the results?

### How to Respond to Reviews

When you get reviews back:

1. **Don't take it personally**. Reviewers are doing their job.
2. **Create a point-by-point response** to each comment
3. **Be respectful** even if you disagree
4. **Make the changes** or explain clearly why you didn't
5. **Highlight changes** in the revised manuscript (use colored text)

### Example Review Response

```
Reviewer 1, Comment 3: "The authors should compare with ExoMiner 
on the same dataset."

RESPONSE: Thank you for this suggestion. We have added a comparison 
with ExoMiner using the Kepler DR25 test set in Table 3 (Section 5.1). 
Our model achieves comparable precision (93.1% vs 95%) with improved 
recall (92.4% vs 93%) while additionally providing explainability 
features that ExoMiner lacks. The relevant text has been updated 
in Section 5.1, paragraph 2 (highlighted in blue).
```

---

## 8. Tools and Resources

### Writing Tools

| Tool | Purpose |
|------|---------|
| **Overleaf** | Online LaTeX editor (collaborative, free tier available) |
| **LaTeX** | The standard typesetting system for scientific papers |
| **BibTeX** | Citation management within LaTeX |
| **draw.io** | Free diagram tool for architecture figures |
| **Zotero** | Free reference manager (imports from arXiv, ADS) |

### Scientific Python

| Library | Purpose |
|---------|---------|
| **matplotlib** | Publication-quality figures |
| **seaborn** | Statistical visualization |
| **scikit-learn** | Baseline ML models, metrics |
| **shap** | SHAP explainability analysis |
| **captum** | PyTorch model interpretability (Grad-CAM, etc.) |
| **pytorch-grad-cam** | Grad-CAM implementation |

### Astronomy-Specific

| Resource | Purpose |
|----------|---------|
| **NASA ADS** | Search for astronomy papers |
| **arXiv astro-ph.EP** | Exoplanet preprints |
| **lightkurve** | TESS/Kepler data access |
| **exoplanet.eu** | Confirmed planet database |
| **AASTeX** | LaTeX template for AAS journals (AJ, ApJ) |
| **MNRAS LaTeX class** | Template for MNRAS |

---

## Key Takeaways

1. **Post to arXiv first** -- establishes priority and gets community feedback
2. **Target AJ or MNRAS** for the full paper
3. **Write Methods first** (you know it best), then Results, then Intro, then Abstract last
4. **Every claim needs evidence**: comparison tables, ablation study, statistical significance
5. **Release your code** -- reviewers and the community value reproducibility
6. **Start with the evaluation infrastructure** -- you can't write results without numbers

---

## Congratulations!

You've completed the entire learning guide. Here's your path forward:

```
YOU ARE HERE
     │
     ▼
┌─────────────────────────────────────┐
│  1. Read the recommended papers     │
│  2. Set up evaluation infrastructure│
│  3. Run experiments                 │
│  4. Write the paper                 │
│  5. Submit!                         │
└─────────────────────────────────────┘
```

Go find some exoplanets.
