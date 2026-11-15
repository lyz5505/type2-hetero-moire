# Type-II heterostructure moiré models

Mathematica code, data and slides for effective models of type-II van der Waals heterostructures (HgCl₂/PtTe₂, HgCl₂/TiTe₂, PbS₂/TiTe₂) and their moiré band structures, together with the LaTeX source of the supplementary information (SI).

## Contents

| Path | What it is |
|---|---|
| `SM_Figures_3_4_5 and figs in summary slide.nb` | Reproduces SI Figs. 3–5 (effective models vs. DFT) and contains the moiré calculations of slides 13–24 of `type2_summary_Yunzhe.pptx` |
| `data/` | DFT band data (JSON) read by the notebook above; keep it next to the notebook |
| `change_moire_length.nb` | Moiré calculations of `change moire length.pptx` (HgCl₂/TiTe₂, varying moiré period) |
| `type2bandgap.csv` | Saved result of the (Δm, λ) phase-diagram scan, read by `change_moire_length.nb` |
| `type2_summary_Yunzhe.pptx`, `change moire length.pptx` | Slides whose figures the notebooks reproduce |
| `thf_typeII_paper_note/` | LaTeX source: `si.tex`, `main.tex`, bibliography, figures, compiled `si.pdf` |

## Requirements and how to run

