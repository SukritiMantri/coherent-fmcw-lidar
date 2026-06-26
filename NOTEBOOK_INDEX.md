# Notebook Index

These are the build-up notebooks behind `coherent_fmcw_lidar.ipynb` and `wavefront_aberration_lab.ipynb`. Each notebook verifies a single new piece of physics or machinery before the next one stacks on top, so any bug surfaces in isolation rather than at the end of a large pipeline.

All build-up notebooks live in the `build_up_notebooks/` subfolder. The two headline notebooks at the repo root are self-contained — you do not need to run the GBD_# series to read them. The series is included so anyone interested can see exactly how each piece was validated against either a closed-form textbook expression or the angular-spectrum FFT reference.

## Era 1 — 1-D propagator: validate the q-parameter formalism

| Notebook | Adds |
|---|---|
| `build_up_notebooks/GBD_5.ipynb` | 1-D single-Gaussian-beamlet propagator using the complex q-parameter form (`q(z) = z + q₀`, `q₀ = −i·zR`). Validated against textbook `w(z)`, `R(z)`, and Gouy phase. |
| `build_up_notebooks/GBD_6.ipynb` | 1-D full-basis sum: solve the source-to-basis decomposition once at z = 0, reuse the same complex coefficients at every other z, sum at the output plane. |
| `build_up_notebooks/GBD_7.ipynb` | 1-D angular-spectrum FFT reference implementation. Used as the gold-standard check on the GBD answer; flags the paraxial-validity limits of the GBD_4–7 toy parameters. |
| `build_up_notebooks/GBD_8.ipynb` | Switches to LiDAR-realistic parameters (λ = 1550 nm, w₀ = 1 mm, range up to 200 m) and validates GBD vs. AS across the full LiDAR operating range. **Closes Era 1.** |

## Era 2 — extend to 2-D

| Notebook | Adds |
|---|---|
| `build_up_notebooks/GBD_9.ipynb` | 2-D single-beamlet propagator (the 2-D analog of GBD_5), with the subtle 2-D-vs-1-D differences (amplitude scaling `w₀/w(z)`, full Gouy `arctan(z/zR)`) called out explicitly. |
| `build_up_notebooks/GBD_10.ipynb` | 2-D full-basis sum: 81 (= 9×9) Gaussian beamlets, "decompose once at z = 0, propagate the basis, weighted-sum at any z" generalized to 2-D. |
| `build_up_notebooks/GBD_11.ipynb` | 2-D angular-spectrum FFT reference + GBD-vs-AS sweep across z. Residuals match the textbook leading paraxial correction `θ_div²·z/zR` to within a factor of 2–3. **Closes Era 2.** |

## Era 3 (Phase 3A) — optical elements and the target

| Notebook | Adds |
|---|---|
| `build_up_notebooks/GBD_12.ipynb` | Thin-lens element via per-beamlet q-update (`1/q′ = 1/q − 1/f`), aperture via grid mask, plus a chirp-aliasing diagnostic showing GBD analytic is strictly more accurate than AS for chirped beams. |
| `build_up_notebooks/GBD_13.ipynb` | Closes the LiDAR loop: source → TX lens → planar mirror at R = 100 m via the image method → return → RX aperture → re-decomposition into a receive-plane basis. Two-basis design pattern. |
| `build_up_notebooks/GBD_14.ipynb` | Adds the scanner as a per-beamlet `(kx, ky)` tilt, verified to land the spot at `(2R·tan(θₓ), 2R·tan(θ_y))` to sub-µm precision. **Closes Phase 3A.** |

## Era 4 — FMCW signal chain

| Notebook | Adds |
|---|---|
| `build_up_notebooks/GBD_15.ipynb` | Adds the FMCW chirp and time domain: spatial overlap with a mode-matched LO drives the photocurrent, FFT extracts the beat frequency, range follows from `R = f_beat · cT/(2B)`. Includes a Nyquist-aliasing demonstration at R = 200 m. |
| `build_up_notebooks/GBD_16.ipynb` | Adds Doppler via a triangular chirp (up-slope + down-slope): joint recovery of range and signed radial velocity from the beat-frequency pair, validated across v ∈ {−20, −10, 0, +10, +20} m/s. **Closes Era 4 minimum.** |
| `build_up_notebooks/GBD_17.ipynb` | Mini point cloud: combines scanner + FMCW + Doppler + mode-matched LO (matching `q_anchor`, position, AND tilt — all three are required for off-axis directions to couple) into a 5×5 scan that produces 25 `(x, y, R, v)` points. The cleaned-up presentation of this notebook is `coherent_fmcw_lidar.ipynb` at the repo root. |

## How the headline notebooks relate to the series

`coherent_fmcw_lidar.ipynb` is a polished, standalone version of `GBD_17.ipynb` — same physics, same numerical results, with the markdown rewritten so a reader does not need any of the prior notebooks to follow it. Use the headline version for reading; refer to `GBD_17.ipynb` if you want to see how it sat in the build-up sequence.

`wavefront_aberration_lab.ipynb` is independent of the GBD_# series. It reuses the same beam-decomposition primitives but builds a separate optical-engineering diagnostic (OPD, RMS, Strehl, heterodyne efficiency η, Zernike decomposition) on top of them, and validates the diagnostic by injecting a known mix of Zernike aberrations and recovering them exactly.

`realistic_time_resolved_fmcw_lidar.ipynb` is a complementary demonstration that builds on `coherent_fmcw_lidar.ipynb`. It addresses one specific shortcoming of that notebook: the FMCW beat there is constructed analytically from `tau = 2R/c` rather than derived from the chirp's physical propagation — there is no time dimension in the spatial calculation. The complementary notebook evaluates the spatial GBD calculation at multiple chirp wavelengths (time-resolved chirp slices) and extracts R from the propagation-phase rotation as the chirp sweeps `k(t)`. The scanner, Doppler, and TX lens are omitted to keep the chirp-as-actual-physics story isolated; the lens is replaced by a 25 mm collimated beam. See its own intro for parameters and results.

## Reading order

If you want the shortest path: read `coherent_fmcw_lidar.ipynb` and `wavefront_aberration_lab.ipynb`. They cover everything important. The complementary `realistic_time_resolved_fmcw_lidar.ipynb` is worth reading if you want to see the chirp actually performing the range extraction (rather than the precomputed-`tau` form used in `coherent_fmcw_lidar.ipynb`).

If you want the full validation story: walk Era 1 → 2 → 3A → 4 in order, in `build_up_notebooks/`. Each notebook lists the specific sanity checks and benchmark numbers it produces, so you can verify any step by re-running it (each takes seconds to a couple of minutes).
