**# ROCkN Simulator v5.0 - Resilient Optical Clock & Quantum PNT Digital Twin**

**First-Principles Optical Lattice Clock + Resilient PNT Simulator**  
*Built from atomic physics, control theory, inertial navigation, MIL-STD-810G vibration, and adversarial signal processing - no black-box approximations.*

**Version**: 5.0 (March 2026)  
**License**: MIT (open-source; feel free to fork, extend, and deploy)  
**Author**: James Squire 

---

## Overview

ROCkN (Resilient Optical Clock & Navigation) Simulator v5.0 is a **complete end-to-end digital twin** of a strontium or ytterbium optical lattice clock integrated into military-grade Positioning, Navigation, and Timing (PNT) systems.  

It models every critical real-world degradation path — vibration, temperature, systematics, Dick effect, quantum projection noise, platform dynamics, inertial drift, clock networking, SWaP-C constraints, and adversarial jamming/spoofing — using **first-principles physics** rather than empirical fudge factors.

**Core innovation of v5.0**: All six major enhancements requested from v4 have been implemented with rigorous fidelity:
- Real MIL-STD-810G vibration PSD libraries + platform harmonics  
- True sigma-point Unscented Kalman Filter (Merwe-scaled, 2n+1 points)  
- Multi-platform clock network with Two-Way Time Transfer (TWTT)  
- Full SWaP-C engineering trade-space model  
- Physics-based Quantum Inertial Navigation Sensor (cold-atom interferometer)  
- Adversarial jamming/spoofing detection via clock stability monitoring  

The simulator runs as a single, dependency-light Python script and delivers an interactive 12-tab GUI built purely with `matplotlib`.

---

## Requirements

### Minimal (tested)
- **Python 3.9+**
- `numpy` (for all physics & linear algebra)
- `matplotlib` (for interactive GUI, sliders, tabs, and export)

**No other external dependencies.**  
No SciPy, no Pandas, no Qt/Tkinter — deliberately kept portable for field/lab deployment and air-gapped environments.

### Recommended Hardware
- Any modern laptop (CPU only; no GPU required)  
- 8 GB RAM sufficient (pre-generates ~10k noise samples)  
- For fastest slider response: run on a machine with ≥4 cores

### Installation

```bash
pip install numpy matplotlib
python ROCkNsim_v5.py
```

The script auto-generates all noise pools (MIL-STD-810 traces, laser noise, temperature) at startup and opens the full 12-tab interface.

---

## Architecture (First-Principles Modular Design)

The codebase is deliberately engineered as a **composable physics library** rather than a monolithic script. Every class maps 1:1 to a real subsystem:

| Class                  | Domain                          | Key First-Principles Implementation |
|------------------------|---------------------------------|-------------------------------------|
| `MilStd810`            | Environmental vibration        | Piecewise log-log PSD breakpoints + coherent blade/rotor harmonics from MIL-STD-810G Table 514.6-II |
| `OpticalLatticeClock`  | Atomic clock physics           | h₀/h₋₁/h₋₂ noise, Dick effect (single & interleaved), quantum projection noise, spin-squeezing ξ², vibration-to-frequency coupling |
| `Systematics`          | Accuracy budget                | 6-term shifts + full 6×6 covariance + correlated Monte-Carlo (30k samples) |
| `Servo`                | Control loop                   | Discrete-time PI + VCO/AOM actuator + real-time environmental coupling (BBR + vibration) |
| `TrueSigmaUKF`         | Sensor fusion                  | Merwe-scaled unscented transform (α/β/κ tunable), dynamic Q modulation from QuantumINS acceleration |
| `QuantumINS`           | Inertial navigation            | Kasevich-Chu atom interferometer: σ_φ = 1/(C√N), σ_a ∝ 1/T², σ_Ω ∝ 1/(vT²), CMR vibration rejection, wavefront bias |
| `PNTSimulator`         | Navigation error propagation   | Clock timing → range error + full QuantumINS position error (double integration + gyro heading) |
| `ClockNetwork`         | Distributed timing             | Heterogeneous nodes + TWTT link noise models + dropout probability + ensemble averaging |
| `SWaPCModel`           | Systems engineering            | Component database + scaling laws (P∝N⁰·⁶, M∝N⁰·¹, σ∝N⁻⁰·⁵) + platform feasibility margins |
| `AdversarialMonitor`   | Electronic warfare             | Threshold + CUSUM + Allan-anomaly detection for jamming, spoofing, meaconing |

**Data flow (first-principles pipeline)**:  
`Laser noise + MIL-STD-810 vibration + temperature → Servo → Locked clock → TrueSigmaUKF (with QuantumINS accel feed) → PNT trajectory + Clock Network + Adversarial detection`

All noise is generated via FFT from power-law PSDs or real MIL-STD-810 profiles — **no synthetic white/flicker/random-walk shortcuts**.

---

