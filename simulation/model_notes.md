# COMSOL FSI Model: Soft Balloon Actuator

## Problem Statement

The hindwing balloon actuator in DraBot works by pressurizing a rectangular
microchannel embedded between a PDMS layer and an Ecoflex 00-30 membrane. As
fluid pressure builds, the membrane inflates asymmetrically — the compliant
Ecoflex side expands freely while the stiffer PDMS side resists — generating a
net bending moment that deflects the wing structure out-of-plane.

**Design question:** What channel length produces a target deflection angle at a
given injected volume?

This cannot be answered analytically because:

1. Both materials operate far outside the linear elastic regime at the target
   deflection angles (> 60°)
2. Fluid pressure and structural deformation are tightly coupled — the expanding
   solid reshapes the fluid domain continuously in time
3. The bending kinematics are geometrically nonlinear; small-angle approximations
   fail above ~15°

A fully coupled FSI model in COMSOL 5.3a was developed to run a parametric
study over channel length, and validated against experiment before use in design.

---

## Geometry and Domain

A 3D cuboid domain represents one balloon actuator unit.

```
                    ┌─────────────────────────┐  ← PDMS layer (stiff)
                    │      Fluid Domain        │  ← microchannel (pressurized)
                    └─────────────────────────┘  ← Ecoflex membrane (compliant)

              Fixed end ──►|                   |◄── Free end (deflects)
```

| Dimension | Value |
|---|---|
| Lateral extent | 12 mm |
| Side extent | 3 mm |
| Channel height | 0.15 mm (fixed) |
| Channel width | 1.8 mm (fixed) |
| Channel length L | 4.1 / 5.6 / 7.1 mm (parametric) |

**Domain truncation rationale:** The physical inlet tube was excluded from the
computational domain. The tube serves only as a fluid passage to the balloon
cavity; pressure drop across a short rigid tube is negligible compared to the
compliance of the balloon cavity at these flow rates. The volume flow rate
boundary condition was applied directly to the right face of the fluid domain.
Validated post-hoc: simulated inlet pressure matched experimental hydraulic
pressure within ~5% across all channel lengths.

The lateral dimension (12 mm) was chosen to capture the full bending arc at
maximum deflection without introducing boundary artifacts. The 3 mm side
dimension was verified sufficient to avoid lateral constraint effects.

---

## Governing Equations

### Fluid — Laminar Flow module

Incompressible Navier-Stokes, time-dependent:

$$\nabla \cdot \mathbf{u} = 0$$

$$\rho_f \left(\frac{\partial \mathbf{u}}{\partial t} + \mathbf{u} \cdot \nabla \mathbf{u}\right) = -\nabla p + \eta \nabla^2 \mathbf{u}$$

where $\rho_f$, $p$, $\eta$, and $\mathbf{u}$ are fluid density, pressure,
dynamic viscosity, and velocity vector.

**Gravity neglected:** Hydrostatic pressure across the 0.15 mm channel height
is ~1.5 Pa. Operating pressures are O(kPa). Body force contribution is
four orders of magnitude smaller — safely neglected.

**Turbulence model:** None. Reynolds number at these flow rates and channel
dimensions is Re << 1 (viscous-dominated microfluidic regime). Laminar flow
assumption is firmly satisfied.

**Moving Mesh (ALE)** applied to the fluid domain. The mesh deforms with the
solid boundary at each timestep, maintaining a valid fluid domain throughout
the large-displacement bending motion.

### Solid Mechanics module

Large-deformation framework using the second Piola-Kirchhoff stress tensor:

$$\rho_m \frac{\partial^2 \mathbf{x}}{\partial t^2} = \nabla \cdot (\mathbf{F} \cdot \mathbf{S})^T$$

$$\mathbf{F} = \mathbf{I} + \nabla \mathbf{x}, \quad \mathbf{S} = 2\frac{\partial W_s}{\partial \mathbf{C}}$$

where $\mathbf{x}$ is displacement, $\mathbf{F}$ is the deformation gradient,
$\mathbf{S}$ is the second Piola-Kirchhoff stress, and $\mathbf{C} = \mathbf{F}^T\mathbf{F}$
is the right Cauchy-Green deformation tensor.

The choice of second Piola-Kirchhoff (rather than Cauchy or first P-K) is
appropriate here: it is energy-conjugate to the Green-Lagrange strain tensor and
remains objective under rigid body rotation — important at the large rotations
seen in wing bending.

---

## Material Models

### PDMS (Sylgard 184) — 5-Parameter Mooney-Rivlin

$$W_s = \sum_{i,j=0}^{n} C_{ij}(I_1 - 3)^i(I_2 - 3)^j + \frac{1}{2}K(J_{el}-1)^2$$

Parameters from Kim et al. (2011), *Microelectronic Engineering* — a
well-characterized benchmark dataset for Sylgard 184. At the strain levels PDMS
experiences in this actuator (relatively small, as it is the stiff constraining
layer), the published parameters are reliable without re-fitting.

### Ecoflex 00-30 — 2-Term Ogden

$$W_s = \sum_{p=1}^{n} \frac{\mu_p}{\alpha_p}\left(\lambda_1^{\alpha_p} + \lambda_2^{\alpha_p} + \lambda_3^{\alpha_p}\right) + \frac{1}{2}K(J_{el}-1)^2$$

