# ECE1512 Project A — Attention-Based MIL for Whole-Slide Histopathology

**ECE1512 Digital Image Processing · University of Toronto · Fall 2026**  
**Author:** Anastasios-Theocharis Sotiropoulos  
**Code, experiment artifacts, and reproducibility notes accompanying the Project A report**

## Overview

This repository implements and evaluates **gated Attention-Based Multiple Instance Learning (ABMIL)** for whole-slide image (WSI) classification using **pre-extracted patch features** in PyTorch. It reproduces the course ABMIL baseline and examines **class-conditional diagnostic evidence learning** through staged extensions and independently initialized ablations on **CAMELYON16 (C16)**, **CAMELYON17 (C17)**, and **BRACS**.

The project investigates two research questions:

1. **Implemented:** Can class-specific attention and prototype-guided evidence learning improve slide-level classification, particularly for imbalanced and heterogeneous diagnostic classes?
2. **Proposed, not fully implemented:** Can predictions remain reliable under missing tissue evidence through a full-bag versus reduced-bag prediction-consistency objective?

The implemented work compares class-prior adjustment, class-specific gated attention, prototype-guided attention, prototype-discrimination regularization, restricted contextual self-attention, and evidence-selection variants. These are **controlled empirical modifications of existing ideas**, not claims of inventing the underlying mechanisms.