## Interactive GUI — 12 Tabs

| Tab # | Tab Name              | What You Can Explore |
|-------|-----------------------|----------------------|
| 0     | Laser Stability       | Allan deviation (free, Dick, QPN, spin-squeezed) with live sliders |
| 1     | Ramsey Fringes        | Contrast loss, QPN band, squeezing effects |
| 2     | Environ (MIL-810)     | Time-domain vibration coupling + PSD comparison across 7 platforms |
| 3     | Holdover & PNT        | UKF fusion, GPS outage, timing error vs Cs reference |
| 4     | Systematics           | 6-term shift bar chart + covariance heatmap + sensitivity curves |
| 5     | Accuracy Budget       | Monte-Carlo histogram + variance pie + cumulative uncertainty |
| 6     | Servo Demo            | Real-time locked vs free-running traces + live PI gains |
| 7     | PNT Trajectory        | 6-hour GPS-denied flight path + error envelopes (Quantum vs v4 heuristic) |
| 8     | Clock Network         | Ensemble TWTT performance + link budget vs distance + node scaling |
| 9     | SWaP-C                | Atom-number trade curves + component pies + platform feasibility margins |
| 10    | Quantum INS           | Cold-atom Allan devs vs MEMS + position error vs config + T-interrogation sensitivity |
| 11    | Adversarial           | Live jamming/spoofing/meaconing scenarios with threshold + CUSUM overlays |

**Export buttons** (bottom right): PNG per tab or full CSV of Allan data.

---

## Potential Uses

### 1. **Research & Publication**
- Quantitative studies of vibration-limited stability on real platforms
- Trade-space exploration for next-generation optical clocks (N atoms, squeezing, interrogation time)
- Validation of new servo/UKF/INS fusion algorithms before hardware

### 2. **Defense & Aerospace Engineering**
- MIL-STD-810 compliance analysis for fighter jets, UAVs, submarines, ships
- Resilient PNT holdover predictions (100 m requirement at 6+ hours)
- SWaP-C feasibility for size-constrained platforms (UAV vs ship)
- GPS-denied navigation error budgeting

### 3. **Education & Training**
- Interactive atomic-clock physics classroom tool
- Control-theory demonstration (Dick effect, servo tuning, UKF sigma points)
- Adversarial PNT awareness (how clocks detect spoofing)

### 4. **Industry & Standards**
- Clock-network design (TWTT optical links, ensemble averaging)
- Quantum-sensor roadmapping (cold-atom INS performance)
- Adversarial resilience testing for modernized PNT systems

---

## Suggested Future Extensions (v6 Roadmap)

From first-principles analysis, the following extensions would push the simulator to production-grade hardware-in-the-loop fidelity:

1. **Full 6-DoF Strapdown Quantum INS**  
   Quaternion-based attitude propagation + Coriolis coupling + real-time vibration feed-forward.

2. **Cavity & Laser Physics Module**  
   Thermal noise, vibration-insensitive mounting, cryogenic operation, and squeezed-light input models.

3. **Multi-Clock Network Kalman Filter**  
   Replace simple averaging with a distributed UKF across heterogeneous nodes.

4. **Machine-Learning Anomaly Detector**  
   Train on simulated adversarial residuals for next-gen spoof detection.

5. **Export to FMU / Hardware-in-the-Loop**  
   Functional Mock-up Unit standard for integration with real-time simulators (e.g. dSPACE, OPAL-RT).

6. **Environmental Realism Extensions**  
   - Dynamic magnetic field + Zeeman compensation  
   - Blackbody radiation real-time model with sensor feedback  
   - Platform-specific manoeuvre libraries (turns, acceleration profiles)

7. **Performance & Usability**  
   - Optional Numba/JIT acceleration for longer Monte-Carlo runs  
   - Dark/light theme toggle + high-DPI support  
   - Parameter presets for common platforms (F-35, MQ-9 Reaper, Virginia-class sub)

8. **Quantum-Enhanced Features**  
   - Entangled clock networks  
   - Spin-squeezed + number-squeezed lattice models  
   - Hybrid optical + microwave clock comparisons

---

## Parameter Quick Reference (excerpt)

See full table in the code for defaults and ranges. Key tunable physics parameters include:
- `h0`, `hm1` — laser noise coefficients
- `N_atoms`, `xi2` — atom number & squeezing
- `platform` — MIL-STD-810 profile
- `T_int_atom`, `N_atoms` (QINS) — cold-atom parameters
- `Kp`, `Ki`, `tau_act` — servo tuning
- `n_nodes`, `link_type` — network configuration

---

**ROCkN v5.0** demonstrates what is possible when simulation is built from fundamental physics rather than curve-fitting. It is ready for immediate use in research, education, and early-stage engineering design.

**Run it. Tweak it. Break the limits of classical PNT.**

**Happy simulating — may your Allan deviation always stay below 10⁻¹⁸.**