- Mathematica 15 (the notebooks were saved with 15.0). The moiré cells use `ParallelTable` and start all available subkernels.
- Open a notebook and evaluate it from top to bottom. Each notebook uses a private context (`CellContext -> Notebook`), so its variables do not clash with other open notebooks.
- Some cells are slow (Wilson loops, Berry curvature, gap scans, the phase-diagram scan). They are marked in the text cells; see [Run time](#run-time).

## Conventions

- **Energies** are in eV.
- **Momenta.** The effective models in sections 1–3 of the SM notebook use the dimensionless momentum q = a₀k. The moiré code uses k in nm⁻¹ with the same parameters rescaled by a₀ⁿ:

  | Heterostructure | a₀ (nm) |
  |---|---|
  | HgCl₂/PtTe₂ | 0.4016 |
  | HgCl₂/TiTe₂ | 0.373 |
  | PbS₂/TiTe₂ | 0.4123 |

- **Notation.** k± = k_x ± i k_y and k² = k_x² + k_y².
- **Time reversal.** All models are time-reversal symmetric; this is checked in the notebook.

## 1. `SM_Figures_3_4_5 and figs in summary slide.nb`

### Sections 1–3: SI Figs. 3–5

Each material section defines the parameters as an `Association` (`p3`, `p4`, …) and the Hamiltonian as a function of {q_x, q_y} (`h3`, `h4`, …). It then reads the DFT bands from `data/` and plots model (orange) against DFT (black: monolayer, green: heterostructure).

After every model definition an extra cell writes the Hamiltonian explicitly:

- `H3`, `H4`, … give the symbolic matrix.
- `smShowH` displays it with k± and k².
- `smNumH` shows it with the numerical parameters substituted.
- `smCheckH` verifies that the explicit matrix equals the numerical function (returns `True`).

| Figure | Model | Parameters |
|---|---|---|
| Fig. 3, HgCl₂/PtTe₂ (`h3`) | 4×4, Rashba SOC in the conduction band (SI Eq. 4) | E₁ = −E₂ = 0.045, A₁ = 0.28856, A₂ = −0.43440, B₀ = 0.257, R₀ = 0.134 |
| Fig. 4(a), TiTe₂ monolayer (`h4mono`) | (E₀ + A q²)σ₀ + W q_y(3q_x² − q_y²)σ_z (SI Eq. 10) | E₀ = −0.14, A = −0.266, W = −0.164 |
| Fig. 4(b), HgCl₂/TiTe₂ (`h4`) | spin-block-diagonal 4×4 with trigonal warping and a quartic valence-band term λq⁴ (SI Eq. 11) | E₁ = −E₂ = 0.024, A₁ = 0.4714, B₀ = 0.09096, A₂ = −0.1558, W = 0.2034, λ = −0.02154 |
| Fig. 5(a), PbS₂ monolayer (`h5pb`) | (E₀ + A q²)σ₀ | E₀ = −3.338, A = 0.51304 |
| Fig. 5(b), TiTe₂ monolayer (`h5ti`) | (E₀ + A q²)σ₀ + R(q_xσ_x + q_yσ_y) | E₀ = −3.054, A = −0.566, R = 0.056 |
| Fig. 5(c), PbS₂/TiTe₂ (`h5`) | 4×4 with interlayer tunnelling (SI, P-4m2 section) | E₀ = −E₁ = −0.0125, B = −0.573, A = 0.398, R₀ = −0.056, R_A = 0.044, T_B = 0.1488i |

The last cell exports the figures to `reproduced_figures/Fig3`–`Fig5` (`.pdf` and `.png`).

Fig. 5(c) keeps two conventions of the original figure:

- The heterostructure data are drawn on the monolayer M–Γ–X axis, although the JSON labels the path Y–Γ–X.
- Points 34 and 37 are removed from the fourth band.

### Section 4: moiré calculations (slides 13–24)

**Source and context.** This code is collected from the original working notebook. Cells are copied unchanged except where a text cell says `NOTE` or `ADDED`. The section runs in its own context `moire`` and is independent of sections 1–3.

**Moiré Hamiltonian.** Plane waves k + G are used for the moiré reciprocal-lattice vectors G inside a cutoff of n = 6, which gives 127 vectors.

- **Moiré period.** 55 nm (`TA = 1/55`).
- **Atomic part.** On the diagonal blocks, the non-moiré model at k + G is shifted by +10 eV. Plotted energies are therefore `eigenvalue − 10`.
- **Moiré potential.** First-shell, C₃-symmetric. It couples neighbouring G by ±iV₀ with V₀ = 37.5/(3√3) meV × V.
- **Λ.** The ratio of the conduction-band potential to the valence-band potential. Λ = 0 means the potential acts only on the valence band.
- **Δm.** The mass shift, +Δm on the conduction band and −Δm on the valence band. It changes the atomic gap by 2Δm.
- **Entry point.** `TotalHam3[k, Δm, V, Λ]`.

The non-moiré Hamiltonians are the same as in sections 1–3:

- **HgCl₂/TiTe₂, PbS₂/TiTe₂.** Identical (checked numerically).
- **HgCl₂/PtTe₂.** Uses the rounded values A₁ = 0.289 and A₂ = −0.434. The difference is ≤ 0.27 meV for |k| ≤ 1.5 nm⁻¹.

| Subsection | Slides | Content |
|---|---|---|
| HgCl₂/PtTe₂ | 13–16 | Δm sweep with the potential on both layers (Λ = 0.3, Z₂ = 0 at Δm = −0.035, Z₂ = 1 at −0.0403); potential only on the conduction band (Λ = 0); gap at Γ vs Δm; Wilson loops |
| HgCl₂/TiTe₂ (spin-up block, matrix size 254) | 17, 19–21 | Potential only on the valence band (C = 1 at Δm = −0.016); Berry curvature and Chern number; ratio 0.1; band crossing and C₃ eigenvalues at Γ and K (`m0 = −0.00965`) |
| PbS₂/TiTe₂ | 22–24 | Λ = 0 and Λ = 0.1 bands, gap scans and Wilson loops |

Notes, also given in the notebook:

- **Slide 15.** The original Wilson-loop cells called a three-argument `TotalHam3`. They now use `TotalHam3[k, Δm, 1, 0.3]`, the parameters of slide 14.
- **Slide 20.** The Δm = −0.008 panel has no corresponding cell in the original code.
- **Slides 23–24.** The P-4m2 gap scans compute `TotalHam3[{0,0}, 0.03 − m, …]` but plot the x-axis as −m, so the axis is shifted by 0.03 eV relative to Δm (original code, unchanged). The slide-23 scan (Λ = 0) is the slide-24 cells with Λ set to 0.

## 2. `change_moire_length.nb`

The model is HgCl₂/TiTe₂, spin-up block, a₀ = 0.373 nm (the same model as Fig. 4(b)).

| Section | Slides | Content |
|---|---|---|
| 1–2 | — | Moiré geometry and the non-moiré model |
| 3 | 2–4 | Moiré Hamiltonian with variable period, `TotalHam3[k, Δm, TA, V, Λ]` with TA = 1/(period in nm): 20 nm and 55 nm band structures, and ΔE₀ vs moiré period |
| 4 | 3, 5, 6 | Fixed 55 nm code (`TotalHam3[k, Δm, V, Λ]`): bands at Δm = −0.01, −0.015, −0.016; Berry curvature and Chern number; (Δm, λ) phase diagram |

Section 3 details:

- `nList` lists the moiré periods: 20, 25, …, 70 nm.
- `MmagicList` gives, for each period, the Δm at which the red band is flattest. The values are hard-coded; the code that determined them is not included.
- `Plotf`/`result` compute ΔE₀, the gap at Γ above the red band, for every period.
- `gapMeVList` stores the computed values, so slide 4 can be plotted directly.

Phase diagram (slide 5):

- The scan covers 101 × 101 points, Δm ∈ [−0.016, −0.008] and λ ∈ [0, 0.1].
- Its result is saved in `type2bandgap.csv`, columns {gap below, gap above, bandwidth}.
- An `ADDED` cell loads the CSV, so the plot cells run without redoing the scan.
- The two scan cells are marked **DO NOT RUN**.

## Run time

The notebooks were saved after evaluation, so they already contain the output figures.

Rough cost of re-running:

- One moiré band structure (301 k points, 508 × 508 matrices): about a minute with parallel kernels.
- Gap scans and the slide-4 loop: minutes.
- Wilson loops and Berry curvature: much longer.
- The phase-diagram scan (10201 points × 961 k points): very long; use the CSV instead.

## LaTeX (`thf_typeII_paper_note/`)

`si.tex` is the SI. Its effective-model equations and parameters match the notebooks:

- Eq. 10 and Eq. 11 use the trigonal-warping term k_y(3k_x² − k_y²).
- Eq. 11 includes the λk⁴ term (λ = −0.02154).

Compile with `pdflatex si`, `bibtex si`, then `pdflatex si` twice. `*.backup.tex` are earlier versions kept for reference.
