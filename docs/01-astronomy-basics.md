# Part 1: Astronomy Basics -- From Zero to Exoplanets

> **Who this is for**: No prior astronomy knowledge assumed. By the end of this doc, you'll understand why finding planets around other stars is one of the most exciting problems in science, and why ML is critical to solving it.

---

## Table of Contents

1. [What Is a Star?](#1-what-is-a-star)
2. [The Life Cycle of Stars](#2-the-life-cycle-of-stars)
3. [Key Properties of Stars](#3-key-properties-of-stars)
4. [What Is an Exoplanet?](#4-what-is-an-exoplanet)
5. [Types of Exoplanets](#5-types-of-exoplanets)
6. [The Habitable Zone](#6-the-habitable-zone)
7. [How Do We Find Exoplanets?](#7-how-do-we-find-exoplanets)
8. [Why Does This Matter?](#8-why-does-this-matter)
9. [Key Numbers to Remember](#9-key-numbers-to-remember)
10. [Glossary](#10-glossary)

---

## 1. What Is a Star?

A star is a massive ball of hot gas (mostly hydrogen and helium) held together by its own gravity. At its core, hydrogen atoms fuse into helium through **nuclear fusion**, releasing enormous amounts of energy. This energy is what makes stars shine.

### The Simple Version

Think of a star as a controlled nuclear explosion that lasts billions of years. Gravity tries to crush it inward; fusion energy pushes outward. This balance is called **hydrostatic equilibrium**.

```
           GRAVITY (inward)
               |
               v
    +-----------------------+
    |                       |
    |    FUSION ENERGY      |
    |    (outward pressure) |
    |                       |
    +-----------------------+
               ^
               |
           RADIATION PRESSURE (outward)
```

Our Sun is a star. It's a pretty average one -- not too hot, not too cold, not too big, not too small. Astronomers call it a **G-type main-sequence star** (or G dwarf).

### Stars vs. Planets

| Property | Star | Planet |
|----------|------|--------|
| Energy source | Nuclear fusion (self-luminous) | Reflects starlight (mostly) |
| Mass | Massive (at least ~80x Jupiter) | Much smaller |
| Temperature | Thousands to tens of thousands of Kelvin | Hundreds to thousands of Kelvin |
| Composition | Mostly hydrogen & helium | Rocky, gaseous, or icy |

---

## 2. The Life Cycle of Stars

Stars don't live forever. Their lifecycle depends primarily on their **mass**:

```
  Molecular Cloud (gas & dust)
         |
         v
    Protostar (collapsing cloud)
         |
         v
    Main Sequence Star (stable fusion, like our Sun)
         |
         v
    +---------+---------+
    |                   |
    v                   v
  Low/Medium Mass     High Mass
  (like our Sun)      (8+ solar masses)
    |                   |
    v                   v
  Red Giant           Red Supergiant
    |                   |
    v                   v
  Planetary Nebula    Supernova Explosion
    |                   |
    v                   v
  White Dwarf         Neutron Star or Black Hole
```

**Why this matters for exoplanets**: We mostly look for planets around **main sequence stars** -- stable stars in the prime of their lives. These are the stars where planets have had time to form and where conditions might support life.

---

## 3. Key Properties of Stars

When you're doing exoplanet detection, you need to know these stellar properties:

### 3.1 Temperature (T_eff -- Effective Temperature)

Measured in **Kelvin (K)**. Our Sun is ~5,778 K.

Stars are classified by their temperature into **spectral types**:

```
  O  ---  B  ---  A  ---  F  ---  G  ---  K  ---  M
  Hot                                              Cool
  Blue     Blue-   White   Yellow  Yellow  Orange  Red
           White                   (Sun)

  >30,000K  10,000K  7,500K  6,000K  5,200K  3,700K  2,400K
```

**Mnemonic**: "Oh Be A Fine Guy/Girl, Kiss Me"

**For exoplanets**: We mostly target **F, G, K, and M** stars:
- G stars (like the Sun): Good baseline for Earth-like planet searches
- K stars: Slightly cooler, very stable, excellent targets
- M stars (red dwarfs): Most common stars in the galaxy (~70%), small and dim, so planet transits are deeper and easier to detect. But they flare a lot, which creates noise.

### 3.2 Radius (R_star)

Measured in **solar radii (R_sun)**, where R_sun = 696,340 km.

This is CRITICAL for transit detection because the transit depth depends on the ratio (R_planet / R_star)^2. A planet transiting a smaller star creates a deeper dip.

**Example**:
- Earth transiting the Sun: depth = (R_earth / R_sun)^2 = (6,371 / 696,340)^2 = **0.0084%** (incredibly tiny!)
- Earth transiting an M dwarf (0.3 R_sun): depth = (6,371 / 208,902)^2 = **0.093%** (~11x deeper, much easier to detect)

### 3.3 Mass (M_star)

Measured in **solar masses (M_sun)**, where M_sun = 1.989 x 10^30 kg.

Mass determines:
- How long the star lives (more massive = shorter life)
- The orbital dynamics of planets around it (Kepler's third law)
- The habitable zone distance

### 3.4 Luminosity

How much total energy a star emits per second. Measured in **solar luminosities (L_sun)**.

Determines the habitable zone distance: more luminous star = habitable zone further out.

### 3.5 Metallicity

In astronomy, anything heavier than helium is a "metal" (yes, even carbon and oxygen). **Metallicity** measures the abundance of these heavy elements.

**For exoplanets**: Higher metallicity stars are more likely to host giant planets. These heavy elements are the building blocks of rocky planets and giant planet cores.

### 3.6 TESS Magnitude (T_mag)

A measure of how bright the star appears to TESS specifically. Lower numbers = brighter. TESS targets stars with T_mag < 12 (bright, nearby stars).

### 3.7 TIC ID

Every star observed by TESS has a unique identifier in the **TESS Input Catalog (TIC)**. This is like a social security number for stars. Example: TIC 150428135 is the star TOI-700, which hosts confirmed planets.

---

## 4. What Is an Exoplanet?

An **exoplanet** (or extrasolar planet) is any planet that orbits a star other than our Sun. That's it. Simple definition, but finding them is incredibly hard.

### Brief History

- **1992**: First confirmed exoplanet found -- around a pulsar (dead star), not a normal star
- **1995**: First exoplanet around a Sun-like star -- **51 Pegasi b** (a hot Jupiter). Won the 2019 Nobel Prize.
- **2009**: NASA's Kepler mission launches -- the exoplanet revolution begins
- **2018**: TESS launches -- the survey of nearby bright stars begins
- **2025**: We've confirmed **5,800+ exoplanets**, with thousands more candidates waiting

### The Key Insight

For most of human history, we didn't know if planets existed around other stars. Now we know that **planets are incredibly common** -- on average, every star in our galaxy has at least one planet. That's ~100-400 billion planets in the Milky Way alone.

---

## 5. Types of Exoplanets

Exoplanets come in wild varieties, many with no analog in our solar system:

### 5.1 Hot Jupiters

- **Size**: Jupiter-sized (radius ~1 R_Jupiter = ~11 R_Earth)
- **Orbit**: Extremely close to their star (orbital period: 1-10 days)
- **Temperature**: 1,000-3,000 K (scorching hot)
- **Example**: WASP-18b (TIC 100100827) -- 1.16 R_Jupiter, orbital period 0.94 days
- **Why they matter**: Easiest to detect (big planet + short period = frequent, deep transits). But they're actually rare (~1% of stars have them).

### 5.2 Super-Earths

- **Size**: 1.2-2.0 R_Earth
- **Mass**: 2-10 Earth masses
- **No solar system analog** -- we don't have anything between Earth and Neptune
- **Example**: Pi Mensae c (TIC 261136679) -- 2.04 R_Earth, 6.27 day period
- **Why they matter**: Most common type of planet in the galaxy. Key targets for habitability studies.

### 5.3 Mini-Neptunes / Sub-Neptunes

- **Size**: 2.0-4.0 R_Earth
- **Likely have thick hydrogen/helium atmospheres** over a rocky/icy core
- **The "radius gap"**: There's a mysterious gap in the planet size distribution between 1.5-2.0 R_Earth. Planets are either super-Earths or mini-Neptunes, rarely in between. Understanding this gap is an active research area.

### 5.4 Neptune-like

- **Size**: ~4 R_Earth
- **Similar to our Neptune/Uranus**
- **"Hot Neptunes" are rare** -- this is called the "hot Neptune desert." We don't fully understand why.

### 5.5 Terrestrial / Earth-like

- **Size**: 0.5-1.2 R_Earth
- **Rocky composition**
- **Example**: TOI-700 d (TIC 150428135) -- 1.07 R_Earth, in the habitable zone!
- **Why they matter**: The ultimate goal -- finding another Earth.

### Size Comparison

```
  Earth     Super-Earth    Mini-Neptune    Neptune      Jupiter
   .          ..             ...           ....        .........
  1 R_E      1.5 R_E        3 R_E         4 R_E       11 R_E
```

---

## 6. The Habitable Zone

The **habitable zone** (HZ), also called the "Goldilocks zone," is the range of orbital distances from a star where liquid water could exist on a planet's surface.

```
                    STAR
                     *
                     |
    Too Hot    |  Habitable  |  Too Cold
    (Venus)    |    Zone     |  (Mars)
               |  (Earth)   |
    <----------|------------|------------>
               Inner edge   Outer edge
```

### Key Points

- The HZ depends on the **star's luminosity and temperature**
- For a Sun-like star: roughly 0.95-1.67 AU (1 AU = Earth-Sun distance)
- For an M dwarf (cooler, dimmer): HZ is much closer, maybe 0.1-0.4 AU
- Being in the HZ doesn't guarantee habitability -- atmosphere, magnetic field, and other factors matter
- Can calculate this using the **Kopparapu et al. (2013)** model

### The Calculation (simplified)

The inner and outer edges depend on the star's effective temperature:

```
Inner edge: a_inner = sqrt(L_star / S_inner)    [in AU]
Outer edge: a_outer = sqrt(L_star / S_outer)    [in AU]
```

Where S_inner and S_outer are the stellar flux limits (derived from climate models), and L_star is the luminosity in solar luminosities.

---

## 7. How Do We Find Exoplanets?

There are several detection methods. Each has strengths and biases:

### 7.1 Transit Method

When a planet passes in front of its star (from our viewpoint), it blocks a tiny fraction of the star's light. We measure this brightness dip.

**We'll cover this in depth in Part 2.** This is the most productive method -- responsible for ~75% of all confirmed exoplanets.

### 7.2 Radial Velocity (Doppler Method)

A planet's gravity tugs on its star, causing the star to "wobble." This wobble shifts the star's light slightly toward blue (approaching) or red (receding). We measure this Doppler shift.

```
  Planet pulls star toward us     Planet pulls star away
  (star light blueshifted)        (star light redshifted)
         <---                           --->
    Star . . . *                   * . . . Star
                 Planet               Planet
```

**Measures**: Planet's **minimum mass** (m * sin(i), where i is orbital inclination)
**Complementary to transits**: Transits give radius, RV gives mass. Together you get **density** and can infer composition.

### 7.3 Direct Imaging

Actually taking a picture of the planet. Extremely hard because stars are ~10 billion times brighter than their planets. Requires a **coronagraph** (to block the star's light) and advanced image processing.

**Best for**: Young, massive planets far from their stars (they glow in infrared from formation heat).

### 7.4 Gravitational Microlensing

When a foreground star with planets passes in front of a background star, its gravity bends the background star's light (Einstein's general relativity). The planet causes an extra blip in the magnification.

**Unique advantage**: Can find planets at any orbital distance, even rogue planets (no host star).
**Disadvantage**: One-time events -- can't follow up or confirm easily.

### 7.5 Astrometry

Precisely measuring a star's position over time. A planet's gravity makes the star trace a tiny ellipse in the sky.

**Gaia mission**: Currently measuring positions of ~2 billion stars. Expected to find thousands of planets via astrometry.

### Detection Method Comparison

| Method | What it measures | Best for | Confirmed planets |
|--------|-----------------|----------|-------------------|
| Transit | Planet radius | Close-in planets | ~4,200+ |
| Radial Velocity | Planet mass | Nearby bright stars | ~1,100+ |
| Direct Imaging | Light from planet | Young, massive, far-out planets | ~70+ |
| Microlensing | Mass ratio | Distant planets, rogues | ~200+ |
| Astrometry | Star wobble in sky | Long-period planets | ~30+ |

---

## 8. Why Does This Matter?

### The Big Questions

1. **Are we alone?** Finding Earth-like planets in habitable zones is the first step toward answering this.
2. **How do planetary systems form?** The diversity of exoplanets (hot Jupiters, super-Earths, etc.) challenges our theories of planet formation.
3. **How common are Earth-like planets?** The answer (called **eta-Earth**) has enormous implications. Current estimates: 10-50% of Sun-like stars may have an Earth-sized planet in their HZ.
4. **Atmospheric characterization**: With JWST, we can now study exoplanet atmospheres. Finding **biosignatures** (oxygen, methane, water vapor) would be groundbreaking.

### Why ML Matters Here

- TESS will observe **~200 million stars** over its mission
- Each star produces a time series (light curve) with thousands of data points
- Humans can't manually inspect 200 million light curves
- Traditional algorithms (BLS) find candidates, but produce many false positives
- ML can learn the subtle differences between real planets and imposters (eclipsing binaries, instrumental artifacts, stellar variability)

---

## 9. Key Numbers to Remember

| Quantity | Value | Why it matters |
|----------|-------|----------------|
| Confirmed exoplanets | ~5,800+ (2025) | The field is growing fast |
| Estimated planets in Milky Way | 100-400 billion | Planets are everywhere |
| Sun's temperature | 5,778 K | Reference point for stellar types |
| Sun's radius | 696,340 km | Used to normalize planet sizes |
| Earth's radius | 6,371 km | our detection target |
| Jupiter's radius | 69,911 km (~11 R_Earth) | Hot Jupiter reference |
| 1 AU | 149.6 million km | Earth-Sun distance |
| Transit depth (Earth-Sun) | 0.0084% | Shows how hard detection is |
| Transit depth (Hot Jupiter-Sun) | ~1% | Shows why these are found first |
| TESS sector observation | ~27 days | Limits detectable orbital periods |
| Kepler mission duration | ~4 years | Much longer baseline than TESS |

---

## 10. Glossary

| Term | Definition |
|------|-----------|
| **AU (Astronomical Unit)** | The average Earth-Sun distance: 149.6 million km |
| **Effective Temperature (T_eff)** | The surface temperature of a star, in Kelvin |
| **Exoplanet** | A planet orbiting a star other than our Sun |
| **Habitable Zone (HZ)** | The orbital region where liquid water could exist on a planet surface |
| **Light curve** | A graph of a star's brightness over time |
| **Main sequence** | The stable phase of a star's life where it fuses hydrogen |
| **Metallicity** | Abundance of elements heavier than helium in a star |
| **R_Earth (R_E)** | Earth's radius, used as a unit for planet sizes |
| **R_Jupiter (R_J)** | Jupiter's radius, ~11 R_Earth |
| **R_Sun (R_solar)** | The Sun's radius, used as a unit for star sizes |
| **Solar luminosity (L_sun)** | The Sun's total energy output per second |
| **Solar mass (M_sun)** | The Sun's mass: 1.989 x 10^30 kg |
| **Spectral type** | Classification of stars by temperature (O, B, A, F, G, K, M) |
| **TIC** | TESS Input Catalog -- the master list of TESS target stars |
| **TIC ID** | Unique identifier for a star in the TIC |
| **TOI** | TESS Object of Interest -- a candidate planet signal |
| **Transit** | When a planet passes in front of its star, blocking some light |
| **Transit depth** | The fractional decrease in starlight during a transit |

---

## What's Next?

Now that we understand the basics of stars, planets, and detection methods, **Part 2** dives deep into the transit method. We'll learn the math behind light curves, how to read them, and what makes a signal look like a real planet vs. a false positive.
