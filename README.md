# Application-Aware Deep Learning for Optical Design

Code and data accompanying the paper:

> **Application-Aware Deep Learning for Optical Design: Enhancing Transmittance
> Prediction with a Domain-Specific Loss Function**
> Erfan Ghapanvari, Mostafa Salahshoor, Mohammad Reza Mohammadi
> *Results in Optics* (under review), 2026.

This repository contains the complete, end-to-end pipeline required to reproduce
every figure and table in the paper: the physics-based data generation (transfer
matrix method), the deep-learning training for all loss configurations, the
custom `focused_loss` implementation, the evaluation/benchmark scripts, and all
plotting code. Random seeds are fixed throughout for reproducibility.

---

## 1. Overview

We train a multilayer perceptron (MLP) to predict the p-polarized transmittance
spectrum of single-layer anti-reflective coatings (MgF₂ and SiO₂ on BK7) as a
function of incidence angle and film thickness. The central contribution is a
spectrally-aware, multi-objective loss function (`focused_loss`) that combines:

- **Shape loss** — a weighted MSE that amplifies accuracy in the high-transmittance region,
- **Value loss** — penalizes error at the peak transmittance, and
- **Location loss** — penalizes error in the wavelength of the peak.

The script also benchmarks this loss against standard baselines (MSE, L1/MAE,
Spectral Angle Mapper) and against non-learning surrogates (linear interpolation
and a nearest-neighbour transfer-matrix lookup).

---

## 2. Repository structure

```
.
├── README.md
├── LICENSE
├── requirements.txt
├── run_pipeline.py            # the full pipeline (training + analysis + reviewer add-on)
└── all_my_results/            # all inputs and outputs live here
    ├── mgf2_base.csv          # refractive index n(λ) for MgF2      [INPUT, required]
    ├── sio2_base.csv          # refractive index n(λ) for SiO2      [INPUT, required]
    ├── bk7_base.csv           # refractive index n(λ) for BK7       [INPUT, required]
    ├── mgf2_data/             # generated 500-sample LHS dataset    [auto-generated]
    ├── sio2_data/             # generated 500-sample LHS dataset    [auto-generated]
    ├── mgf2_ood_data/         # 56-sample interpolative-grid set    [auto-generated]
    ├── sio2_ood_data/         # 56-sample interpolative-grid set    [auto-generated]
    ├── *_model_*.h5           # trained model weights               [output]
    ├── *_scaler.pkl           # fitted input scalers                [output]
    ├── *_history.csv          # per-model training histories        [output]
    ├── *_full_error_analysis.csv   # MSE/RMSE/MAE/MaxAE per model   [output]
    ├── *_baseline_vs_dl_table.csv  # non-learning baselines         [output]
    ├── *_table1_clean.tex     # publication-ready Table I rows       [output]
    └── *.png / *.jpg          # all figures                         [output]
```

> **Minimum required inputs.** Only the three refractive-index CSVs
> (`mgf2_base.csv`, `sio2_base.csv`, `bk7_base.csv`) inside `all_my_results/`
> are needed to run the pipeline from scratch. Everything else is regenerated.
> Pre-trained weights are also included so the analysis can be reproduced
> without retraining.

### Format of the refractive-index CSVs

Each base CSV has two columns, a header row, and one row per wavelength:

```
Wavelength,n
400.0,1.38291
401.0,1.38275
...
1200.0,1.37901
```

The first column is wavelength in nm; the second is the (real) refractive index.

---

## 3. Installation

Requires Python 3.9–3.11.

```bash
git clone https://github.com/USERNAME/REPO.git
cd REPO
python -m venv .venv
source .venv/bin/activate        # on Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

---

## 4. How to run

A single command runs the whole pipeline:

```bash
python run_pipeline.py
```

The script is **idempotent and incremental**:

- If a dataset already exists in `all_my_results/`, it is **not** regenerated.
- If a trained model `*.h5` already exists, training for it is **skipped**.
- Delete the relevant files in `all_my_results/` if you want to force a clean
  re-run (e.g. remove `*_model_*.h5` to retrain from scratch).

The run proceeds in three stages:

1. **Simulation & training** — generates the LHS and interpolative-grid datasets
   via the transfer matrix method, then trains all eight loss configurations.
2. **Visualization & reporting** — produces the comparison table, metric bar
   charts, 3D surfaces, contour maps, learning curves, and scatter plots.
3. **Reviewer add-on** — produces the full error analysis (MSE/RMSE/MAE/MaxAE),
   the non-learning baseline comparison, and the clean LaTeX Table I rows.

### Reproducibility

Global seeds are set at the top of the script:

```python
np.random.seed(42)
tf.random.set_seed(42)
```

The train/validation split uses `random_state=42`, and the Latin hypercube
sampler uses `seed=42`. Note that exact bit-for-bit reproducibility of the
trained weights can still vary slightly across GPU/CPU hardware and TensorFlow
builds; the pre-trained weights used in the paper are therefore included in
`all_my_results/`.

---

## 5. Key outputs for the paper

| File | Paper element |
|------|---------------|
| `*_full_error_analysis.csv`   | Full error metrics (Table I, MaxAE column) |
| `*_baseline_vs_dl_table.csv`  | Non-learning surrogate comparison |
| `*_table1_clean.tex`          | LaTeX rows for Table I |
| `*_publication_comparison_plot.png` | Optimal-spectra comparison (Fig. 9) |
| `*_Learning_Curve.jpg`        | Learning curves (Fig. 7) |
| `*_Actual_vs_Predicted_Scatter.jpg` | Scatter plots (Fig. 8) |

---

## 6. Citation

If you use this code, please cite the paper:

```bibtex
@article{Ghapanvari2026AppAwareDL,
  title   = {Application-Aware Deep Learning for Optical Design:
             Enhancing Transmittance Prediction with a Domain-Specific Loss Function},
  author  = {Ghapanvari, Erfan and Salahshoor, Mostafa and Mohammadi, Mohammad Reza},
  journal = {Results in Optics},
  year    = {2026},
  note    = {Under review}
}
```

---

## 7. License

Released under the MIT License. See [LICENSE](LICENSE).
