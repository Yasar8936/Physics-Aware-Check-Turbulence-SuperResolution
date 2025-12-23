Project: TurbulenceSR-Channel5200
--------------------------------

1) What this project is
This project builds a small machine learning (ML) baseline for “super-resolution” of turbulence data.
The goal is to reconstruct a higher-resolution 3D velocity field (HR) from a lower-resolution version (LR),
and compare the ML model against a simple baseline: trilinear interpolation.

In short:
- Input  : LR velocity block (coarse 3D grid)
- Output : HR velocity block (finer 3D grid)
- Baseline: Trilinear upsampling
- Model   : A neural network trained to reduce reconstruction error

2) What dataset we use
We work with small cutouts (patches) from a turbulent channel-flow dataset (Channel5200 style naming).
Each cutout is stored as HDF5 (.h5) and contains:
- velocity_0001  : velocity components packed together
- xcoor, ycoor, zcoor : grid coordinates
- metadata in the filename: X####-####, Y####-####, Z####-####, T#

Each sample has 3 velocity components:
- u: streamwise velocity
- v: wall-normal velocity
- w: spanwise velocity

The data is treated as 3D blocks:
- HR block size example: (16, 16, 16) per component
- LR block size example: (8, 8, 8) per component

3) What problem we are solving (in simple words)
Trilinear interpolation can upscale LR data, but it tends to smooth out important turbulent structures.
We train an ML model to predict HR fields that are closer to the ground-truth HR data than trilinear.

4) Current results (main takeaway)
The ML model beats trilinear interpolation on standard reconstruction metrics.

Example output (your numbers may differ depending on run/training seed):
- Validation MSE (baseline): 0.061085...
- Validation MSE (model)   : 0.054584...
- Relative improvement     : ~10.64%

We also track MAE similarly (lower is better).

So the key point:
✅ ML improves accuracy (MSE/MAE) compared to trilinear interpolation.

5) Physics limitation (important note)
Even if MSE/MAE improves, turbulence fields also need to respect physics.
For incompressible flow, one simple sanity check is “divergence”:

div(u) = du/dx + dv/dy + dw/dz

Ideally, divergence should be close to 0 everywhere.
In our checks, the ML output can have worse divergence than the baseline.
That means:

⚠️ The model is not physics-constrained yet.
It can be numerically accurate (good MSE/MAE), but still physically inconsistent.

This project intentionally stops here (accuracy-first).
Physics constraints are listed as future work.

6) What’s included in this repo
- DEMO_Getcutout_local.ipynb
  A single notebook that covers:
  (a) loading / inspecting cutout files
  (b) running ML vs baseline comparison
  (c) plotting RMS profiles / spectra (optional)
  (d) a simple physics check using divergence (optional)

- data/ 
  The HDF5 cutout files are large.
  It is recommended NOT to push raw data to GitHub unless you use Git LFS.

7) How to run (basic steps)
1) Create a Python environment and install dependencies (example):
   - numpy
   - h5py
   - torch
   - matplotlib

2) Put cutout .h5 files in a folder (example: giverny_output/ or data/raw/)

3) Open and run the notebook:
   DEMO_Getcutout_local.ipynb

4) Look at printed metrics:
   - Validation MSE/MAE (baseline vs model)
   - Optional divergence RMS (physics proxy)

8) How to interpret the plots/terms 
- Streamwise (u): flow direction along the channel
- Wall-normal (v): direction from wall toward the center of the channel
- Spanwise (w): side-to-side direction across the channel width
- RMS profile: how turbulence intensity changes from wall → center
- Spectrum: how energy is distributed across spatial frequencies (structures of different sizes)
- Divergence: physics sanity check for incompressible flow (should be small)

9) Where this project stops 
This repo focuses on the “ML beats trilinear in MSE/MAE” milestone.
Anything deeper (physics constraints, advanced turbulence diagnostics, stronger architectures)
is intentionally as future work.

10) Future work (next improvements)

Teach the model to follow basic fluid rules better (for example, reduce “mass-balance” errors like divergence).
Handle the wall region more carefully so predictions near the wall look more realistic.
Try training methods that help the model keep small swirling details, not just smooth shapes.
Test on more cutouts and more time steps to see if the results stay consistent.
Run the training a few times and report an average result (so it’s not just one lucky run).
Compare against a few more baselines, not only trilinear (stronger simple methods and other ML upscalers).

Author / Notes
“ML reconstruction improves MSE/MAE over trilinear, but physics constraints are future work.”
