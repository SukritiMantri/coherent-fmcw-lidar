# Coherent FMCW LiDAR — Beam-Decomposition Simulation

A self-directed engineering project implementing a complete coherent frequency-modulated continuous-wave (FMCW) LiDAR signal chain from first principles using **Gaussian Beam Decomposition (GBD)** — building each piece of the pipeline up incrementally against closed-form physics checks, then assembling them into an end-to-end simulation where the chirp itself drives range and velocity extraction from physical propagation — not from a plugged-in `tau = 2R/c`. The simulation lives in a single headline notebook; a staged lineage of precursor notebooks (which I built incrementally on the way to the headline) is included too, so each piece of physics can be checked in isolation. A standalone wavefront-aberration diagnostic toolkit (RMS wavefront error, Strehl ratio, heterodyne efficiency η, Zernike decomposition) sits alongside.

> **Assumptions / scope.** "Complete signal chain" here means all the major coherent-FMCW blocks are present in their idealized form. The source is monochromatic with no laser phase noise, the lens is a thin perfect lens, the target is a perfect planar mirror with no speckle, propagation is in vacuum (no atmosphere), and there is no detector noise.

## Repository contents

The headline deliverable is `realistic_time_resolved_with_doppler.ipynb` at the repo root (see Section 1 below). Three precursor notebooks at the repo root (`coherent_fmcw_lidar.ipynb`, `realistic_time_resolved_fmcw_lidar.ipynb`, `realistic_time_resolved_with_optics.ipynb`) are the stages that led up to it; each isolates one piece of the final pipeline. `wavefront_aberration_lab.ipynb` is a standalone diagnostic independent of the lineage. The rest of the files are companion material:

- `coherent_fmcw_lidar.html` — single-page workflow visualization of the precursor `coherent_fmcw_lidar.ipynb` (stages, per-direction measurement loop, parameters, sanity-check results).
- `realistic_time_resolved_fmcw_lidar.html` — workflow visualization of the first time-resolved precursor.
- `NOTEBOOK_INDEX.md` — one-line description of every `GBD_#.ipynb` build-up notebook (Era 1 → Era 4), so you can see at a glance what each step in the staged build-up adds.
- `NEXT_STEPS.md` — planned extensions (atmospheric turbulence, diffuse-target speckle, full production source modeling, 5×5 scan extension to the headline notebook).
- `build_up_notebooks/` — the GBD_5 through GBD_17 notebooks behind the pipeline. Each one verifies a single new piece of physics or machinery (q-parameter propagator → 2-D extension → angular-spectrum validation → thin lens → mirror target → scanner → FMCW chirp → Doppler → mini point cloud) before the next one stacks on top. They are not required reading — the repo-root notebooks are self-contained — but they are included so anyone who wants to verify a specific step can re-run it in seconds.

## What's in here

### 1. `realistic_time_resolved_with_doppler.ipynb` — End-to-end LiDAR signal chain with physical chirp

A complete coherent FMCW LiDAR simulation in which the chirp itself does the range and velocity extraction from physical propagation. The notebook builds the transmit beam from an 81-Gaussian-beamlet basis, steers it with a scanner, focuses it through a thin TX lens (f = 1 m), propagates it to a moving planar mirror at R = 50 m with v = +10 m/s approaching, mixes the return with a mode-matched local oscillator (designed for R_nominal = 100 m — i.e., deliberately off-nominal, so the LO and the actual signal are not mode-matched), and extracts both target range and radial velocity from a triangular (up + down) FMCW chirp. The spatial GBD calculation is re-evaluated at every chirp wavenumber `k_i` across one chirp duration — target range and Doppler shift fall out of how the beat phase rotates as the chirp sweeps `k(t)` and as the target moves.

**Key technical pieces:**
- Gaussian beam decomposition with 81-beamlet basis and Tikhonov-regularized least-squares source decomposition
- Per-beamlet q-parameter tracking through scanner, thin lens, and free-space propagation
- Image-method round-trip for the planar mirror target
- Mode-matched LO design at nominal range (q-parameter matched)
- Time-resolved chirp loop: spatial GBD re-evaluated at each `k_i`; R extracted from overlap phase slope (Method A) and photocurrent FFT peak (Method B)
- Target motion handled physically via moving receive plane `z_receive_i = 2 · R(t_i)` per slice — Doppler emerges, not plugged in
- Triangular chirp (up + down) for joint R + v recovery
- Realistic parameters: 1550 nm wavelength, 1 GHz chirp bandwidth, 10 µs chirp duration

**Scope.** Single on-axis scan direction only; no 5×5 point-cloud output. The scanner primitive is wired in — extending to a multi-direction scan is planned future work (see `NEXT_STEPS.md`). Runtime ≈ 20 minutes (thousands of chirp slices for Nyquist, for both up and down chirps).

### 2. Lineage — how we got here

The headline notebook is the end of a staged build-up. Each step validated one new piece of physics in isolation before the next one stacked on top, so any bug could surface at its own level rather than at the end of a large pipeline. Reading them in order is not required (the headline is self-contained), but they are included so any step can be re-run and checked.

**`build_up_notebooks/GBD_5.ipynb` through `GBD_17.ipynb`** — the staged validation series behind the beam-decomposition primitives. Era 1 validates the 1-D q-parameter propagator against textbook `w(z)`, `R(z)`, and Gouy phase. Era 2 extends to 2-D and validates against the angular-spectrum FFT reference. Era 3A adds thin lens, mirror target (image method), and scanner elements. Era 4 adds the FMCW chirp, Doppler via triangular chirp, and the 5×5 scan. See `NOTEBOOK_INDEX.md` for a one-line description of each.

