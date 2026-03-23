# N-body Simulation of Water Content Estimation in Terrestrial Planets

**Bachelor’s Thesis — Nihon University, Department of Physics (Astrophysics Lab)**  
**Author:** Teriya Tsukui  
**Supervisor:** Prof. Shimami Fujii  
**Submitted:** February 2022

-----

## Overview

This project implements a gravitational N-body simulation to estimate the water content of terrestrial planets during their formation.

The simulation is based on the model proposed by **Machida & Abe (2010)**, which assumes that a dense dust cloud lowers the temperature inside the protoplanetary disk, pushing the H₂O snowline inward to approximately **0.75 AU** — well within Earth’s orbital region. Under this model, protoplanets forming inside the snowline can still retain water (water fraction: ~0.59 wt%), unlike the classical standard model (Hayashi model) which predicts a dry inner disk.

The goal is to track the orbital evolution and collisional accretion of protoplanets, and thereby estimate the final water content of terrestrial planets.

-----

## Physical Background

|Parameter                                     |Value                 |
|----------------------------------------------|----------------------|
|Snowline position (this model)                |0.75 AU               |
|Snowline position (standard model)            |~2.7 AU               |
|Water fraction of protoplanets beyond snowline|0.59 wt%              |
|Protoplanet mass range                        |~0.1 M⊕ (10²²–10²³ kg)|
|Disk ring width                               |0.5–1.5 AU            |
|Softening parameter ε                         |1×10⁻⁶ AU             |

-----

## Numerical Method

**4th-Order Hermite Integrator** (Makino & Aarseth 1992)

This is a predictor-corrector method widely used in collisional N-body simulations. It integrates the equations of motion using both acceleration **a** and jerk **ȧ**, then applies Hermite interpolation to compute higher-order correction terms.

### Algorithm (per timestep, per particle i)

1. Compute acceleration **a₀** and jerk **ȧ₀** from current positions/velocities
1. Compute predictor positions **xₚ** and velocities **vₚ** (Taylor expansion to jerk term)
1. Using predicted positions, compute new acceleration **a₁** and jerk **ȧ₁**
1. Compute Hermite interpolation coefficients **a₀⁽²⁾** and **a₀⁽³⁾**
1. Apply corrector to obtain refined **xc** and **vc**

### Collision Model

- Perfect merging assumed (no fragmentation)
- Collision condition: `α(r₁ + r₂) ≥ d` where α = 5.0
- Post-collision mass, radius, density, and water fraction updated via conservation laws
- The smaller-index particle absorbs the other; the consumed particle is moved to the origin with zero mass

-----

## Code Structure

|File                     |Description                                                                         |
|-------------------------|------------------------------------------------------------------------------------|
|`init_pos_pp_v2.c`       |Generates initial conditions for protoplanets (positions, velocities, masses, radii)|
|`hermite_nbody_v9.c`     |Main N-body integrator using the 4th-order Hermite scheme                           |
|`display_hermite_2D_v3.c`|Visualizes simulation snapshots as a 2D animated GIF via gnuplot                    |
|`display_hermite_ELA.c`  |Plots conserved quantities (total energy E, angular momentum L) over time           |

-----

## How to Build and Run

### Requirements

- GCC (C compiler)
- GNU Make (optional)
- gnuplot (for visualization)

### Compile

```bash
gcc -o init_pos init_pos_pp_v2.c -lm
gcc -o nbody hermite_nbody_v9.c -lm
gcc -o display_2d display_hermite_2D_v3.c -lm
gcc -o display_ela display_hermite_ELA.c -lm
```

### Step 1: Generate Initial Conditions

```bash
./init_pos
```

Follow the interactive prompts to set disk radius range, mass range, softening parameter, and initial velocity mode (Keplerian rotation recommended).

### Step 2: Run the Simulation

```bash
./nbody
```

Prompts will ask for simulation duration, timestep, softening parameter, collision handling (y/n), and energy/angular momentum output (y/n).

### Step 3: Visualize Results

```bash
# Animate particle positions
./display_2d

# Plot conserved quantities
./display_ela
```

-----

## Known Issues / Current Status

> ⚠️ **The simulation does not currently produce physically correct results.**

As of the thesis submission (February 2022), after 40+ rounds of debugging and revision:

- **Author’s code (`hermite_nbody_v9.c`):** Protoplanets undergo straight-line motion instead of Keplerian orbits. Kinetic energy remains constant while potential energy increases monotonically — indicating that gravitational acceleration and jerk are not being correctly applied to the equations of motion. Total energy becomes positive almost immediately (E > 0).
- **Reference code (`taylor_t_modi.c`, provided by a senior lab member):** Conserved quantities appear correct over short timescales (~1 yr), but total energy diverges significantly over longer runs (~10⁶ yr), suggesting accumulated numerical error from the fixed timestep being too large.

**Root cause hypothesis:** The acceleration and jerk terms are computed but not correctly propagated into the position/velocity updates.

**Planned fixes (future work):**

- Validate acceleration/jerk computation with simple two-body test cases
- Implement adaptive (individual) timesteps per particle
- Add Runge-Lenz vector conservation check

-----

## Physical Quantities and Units

|Quantity                |Unit used in code        |
|------------------------|-------------------------|
|Length                  |AU (1 AU = 1.496×10¹¹ m) |
|Mass                    |M☉ (1 M☉ = 1.988×10³⁰ kg)|
|Time                    |yr (1 yr = 3.156×10⁷ s)  |
|Gravitational constant G|39.466 AU³ M☉⁻¹ yr⁻²     |

-----

## Future Extensions

The following physics were identified but not yet implemented:

- Orbital evolution of collision-generated debris (smaller planetesimals)
- Jupiter’s gravitational perturbation and secular resonance sweeping
- Circularization of protoplanet orbits by residual disk gas
- Sublimation of ice from icy planetesimals due to solar radiation
- Water loss through magma ocean processes
- Differentiation effects on water retention capacity

-----

## References

1. Machida R., Abe Y. (2010). *Terrestrial Planet Formation Through Accretion of Sublimating Icy Planetesimals in a Cold Nebula.* ApJ, 716, 1252–1262.
1. Makino J., Aarseth S.J. (1992). *On a Hermite Integrator with Ahmad-Cohen Scheme for Gravitational Many-Body Problems.* PASJ, 44, 141–151.
1. Tagawa S., Sakamoto N., Hirose K., et al. (2021). *Experimental evidence for hydrogen incorporation into Earth’s core.* Nature Communications, 12, 2588.
1. Watanabe J., Ida S., Sasaki S. (2008). 太陽系と惑星 シリーズ現代の天文学 第9巻. 日本評論社.
1. Ida S., Nakamoto T. (2015). 惑星形成の物理 太陽系と系外惑星系の形成論入門. 共立出版.

-----

## Acknowledgements

Special thanks to Yuki Odanaka (lab alumnus) for providing the reference N-body code and permitting its modification, and to Prof. Shimami Fujii for patient guidance throughout this project.

-----

## License

This code was developed as undergraduate thesis work. Please contact the author before reuse or redistribution.
