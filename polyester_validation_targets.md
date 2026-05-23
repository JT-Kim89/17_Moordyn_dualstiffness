# Polyester MoorDyn Validation Targets

These are practical comparison targets for the two MoorDyn polyester
dual-stiffness examples in this folder.
The purpose is to avoid validating only by eye. A polyester stiffness model
should reproduce at least the material curve and, when the same system is
available, comparable tension statistics.

## 1. Material-Curve Check

Use this first. It is independent of platform geometry.
This check confirms that the input values produce the intended axial stiffness
before any hydrodynamics, platform motion, seabed contact, or solver settings
can confuse the interpretation.

For the 147 mm polyester rope example:

- MBS = 8.678885e6 N
- Krs = 12
- Krd = 29
- EA_s = 1.041466e8 N
- EA_d = 2.516877e8 N
- Transition mean load = 32% MBS = 2.777243e6 N
- Transition strain = 0.026667

Expected static-dynamic checks:

| Strain | Tension [N] | Tension/MBS |
|---:|---:|---:|
| 0.005000 | 5.207331e5 | 0.060 |
| 0.010000 | 1.041466e6 | 0.120 |
| 0.020000 | 2.082932e6 | 0.240 |
| 0.026667 | 2.777243e6 | 0.320 |
| 0.040000 | 6.133079e6 | 0.707 |
| 0.050000 | 8.649956e6 | 0.997 |

This should match `polyester_bilinear_tension_strain.txt`.
For method 1, the same numbers are not a direct table lookup, but the EA values
should still correspond to the same Krs and Krd ratios.

## 2. ABS Static-Dynamic Strength Example

ABS Appendix 2 gives direct response statistics for a static-dynamic polyester
mooring example using Krs = 12 and Krd = 29. This is the cleanest published
numeric target for the same stiffness-ratio pair used here.
Use this as a system-level benchmark only when the rest of the model is also
made consistent with the ABS example.

Frequency-domain example:

| Quantity | Static Krs = 12 | Dynamic Krd = 29 |
|---|---:|---:|
| Mean offset [m] | 40.92 | - |
| Sig. LF offset [m] | - | 2.27 |
| Sig. WF offset [m] | - | 3.85 |
| Mean tension [kN] | 7771 | - |
| Sig. LF tension [kN] | - | 616 |
| Sig. WF tension [kN] | - | 1010 |
| Combined 3 h max offset [m] | 50.35 | - |
| Combined 3 h max tension [kN] | 10266 | - |

Time-domain example:

| Quantity | Static Krs = 12 | Dynamic Krd = 29 |
|---|---:|---:|
| Mean offset [m] | 40.92 | 18.0 |
| Max offset [m] | - | 26.38 |
| Mean tension [kN] | 7771 | 7930 |
| Max tension [kN] | - | 9940 |
| Combined max offset [m] | 49.30 | - |
| Combined max tension [kN] | 9781 | - |

Important: these are not for the 147 mm Sorum rope. They are system-level
ABS example results. Use them as an apples-to-apples benchmark only if you
build the same mooring system. Otherwise use the normalized stiffness ratios
and trend checks.

Source: ABS, "Guidance Notes on the Application of Fiber Rope for Offshore
Mooring", Appendix 2, Section 4.4.

## 3. Sorum et al. 2023 Full-System Trend Checks

Sorum et al. use a 147 mm polyester fibre rope with MBL = 885 tonnes, matching
the rope scale used in the example files.
This makes the paper useful for scale and trend checks even when the full
platform, environmental loading, and solver are not exactly replicated.

Useful checks against their results:

- For loads above mean tension, bi-linear and Syrope models give similar
  tension estimates.
- Below mean tension, the bi-linear model reduces tension amplitudes.
- The Syrope fatigue lifetime is about 15 years shorter than the bi-linear new
  model and about 5 years shorter than the bi-linear aged model.
- For ULS, the windward-line design tension is about 10% higher with the
  bi-linear new model than with Syrope.
- Bi-linear aged and Syrope extreme tensions are close to identical.
- Reported dynamic/quasi-static stiffness ratio is 2.6-2.8 for the bi-linear
  new model and 1.18-1.26 for the bi-linear aged model.

Source: Sorum et al. (2023), "Modelling of Synthetic Fibre Rope Mooring for
Floating Offshore Wind Turbines", JMSE 11(1), 193.

## 4. Recommended MoorDyn Comparison Outputs

For a MoorDyn run, request at least:

- Segment tension output: `t`
- Segment strain output: `s`
- Segment strain-rate output: `d`
- Fairlead tension or line end tension in the main output, if available

Compare:

- Mean fairlead tension
- Standard deviation or significant LF/WF tension, if separated
- Max fairlead tension
- Tension-strain loop or scatter around the mean load
- For method 1 vs method 2, whether both give similar response above the mean
  load and whether method 2 shows the expected bilinear slope change.

Good acceptance criteria for an early validation run:

- The static stretch should follow EA_s near the selected mean tension.
- Dynamic tension increments should be closer to EA_d than EA_s when the motion
  is mostly wave-frequency.
- Tension should remain below MBS for ordinary operating and design cases.
- If method 2 is used, the tension-strain plot should show a visible slope
  change near eps = 0.026667 for the current example.
