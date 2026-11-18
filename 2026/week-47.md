# 2026 · Week 47: Why plant stems need geometry, drag and per-plant solving

> Draft notes taken from my [Plant Biomechanics README](https://github.com/SudharshanSuman/gpu-accelerated-plant-biomechanics). Edit anything that does not match what I actually understood.

## What I learned

- **Stiffness comes from geometry.** Flexural rigidity is `EI` with `I = πr⁴/4`. A stem that tapers from 5 mm to 2 mm is about 39× stiffer at the base with no tuning. "Stiffer at the bottom" is a consequence of the taper, not a rule I add on top.
- **Wind is a drag force, not an acceleration.** `F = ½ρC_dA|u−v|(u−v)` depends on air velocity relative to the moving plant, so it self-limits as the plant yields. A constant wind acceleration has no feedback and pushes the stem flat however I set the damping.
- **Parallelise over plants, not constraints.** A stem is a chain, so bending corrections travel one hinge per pass. Colour-parallel solving left a 17-node stem about 7× too compliant. One thread per plant sweeping its chain in order solves it almost directly.

## What surprised me

- The speed-vs-damage result is negative: peak impact quantities do not converge without strain-rate damping. I documented it instead of hiding it (`Documentation/SPEED_DAMAGE_FINDING.md`).

## Evidence

- Mesh refinement against the analytic `qL⁴/(8EI)` beam result: error falls 25% → 12% → 8.4% at 9/17/33 nodes.
- Warp kernels agree with an independent NumPy solver to about 5×10⁻⁵ m over 300 frames.

## Known gaps

- No GPU timings yet. A Secure Boot key blocks the driver, so all numbers are from the CPU backend.
- No real plant measurements yet. Validation is against beam theory only.

## Next week

- Fix the GPU driver block and get a real GPU benchmark.
