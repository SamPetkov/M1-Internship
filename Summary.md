# Project Summary

This repository documents a progression of muscular-hydrostat simulation work.
The main contribution is not a single solver, but a set of paper-inspired models
that move from simpler mass-spring octopus mechanics toward more detailed
finite-element squid, octopus, and tongue formulations. The `Vavourakis/`
subfolder contains the main PDF references used to guide the later FEM work,
including Liang 2006 and the Vavourakis nonlinear dynamic FEM paper.

## 1. Yekutieli-Based Octopus Arm Models

The first major section of the work is based on Yekutieli-style octopus arm
extension. These codes model the octopus arm as an active, tapered,
muscular-hydrostat structure with activation waves, drag, and approximate
area/volume preservation. This branch is the main octopus mass-spring part of
the project.

### `main.c`: Main 2D Mass-Spring Octopus Model

`main.c` is the core Yekutieli-based model in the repository. It implements a
2D tapered octopus arm using paired masses per ring, longitudinal and transverse
springs, propagating muscle activation, nonlinear muscle behavior, anisotropic
drag, and area conservation constraints.

This file is the main starting point for the Yekutieli-based octopus work. It
captures the reduced 2D mechanics of arm extension while still keeping the key
biological ingredients: taper, activation timing, passive/active muscle
response, and hydrostatic-style conservation.

### `3d.c`: 3D Yekutieli-Inspired Arm

`3d.c` extends the 2D mass-spring idea into a 3D arm with four masses per ring.
The cross-section is represented by a square-like ring, and each segment uses a
tetrahedral decomposition to impose volume conservation. This was a step toward
testing whether the Yekutieli-inspired mechanics could be lifted from 2D area
constraints into 3D volume constraints.

### `cylinder.c`: 3D Cylindrical Variant

`cylinder.c` is the second 3D Yekutieli-inspired robot/arm model. It uses a more
cylindrical representation with outer nodes and center nodes per ring. This
version explores a different discretization for the same broad goal: creating a
3D active hydrostat arm that is closer to a robotic or physical arm geometry
than the reduced 2D model.

Together, `main.c`, `3d.c`, and `cylinder.c` form the Yekutieli-inspired octopus
and robot-arm section of the project.

## 2. Perkell 1974 Tongue Model

The next section of the work is `perkell1974.c`, based on Perkell's 1974 tongue
modeling paper. This code shifts from octopus arms to another muscular
hydrostat-like biological system: the tongue.

`perkell1974.c` implements a 2D tongue mechanics model with active and passive
components, structural damping, and stabilization constraints. The goal of this
section is to connect the broader muscular-hydrostat modeling effort to a
classic speech/tongue biomechanics reference rather than only cephalopod
locomotion or prey-capture systems.

Supporting output and visualization files include:

- `perkell_rk23_tongue_traj.csv`
- `visualize_perkell.py`

## 3. Liang 2006 Squid Tentacle Models

The Liang section develops a squid tentacle finite-element model inspired by
Liang 2006. This part of the project contains many iterative versions because
the model required repeated corrections to geometry, muscle-region assignment,
time integration, material updates, and hourglass/volume behavior.

The Liang paper in `Vavourakis/liang2006.pdf` describes an explicit finite
element scheme for biological muscular hydrostats such as squid tentacles,
octopus arms, and elephant trunks. The key idea is to embed muscle fibers inside
finite elements, allow arbitrary fiber orientations and multiple muscle
directions, and compute muscle stress as the sum of active and passive parts.
In that framework, active stress depends on activation state, fiber shortening
velocity, and fiber strain, while passive stress depends on strain.

The current SOTA Liang code in this repository is:

```text
liang_coarse_timefix_xy.c
```

This version keeps the Liang-style coarse Fig. 2(b) mesh, uses corrected
explicit step ordering, updates constitutive state after the kinematic step, and
uses fixed orthogonal transverse-muscle directions in the x/y plane. It also
writes solver pressure and debug fields to VTK, making it the best current
reference for the Liang branch.

Solver and model choices in the Liang branch:

- Solver: explicit finite-element time stepping, with the corrected Liang-style
  order of operations: assemble forces from the old state, update acceleration
  and half-step velocity, update displacement, then update constitutive state.
- Time integration: central-difference/leapfrog-like explicit dynamics rather
  than an implicit Newton solve.
- Material/model: Liang-style embedded muscle fibers inside finite elements,
  with active and passive fiber stress. Active stress depends on activation,
  fiber strain, and shortening velocity; passive stress depends on strain.
