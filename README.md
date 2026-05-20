# Muscular Hydrostat Simulation Codes

This repository contains the simulation code developed during the internship for
paper-inspired muscular-hydrostat models. The main work is grouped by the model
families implemented: Yekutieli-style octopus arms, 3D Yekutieli-inspired robot
arms, Perkell's tongue model, Liang's squid tentacle model, and Vavourakis-style
squid/octopus finite-element models.

For the longer narrative summary, see [Summary.md](Summary.md).

## Main Implemented Work

### 1. Yekutieli-Based Octopus Model

The central starting point of the project is the Yekutieli-based octopus arm
model:

| Main file | Implementation focus |
| --- | --- |
| `main.c` | Main 2D mass-spring octopus arm implementation. It models a tapered octopus arm with active muscle waves, nonlinear muscle behavior, anisotropic drag, and hydrostatic-style area conservation. |

This file is the main reduced octopus-arm model. It translates Yekutieli-style
arm extension mechanics into an executable 2D mass-spring simulation.

### 2. 3D Yekutieli-Inspired Robot / Arm Models

After the 2D model, the work extends the Yekutieli-inspired idea into two 3D
robot/arm implementations:

| Main file | Implementation focus |
| --- | --- |
| `3d.c` | 3D mass-spring arm using four nodes per ring, square-like cross-sections, diagonal springs, and volume constraints through tetrahedral decomposition. |
| `cylinder.c` | 3D cylindrical variant with outer nodes plus center nodes per ring, designed to test a more robot-like arm geometry and volume-preserving actuation. |

These two files are the main 3D Yekutieli-inspired implementations in the
repository.

### 3. Perkell 1974 Tongue Model

The Perkell branch applies the same broader active/passive soft-body modeling
interest to tongue biomechanics:

| Main file | Implementation focus |
| --- | --- |
| `perkell1974.c` | 2D tongue mechanics model inspired by Perkell 1974, with active muscle-like forces, passive connective behavior, damping, and stabilization. |

This code connects the project to a classic tongue-modeling paper rather than
only cephalopod muscular hydrostats.

### 4. Liang 2006 Squid Tentacle Model

The Liang branch implements and iteratively improves a squid tentacle FEM model
inspired by Liang, McMeeking, and Evans 2006. The paper develops an explicit FEM
scheme for biological muscular hydrostats by embedding muscle fibers inside
finite elements, with active stress depending on activation, strain, and
shortening velocity, and passive stress depending on strain.

| Main file | Implementation focus |
| --- | --- |
| `liang_coarse_timefix_xy.c` | Current SOTA Liang implementation in this repository. It uses the Liang-style coarse Fig. 2(b) mesh, corrected explicit time-step ordering, fixed x/y transverse muscle directions, post-step constitutive updates, and VTK/debug fields. |

Solver/model details:

- Solver: explicit finite-element time stepping in the Liang style, using
  central-difference/leapfrog-like update ordering.
- Material/model: embedded muscle-fiber formulation with active and passive
  fiber stress; active stress depends on activation, strain, and shortening
  velocity, while passive stress depends on strain.
- Numerical stabilization: reduced-integration HEX-style elements with
  geometry-aware viscous Flanagan-Belytschko hourglass control, raw volume/J
  pressure diagnostics, and Rayleigh mass damping.
- This is not a Mooney-Rivlin Vavourakis model; it is the Liang-style explicit
  muscular-hydrostat formulation.

This is the current best Liang code path and the one to emphasize when
discussing the final Liang implementation.

### 5. Vavourakis-Inspired Squid and Octopus Models

The Vavourakis branch implements nonlinear dynamic FEM-style muscular-hydrostat
models for both squid and octopus. The reference paper uses nearly
incompressible hyperelastic muscle, Mooney-Rivlin connective tissue, and
active/passive fiber stresses, with an implicit nonlinear FEM formulation.

#### Standalone / OpenMP Implementations

| Main file | Implementation focus |
| --- | --- |
| `vavourakis_squid_standalone_explicit_omp.c` | Current SOTA standalone squid implementation. It is a dependency-light C/OpenMP solver inspired by the Vavourakis squid benchmark. |
| `octopus_openmp_crossplatform_mt.cpp` | Current SOTA standalone octopus implementation. It is a cross-platform C++/OpenMP multi-threaded version of the Vavourakis-inspired octopus arm model. |

Solver/model details:

