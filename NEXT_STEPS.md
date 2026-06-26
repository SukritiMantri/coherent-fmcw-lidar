# Next steps

Four planned extensions, listed in rough order of complexity:

## 1. Atmospheric turbulence

Add random Kolmogorov phase screens along the propagation path and run Monte Carlo over independent atmospheric realizations to get distributions of Strehl ratio, heterodyne efficiency, and the resulting degradation in FMCW range and velocity precision. The wavefront-aberration toolkit is set up to receive these aberrations cleanly.

## 2. Diffuse-target speckle

Replace the planar mirror with a diffusely scattering surface. Each random scatterer realization gives a different speckle pattern at the receive aperture and a different overlap with the LO. This is what real automotive targets (asphalt, paint, foliage) look like and is typically the dominant SNR penalty in production systems.

## 3. Add Doppler velocity to the time-resolved notebook

Extend `realistic_time_resolved_fmcw_lidar.ipynb` from a single up-slope chirp at a stationary target to a triangular chirp (up + down slope) at a moving target. Same architecture, just two chirp segments — sum of beats gives range, difference gives Doppler.

## 4. Full production source modeling

Replace the "25 mm collimated beam" abstraction with the actual source chain: laser diode → fast/slow-axis collimation → fiber coupling → more necessary optical elements → scanner. Captures source astigmatism, coupling losses, and scanner kinematics that the current notebooks abstract away.
