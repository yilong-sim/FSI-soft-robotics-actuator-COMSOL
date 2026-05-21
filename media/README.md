# Media

Place the following files here:

| Filename | Source | Description |
|---|---|---|
| `balloon_exp.gif` | Your experimental video | Balloon actuator expansion, 7.1 mm channel |
| `balloon_sim.gif` | COMSOL animation export | Corresponding simulation output |

## How to prepare the GIFs

**Target specs:**
- Duration: 3–6 seconds, looping
- Width: 400–500 px (GitHub renders these side-by-side in the README table)
- Frame rate: 15–20 fps
- File size: keep under 5 MB each for fast loading

**Experimental video → GIF:**
Use ffmpeg:
```bash
ffmpeg -i balloon_exp.mp4 -vf "fps=15,scale=450:-1" -loop 0 balloon_exp.gif
```

**COMSOL animation → GIF:**
Export as AVI from COMSOL (Results → Export → Animation), then convert:
```bash
ffmpeg -i balloon_sim.avi -vf "fps=15,scale=450:-1" -loop 0 balloon_sim.gif
```

## Recommended visualization for simulation GIF

Export the pressure field (p) or von Mises stress on the solid overlaid with
the deformed geometry. A color scale showing pressure buildup in the fluid
cavity alongside the bending deformation communicates the FSI coupling visually.
Alternatively, export displacement magnitude on the solid — this makes the
asymmetric inflation immediately apparent.

If side-by-side comparison is possible (experiment left, simulation right,
synchronized in time), that is the strongest single visual in the repo.
