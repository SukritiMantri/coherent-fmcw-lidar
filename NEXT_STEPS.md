# Next steps

The wavefront-aberration toolkit (`wavefront_aberration_lab.ipynb`) is set up to receive each of the disturbances below cleanly with no further code changes — same aperture, same Zernike basis, same metrics.

## 1. Atmospheric turbulence

**Goal:** quantify how moderate atmospheric turbulence over a 100 m horizontal automotive path degrades the heterodyne efficiency, the Strehl ratio, and the recovered FMCW range and velocity precision.



## 2. Diffuse-target speckle

Replace the planar mirror with a diffusely scattering surface (Lambertian-style amplitude × random phase per scatterer). Each independent realization of the random scatterer phase gives a different speckle pattern at the receive aperture, which means a different overlap with the LO and a different beat-peak SNR. Doppler signature also changes — multiple sub-resolution scatterers at slightly different radial velocities produce a Doppler spread instead of a clean peak.

This is what real automotive targets (asphalt, vehicle paint, foliage) actually look like, and it sets a more realistic lower bound on the velocity precision than the perfect-mirror geometry does.