- `vavourakis_squid_standalone_explicit_omp.c`: standalone explicit
  lumped-mass central-difference solver. It keeps the Vavourakis-style
  constitutive model but replaces the paper's full implicit PETSc/libMesh solve
  with a portable OpenMP explicit update.
- `octopus_openmp_crossplatform_mt.cpp`: standalone implicit solver using
  Newton iterations with trapezoidal time integration. The internal linear
  algebra uses a CSR matrix with BiCGSTAB and Jacobi preconditioning.
- Material/model: nearly incompressible modified-invariant Mooney-Rivlin
  connective tissue plus additive active/passive muscle-fiber stress.
- Extra numerical ingredients include bulk/near-incompressibility control,
  damping, reduced/selective integration ideas, and VTK diagnostic fields for
  checking deformation and stress behavior.

These two files are the practical standalone implementations for the
Vavourakis-inspired squid and octopus systems.

#### PETSc / libMesh Implementations

| Main file | Implementation focus |
| --- | --- |
| `vasvourakis.cpp` | PETSc/libMesh squid reference implementation, closer to the implicit sparse-solver direction of the Vavourakis paper. |
| `vasvourakis_octopus.cpp` | PETSc/libMesh octopus reference implementation with internal mesh construction, muscle-region tags, and global nonlinear solve structure. |

Solver/model details:

- Solver: implicit nonlinear finite-element solve with global sparse residual
  and Jacobian assembly, Newton iterations, and PETSc/libMesh linear algebra.
- Material/model: Vavourakis-style nearly incompressible Mooney-Rivlin
  connective tissue with active/passive fiber stress superposition.

These are the heavier research-reference versions. They require external
scientific libraries, unlike the standalone OpenMP implementations.

## Runtime Flags and File Headers

Each main source file has a header or top configuration block describing the
implementation, model assumptions, constants, and run-time behavior. For the
older C models, most settings are edited through `#define` constants near the
top of the file. For the newer Vavourakis-style solvers, many settings can be
changed from the command line.

### Yekutieli, Perkell, and Liang Configuration

These files are mainly controlled by compile-time constants in the source:

| File | Important top-of-file controls |
| --- | --- |
| `main.c` | `USE_NONLINEAR`, segment count, arm geometry, drag coefficients, maximum stress, activation timing, RK23 time-step limits, and area-constraint parameters. |
| `3d.c` | `USE_NONLINEAR`, 3D segment/ring layout, tetrahedral volume constraints, gravity/buoyancy/drag, anchor springs, diagonal springs, and RK23 time-step limits. |
| `cylinder.c` | `USE_NONLINEAR`, cylindrical ring layout, center-node springs, volume constraints, gravity/buoyancy/drag, and RK23 time-step limits. |
| `perkell1974.c` | tongue mesh size, active/passive stiffnesses, damping, stabilization constants, and RK23 time-step limits. |
| `liang_coarse_timefix_xy.c` | material constants such as `MU_SHEAR` and `K_BULK`, activation constants, mesh resolution, output interval, hourglass coefficients, bulk-viscosity constants, and Rayleigh damping. |

### Vavourakis Standalone Squid Flags

`vavourakis_squid_standalone_explicit_omp.c` and
`vavourakis_squid_explicit_fixed_root.c` expose command-line flags including:

| Flag | Purpose |
| --- | --- |
| `--dt` | Override explicit time step. |
| `--t_end` | Set final simulation time. |
| `--activation_delay_tip` | Add tipward activation onset delay. |
| `--cfl` | Set CFL factor for stable time-step estimate. |
| `--hg_stiff`, `--hg_visc` | Tune stiffness/viscous hourglass control. |
| `--bv_linear`, `--bv_quad` | Tune bulk-viscosity terms. |
| `--rayleigh_alpha` | Set mass-proportional Rayleigh damping. |
| `--root_plane_dashpot` | Add root-plane dashpot in the axial direction. |
| `--eta_pas_lm`, `--eta_pas_club_lm` | Set passive-fiber viscosity for stalk/club longitudinal fibers. |
| `--prism_sri` | Toggle prism selective/reduced integration behavior. |
| `--vtk_stride`, `--csv_stride`, `--console_stride` | Control VTK, CSV, and console output frequency. |
| `--recompute_dt_stride` | Recompute stable time step every `n` steps. |
| `--checkpoint_enabled`, `--checkpoint_restart`, `--checkpoint_interval`, `--checkpoint_file` | Enable and configure binary restart checkpoints. |
| `--out_dir`, `--vtk_prefix`, `--csv` | Configure output directory and file names. |
| `--fixed_root_z`, `--root_rigid_plane_z` | Change root axial boundary constraints. |
| `--club_connective_only` | Disable passive fibers in the club so the club behaves as connective tissue only. |
| `--help` | Print the available options from the executable. |

