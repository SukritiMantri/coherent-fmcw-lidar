# Next steps

Four planned extensions, listed in rough order of complexity:

## 1. 5×5 scan grid on the headline notebook

Extend `realistic_time_resolved_with_doppler.ipynb` from a single on-axis direction to the 25-direction scan that `coherent_fmcw_lidar.ipynb` already does — producing a visible mini point cloud with physical-chirp range and velocity per point. The scanner primitive is already wired in and the position+tilt-matched LO logic exists in `coherent_fmcw_lidar.ipynb` for reference; primarily a runtime concern (25 directions × ~15 min each).

## 2. Atmospheric turbulence

Add random Kolmogorov phase screens along the propagation path and run Monte Carlo over independent atmospheric realizations to get distributions of Strehl ratio, heterodyne efficiency, and the resulting degradation in FMCW range and velocity precision. The wavefront-aberration toolkit is set up to receive these aberrations cleanly.

## 3. Diffuse-target speckle

Replace the planar mirror with a diffusely scattering surface. Each random scatterer realization gives a different speckle pattern at the receive aperture and a different overlap with the LO. This is what real automotive targets (asphalt, paint, foliage) look like and is typically the dominant SNR penalty in production systems.

## 4. Full production source modeling

Replace the "25 mm collimated beam" abstraction with the actual source chain: laser diode → fast/slow-axis collimation → fiber coupling → more necessary optical elements → scanner. Captures source astigmatism, coupling losses, and scanner kinematics that the current notebooks abstract away.
