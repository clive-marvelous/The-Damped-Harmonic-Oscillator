# The Damped Harmonic Oscillator
Author: Clive Marvelous 
Institution: University of Manchester — PHYS20762: Computational Physics 
Supervisor: Dr. Draga Pihler-Puzović 
Created: Feb 2023 | Updated: April 2026

---

## Overview

This project numerically investigates the dynamics of a forced spring-mass system governed by the damped harmonic oscillator equation. The two core goals are: (1) derive and explore analytical solutions for special cases of the forcing function, and (2) implement, compare, and critically evaluate four numerical integration schemes against those known solutions.

The project was completed as a Jupyter notebook in Python, using NumPy for computation and Matplotlib/Seaborn for visualisation. All integrators were written from scratch as modular functions to allow direct substitution when benchmarking methods.

---

## Physical System

The system models a mass on a spring with linear damping, described by:

> **mx'' + bx' + kx = F(t)**

| Parameter | Value |
|-----------|-------|
| Mass *m* | 2.37 kg |
| Spring constant *k* | 0.95 N/m |
| Default damping *b* | 0.08 kg/s (underdamped) |
| Critical damping *b_cr* | ≈ 2.997 kg/s |
| Initial displacement *x₀* | 0 m |
| Initial velocity *v₀* | −1 m/s |

The default damping constant is far below the critical value, placing the system in the **underdamped** regime where the mass oscillates with exponentially decaying amplitude.

---

## Numerical Methods Implemented

Four integrators were implemented, each derived from different truncations of the Taylor series:

| Method | Per-step Error | Key Property |
|--------|---------------|--------------|
| **Euler** | O(h²) | Simplest; energy drifts upward |
| **Improved Euler** | O(h³) on position | More accurate than Euler; still gains energy |
| **Euler-Cromer** | O(h²) | Symplectic — conserves *average* energy per cycle |
| **Verlet** | O(h⁴) | Most accurate; near-exact energy conservation |

**Verlet's method** is not self-starting — it requires an auxiliary integrator to compute the second position value *x[1]* before the main recurrence can begin. The Improved Euler method was selected for this role based on its superior accuracy.

Analytical solutions were also implemented for all three damping regimes (underdamped, critically damped, overdamped) and used as ground-truth benchmarks throughout.

---

## Project Structure (8 Objectives)

### 1. Implementing the Integrators
All four numerical methods and the analytical solutions are defined as reusable Python functions, with no external forcing (*F* = 0) and standard initial conditions.

### 2. Step-Size Sensitivity
Each method was run with step sizes *h* ∈ {0.005, 0.01, 0.05, 0.1} s and compared against the analytical solution. Smaller *h* consistently improves accuracy, with *h* = 0.005 s giving the best results. Verlet's method tolerates larger step sizes better than the others due to its lower truncation error.

### 3. Accuracy Across Damping Values
At a fixed step size (*H* = 0.0001 s), each method was tested across five damping values from *b* = 0 to *b* = 1 kg/s:
- **Euler / Improved Euler** — accurate for large *b*, but amplitude grows without bound for small or zero damping due to spurious energy gain.
- **Euler-Cromer** — accurate for small/zero damping (energy conserved on average), but fails for large *b* as it cannot reproduce amplitude decay.
- **Verlet** — accurate across all tested damping values.

### 4. Energy Conservation Test
Total mechanical energy *E = ½(mv² + kx²)* was computed numerically for the undamped case, where exact conservation is expected. This is a stringent accuracy test:
- Euler and Improved Euler show monotonically growing energy — explaining the amplitude blow-up seen in Objective 3.
- Euler-Cromer's energy oscillates but averages to the correct value — confirming its symplectic character.
- Verlet's energy remains essentially constant for small *h*, confirming it as the most physically faithful integrator.

### 5. Damping Regimes
Verlet's method was used to reproduce all three canonical cases — underdamped, critically damped, and overdamped — against the respective analytical solutions. Close agreement was achieved in all cases.

### 6. Impulse Forcing
Since a truly instantaneous force cannot be represented on a discrete time grid, the impulse was modelled as a **Gaussian pulse** centred at the desired application time. Four scenarios were tested (positive/negative force, applied at different cycle phases). Key findings:
- A force aligned with the instantaneous velocity increases the amplitude and preserves phase.
- A force opposing the velocity decreases the amplitude and introduces a phase shift proportional to the timing of the impulse.

### 7. Sinusoidal Forcing and Steady-State Behaviour
With a sinusoidal driving force *F(t) = F₀ cos(ωt)*, the equation becomes an inhomogeneous ODE. The general solution is the sum of a transient (complementary function) and a steady-state (particular integral). Verlet's method was applied for lightly damped, heavily damped, and critically damped cases — all matching the analytical solution closely. The transient component decays over time, leaving the system in the steady-state response.

### 8. Resonance
The steady-state amplitude was computed numerically across 200 driving frequencies spanning ±0.1 Hz around the natural frequency *f₀*, for three values of *b*:
- The amplitude peaks sharply near *f₀* — the resonance effect.
- Smaller *b* produces a taller, narrower peak.
- As *b* increases, the peak shifts slightly below *f₀* and broadens.
- For zero damping, the amplitude grows without bound as *f → f₀*.

### Round-Off Error Analysis
The optimal step size for Verlet's method was determined by computing the error in the centred finite-difference velocity approximation across 16 decades of *h* (from *h* = 1 to *h* = 10⁻¹⁵). The minimum error occurs at **h ≈ 10⁻⁵ s** — below this, floating-point cancellation dominates; above it, truncation error dominates.

---

## Key Conclusions

- **Verlet's method** is the most accurate and general integrator for this problem, owing to its O(h⁴) truncation error and near-exact energy conservation.
- **Euler-Cromer** is the best choice for undamped or lightly damped systems specifically, due to its symplectic energy behaviour.
- **Euler and Improved Euler** are unsuitable for long-time or low-damping simulations due to systematic energy drift.
- The optimal integration step size is **h ≈ 10⁻⁵ s**, balancing truncation and floating-point round-off errors.

---

## Dependencies

```
numpy
matplotlib
seaborn
scipy
cmath (standard library)
```

Install with:
```bash
pip install numpy matplotlib seaborn scipy
```
