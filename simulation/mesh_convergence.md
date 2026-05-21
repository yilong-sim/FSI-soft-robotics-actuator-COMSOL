# Mesh Convergence Study

## Purpose

FSI simulations with large deformations and moving meshes are sensitive to
spatial discretization — particularly in regions of high stress gradient and
near the FSI interface where the mesh must deform significantly. Mesh
independence was verified before using simulation results for design decisions.

---

## Study Setup

Convergence checked on the primary output quantity of interest: displacement
angle θ at maximum injection volume (40 µL), for the 7.1 mm channel (the
geometry selected for device fabrication — most critical to get right).

Three mesh densities were evaluated:

| Mesh | Global element size | FSI interface refinement | DOF (approx.) |
|---|---|---|---|
| Coarse | 0.3 mm | None | ~15,000 |
| Medium | 0.15 mm | 2× local refinement | ~45,000 |
| Fine | 0.08 mm | 3× local refinement | ~130,000 |

All three meshes used swept elements through the channel height (0.15 mm) to
maintain good aspect ratios in the thin fluid layer — this is the critical
dimension and free meshing here produces poorly conditioned elements.

---

## Results

| Mesh | θ at 40 µL (°) | Change from previous | Runtime |
|---|---|---|---|
| Coarse | 74.2 | — | ~8 min |
| Medium | 78.6 | 5.9% | ~25 min |
| Fine | 79.1 | 0.6% | ~90 min |

**Decision:** Medium mesh used for all production runs. The 0.6% change from
medium to fine is well within the experimental scatter (±3–5° across 3 cycles),
and the 3.6× runtime increase is not justified.

---

## Refinement Strategy

Local mesh refinement was applied in two regions:

**Channel corner regions:** The junction between the rigid PDMS channel wall
and the compliant Ecoflex membrane is a stress concentration site. Coarse
elements here under-resolve the stress gradient and cause the ALE mesh to
distort excessively at large deflections. 3× local refinement applied at all
four channel corners.

**FSI interface:** The accuracy of the fluid-to-solid traction transfer depends
on how well the interface is resolved. 2× refinement along the FSI boundary
ensures consistent traction integration across the deforming surface.

---

## ALE Mesh Quality

The ALE moving mesh was monitored throughout each simulation for element
quality degradation. At maximum deflection (θ ≈ 80°, 40 µL), the minimum
mesh quality metric (scaled Jacobian) remained above 0.35 on the medium mesh —
within acceptable range for COMSOL's laminar flow solver. Remeshing was not
required for any of the three channel lengths studied.

If this simulation were extended to higher flow rates or longer channels (larger
deflections), ALE mesh quality would need to be re-evaluated and adaptive
remeshing considered.

---

## Timestep Sensitivity

A brief timestep study was also conducted at medium mesh refinement:

| Timestep | θ at 40 µL (°) | Change |
|---|---|---|
| 0.5 s | 78.1 | — |
| 0.2 s | 78.6 | 0.6% |
| 0.1 s | 78.7 | 0.1% |

**Decision:** 0.2 s timestep used for all runs. The nonlinear Newton-Raphson
solver converged reliably at this step size across all channel lengths without
requiring step reduction.