- Stabilization/numerics: reduced-integration hexahedral elements, viscous
  Flanagan-Belytschko hourglass control, pressure/volume diagnostics, and
  Rayleigh mass damping.
- Constitutive distinction: this branch is not Mooney-Rivlin; it follows the
  Liang explicit muscular-hydrostat formulation.

Important Liang development files:

- `liang2006.c`: early strict reproduction attempt.
- `liangogrid.c`: O-grid and hourglass-control development.
- `liangogridfiner.c`: refined O-grid experiment.
- `liang2006_v4.c` through `liang2006_v7.c`: successive physics and solver
  correction attempts.
- `liang_coarse.c`: coarse Liang mesh before the time-ordering correction.
- `liang_coarse_timefix.c`: corrected explicit time stepping.
- `liang_coarse_timefix_xy.c`: current best Liang implementation.
- `liang_xy_crossection_scale2.c`: cross-section refinement experiment.

## 4. Vavourakis-Inspired Squid and Octopus Models

The final major section is based on Vavourakis et al.'s nonlinear dynamic FEM
formulation for muscular hydrostats. This section includes both standalone
OpenMP implementations and PETSc/libMesh reference implementations.

The Vavourakis paper in `Vavourakis/A nonlinear dynamic finite element approach
for simulating muscular hydrostats.pdf` develops an implicit nonlinear FEM
method for dynamic, three-dimensional, nearly incompressible, hyperelastic
muscle undergoing large deformation. The paper models total stress as a
superposition of connective-tissue stress and fiber stress: the connective
tissue uses a Mooney-Rivlin law, while the fibers combine active and passive
stress contributions. The paper validates the approach on squid arm extension
and frog gastrocnemius contraction, then applies it to octopus arm maneuvers
such as bending and reaching.

### Standalone Squid: `vavourakis_squid_standalone_explicit_omp.c`

`vavourakis_squid_standalone_explicit_omp.c` is the current standalone OpenMP
squid code. It is designed to be dependency-light and portable while preserving
the main Vavourakis-inspired ingredients: nearly incompressible connective
tissue, muscle-fiber stress, squid benchmark subdomains, internal O-grid mesh
structure, and explicit time stepping. It deliberately trades the paper's full
implicit PETSc/libMesh setup for a standalone OpenMP implementation that is
easier to build and test.

Solver/model details:

- Solver: explicit lumped-mass central-difference update.
- Material/model: modified-invariant Mooney-Rivlin connective tissue plus
  additive active/passive muscle-fiber stresses.
- Numerical additions: OpenMP force assembly, hourglass control,
  near-incompressibility/bulk response, damping, and VTK/CSV diagnostics.

This file is the practical current SOTA for the standalone squid branch.

Related standalone squid files:

- `vavourakis_squid_explicit_fixed_root.c`
- `vavourakis_squid_standalone_implicit_newtonkrylov.c`

### Standalone Octopus: `octopus_openmp_crossplatform_mt.cpp`

`octopus_openmp_crossplatform_mt.cpp` is the current multi-threaded,
cross-platform standalone octopus implementation. It ports the Vavourakis
octopus arm examples into a C++17/OpenMP code path that can be built without
PETSc, libMesh, or MPI. It represents the practical octopus version of the
Vavourakis-inspired standalone branch.

Solver/model details:

- Solver: implicit Newton iterations with trapezoidal time integration.
- Linear solve: internal CSR sparse matrix with BiCGSTAB and Jacobi
  preconditioning.
- Material/model: Vavourakis-style nearly incompressible Mooney-Rivlin
  connective tissue plus active/passive fiber stresses, with octopus-specific
  muscle groups and activation patterns.

This is the practical current SOTA for the standalone octopus branch.

Related standalone octopus files:

- `octopus_openmp_standalone_annotated.cpp`
- `octopus_opemp_slow.cpp`

### PETSc/libMesh Reference Versions

The PETSc/libMesh versions are more reference-oriented and closer to a full
implicit finite-element research-code structure:

- `vasvourakis.cpp`: PETSc/libMesh squid reference implementation.
- `vasvourakis_octopus.cpp`: PETSc/libMesh octopus reference implementation.
- `vasvav.c`: earlier PETSc-based Vavourakis-style solver.
- `Makefile.vasvav`: build helper for `vasvav.c`.

These versions require external scientific libraries and are not as portable as
the standalone OpenMP versions, but they are important because they preserve the
implicit sparse-solver direction of the Vavourakis formulation.