**Quick navigation:** [Four notebooks](#the-four-jupyter-notebooks) · [Results](#reported-results) · [Data](#datasets-and-splits) · [How to run](#setup-and-execution) · [Reproducibility](#reproducibility-and-limitations)

## The four Jupyter notebooks

The three dataset-specific notebooks are the recommended starting point. Each contains the completed course workflow: dataset loading, ABMIL model, training/validation, checkpoint selection, held-out testing, optional attention inspection, and the staged A0–A4 research extensions.

| Notebook | Task and role | Where to start |
|---|---|---|
| [`ECE1512_ProjectA_Student_C16.ipynb`](ECE1512_ProjectA_Student_C16.ipynb) | **CAMELYON16:** binary normal/tumor WSI classification. Includes baseline and staged extensions, held-out test diagnostics, and C16 attention visualization. | Start at Step 0; `DATASET = 'C16'`. |
| [`ECE1512_ProjectA_Student_C17.ipynb`](ECE1512_ProjectA_Student_C17.ipynb) | **CAMELYON17:** four-class classification; the test data come from held-out centers. Useful for studying class imbalance and generalization across acquisition sites. | Start at Step 0; `DATASET = 'C17'`. |
| [`ECE1512_ProjectA_Student_BRACS.ipynb`](ECE1512_ProjectA_Student_BRACS.ipynb) | **BRACS:** seven-class breast-lesion classification. Includes baseline and staged extensions with classwise metrics and imbalance-sensitive evaluation. | Start at Step 0; `DATASET = 'BRACS'`. |
| [`ECE1512_ALL_DATASETS_ONE_NOTEBOOK.ipynb`](ECE1512_ALL_DATASETS_ONE_NOTEBOOK.ipynb) | **Unified experiment orchestrator:** processes C16 → C17 → BRACS in one kernel; runs independently initialized X-series component ablations, saves progress/results, and compares configurations. | Begin with **Cell 1: User Controls**. This is the advanced/long-running notebook. |

**What the notebooks share:** `dataloader.py` for HDF5 features/splits, `utils.py` for reproducibility and classification metrics, and the YAML configurations under `config/`. The dataset-specific notebooks retain the instructional Step 0–9 organization and additionally contain student-authored reporting/export and experimental extensions.

**Important controls in the unified notebook:**

```python
PROJECT_DIR = Path('.').resolve()  # open Jupyter from repository root
DATASETS = ('C16', 'C17', 'BRACS')
EXPERIMENT_PLAN = 'focused'       # also supports 'all_singles_pairs', 'all', 'custom'
RUN_EXISTING_STAGES = False       # do not rerun A0–A4 when False
RUN_EXTRA_ABLATIONS = True
SKIP_COMPLETED_ABLATIONS = True
EVALUATE_TEST = False            # validation first; deliberate final testing separately
EPOCH_OVERRIDE = None            # original 30 epochs when None
```

These are the defaults in the submitted version. **`RUN_EXISTING_STAGES=False` does not recreate the A0–A4 outputs on a fresh clone.** To regenerate those results, use the individual notebooks or deliberately enable the stages after confirming the data and output settings. The optional `run_one_notebook.py` mentioned inside the combined notebook is **not included in this repository**; open and execute the `.ipynb` file directly with Jupyter unless that runner is supplied separately.

## Experiments and method notation

### Staged experiments A0–A4

The dataset-specific notebooks evaluate the same sequential design on all three datasets:

| Stage | Description |
|---|---|
| **A0** | Original course **gated ABMIL baseline**. |
| **A1** | A0 architecture with **class-prior logit-adjusted cross-entropy** during training (L). |
| **A2** | Class-specific gated attention + prototype-guided attention, with the A1 loss (**L + H + P**). |
| **A3** | A2 plus a **prototype-discrimination auxiliary objective** (**L + H + P + R**). |
| **A4** | A3 model with **restricted selected-patch contextual multi-head attention** (C). |

These stages are **separately trained experiments**, not sequential fine-tuning of a single checkpoint. For exact implementation details and hyperparameters, consult the notebooks and the accompanying report.

### Independently initialized X-series ablations

The fourth notebook examines separate contributions and combinations, using the following notation:

| Symbol | Meaning |
|---|---|
| **L** | Class-prior / logit-adjusted classification loss |
| **H** | Class-specific gated attention heads |
| **P** | Prototype-guided attention scores |
| **R** | Prototype-discrimination auxiliary loss |
| **S** | Attention-ranked evidence selection / retention variants |
| **C** | Restricted contextual self-attention over selected patch features |

Examples: `X_H` isolates class-specific attention; `X_HP` combines class heads with prototype guidance; `X_LHPR` combines L, H, P, and R. Additional suffixes in the results directories describe selection fractions, attention-mass thresholds, regularization strengths, and context placement. **The proposed full-bag/reduced-bag prediction-consistency loss is not implemented by the S-only ablations.**

## Reported results

The following are **held-out test** results from the accompanying 17-page Project A report. Values correspond to checkpoints chosen using validation macro-AUROC + macro-F1. Accuracy is a percentage; AUROC and F1 are macro-averaged.

| Dataset | Model | Test accuracy (%) | Test macro-F1 | Test macro-AUROC |
|---|---|---:|---:|---:|
| C16 | ABMIL **A0** | 82.95 | 0.8159 | 0.8704 |
| C16 | **H** (class-specific heads) | **93.02** | **0.9236** | **0.9684** |
| C16 | HP (heads + prototypes) | 88.37 | 0.8668 | 0.9505 |
| C17 | ABMIL **A0** | 79.50 | 0.5289 | 0.8357 |
| C17 | H | 81.00 | 0.6175 | 0.8661 |
| C17 | **HP** | **87.50** | **0.7354** | **0.8949** |
| BRACS | ABMIL **A0** | 40.70 | 0.3439 | 0.7920 |
| BRACS | **H** | **45.35** | **0.4177** | 0.7857 |
| BRACS | HP | 40.70 | 0.3199 | 0.7717 |

**Interpretation:** Class-specific gated heads improve test macro-F1 relative to A0 in all three evaluated tasks. Adding prototypes to H helps markedly on C17 but is not consistently beneficial on C16 or BRACS. The table shows **exploratory, primarily single-seed results**; these are not statistically established improvements. Reviewing numerous test results while exploring architectures is a limitation, even when individual checkpoint epochs were selected on validation data.

**Find the underlying artifacts:**

- **A0–A4 metrics, learning curves and per-slide predictions:** [`Experiments_A0_A1_A2_A3_A4/`](Experiments_A0_A1_A2_A3_A4/); see [`C16 A0 metrics`](Experiments_A0_A1_A2_A3_A4/results_C16_A0/metrics.json), [`C17 A0 metrics`](Experiments_A0_A1_A2_A3_A4/results_C17_A0/metrics.json), and [`BRACS A0 metrics`](Experiments_A0_A1_A2_A3_A4/results_BRACS_A0/metrics.json).
- **Independent ablations:** [`Experiments_Ablations/`](Experiments_Ablations/); see [`C16 X_H test metrics`](Experiments_Ablations/results_C16_X_H/test_metrics.json), [`C17 X_HP test metrics`](Experiments_Ablations/results_C17_X_HP/test_metrics.json), and [`BRACS X_H test metrics`](Experiments_Ablations/results_BRACS_X_H/test_metrics.json).
- **Figures and underlying plot data used in the report:** [`report_figures/`](report_figures/), including [`dataset overview`](report_figures/dataset_overview.png) and [`C16 test ROC`](report_figures/C16_test_ROC.png).

Each experiment folder typically includes metrics (`metrics.json` / `test_metrics.json`), validation/test predictions, training histories and generated diagnostic PNGs. The complete A0–A4 and focused-ablation tables, methodology and interpretation are in the **accompanying PDF report submitted separately through Quercus**.

## Datasets and splits

**Official ECE1512 pre-extracted feature archive:** [WSI Feature, Zenodo record 23130543](https://zenodo.org/records/23130543), DOI [`10.5281/zenodo.23130543`](https://doi.org/10.5281/zenodo.23130543). The archive contains C16, C17 and BRACS HDF5 features. The large feature datasets and raw slide images are **not duplicated in Git**.

Download and extract the relevant course feature files to:

```text
feature/
  C16/patch_feats_pretrain_medical_ssl.h5
  C17/patch_feats_pretrain_medical_ssl.h5
  BRACS/patch_feats_pretrain_medical_ssl.h5
```

Update `data_dir` in the relevant YAML file if your files are stored elsewhere:

- [`config/camelyon16_medical_ssl_config.yml`](config/camelyon16_medical_ssl_config.yml)
- [`config/camelyon17_medical_ssl_config.yml`](config/camelyon17_medical_ssl_config.yml)
- [`config/bracs_medical_ssl_config.yml`](config/bracs_medical_ssl_config.yml)

Each HDF5 top-level group corresponds to a slide; `feat` stores an `N × 384` patch-feature matrix, `coords` optionally stores patch coordinates, and a slide-level `label` is stored in HDF5 attributes. The pre-extracted encoder is **not retrained** in these experiments. Its original checkpoint and patch-extraction settings are not fully documented in the course files; see the report's limitations.

| Dataset | Classes | Training slides | Validation slides | Test slides | Total |
|---|---:|---:|---:|---:|---:|
| CAMELYON16 | 2 | 242 | 27 | 129 | 398 |
| CAMELYON17 | 4 | 269 | 30 | 200 | 499 |
| BRACS | 7 | 389 | 62 | 86 | 537 |

The C16 held-out test partition is identified by slide names. C17 holds out centers 3–4 for testing, with centers 0–2 used for train/validation. The C16 and C17 train/validation partitions are stratified with seed 42 and validation fraction 0.1. BRACS uses split labels in [`dataset_csv/bracs.csv`](dataset_csv/bracs.csv); C17 requires [`dataset_csv/camelyon17.csv`](dataset_csv/camelyon17.csv). Split counts describe the **course feature files used in the report**, not necessarily every upstream public dataset release.

## Setup and execution

**Dependencies:** Python, PyTorch, TorchMetrics, NumPy, pandas, scikit-learn, h5py, PyYAML, matplotlib, and optional OpenSlide for raw WSI attention overlays. See [`requirements.txt`](requirements.txt). **Package versions are not pinned in the submitted requirements file**; use a compatible environment and record versions when attempting a strict replication.

From a terminal opened **at the repository root**:

```bash
python -m venv .venv
# On Windows PowerShell:
.venv\Scripts\python -m pip install -r requirements.txt
.venv\Scripts\python -m pip install jupyter
.venv\Scripts\python -m jupyter notebook
```

On Linux/macOS, activate the virtual environment and run `python -m pip install -r requirements.txt`, `python -m pip install jupyter`, then `python -m jupyter notebook`.

**Recommended run path:**

1. Download the HDF5 features and confirm each YAML `data_dir` points to an existing file.
2. Open one dataset-specific notebook (C16, C17, or BRACS). Its Step 0 sets `PROJECT_DIR` and `DATASET`; when launching Jupyter from the repository root, `PROJECT_DIR = './'` is appropriate.
3. Run **Steps 1–7** in order to load feature bags, train ABMIL, choose the best epoch on validation, and evaluate the held-out test split. Step 8 optionally inspects attention; whole-slide overlays require additional raw slides under `slide/` and matching annotations when available.
4. Run **Step 9** to train A1–A4 and save each stage's results. **By default the notebooks set `RESULTS_ROOT` to the project root, so newly generated `results_<DATASET>_<STAGE>/` folders appear there.** The committed copies have been organized under `Experiments_A0_A1_A2_A3_A4/` for browsing; move/organize new outputs deliberately or change `RESULTS_ROOT` before running. Use the saved comparison files to compare against A0.
5. For the independent focused ablations, use the combined notebook, verify Cell 1 controls, and **do not automatically enable test evaluation for exploratory configurations**. New ablation folders likewise default to `RESULTS_ROOT = PROJECT_DIR` rather than the archive folder `Experiments_Ablations/`.

**Reported training protocol:** 384-dimensional fixed patch features → 256-dimensional trainable projection → 128-dimensional gated attention → slide classifier; batch size 1, 30 epochs, AdamW, initial learning rate `1e-4`, weight decay `1e-5`, seed 42, cross-entropy for A0, and validation macro-AUROC + macro-F1 for model selection. Study the notebooks and config files before changing a run; the full ablation sweep is computationally expensive.

**Checkpoint availability:** local `.pt` checkpoints were excluded from the initial Git commit. The published metrics and predictions can be inspected without retraining, but loading the trained model weights requires separately supplied checkpoints or rerunning training. No external checkpoint URL is claimed here.

## Repository organization

```text
README.md
requirements.txt
.gitignore
dataloader.py                # HDF5 data loading and split logic
utils.py                     # configuration, seeding and metrics
wsi_visualization.py        # optional C16 raw-WSI attention overlays
config/                     # three dataset-specific YAML files
dataset_csv/                # C17 and BRACS metadata
ECE1512_ProjectA_Student_C16.ipynb
ECE1512_ProjectA_Student_C17.ipynb
ECE1512_ProjectA_Student_BRACS.ipynb
ECE1512_ALL_DATASETS_ONE_NOTEBOOK.ipynb
Experiments_A0_A1_A2_A3_A4/  # saved baseline and staged runs
Experiments_Ablations/       # saved independent X-series results
report_figures/              # report images and figure data
docs/
  REPORT_REPRODUCIBILITY.md
  CODE_REPRODUCIBILITY.md
feature/                     # local ONLY; excluded by .gitignore
slide/                       # local ONLY; excluded by .gitignore
```

## Reproducibility and limitations

The **PDF report** and the **code repository** are complementary artifacts. The report presents the research motivation, mathematical formulation, design decisions, tables, discussion and limitations. This repository supplies the implementation, dataset metadata, experiment configurations and saved quantitative artifacts.

- [**Report / paper reproducibility checklist**](docs/REPORT_REPRODUCIBILITY.md) — scientific methods, splits, claims, metrics and limitations.
- [**Code completeness and reproducibility checklist**](docs/CODE_REPRODUCIBILITY.md) — dependencies, training/evaluation notebooks, artifact paths, checkpoints and practical rerun instructions.

Known limitations include primarily single-seed main comparisons, small validation partitions (especially C16/C17), unpinned dependency versions, absence of pretrained or trained `.pt` weights from the first Git commit, incomplete upstream feature-extraction details, and exploratory examination of multiple test-set outcomes. **Attention weight is not a calibrated tumor probability.** The proposed full-bag/reduced-bag prediction-consistency objective is a future experiment, not an evaluated result.

## Attribution and references

This work builds on the **ECE1512 Project A course starter materials** supplied by Linfeng Ye (University of Toronto, 2026). The student completed the course model/training tasks, added evaluation exports, implemented and investigated the stated research extension, and performed additional independent ablations. Original research contributions described in the literature are cited rather than claimed as novel algorithms.

1. M. Ilse, J. M. Tomczak, and M. Welling, *Attention-based Deep Multiple Instance Learning*, ICML, 2018. [Paper](https://proceedings.mlr.press/v80/ilse18a.html).
2. M. Y. Lu et al., *Data-efficient and weakly supervised computational pathology on whole-slide images*, Nature Biomedical Engineering, 2021 (CLAM).
3. B. Li et al., *Dual-stream multiple instance learning network for whole slide image classification with self-supervised contrastive learning*, CVPR, 2021 (DSMIL).
4. Z. Shao et al., *TransMIL: Transformer based correlated multiple instance learning for whole slide image classification*, NeurIPS, 2021.
5. H. Zhang et al., *DTFD-MIL: Double-tier feature distillation multiple instance learning*, CVPR, 2022.
6. L. Ye, [*WSI Feature* (ECE1512 pre-extracted features)](https://doi.org/10.5281/zenodo.23130543), Zenodo, 2026.

**Note:** This is a course research project using pre-extracted features; reported results are research/educational observations, **not** a clinically validated diagnostic system.