### Vavourakis Standalone Implicit Squid Flags

`vavourakis_squid_standalone_implicit_newtonkrylov.c` adds implicit-solver
controls, including:

| Flag | Purpose |
| --- | --- |
| `--dt`, `--t_end`, `--dt_min` | Time-step and end-time controls. |
| `--newton_abs_tol`, `--newton_rel_tol`, `--newton_corr_tol`, `--newton_relax`, `--newton_maxit`, `--newton_monitor` | Newton convergence and monitoring controls. |
| `--line_search_max_backtracks`, `--line_search_reduction`, `--line_search_armijo`, `--line_search_min_alpha` | Backtracking line-search controls. |
| `--dt_cutback_factor`, `--dt_growth_factor`, `--newton_easy_iter`, `--max_step_retries` | Adaptive time-step cutback/regrowth controls. |
| `--gmres_restart`, `--gmres_maxit`, `--gmres_tol`, `--jv_fd_eps` | Newton-Krylov/GMRES and Jacobian-vector finite-difference controls. |
| `--hg_visc`, `--bv_linear`, `--bv_quad`, `--bv_expand`, `--prism_sri` | Stabilization and integration controls. |
| `--vtk_stride`, `--csv_stride`, checkpoint flags, `--out_dir`, `--vtk_prefix`, `--csv` | Output and restart controls. |
| `--fixed_root_z`, `--club_connective_only` | Boundary/model switches. |

### Vavourakis Standalone Octopus Flags

`octopus_openmp_crossplatform_mt.cpp` accepts `--key value` or `--key=value`
style flags. Important options include:

| Flag | Purpose |
| --- | --- |
| `--c1_ct`, `--c2_ct`, `--K_bulk` | Mooney-Rivlin and bulk material constants. |
| `--dt`, `--t_end`, `--dt_min`, `--dt_reduction`, `--dt_growth` | Time-step controls. |
| `--newton_tol`, `--newton_resid_abs_tol`, `--newton_resid_rel_tol`, `--newton_relax`, `--newton_maxit` | Newton controls. |
| `--line_search_max`, `--line_search_shrink`, `--line_search_accept` | Line-search controls. |
| `--linear_tol`, `--linear_maxits` | BiCGSTAB linear-solver controls. |
| `--use_sri`, `--dump_failure_vtk` | Integration/debug controls. |
| `--arm_len`, `--root_radius`, `--tip_radius`, `--tm_core_fraction` | Octopus geometry controls. |
| `--scenario`, `--primary_group`, `--secondary_group` | Activation scenario and muscle-group controls. |
| `--vtk_stride`, `--vtk_dt`, `--output_dir`, `--vtk_prefix`, `--csv`, `--centerline_history_csv`, `--layer_profile_history_csv` | Output controls. |
| `--checkpoint_enabled`, `--checkpoint_restart`, `--checkpoint_interval`, `--checkpoint_file` | Restart/checkpoint controls. |
| `--omp_threads`, `--deterministic_assembly`, `--deterministic_post`, `--progress_stride`, `--log_newton` | Parallelism, reproducibility, and logging controls. |

### PETSc / libMesh Flags

`vasvourakis.cpp` and `vasvourakis_octopus.cpp` use GetPot-style options plus
PETSc options. In practice, options are passed without the double dash, for
example `dt=...`, `vtk_stride=...`, or `club_connective_only=1`, alongside PETSc
flags such as `-ksp_type`, `-pc_type`, or `-snes_type`.

Important `vasvourakis.cpp` squid options include `dt`, `t_end`,
`newton_*`, `line_search_*`, `dt_cutback_factor`, `dt_growth_factor`,
`vtk_stride`, `write_initial_vtk`, checkpoint options, `lm_fraction`,
`lm_interface_radius`, `r_outer`, `stalk_len`, `club_len`, `fixed_root_z`,
`root_rigid_plane_z`, `club_connective_only`, `hex8_integration_mode`,
`hg_alpha_k`, `hg_alpha_v`, `fv_ecc_coeff`, `vtk_prefix`, and `csv`.