Solver/model details:

- Solver: global implicit nonlinear FEM with sparse residual/Jacobian assembly,
  Newton iterations, and PETSc/libMesh linear algebra.
- Material/model: Vavourakis-style Mooney-Rivlin connective tissue and
  active/passive fiber stress superposition.

## Local Paper References

The `Vavourakis/` folder contains the main PDFs used as references:

- `A nonlinear dynamic finite element approach for simulating muscular hydrostats.pdf`:
  Vavourakis et al. nonlinear dynamic FEM paper, central for the squid/octopus
  FEM branch.
- `liang2006.pdf`: Liang, McMeeking, and Evans explicit FEM scheme for
  biological muscular hydrostats.
- `1-s2.0-S002251930092109X-main.pdf`: Johansson, Meier, and Blickhan finite
  element skeletal-muscle model, useful background for nonlinear continuum
  muscle mechanics.
- `1-s2.0-S0021929009000475-main.pdf`: Tang, Zhang, and Tsui 3D skeletal muscle
  FEM with Hill-type active contraction and hyperelastic passive behavior.
- `vanLeeuwen_Kier_1997.pdf`: squid/tentacle biomechanics reference material
  used as background for cephalopod muscular-hydrostat mechanics.

## Overall Contribution

The repository shows a research path through multiple muscular-hydrostat
models:

1. A Yekutieli-inspired 2D octopus mass-spring model as the central starting
   point.
2. Two 3D Yekutieli-inspired extensions for arm/robot geometries:
   `3d.c` and `cylinder.c`.
3. A Perkell 1974 tongue model connecting the work to classic tongue
   biomechanics.
4. A Liang 2006 squid tentacle FEM branch, with `liang_coarse_timefix_xy.c` as
   the current best implementation.
5. A Vavourakis-inspired squid/octopus FEM branch, with both standalone OpenMP
   versions and PETSc/libMesh reference versions.

The main theme across the project is translating biological muscular-hydrostat
papers into executable simulation code, then progressively improving geometry,
constraints, activation, constitutive behavior, and numerical robustness.

## Runtime Control

The main files are documented directly in their headers or top configuration
blocks. The Yekutieli, Perkell, and Liang files are mostly controlled by
top-of-file constants such as geometry, material constants, activation timing,
time-step limits, damping, and stabilization coefficients. The Vavourakis-style
files expose more command-line controls.

Examples of important runtime flags:

- `vavourakis_squid_standalone_explicit_omp.c`: `--dt`, `--t_end`,
  `--activation_delay_tip`, `--cfl`, `--hg_stiff`, `--hg_visc`,
  `--bv_linear`, `--bv_quad`, `--rayleigh_alpha`, `--root_plane_dashpot`,
  `--eta_pas_lm`, `--eta_pas_club_lm`, `--prism_sri`, output/checkpoint flags,
  root constraint flags, and `--club_connective_only`.
- `vavourakis_squid_standalone_implicit_newtonkrylov.c`: Newton tolerances,
  line-search controls, adaptive cutback/regrowth controls, GMRES controls,
  stabilization flags, output/checkpoint flags, `--fixed_root_z`, and
  `--club_connective_only`.
- `octopus_openmp_crossplatform_mt.cpp`: Mooney-Rivlin constants, bulk modulus,
  time-step controls, Newton controls, BiCGSTAB controls, geometry controls,
  `--scenario`, `--primary_group`, `--secondary_group`, VTK/CSV controls,
  checkpoint controls, OpenMP thread count, deterministic assembly/postprocess
  switches, and logging flags.
- `vasvourakis.cpp` and `vasvourakis_octopus.cpp`: GetPot-style options such as
  `dt`, `t_end`, `vtk_stride`, `csv`, `checkpoint_enabled`, material constants,
  geometry parameters, Newton/line-search controls, and PETSc options such as
  `-ksp_type`, `-pc_type`, and `-snes_type`.
- `vasvav.c`: PETSc/SNES options plus the direct `-sigma_max` control.

The flag lists in `README.md` give the practical map; the source headers remain
the most detailed place to check assumptions and defaults before running a
specific experiment.

## Visualization and Verification

The code is not only written as numerical solvers; it is also checked through
post-processing. CSV files from the simulations are used to plot graphs of
motion, length change, velocity, activation, and other diagnostic quantities.
VTK files are opened in ParaView to visualize the 3D deformation, verify that
the simulated squid/octopus behavior is physically reasonable, and create videos
of the simulations.
