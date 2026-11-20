# Generalized Aris Analysis of Longitudinal and Temporal Moments in River Solute Transport

Code accompanying the manuscript *Generalized Aris Analysis of Longitudinal and Temporal Moments in River Solute Transport* (Manuel M. Reyna).

The repository contains a Jupyter notebook that implements the quasi-three-dimensional advection–diffusion (Q3D-ADE) moment analysis described in the paper, using the St. Clair River ADCP cross-section as the worked example. Starting from a measured velocity section, it builds the cross-sectional diffusion operator, computes its eigenmodes, and uses them to obtain the full time evolution of longitudinal and temporal moments, concentration distributions and breakthrough curves, and the asymptotic link to memory-function exchange (MFE) models.

## What the method does

Classical Aris analysis gives evolution equations for the longitudinal moments of a solute plume. Here they are solved with a modal decomposition of the cross-sectional diffusion operator, so that:

- **Moments** (any order, nonuniform injections allowed) are obtained in Laplace space and inverted numerically with the de Hoog algorithm.
- **Concentration distributions** are obtained from the characteristic function of the modal system and inverted with the FFT. Breakthrough curves are read along the characteristic line ξ′(τ) = ξ − μτ.
- **Temporal moments** (longitudinal dependence) follow from the same modal system, with the roles of θ and s exchanged.
- **Asymptotic cumulant growth rates** are computed from the dominant eigenvalue of a small matrix pencil, and matched to first-order (or higher) MFE models.

## Repository contents

| File | Description |
|---|---|
| `generalized_aris_analysis.ipynb` | Main notebook: full workflow from the ADCP section to moments, concentration fields, breakthrough curves and MFE matching. |
| `StClairCrossSection.xlsx` | St. Clair River ADCP section. *Add ADCP data (placeholder name for St Clair River data but other data can be used)* |

## Notebook walkthrough

| Section | What it does |
|---|---|
| **Loading and completing the velocity profile** | Reads the ADCP spreadsheet and extrapolates velocities to the bed (law of the wall, κ = 0.41) and to the free surface (Coles wake profile, fitted per vertical by least squares). Estimates the shear velocity u\* at each position, then resamples onto a uniform transverse grid. |
| **Eigenvalue problem** | Sets the dimensionless diffusivities (D_y = 0.6 u\*d, D_z = 0.067 u\*d, following Fischer et al. 1979), builds the sparse cross-sectional diffusion operator with no-flux boundaries (finite volumes, harmonic face averaging), computes the slowest modes with `eigsh`, and projects the initial conditions (uniform or point release). Reports the asymptotic dispersion coefficient, Reynolds and Froude numbers, and compares with a 1D integral formula and a regression estimate (Deng et al. 2001). Includes an animation of cross-sectional mixing. |
| **Reduction of number of modes** | Keeps the modes with the largest contribution to the asymptotic dispersion (default `N_ = 20` of the 30 computed) and assembles the modal matrices `M_1`, `M_2`, `M_2_adv`. |
| **Function** | Python port of the `invlap` de Hoog numerical Laplace inversion (Hollenbeck, 1998; see Credits). |
| **Spatial moments** | Time evolution of the longitudinal cumulant growth rates for uniform and central injections, checked against the asymptotic expressions. |
| **Concentration distributions** | Characteristic function from the modal ODE system, inverted by FFT with zero padding. Produces the concentration field, breakthrough curves, and moment comparisons against the semi-analytical moments. |
| **Asymptotic matching of moments** | Cumulant growth rates from the dominant pole (using `numdifftools`), and fitting of MFE parameters by matching the first 2 + 2*N_rates cumulant rates (differential evolution followed by Nelder–Mead). Compares 1D ADE and first-order MFE breakthrough curves against Q3D-ADE. |
| **Temporal moments** | Longitudinal dependence of temporal raw and central moments, cumulant growth rates and their asymptotics. |

## Requirements

- Python ≥ 3.10
- `numpy` ≥ 2.0 (the notebook uses `np.trapezoid`)
- `scipy`
- `pandas` and `openpyxl` (to read the Excel file)
- `matplotlib`
- `numdifftools`

## Data

The ADCP velocity data for the St. Clair River were collected by the U.S. Geological Survey Illinois Water Science Center as part of a study commissioned by the International Joint Commission on the conveyance capacity of the river. The notebook uses cross-section 17, located 500 m downstream of the Black River.

The Los Molinos–Córdoba Canal application in the manuscript (ADCP section and tracer breakthrough curves) is not part of this notebook. 

These data are not our own.


## Credits

- The numerical Laplace inversion is a Python translation of `invlap.m` by Karl J. Hollenbeck (1998), based on the algorithm of de Hoog, Knight and Stokes (1982). If you publish work using it, please cite: Hollenbeck, K. J. (1998), *INVLAP.M: A MATLAB function for numerical inversion of Laplace transforms by the de Hoog algorithm*.
- The framework extends the moment analysis of Aris (1956) and the modal-decomposition approach of Barton (1983).
