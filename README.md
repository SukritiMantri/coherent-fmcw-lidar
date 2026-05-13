# Coherent FMCW LiDAR — Beam-Decomposition Simulation

A pair of Jupyter notebooks implementing a complete coherent frequency-modulated continuous-wave (FMCW) LiDAR signal chain using **Gaussian Beam Decomposition (GBD)**, plus a wavefront-aberration diagnostic toolkit using the standard optical-engineering metrics (RMS wavefront error, Strehl ratio, heterodyne efficiency η, Zernike decomposition).

Built as a self-study to deepen my understanding of coherent automotive LiDAR fundamentals.

> **Assumptions / scope.** "Complete signal chain" here means all the major coherent-FMCW blocks are present in their idealized form. The source is monochromatic with no laser phase noise, the lens is a thin perfect lens, the target is a perfect planar mirror with no speckle, propagation is in vacuum (no atmosphere), and there is no detector noise. The recovered range and velocity therefore have σ ≈ 0 across all 25 scan directions — that's the ideal-physics reference, not a model of production hardware.

## What's in here

### `coherent_fmcw_lidar.ipynb` — End-to-end LiDAR signal chain

For each direction in a 5×5 scan, the notebook builds the transmit beam from an 81-Gaussian-beamlet basis, steers it with a scanner, focuses it through a TX lens, propagates it to a planar mirror at R = 100 m, mixes the return with a mode-matched local oscillator, and extracts target range and radial velocity from a triangular FMCW chirp. Output: 25 LiDAR data points forming a small point cloud with (azimuth, elevation, R, v) per point.

Key technical pieces: 81-beamlet Gaussian Beam Decomposition with Tikhonov-regularized source decomposition; per-beamlet q-parameter tracking through scanner / thin lens / free-space propagation; image-method round-trip for the planar mirror target; mode-matched LO design (matching q-parameter, position, AND tilt simultaneously — critical for off-axis scan directions); triangular-chirp FMCW with up/down beat-frequency separation for joint R + v recovery. Realistic parameters throughout: 1550 nm wavelength, 1 GHz chirp bandwidth, 10 µs chirp duration, 200 MHz ADC sample rate.

`coherent_fmcw_lidar.html` is a single-page workflow visualization of the same pipeline (stages, per-direction measurement loop, parameters, sanity-check results).

### `wavefront_aberration_lab.ipynb` — Optical-engineering diagnostic toolkit

A standalone diagnostic that computes the standard wavefront-aberration metrics used throughout coherent LiDAR optical engineering. For each of five optical configurations (free space, +lens, +scanner, +mirror, +injected aberration), it reports the OPD map (2-D wavefront error in waves), RMS wavefront error (Maréchal criterion: λ/14 = diffraction-limited), Strehl ratio (extended Maréchal approximation), heterodyne efficiency η (the figure of merit for coherent detection — directly multiplies SNR), and a Zernike decomposition into the first 15 Noll-indexed modes.

The C4 demo injects a known mix of Zernike aberrations (0.10 λ defocus + 0.05 λ coma + 0.04 λ spherical) and recovers them exactly, verifying the diagnostic works correctly. The framework is set up to receive disturbances (atmospheric turbulence, lens defects, alignment errors) cleanly with no further code changes.

## Why coherent FMCW LiDAR

Coherent FMCW is the dominant 4D LiDAR technology in production automotive systems. Unlike time-of-flight LiDAR, it recovers velocity directly from a single chirp via the Doppler shift on the optical carrier, achieves sub-mm range precision through coherent peak-fitting, operates with eye-safe per-pulse energy at 1550 nm, and is naturally robust against ambient light and inter-LiDAR interference.

## Why Gaussian Beam Decomposition

GBD represents an arbitrary optical beam as a sum of complex-weighted Gaussian beamlets, each with its own q-parameter. Each beamlet propagates analytically through optical elements via closed-form q-parameter rules — no FFT required for the propagation itself. Compared to angular-spectrum FFT propagation, GBD is faster for tracking a beam through many optical elements, more accurate in the presence of strong phase chirps, and naturally extensible to non-Gaussian fields via re-decomposition. Production simulation tools use related decomposition methods.

## Running the notebooks

```bash
pip install numpy matplotlib jupyter
jupyter notebook coherent_fmcw_lidar.ipynb
```

Both notebooks are self-contained — no external dependency files. Each takes 1–2 minutes to execute end-to-end. They ship with their cell outputs already populated, so they're readable without re-execution.

## What's next

The natural extension is **atmospheric turbulence via split-step beam-propagation method (BPM) with random Kolmogorov phase screens** at moderate AV-typical conditions (Cₙ² ≈ 10⁻¹⁴ m⁻²/³ ground-level horizontal). The wavefront-aberration toolkit is set up to receive the resulting random aberrations cleanly: a Monte Carlo over many independent atmospheric realizations would give ensemble distributions of Strehl ratio, heterodyne efficiency, and the corresponding degradation in FMCW range and velocity precision. Diffuse-target speckle is the other natural addition.

---

*Built as a self-study project. Comments, corrections, and feedback welcome.*
