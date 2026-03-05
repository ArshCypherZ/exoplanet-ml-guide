# Part 2: The Transit Method -- How We Detect Planets by Watching Stars Dim

> **Prerequisites**: You've read [Part 1: Astronomy Basics](01-astronomy-basics.md).  
> **What you'll learn**: The complete physics behind transit detection, how light curves work, what BLS does, and how to tell real planets from imposters.

---

## Table of Contents

1. [The Core Idea](#1-the-core-idea)
2. [Anatomy of a Transit](#2-anatomy-of-a-transit)
3. [Light Curves Explained](#3-light-curves-explained)
4. [The Math Behind Transits](#4-the-math-behind-transits)
5. [Phase Folding](#5-phase-folding)
6. [Box Least Squares (BLS)](#6-box-least-squares-bls)
7. [False Positives -- The Imposters](#7-false-positives----the-imposters)
8. [U-Shape vs V-Shape](#8-u-shape-vs-v-shape)
9. [Signal-to-Noise Ratio (SNR)](#9-signal-to-noise-ratio-snr)
10. [Limb Darkening](#10-limb-darkening)
11. [Glossary](#11-glossary)

---

## 1. The Core Idea

Imagine you're staring at a lightbulb. Now imagine an ant walks across the face of the lightbulb. You'd see a **tiny, brief dimming** as the ant blocks some of the light. That's essentially what a transit is -- except the "lightbulb" is a star, and the "ant" is a planet.

```
BEFORE TRANSIT          DURING TRANSIT         AFTER TRANSIT
                                         
   * * *                  * * *                  * * *
  * * * * *             * * . * *              * * * * *
 * * * * * *           * * .O. * *            * * * * * *
  * * * * *             * * . * *              * * * * *
   * * *                  * * *                  * * *
                                         
  (Full brightness)    (Planet blocks       (Full brightness
                        some light)          restored)
```

The key insight: **we don't see the planet directly. We infer its existence from the periodic dimming it causes.**

---

## 2. Anatomy of a Transit

A transit event has distinct phases:

```
Flux
(brightness)
  1.00 ─────────┐                              ┌─────────
                 │  Ingress                     │ Egress
                 │    ↓                         │   ↓
  0.99 ─         └────┐                    ┌───┘
                       │                    │
  0.98 ─               └────────────────────┘
                        ↑                  ↑
                     Contact 2          Contact 3
                ↑                                ↑
             Contact 1                        Contact 4
                │←─────── Transit Duration ────────→│
                       │←── Flat Bottom ──→│
  
                ──────────── Time ────────────────────→
```

### The Four Contact Points

| Contact | What happens |
|---------|-------------|
| **1st Contact** (T1) | Planet's leading edge touches the stellar disk. Dimming begins. |
| **2nd Contact** (T2) | Planet is fully on the stellar disk. Maximum dimming begins. |
| **3rd Contact** (T3) | Planet's trailing edge starts to leave. Maximum dimming ends. |
| **4th Contact** (T4) | Planet has fully left. Brightness restored. |

### Key Transit Parameters

| Parameter | Symbol | What it tells you |
|-----------|--------|-------------------|
| **Transit depth** | delta | Planet radius (relative to star) |
| **Transit duration** | T_dur | Orbital velocity, impact parameter |
| **Ingress/Egress time** | tau | Planet size, orbital inclination |
| **Period** | P | Orbital period (time between transits) |
| **Epoch** | T0 | Time of the first transit midpoint |

---

## 3. Light Curves Explained

A **light curve** is simply a graph of a star's brightness (flux) over time. Here's what a real light curve looks like:

### Raw Light Curve (noisy, full time series)

```
Flux
1.02 ─  .  .     . .   .      .  .     .      .
1.01 ─ . .. .  .. .  . . .. .. .. .  .. . . .. .
1.00 ─.......  ..... .. .......  .... ...........
0.99 ─ . ..  .. . .   .  . ..  .  .  . .  .  .
0.98 ─       V         V         V         V
0.97 ─     (dip 1)  (dip 2)   (dip 3)  (dip 4)
     └──────────────────────────────────────────→ Time (days)
       0     5     10    15    20    25    30
```

### What You See

- **Baseline**: The normal brightness of the star (~1.0 after normalization)
- **Noise**: Random scatter around the baseline (from photon noise, stellar variability, instrument effects)
- **Transit dips**: Periodic drops in brightness (the V-shapes at regular intervals)
- **Other features**: Stellar flares (sudden brightness spikes), stellar rotation (slow sinusoidal variation), instrumental systematics

### Types of Noise

| Noise Source | Looks Like | Effect on Detection |
|-------------|-----------|-------------------|
| **Photon noise** | Random scatter (white noise) | Limits detection of shallow transits |
| **Stellar variability** | Slow waves (rotation spots) | Can mimic or hide transit signals |
| **Stellar flares** | Sharp upward spikes | Can confuse period-finding algorithms |
| **Instrumental systematics** | Correlated trends, jumps | Must be removed (detrending) |
| **Cosmic rays** | Single-point outliers | Usually removed by sigma-clipping |

### Why Preprocessing Matters

Raw TESS data is messy. Before we can find planets, we need to:
1. **Remove outliers** (cosmic rays, flares)
2. **Detrend** (remove stellar variability and instrumental systematics)
3. **Normalize** (center the flux at 1.0 so we can compare different stars)
4. **Stitch** (combine data from multiple TESS sectors)

---

## 4. The Math Behind Transits

### Transit Depth

The most fundamental equation in transit photometry:

```
                    (R_planet)^2
    delta  =  ─────────────────────
                    (R_star)^2
```

Where:
- `delta` = transit depth (fractional decrease in flux)
- `R_planet` = planet radius
- `R_star` = star radius

**Rearranging to find planet radius**:

```
    R_planet  =  R_star * sqrt(delta)
```


### Example Calculations

| Scenario | R_planet | R_star | Depth | Detectable? |
|----------|---------|--------|-------|------------|
| Hot Jupiter around Sun | 11 R_E | 1 R_sun | 1.0% | Easy |
| Super-Earth around Sun | 2 R_E | 1 R_sun | 0.033% | Hard |
| Earth around Sun | 1 R_E | 1 R_sun | 0.0084% | Very hard |
| Earth around M dwarf | 1 R_E | 0.3 R_sun | 0.093% | Moderate |

TESS can reliably detect transits with depths > ~200 ppm (0.02%) for bright stars.

### Kepler's Third Law (Orbital Period to Distance)

```
    a^3  =  (P^2 * M_star)
```

Where:
- `a` = semi-major axis in AU
- `P` = orbital period in years
- `M_star` = stellar mass in solar masses


### Geometric Transit Probability

Not every planet transits from our perspective. The probability is:

```
    P_transit  =  R_star / a
```

For Earth around the Sun: P = R_sun / 1 AU = 0.47% (only 1 in 210 randomly oriented solar systems would show Earth's transit).

**This is why we need to observe millions of stars** -- most planets don't transit.

---

## 5. Phase Folding

Phase folding is one of the most important techniques in transit detection. Here's why:

### The Problem

A single transit is weak and buried in noise. But if the planet orbits every P days, it transits repeatedly. Can we combine all the transits to boost the signal?

### The Solution: Phase Folding

1. **Know the period** (P) and the time of first transit (T0)
2. **Calculate the phase** of each data point: `phase = ((time - T0) % P) / P`
3. **Plot all data points against phase** (instead of time)

This stacks all transits on top of each other:

```
BEFORE FOLDING (vs time):
1.00 ─........  ..... .. .......  .... ...........
0.98 ─       V         V         V         V
     └──────────────────────────────────────────→ Time
       0     5     10    15    20    25    30

AFTER FOLDING (vs phase):
1.00 ─ :::::::::::::::............:::::::::::::::
0.99 ─ ::::::::::::::::..........::::::::::::::::
0.98 ─ ::::::::::::::::::.....::::::::::::::::::
0.97 ─ :::::::::::::::::::VVV:::::::::::::::::::
     └──────────────────────────────────────────→ Phase
      -0.5              0.0              +0.5
                    (transit center)
```

The noise averages down (by sqrt(N_transits)), but the transit signal stays. This is how we detect planets with tiny transit depths.

---

## 6. Box Least Squares (BLS)

BLS is the standard algorithm for finding periodic transit-like signals in light curves.

### The Idea

A transit looks like a periodic "box" (rectangular dip) in the light curve. BLS searches for the period and phase of the best-fitting periodic box.

```
Model signal:
             ┌──────────────┐         ┌──────────────┐
 ────────────┘              └─────────┘              └────────
              ← box depth →
 ← period P →
```

### How BLS Works (Step by Step)

1. **Choose a grid of trial periods**: P_1, P_2, ..., P_N

2. **For each trial period P_i**:
   - Phase-fold the light curve at P_i
   - Try different box widths (transit durations): like [1.5, 2.5, 3.5, 5.0] hours
   - For each width, slide the box across the phase and compute the "BLS power" -- essentially, how well the box model fits the data
   - Record the best-fitting box position and its power

3. **Build a periodogram**: Plot BLS power vs. period. The peak indicates the most likely orbital period.

4. **Extract the best signal**: The period, epoch (T0), duration, and depth of the strongest peak.

```
BLS Periodogram:
Power
  |           *
  |          ***
  |    *    *   *
  |   ***  *     *    *
  |  *   **       *  ***
  | *              **   *
  |*                     ****
  └─────────────────────────────→ Period (days)
       1     5     10    15    20    25
              ↑
        Best period (peak)
```

---

## 7. False Positives -- The Imposters

This is one of the biggest challenges. Many things can mimic a planetary transit:

### 7.1 Eclipsing Binaries (EBs)

Two stars orbiting each other. When one star passes in front of the other, it creates a dip -- just like a planet transit!

```
PLANET TRANSIT:                ECLIPSING BINARY:
                               
  ──────┐    ┌──────           ──────┐    ┌──────
        │    │                       │    │
        └────┘                        └──┘
     (U-shape, shallow)           (V-shape, can be deep)
     depth: 0.01-1%              depth: can be >10%
```

**Key differences**:
- EBs often show **V-shaped** dips (gradual ingress/egress) vs. U-shaped for planets
- EBs may show a **secondary eclipse** (when the other star is eclipsed)
- EBs may show **odd-even depth differences** (primary and secondary eclipses have different depths)

### 7.2 Background Eclipsing Binaries (BEBs)

A faint eclipsing binary in the background, blended with the target star in the TESS pixel. The EB's deep eclipse gets diluted by the bright target star, making it look like a shallow planetary transit.

**TESS has large pixels** (21 arcseconds), so this is a significant problem. Multiple stars often fall within one TESS pixel.

### 7.3 Stellar Variability

Star spots (dark regions on the star's surface) rotating in and out of view can create periodic brightness changes that look like transits.

### 7.4 Instrumental Systematics

Spacecraft pointing jitter, temperature changes, and scattered light can create periodic signals that mimic transits.

### False Positive Rates

In the raw TESS data, the false positive rate for transit-like signals is estimated at **40-70%**. This is why ML vetting is so important -- it's the primary tool for separating real planets from imposters.

---

## 8. U-Shape vs V-Shape

This is one of the most important diagnostic tools in transit classification.

### U-Shape (Planet!)

```
  ─────────┐                    ┌─────────
           │                    │
           │  ┌──────────────┐  │
           └──┘              └──┘
              ← flat bottom →
```

**Why it's U-shaped**: The planet is much smaller than the star. During ingress, the planet gradually covers more stellar surface. Once fully on the disk (2nd contact), the depth is constant until egress begins (3rd contact). The "flat bottom" is the signature of a small opaque body fully on the stellar disk.

### V-Shape (Probably an Eclipsing Binary!)

```
  ─────────\                  /─────────
            \                /
             \              /
              \            /
               \    /\    /
                \  /  \  /
                 \/    \/
```

**Why it's V-shaped**: Two roughly equal-sized stars. There's no flat bottom because the eclipsing star is comparable in size to the eclipsed star -- the eclipse depth is always changing as the overlap area changes.

### Grazing Transit (Ambiguous)

```
  ─────────\              /─────────
            \            /
             \          /
              \        /
               └──────┘
```

A planet that only grazes the edge of the stellar disk. It looks V-shaped even though it IS a planet. This is one of the hardest cases for ML classifiers.

---

## 9. Signal-to-Noise Ratio (SNR)

SNR measures how confident we are that a signal is real (not just noise). Higher SNR = more confident detection.

### Basic SNR Formula

```
                   transit_depth
    SNR  =  ──────────────────────────
             noise_per_point / sqrt(N)
```

Where:
- `transit_depth` = the depth of the transit dip
- `noise_per_point` = the standard deviation of the out-of-transit flux
- `N` = the total number of data points in-transit (across all transits)

### What SNR Values Mean

| SNR | Meaning |
|-----|---------|
| < 3 | Not significant (could easily be noise) |
| 3-5 | Marginal detection (needs more data or follow-up) |
| 5-10 | Good detection |
| 10-20 | Strong detection |
| > 20 | Very strong detection |

### How to Increase SNR

1. **More transits** (longer observation): SNR scales as sqrt(N_transits)
2. **Brighter star**: Less photon noise per point
3. **Deeper transit**: Bigger planet and/or smaller star
4. **Less stellar variability**: Quieter stars are better targets

---

## 10. Limb Darkening

Stars aren't uniformly bright disks. The center of the stellar disk appears brighter than the edges (limb). This is called **limb darkening**.

```
  ┌───────────────────┐
  │  dim   ···   dim  │
  │  ··  BRIGHT  ··   │
  │ ·   (center)   ·  │
  │  ··  BRIGHT  ··   │
  │  dim   ···   dim  │
  └───────────────────┘
      ← limb darkening →
```

### Why It Matters

Limb darkening affects the transit shape:
- When the planet crosses the bright center, it blocks more light
- When it crosses the dim limbs (ingress/egress), it blocks less light
- This makes the transit shape slightly **rounded** instead of perfectly flat-bottomed

```
WITHOUT limb darkening:        WITH limb darkening:
  ──────┐        ┌──────        ──────╲        ╱──────
        │        │                     ╲      ╱
        └────────┘                      ╰────╯
     (perfectly flat)              (slightly rounded)
```

---

## 11. Glossary

| Term | Definition |
|------|-----------|
| **BLS** | Box Least Squares -- algorithm for finding periodic box-shaped transit signals |
| **Contact points** | The four moments when the planet's edge touches the star's edge during a transit |
| **Detrending** | Removing long-term trends (stellar variability, instrumental drift) from a light curve |
| **Eclipsing binary** | Two stars orbiting each other, where one periodically blocks the other's light |
| **Epoch (T0)** | The time of the first transit midpoint |
| **Flat bottom** | The constant-depth portion of a U-shaped transit (planet fully on the stellar disk) |
| **Flux** | The brightness of a star, usually normalized to 1.0 |
| **Ingress** | The beginning of a transit (planet entering the stellar disk) |
| **Egress** | The end of a transit (planet leaving the stellar disk) |
| **Limb darkening** | Stars appear dimmer at their edges than their center |
| **Periodogram** | A graph showing signal strength vs. trial period |
| **Phase folding** | Stacking all transits on top of each other by plotting data vs. orbital phase |
| **Savitzky-Golay filter** | A smoothing filter used to remove stellar variability while preserving transit shapes |
| **SNR** | Signal-to-noise ratio -- confidence that a signal is real |
| **Transit depth** | The fractional decrease in brightness during a transit = (R_planet/R_star)^2 |
| **Transit duration** | Total time from 1st to 4th contact |

---

## What's Next?

**Part 3: TESS and Space Missions** -- Everything about TESS, Kepler, JWST, and the other telescopes that provide the data we will work with.