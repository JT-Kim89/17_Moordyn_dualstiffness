# Polyester Rope Dual-Stiffness Basis

This note records the literature basis used in the MoorDyn example files.
It is deliberately more verbose than the MoorDyn input snippets so that the
actual input files can stay readable while the assumptions remain traceable.

## Selected Modeling Reference

Tahar and Kim (2008), "Coupled-dynamic analysis of floating structures with
polyester mooring lines", Ocean Engineering, 35(17-18), 1676-1685.

- DOI: 10.1016/j.oceaneng.2008.09.004
- Semantic Scholar citation count checked on 2026-05-20: 106
- Relevance: highly cited polyester mooring-line modeling paper. It extends
  mooring-line dynamics to large elongation and nonlinear polyester
  stress-strain behavior, adopting the Bosman and Hooker polyester empirical
  stiffness approach.

How it is used here:

- The paper is used as the high-citation modeling reference showing that
  polyester mooring lines need nonlinear/large-elongation treatment rather than
  a simple steel-chain-style constant stiffness.
- The exact numerical MoorDyn values are not copied directly from this paper,
  because the most convenient openly tabulated values for the dual-stiffness
  pair come from ABS guidance and from the Sorum et al. rope scale described
  below.

## Rope Scale Used For MoorDyn Numbers

Sorum et al. (2023), "Modelling of Synthetic Fibre Rope Mooring for Floating
Offshore Wind Turbines", JMSE 11(1), 193.

- DOI: 10.3390/jmse11010193
- Polyester fibre-rope case:
  - Diameter = 147 mm
  - MBL = 885 tonnes
- Converted MBS:
  - MBS = 885 * 1000 * 9.80665 = 8.678885e6 N

## Stiffness Ratios

ABS fiber-rope mooring guidance Appendix 2 gives a static-dynamic strength
analysis example with:

- Krs = 12
- Krd = 29
- Kr = EA / MBS

Converted MoorDyn stiffness values:

- EA_s = 12 * 8.678885e6 = 1.041466e8 N
- EA_d = 29 * 8.678885e6 = 2.516877e8 N

Why MBS scaling is useful:

- Polyester stiffness is often reported as a ratio of breaking strength rather
  than as one universal EA value.
- This makes the values portable between rope sizes.
- For a different rope, keep Krs and Krd if they remain appropriate, replace
  MBS, and recompute EA_s and EA_d.

For the nonlinear table example, the transition was placed at the ABS
100-year intact mean tension example:

- Mean tension = 32% MBS
- eps_k = 0.32 / 12 = 0.026667
- T_k = 0.32 * 8.678885e6 = 2.777243e6 N

The bilinear table is then:

- T = EA_s * eps, for eps <= eps_k
- T = T_k + EA_d * (eps - eps_k), for eps > eps_k
- The example table is stopped near 99.7% MBS to avoid extrapolating beyond
  the nominal breaking strength.

Why the table uses tension rather than EA:

- MoorDyn's nonlinear stiffness-table input is a strain-tension curve.
- The effective stiffness is the slope dT/deps between table points.
- Therefore, a bilinear stiffness law must be integrated into a continuous
  tension-strain curve before being written to the table.

## Caution

Only stiffness values were literature-calibrated. MoorDyn line fields such as
mass per length, drag, added mass, internal damping, and seabed interaction
should be replaced with project-specific or vendor-provided values before
using the model for design decisions.

Suggested workflow:

1. Use the current files only to validate the stiffness implementation.
2. Replace non-stiffness line properties with vendor/project values.
3. Run a static case and check that mean tension and stretch are plausible.
4. Run a dynamic case and compare mean, standard deviation, maximum tension,
   and tension-strain scatter against the targets in
   `polyester_validation_targets.md`.
