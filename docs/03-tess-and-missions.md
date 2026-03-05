# Part 3: TESS and Space Missions -- Where the Data Comes From

> **Prerequisites**: [Part 1](01-astronomy-basics.md) and [Part 2](02-transit-method.md).  
> **What you'll learn**: How TESS works, what data it produces, how to access it, and how other missions (Kepler, JWST, PLATO) fit into the picture.

---

## Table of Contents

1. [TESS -- The Big Picture](#1-tess----the-big-picture)
2. [How TESS Observes the Sky](#2-how-tess-observes-the-sky)
3. [TESS Data Products](#3-tess-data-products)
4. [Accessing TESS Data](#4-accessing-tess-data)
5. [The Kepler Mission](#5-the-kepler-mission)
6. [JWST -- The Atmosphere Revealer](#6-jwst----the-atmosphere-revealer)
7. [Upcoming Missions](#7-upcoming-missions)
8. [Other Data Sources](#8-other-data-sources)
9. [Mission Comparison Table](#9-mission-comparison-table)

---

## 1. TESS -- The Big Picture

**TESS** = Transiting Exoplanet Survey Satellite

| Property | Value |
|----------|-------|
| Launch date | April 18, 2018 |
| Operator | NASA / MIT |
| Orbit | Unique 13.7-day elliptical orbit around Earth (2:1 resonance with Moon) |
| Cameras | 4 wide-field CCD cameras |
| Field of view | Each camera: 24x24 degrees. Combined: 24x96 degrees per sector |
| Sky coverage | ~85% of the sky over 2 years (primary mission) |
| Target stars | ~200,000 pre-selected bright stars (2-min cadence) + all visible stars (30-min FFI) |
| Primary mission | 2018-2020 (2 years, 26 sectors) |
| Extended mission | 2020-present (ongoing, re-observing sky with improved cadence) |

### TESS's Mission Goal

Find small planets around **bright, nearby stars** that can be followed up by ground-based telescopes and JWST for atmospheric characterization.

This is different from Kepler, which found planets around **faint, distant stars** that are harder to study further.

---

## 2. How TESS Observes the Sky

### The Sector Strategy

TESS divides the sky into overlapping "sectors." Each sector is observed for ~27 days before moving to the next.

```
TESS SKY COVERAGE (Simplified):

Northern Hemisphere (Year 2):     Southern Hemisphere (Year 1):
                                   
   ┌──┬──┬──┬──┬──┬──┐              ┌──┬──┬──┬──┬──┬──┐
   │14│15│16│17│18│19│              │ 1│ 2│ 3│ 4│ 5│ 6│
   ├──┼──┼──┼──┼──┼──┤              ├──┼──┼──┼──┼──┼──┤
   │20│21│22│23│24│25│              │ 7│ 8│ 9│10│11│12│
   ├──┼──┼──┼──┼──┼──┤              ├──┼──┼──┼──┼──┼──┤
   │26│  │  │  │  │13│              │13│  │  │  │  │26│
   └──┴──┴──┴──┴──┴──┘              └──┴──┴──┴──┴──┴──┘
   
   Each box = one sector (~27 days)
   Overlap at poles = longer observation = detect longer-period planets
```

### Key Observation Facts

- **27 days per sector**: This limits detection to planets with periods < ~13 days (need at least 2 transits)
- **Overlap regions**: Stars near the ecliptic poles get observed in multiple sectors (up to ~1 year), enabling longer-period planet detection
- **Continuous viewing zone**: A small region near each pole is observed for ~1 year straight
- **Extended mission**: TESS is now re-observing sectors with better cadence (200-second and even 20-second for some targets)

### The 27-Day Limitation

This is crucial to understand. With only ~27 days of data per sector:

```
Period = 2 days:  ~13 transits observed  → Easy to detect
Period = 5 days:  ~5 transits observed   → Doable
Period = 10 days: ~2 transits observed   → Minimum for detection
Period = 15 days: ~1 transit observed    → NOT detectable (need >1 transit)
Period = 30 days: ~0 transits observed   → Impossible in 1 sector
```

This is why TESS primarily finds short-period planets (hot Jupiters, ultra-short period planets, close-in super-Earths). For longer periods, you need multi-sector coverage or Kepler data.

---

## 3. TESS Data Products

### 3.1 Light Curves (2-minute cadence)

TESS focuses on about 200,000 bright stars chosen in advance. For these stars, it takes small "postage stamp" images (tiny cutouts of pixels around each star) every 2 minutes. NASA's SPOC team processes these images into light curves, which show how the star's brightness changes over time.

**There are two main types of brightness data (flux)**:
- **SAP (Simple Aperture Photometry)**: This is the raw brightness from adding up pixels in a circle around the star. It includes all the telescope's errors and noise.
- **PDCSAP (Pre-search Data Conditioning SAP)**: This is the cleaned-up version of SAP, with telescope errors removed. **This is usually the one we use for finding planets.**

### 3.2 Full Frame Images (FFI)

Every 30 minutes (now every 200 seconds in the extended mission), TESS captures a full picture from all 4 of its cameras. This includes **every star** in the sky area it's looking at, not just the pre-selected ones.

**Advantage**: Covers millions of stars
**Disadvantage**: Takes pictures less often, needs more computer work to analyze, and can be noisier

### 3.3 Target Pixel Files (TPF)

These are the raw pixel data from the small "postage stamp" images for the target stars. They're useful if you want to create your own light curves using custom settings for how to measure the star's brightness.

### 3.4 Data Validation Reports (DVR)

For each possible planet signal (called a TCE, or Threshold Crossing Event), NASA's SPOC team creates a report. It includes details like the detection method, graphs, checks on where the signal comes from, and other ways to confirm if it's real.

### Data Flow

```
TESS Cameras → Raw Images → SPOC Pipeline → Light Curves
                                    │
                                    ├── SAP flux (raw)
                                    ├── PDCSAP flux (cleaned)
                                    ├── FFIs (all stars)
                                    └── DVRs (for detected signals)
```

---

## 4. Accessing TESS Data

### 4.1 MAST (Mikulski Archive for Space Telescopes)

The primary archive for TESS data. URL: https://mast.stsci.edu

All TESS data is public and free. No login needed.

### 4.2 Lightkurve (Python Library)

Lightkurve is a Python package that makes it easy to search, download, and analyze Kepler and TESS data.

```python
import lightkurve as lk

# Search for TESS data for a star
search_result = lk.search_lightcurve(
    "TIC 150428135",      # TIC ID
    mission="TESS",        # Mission
    author="SPOC"          # Pipeline (SPOC is NASA's official pipeline)
)

# Download the light curves
lc_collection = search_result.download_all()

# Stitch sectors together and normalize
lc = lc_collection.stitch().normalize()

# Access the data
time = lc.time.value       # Array of time values (days)
flux = lc.flux.value       # Array of flux values (normalized ~1.0)
```

### 4.3 ExoFOP-TESS

The Follow-up Observing Program portal for TESS. Contains community-contributed follow-up observations, stellar parameters, and disposition information for TOIs.

URL: https://exofop.ipac.caltech.edu/tess/

### 4.4 NASA Exoplanet Archive

The definitive database of confirmed and candidate exoplanets from all missions.

URL: https://exoplanetarchive.ipac.caltech.edu

### 4.5 TESS Input Catalog (TIC)

The master catalog of ~1.7 billion stars that TESS could observe. Contains stellar parameters (temperature, radius, mass, magnitude) needed for transit analysis.

Every star has a unique **TIC ID** (e.g., TIC 150428135).

---

## 5. The Kepler Mission

### Overview

| Property | Value |
|----------|-------|
| Launch | March 2009 |
| End | October 2018 (spacecraft retired) |
| Primary mission | 2009-2013 (Kepler) |
| Extended mission | 2014-2018 (K2) |
| Field of view | 115 square degrees (MUCH smaller than TESS) |
| Target stars | ~150,000 stars in one patch of sky |
| Confirmed planets | ~2,700 |
| Observation duration | 4 years continuous (primary mission) |

### Kepler vs TESS

```
KEPLER:                           TESS:
┌─────────┐                       ┌─────────────────────────┐
│         │                       │                         │
│  Small  │                       │    Nearly all-sky       │
│  field  │                       │    coverage             │
│  4 yrs  │                       │    27 days per sector   │
│  deep   │                       │    bright stars         │
│         │                       │                         │
└─────────┘                       └─────────────────────────┘

Kepler:                           TESS:
- Faint, distant stars            - Bright, nearby stars
- Long baselines (4 years)        - Short baselines (27 days)
- Found many long-period planets  - Finds mostly short-period
- Hard to follow up               - Easy to follow up
- ~2,700 confirmed planets        - ~700+ confirmed (growing fast)
- Mission over                    - Still active
```

### Why Kepler Matters for ML

Kepler has the **best-labeled training dataset** for transit classification:

- **Kepler DR25**: The final data release with ~34,000 TCEs, each labeled as:
  - Confirmed Planet (CP)
  - False Positive (FP) -- eclipsing binary, background EB, etc.
  - Candidate (PC)

**Transfer learning**: Train on Kepler (large, well-labeled) and fine-tune on TESS (different noise profile, shorter baselines). This is one of the research directions.

---

## 6. JWST -- The Atmosphere Revealer

### Overview

The James Webb Space Telescope (launched December 2021) is not primarily a planet-finder, but it's the most powerful tool for studying planets TESS finds.

### What JWST Does for Exoplanets

**Transmission spectroscopy**: When a planet transits its star, starlight passes through the planet's atmosphere. Different molecules absorb different wavelengths of light. JWST measures this absorption spectrum.

```
Star                    Planet (transiting)           Observer
  *  ───────────────── ┌────┐ ─────────────────────   🔭
  *  ─── some light ── │atm │ ── absorbed ──────────   
  *  ─── passes thru → │    │ ── filtered light ───→  JWST
  *  ───────────────── └────┘ ─────────────────────   
```

### What JWST Can Detect in Atmospheres

- **Water vapor (H2O)**: Key for habitability
- **Carbon dioxide (CO2)**: Greenhouse gas
- **Methane (CH4)**: Potential biosignature
- **Oxygen/Ozone (O2/O3)**: Strong biosignature
- **Sodium/Potassium (Na/K)**: Hot Jupiter atmospheres

---

## 7. Upcoming Missions

### PLATO (ESA, launch ~2026-2027)

| Property | Value |
|----------|-------|
| Full name | PLAnetary Transits and Oscillations of stars |
| Agency | European Space Agency (ESA) |
| Goal | Find Earth-like planets around Sun-like stars in their habitable zones |
| Cameras | 26 cameras (vs TESS's 4!) |
| Observation | 2-3 years continuous on same fields |
| Key advantage | Long baselines + bright stars = Earth-analog detection |

PLATO is essentially "Kepler + TESS combined" -- long observation baselines (like Kepler) targeting bright stars (like TESS).

**For ML**: PLATO will produce a new wave of data needing ML classification. Models trained on Kepler/TESS could be adapted -- this is a research opportunity.

### Nancy Grace Roman Space Telescope (NASA, launch ~2027)

| Property | Value |
|----------|-------|
| Detection method | Gravitational microlensing + coronagraphy |
| Goal | Find planets at large orbital distances, including free-floating planets |
| Unique angle | Completely different from transit method |

### Ariel (ESA, launch ~2029)

Dedicated to measuring atmospheric spectra of ~1,000 known exoplanets. Will complement JWST with a larger sample size.

---

## 8. Other Data Sources

### Ground-Based Transit Surveys

| Survey | Location | Specialty |
|--------|----------|-----------|
| **WASP** (Wide Angle Search for Planets) | La Palma, South Africa | Hot Jupiters around bright stars |
| **HATNet** / **HATSouth** | Arizona, Chile, Australia | Hot Jupiters, network coverage |
| **NGTS** (Next Generation Transit Survey) | Chile | Neptune-sized planets |
| **MEarth** | Arizona, Chile | Planets around M dwarfs |
| **TRAPPIST** | Chile, Morocco | Small planets around ultracool dwarfs (found TRAPPIST-1 system!) |

### Radial Velocity Surveys

| Instrument | Location | Precision |
|-----------|----------|-----------|
| **HARPS** | ESO 3.6m, Chile | ~1 m/s (the gold standard for 20 years) |
| **ESPRESSO** | VLT, Chile | ~0.1 m/s (next generation, can detect Earth-mass planets) |
| **NEID** | WIYN telescope, Arizona | ~0.3 m/s |
| **Keck/HIRES** | Keck, Hawaii | ~1 m/s |

### Astrometry

**Gaia** (ESA): Measuring positions of ~2 billion stars with unprecedented precision. Expected to find thousands of planets via astrometric wobble. Gaia DR4 (expected ~2026) will be a goldmine.

### Key Databases

| Resource | URL | What It Contains |
|----------|-----|-----------------|
| NASA Exoplanet Archive | exoplanetarchive.ipac.caltech.edu | All confirmed planets, parameters |
| ExoFOP-TESS | exofop.ipac.caltech.edu/tess/ | TESS candidate follow-up data |
| MAST | mast.stsci.edu | TESS/Kepler/HST raw data |
| Lightkurve | docs.lightkurve.org | Python library for light curve analysis |
| Exoplanet.eu | exoplanet.eu | European exoplanet encyclopedia |

---

## 9. Mission Comparison Table

| Feature | Kepler | K2 | TESS | PLATO |
|---------|--------|-----|------|-------|
| Years active | 2009-2013 | 2014-2018 | 2018-present | ~2027+ |
| Sky coverage | 115 sq deg | Multiple fields | ~85% sky | 2 fields |
| Observation per target | 4 years | 80 days | 27 days | 2-3 years |
| Target brightness | Faint (V~12-16) | Moderate | Bright (V<12) | Bright |
| Cadence | 30 min (long), 1 min (short) | 30/1 min | 30/2 min (now 200s/20s) | 25 sec |
| Confirmed planets | ~2,700 | ~550 | ~700+ | TBD |
| Best for | Long-period planets | Various | Short-period, bright | Earth-analogs |
| Best for ML | Training data | Extra training | Real-time detection | Future |
| Status | Retired | Retired | Active | Pre-launch |

---

## Key Takeaways

1. **TESS** surveys bright, nearby stars but only observes each sector for ~27 days.
2. **Kepler** - 4 years of continuous observation with well-labeled TCEs.
3. **JWST** is the downstream consumer
4. **PLATO** (launching soon) will be the next big thing -- long baselines + bright stars.
5. **lightkurve** is the Python library that connects to all this data.
6. The 27-day TESS baseline is a fundamental limitation -- it biases detection toward short-period planets.

---

## What's Next?

**Part 4: Machine Learning for Exoplanets** -- How CNNs, LSTMs, and Transformers are used to classify transit signals. This is where our ML expertise connects to the astronomy.
