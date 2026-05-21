# Figures

Place the following files here:

| Filename | Source | Description |
|---|---|---|
| `sim_vs_exp_validation.png` | Fig 1g / Fig S3b | Displacement angle vs. injected volume, simulation vs. experiment, all 3 channel lengths |
| `mesh_domain.png` | Fig S3a | Isometric view of 3D cuboid FSI domain |
| `ogden_fit.png` | Fig S2b | Ecoflex 00-30 stress-strain curve with Ogden model fit (R²=0.995) |

## Tips for figure preparation

**sim_vs_exp_validation.png:** Export from COMSOL or compose from paper figure.
Show all 3 channel lengths (4.1, 5.6, 7.1 mm) with experiment (solid) and
simulation (dashed) clearly distinguished. Include axis labels and units.
Target width: 800–1000 px for clean GitHub rendering.

**mesh_domain.png:** A clean isometric screenshot of the COMSOL mesh, showing
the two-layer solid structure and fluid domain. Label PDMS, Ecoflex, and fluid
regions.

**ogden_fit.png:** Stress (kPa) vs. strain (%) with the fitted curve overlaid.
Include R² = 0.995 annotation. Can be re-plotted in Python/matplotlib from
the raw tensile data for a cleaner look than the paper figure.
