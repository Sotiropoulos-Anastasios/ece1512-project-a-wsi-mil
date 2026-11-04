# ECE1512 Project A — Code Completeness / Reproducibility Checklist

**Artifact:** [GitHub code repository](https://github.com/Sotiropoulos-Anastasios/ece1512-project-a-wsi-mil).  
**Companion artifact:** Project A PDF report submitted separately through Quercus.  
**Basis:** [ML Code Completeness Checklist](https://medium.com/paperswithcode/ml-code-completeness-checklist-e9127b168501), alongside the course's [ML Reproducibility Checklist v2.0](https://www.cs.mcgill.ca/~jpineau/ReproducibilityChecklist-v2.0.pdf).

The external ML Code Completeness Checklist covers **dependencies, training scripts, evaluation scripts, pretrained models, and results/reproduction guidance**. The statuses below are based on the submitted notebook files and the staged/committed-file list shared during repository setup. **Recheck against the final GitHub branch.**

| Code completeness item | Status | Evidence / limitation | Practical improvement |
|---|---|---|---|
| **1. Dependencies / installation** | **Partial** | [`requirements.txt`](../requirements.txt) and README installation instructions are included; package versions are not pinned. | Record Python/PyTorch/CUDA versions; produce a tested lockfile if time permits. |
| **2. Training implementation** | **Yes** | [`C16`](../ECE1512_ProjectA_Student_C16.ipynb), [`C17`](../ECE1512_ProjectA_Student_C17.ipynb), and [`BRACS`](../ECE1512_ProjectA_Student_BRACS.ipynb) notebooks implement A0–A4; [`combined`](../ECE1512_ALL_DATASETS_ONE_NOTEBOOK.ipynb) supports additional X ablations. | Keep notebook cells executable in order and document stage controls. |
| **3. Evaluation implementation** | **Yes** | Validation/test routines, saved metrics JSON, class reports, prediction CSVs, learning curves and confusion matrices; source in notebooks and `utils.py`. | Make explicit which evaluations are validation-only and which are final held-out test. |
| **4. (Pre)trained model weights** | **No (in Git)** | `.pt` / `.pth` checkpoints were excluded from the initial Git commit; published metrics do not by themselves permit model-weight inference. | Provide a lawful external download/checkpoint link **or** document exact retraining steps and disclose the omission. |
| **5. Results + reproduction instructions** | **Partial** | README contains tables and navigable result links; independent ablations have `test_metrics.json` etc. Reproduction requires executing notebooks manually; no complete verified single command. | Record the exact notebook/settings/seed for each report table and, if available, a complete script command. |

## Four notebooks: what each demonstrates

| Notebook | Reproduction role |
|---|---|
| [`C16`](../ECE1512_ProjectA_Student_C16.ipynb) | The binary ABMIL baseline, A1–A4 extensions, held-out test, C16-specific attention inspection. |
| [`C17`](../ECE1512_ProjectA_Student_C17.ipynb) | Four-class held-out-center classification; baseline vs staged research extension. |
| [`BRACS`](../ECE1512_ProjectA_Student_BRACS.ipynb) | Seven-class breast-lesion classification; baseline vs staged extension with classwise diagnostics. |
| [`Combined`](../ECE1512_ALL_DATASETS_ONE_NOTEBOOK.ipynb) | C16 → C17 → BRACS orchestration; focused independent ablations and configurable additional experiments. |

## Dataset availability and expected layout

Download [ECE1512 WSI Feature (Zenodo 23130543)](https://doi.org/10.5281/zenodo.23130543) and place the extracted medical-SSL HDF5 files at:

```text
feature/C16/patch_feats_pretrain_medical_ssl.h5
feature/C17/patch_feats_pretrain_medical_ssl.h5
feature/BRACS/patch_feats_pretrain_medical_ssl.h5
```

The `feature/` and `slide/` directories are intentionally excluded from Git due to file sizes. `dataset_csv/` is provided and is required by the loader for C17 and BRACS. Confirm `data_dir` in each `config/*.yml` matches your local files.

## Exact execution path available in the repository

1. Run `python -m pip install -r requirements.txt` in a compatible environment, then install Jupyter separately if needed.
2. Launch Jupyter in the repository root and open a dataset-specific notebook.
3. For the **baseline (A0)**, execute the notebook through Step 7; the best checkpoint is selected on validation macro-AUROC + macro-F1.
4. For **A1–A4**, execute Step 9 with the intended stage selection and preserved baseline settings. The notebooks normally write new `results_<DATASET>_<STAGE>/` folders **at the project root**. The committed results were subsequently organized under `Experiments_A0_A1_A2_A3_A4/` for browsing.
5. For **X** ablations, open the combined notebook, review Cell 1 (`DATASETS`, `EXPERIMENT_PLAN`, `RUN_EXISTING_STAGES`, `RUN_EXTRA_ABLATIONS`, `EVALUATE_TEST`), then run relevant experiments. Newly produced X-folders also default to the project root, while submitted copies are grouped under `Experiments_Ablations/`.
6. Use the published prediction CSVs/metrics to inspect reported analyses without retraining; `.pt` weights are **not** published with the current snapshot.

> The combined notebook mentions an optional external `run_one_notebook.py` runner; that script was **not included in the published source files** and is not required for opening the notebooks manually.

## Results locations and traceability

- Baseline metric examples: [`C16 A0`](../Experiments_A0_A1_A2_A3_A4/results_C16_A0/metrics.json), [`C17 A0`](../Experiments_A0_A1_A2_A3_A4/results_C17_A0/metrics.json), [`BRACS A0`](../Experiments_A0_A1_A2_A3_A4/results_BRACS_A0/metrics.json).
- Extension metric examples: [`C16 H`](../Experiments_Ablations/results_C16_X_H/test_metrics.json), [`C17 HP`](../Experiments_Ablations/results_C17_X_HP/test_metrics.json), [`BRACS H`](../Experiments_Ablations/results_BRACS_X_H/test_metrics.json).
- Visualization/data: [`report_figures/`](../report_figures/), [`A0–A4 results`](../Experiments_A0_A1_A2_A3_A4/), [`X-series ablations`](../Experiments_Ablations/).

## Transparency about remaining gaps

- **Reproducibility ≠ simply uploading a notebook.** Source files and recorded results are publicly inspectable, but full checkpoint-level re-evaluation still needs checkpoints, or retraining with sufficient compute.
- **Environment:** `requirements.txt` has unpinned versions; exact installed environment should be recorded when known.
- **Upstream extraction:** model is trained on pre-extracted features, not raw WSI pixels; some encoder and patch-generation parameters are not documented in provided course files.
- **Statistics:** main reported results are primarily single-seed, exploratory comparisons; avoid claiming statistical significance.
- **Evaluation:** final test-set metrics may be observed after checkpoint selection; architecture comparison after inspecting many test runs can still create selection bias.
- **Privacy:** the PDF report includes a student identifier; consider redacting it before publishing the report itself on a public GitHub repository. The official Quercus submission can remain separate.

See also the [report/paper checklist](REPORT_REPRODUCIBILITY.md), which covers the scientific claims rather than repository completeness.