Important `vasvourakis_octopus.cpp` options include `c1_ct`, `c2_ct`, `K_bulk`,
`dt`, `t_end`, `newton_*`, `line_search_*`, `dt_min`, `dt_reduction`,
`dt_growth`, `use_sri`, `dump_failure_vtk`, checkpoint options, `linear_tol`,
`linear_maxits`, `arm_len`, `root_radius`, `tip_radius`, `tm_core_fraction`,
`scenario`, `primary_group`, `secondary_group`, `vtk_prefix`, `csv`,
`progress_stride`, and `log_newton`.

`vasvav.c` is PETSc/SNES-based and currently exposes `-sigma_max` directly,
while PETSc runtime options such as `-snes_type`, `-ksp_type`, and `-pc_type`
can also be used.

## Related Development Files

The files above are the main implementations to highlight. The repository also
contains supporting and intermediate files from the development process.

## Visualization and Verification Workflow

The simulations are checked visually and quantitatively after running:

- CSV trajectory/history files are used to plot graphs of motion, length,
  velocity, activation, and other simulation quantities.
- VTK files are opened in ParaView to inspect the simulated geometry, verify
  deformation behavior, and create videos of the squid/octopus motion.
- These plots and ParaView animations are used as practical validation tools
  while comparing the implementations against the behavior described in the
  papers.

### Yekutieli / Octopus Helpers

| File | Role |
| --- | --- |
| `octopus_colab.py` | Python/Colab version of the 2D octopus arm model. |
| `visualize_octopus.py` | Visualization helper for octopus trajectories. |

### Perkell Helpers

| File | Role |
| --- | --- |
| `visualize_perkell.py` | Visualization helper for the Perkell tongue model. |

### Liang Development Versions

| File | Role |
| --- | --- |
| `liang2006.c` | Early strict Liang reproduction attempt. |
| `liangogrid.c` | O-grid/hourglass-control development version. |
| `liangogridfiner.c` | Refined O-grid development version. |
| `liang2006_v4.c` to `liang2006_v7.c` | Iterative physics and solver correction versions. |
| `liang_coarse.c` | Coarse Liang mesh before the time-step ordering fix. |
| `liang_coarse_timefix.c` | Corrected time-step ordering before the final x/y transverse-muscle version. |
| `liang_xy_crossection_scale2.c` | Cross-section refinement experiment. |

### Vavourakis Development Versions

| File | Role |
| --- | --- |
| `vavourakis_squid_explicit_fixed_root.c` | Squid explicit solver with fixed-root variant. |
| `vavourakis_squid_standalone_implicit_newtonkrylov.c` | Standalone implicit Newton-Krylov squid development version. |
| `octopus_openmp_standalone_annotated.cpp` | Annotated standalone octopus implementation. |
| `octopus_opemp_slow.cpp` | Earlier slower OpenMP octopus implementation. |
| `vasvav.c` | Earlier PETSc-based Vavourakis-style implementation. |
| `Makefile.vasvav` | Build helper for `vasvav.c` in a PETSc/WSL environment. |

## Build Notes

The repository contains a `CMakeLists.txt` with targets for many of the C and
C++ implementations.

Example CMake workflow:

```bash
cmake -S . -B cmake-build-debug
cmake --build cmake-build-debug
```

Selected target names from `CMakeLists.txt`:

| Target | Source |
| --- | --- |
| `sim_main` | `main.c` |
| `sim_3d` | `3d.c` |
| `sim_cylinder` | `cylinder.c` |
| `sim_perkell1974` | `perkell1974.c` |
| `coarsefxy` | `liang_coarse_timefix_xy.c` |
| `vavourakis` | `vavourakis_squid_standalone_explicit_omp.c` |
| `octm` | `octopus_openmp_crossplatform_mt.cpp` |

PETSc/libMesh versions require external scientific-computing libraries that are
not bundled with this repository.

## Reference Papers

- Yekutieli et al.: octopus arm extension and muscular-hydrostat mechanics.
- Perkell 1974: biomechanical tongue modeling.
- Liang, McMeeking, and Evans 2006: explicit FEM scheme for biological
  muscular hydrostats, tested on squid tentacle extension.
- Vavourakis, Kazakidi, Tsakiris, and Ekaterinaris 2014: nonlinear dynamic FEM
  for muscular hydrostats, including squid validation and octopus maneuvers.
- Additional PDFs in `Vavourakis/`: Johansson et al. 2000, Tang et al. 2009,
  and van Leeuwen/Kier material related to muscle and squid mechanics.