**`coherent_fmcw_lidar.ipynb`** — the first end-to-end assembly of the GBD_# primitives into a working LiDAR simulation. Scans 25 directions and produces a mini point cloud with `(x, y, R, v)` per point. The chirp here is constructed analytically from `tau = 2R/c` rather than from time-resolved spatial GBD — but that lets the 25-direction scan run in ~1-2 minutes. A polished, standalone version of `GBD_17.ipynb`.

**`realistic_time_resolved_fmcw_lidar.ipynb`** — the first notebook where the chirp runs as actual physics. Scanner, Doppler, and TX focusing lens were stripped out so the chirp-as-physics story would be clean; the lens was replaced by a 25 mm collimated beam. R sweep at 1 / 100 / 200 m demonstrates that range extraction works from genuine propagation phase rotation.

**`realistic_time_resolved_with_optics.ipynb`** — puts the TX scanner and f = 1 m thin lens back in, so the time-resolved chirp now runs through the same optical train as `coherent_fmcw_lidar.ipynb`. Single on-axis direction, R sweep at 50 / 100 / 150 m. Confirms the chirp-as-physics story survives the full optical chain, not just a collimated beam.

The headline `realistic_time_resolved_with_doppler.ipynb` then adds target motion to that geometry and recovers both R and v from a triangular chirp.

### 3. `wavefront_aberration_lab.ipynb` — Optical-engineering diagnostic toolkit (standalone)

A standalone diagnostic that computes the standard wavefront-aberration metrics used throughout coherent LiDAR optical engineering. For each of five optical configurations (free space, +lens, +scanner, +mirror, +injected aberration), it reports the OPD map (2-D wavefront error in waves), RMS wavefront error (Maréchal criterion: λ/14 = diffraction-limited), Strehl ratio (extended Maréchal approximation), heterodyne efficiency η (the figure of merit for coherent detection — directly multiplies SNR), and a Zernike decomposition into the first 15 Noll-indexed modes.

The C4 demo injects a known mix of Zernike aberrations (0.10 λ defocus + 0.05 λ coma + 0.04 λ spherical) and recovers them exactly, verifying the diagnostic works correctly. The framework is set up to receive disturbances (atmospheric turbulence, lens defects, alignment errors) cleanly with no further code changes. Independent of the lineage — reuses the same beam-decomposition primitives but builds a separate analysis on top.

## Why coherent FMCW LiDAR

Coherent FMCW is the dominant 4D LiDAR technology in production automotive systems. Unlike time-of-flight LiDAR, it recovers velocity directly from a single chirp via the Doppler shift on the optical carrier, achieves sub-mm range precision through coherent peak-fitting, operates with eye-safe per-pulse energy at 1550 nm, and is naturally robust against ambient light and inter-LiDAR interference.

## Why Gaussian Beam Decomposition

GBD represents an arbitrary optical beam as a sum of complex-weighted Gaussian beamlets, each with its own q-parameter. Each beamlet propagates analytically through optical elements via closed-form q-parameter rules. It is naturally extensible to non-Gaussian fields via re-decomposition. Production simulation tools use related decomposition methods.

## Running the notebooks

```bash
pip install numpy matplotlib jupyter
jupyter notebook realistic_time_resolved_with_doppler.ipynb
```

All notebooks are self-contained — no external dependency files. The headline notebook (`realistic_time_resolved_with_doppler.ipynb`) runs in ~20 minutes (thousands of chirp slices for Nyquist, for both up and down chirps). The three precursor notebooks `coherent_fmcw_lidar.ipynb`, `realistic_time_resolved_fmcw_lidar.ipynb`, and `realistic_time_resolved_with_optics.ipynb` run in roughly 1-2 min, 10 min, and 20 min respectively. `wavefront_aberration_lab.ipynb` runs in ~1 min.

The notebooks ship with their cell outputs already populated, so they're readable without re-execution.

## Idealized Simplifications

**Idealized** for clarity: source is monochromatic with no laser phase noise; lens is a thin perfect lens with no aberrations; target is a perfect planar mirror with no speckle; vacuum propagation (no atmosphere); no detector noise. Real LiDAR systems sit below this performance ceiling because of the degradations not yet modeled.

## What's next

The natural next step is extending the headline notebook to a **5×5 scan grid**, producing a visible point cloud like `coherent_fmcw_lidar.ipynb` but with the physical chirp. The scanner primitive is already wired in — this is primarily a runtime concern (25 directions × ~20 min each).

After that, the main extension is **atmospheric turbulence via split-step beam-propagation method (BPM) with random Kolmogorov phase screens** at moderate AV-typical conditions (Cₙ² ≈ 10⁻¹⁴ m⁻²/³ ground-level horizontal, L₀ = 10 m outer scale). The wavefront-aberration toolkit (Section 3) is set up to receive the resulting random aberrations cleanly: a Monte Carlo over many independent atmospheric realizations would give ensemble distributions of Strehl ratio, heterodyne efficiency, and Zernike-mode amplitudes, plus the corresponding degradation in FMCW range and velocity precision. Diffuse-target speckle is the other natural addition.

See `NEXT_STEPS.md` for the planned extensions (5×5 scan on the headline notebook, atmospheric turbulence, diffuse-target speckle, and the full production source-modeling chain).

---

*This is a self-directed project. Comments, corrections, and pull requests welcome.*