**Why Ogden for Ecoflex, not Mooney-Rivlin:**
Ecoflex (Shore 00-30) is one of the softest commercially available silicone
elastomers. At the stretch ratios reached during balloon inflation, Mooney-Rivlin
tends to over-predict stiffness because its polynomial form in the strain
invariants cannot adequately capture the pronounced upturn in stress at large
stretches. The Ogden model's power-law form in principal stretches is better
suited to this regime. The 2-term form was sufficient given the fit quality
(R² = 0.995); adding a third term did not meaningfully improve fit and would
introduce parameter non-uniqueness.

**Ogden constants — fit from our own tensile measurements:**

| Parameter | Value | Units |
|---|---|---|
| $\alpha_1$ | 3.034 ± 0.1123 | — |
| $\alpha_2$ | 13.02 ± 10.56 | — |
| $\mu_1$ | 1.241 ± 0.1524 | kPa |
| $\mu_2$ | 7.879×10⁻⁹ ± 1.512×10⁻⁸ | kPa |

See [`material_params/ecoflex_ogden.md`](material_params/ecoflex_ogden.md) for
experimental protocol and fitting details.

---

## Boundary Conditions

### Fluid

| Boundary | Condition |
|---|---|
| Inlet (right face) | Volume flow rate: 2.5 µL/s |
| All other walls | No-slip: $\mathbf{u} \cdot \mathbf{n} = 0$ |
| Initial conditions | $\mathbf{u} = 0$, $p = 0$ |

### Solid (both PDMS and Ecoflex layers)

| Boundary | Condition |
|---|---|
| Right end faces | Fixed constraint: $\mathbf{x} = 0$ |
| All outer surfaces | Traction-free (zero stress) |
| Initial conditions | $\mathbf{x} = 0$, $\partial\mathbf{x}/\partial t = 0$ |

The fixed constraint at the right end replicates the physical clamping of the
actuator body to the DraBot wing root. The free outer surfaces correctly
represent the exposed silicone faces in air.

### FSI Coupling

Fluid-structure interaction assigned at all interfaces between the fluid domain
and both solid layers. At each timestep:

1. Fluid pressure acts as a traction load on the solid faces
2. Solid displacement updates the ALE mesh in the fluid domain
3. Segregated iterations continue until residuals converge

---

## Solver Configuration

| Setting | Value |
|---|---|
| Study type | Time-dependent |
| Fluid solver | Laminar Flow module, ALE moving mesh |
| Solid solver | Nonlinear Solid Mechanics module |
| Coupling | Segregated FSI iterations per timestep |
| PDMS constitutive law | 5-parameter Mooney-Rivlin (Eq. 6) |
| Ecoflex constitutive law | 2-term Ogden (Eq. 7) |

Convergence was monitored on both displacement and pressure residuals. The
nonlinear solid solver used a Newton-Raphson scheme with load-stepping to
handle the large-deformation regime stably.

---

## Validation Results

Displacement angle θ vs. injected volume compared against experiment for all
three channel lengths, 3 cycles each.

![validation](../figures/sim_vs_exp_validation.png)

**Key findings:**

- Simulation tracks experimental trend across all three geometries without
  geometry-specific parameter adjustment — the constitutive model generalizes
- Longer channels produce larger deflection at equivalent injected volume,
  consistent with the greater moment arm available for bending
- The nonlinear stiffening behavior above ~60° is reproduced — this is the
  regime where a linear elastic model would predict continued linear growth
  and significantly over-estimate deflection
- Maximum deviation between simulation and experiment: < 8% across all
  conditions

**Design decision:** The 7.1 mm channel was selected for all subsequent DraBot
experiments based on this parametric study, as it achieves the target deflection
range (0–80°) within the available hydraulic pressure budget.

---

## Design Judgment Notes

These are the physical reasoning decisions that go beyond running the solver —
the kind of judgment that determines whether a simulation is trustworthy before
you act on its output.

**On domain truncation:**
Including the inlet tube in the computational domain would add substantial mesh
complexity (long thin cylinder adjoining a wide flat cavity — poor aspect ratio)
without changing the answer. The pressure drop in a short, rigid, millimeter-scale
tube at µL/s flow rates is negligible compared to the compliance-dominated
pressure in the balloon. Excluded by design, not by oversight.

**On gravity:**
$\Delta p_{hydrostatic} = \rho g h \approx 1000 \times 9.8 \times 0.00015 \approx 1.5$ Pa,
vs. operating pressure O(kPa). Four orders of magnitude smaller. Not a modeling
choice — a physics argument.

**On constitutive model selection:**
Using Mooney-Rivlin for both layers would have been the path of least resistance.
It was rejected for Ecoflex because the material enters a stretch regime where
the Ogden power-law form is physically more appropriate. This required fitting
our own constants from tensile data rather than pulling published Mooney-Rivlin
values — more work, but the validation result (R² = 0.995 fit, < 8% simulation
error) confirms the choice was correct.

**On the small μ₂ value:**
The second Ogden term (μ₂ ~ 10⁻⁹ kPa) appears negligibly small, but contributes
at large stretches (λ > 3) through the high exponent α₂ = 13. At the strain
levels in this actuator (λ_max ~ 1.5–2), the first term dominates and μ₂ has
minimal influence. The term was retained because it stabilizes the fit without
adding computational cost.

**On the 2D vs 3D domain choice:**
A 2D plane-strain model would reduce computational cost but cannot capture the
out-of-plane bending kinematics correctly — the wing deflects in 3D and the
moment arm changes geometry as it bends. 3D was required.
