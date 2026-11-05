# ECE1512 Project A — Report / Paper Reproducibility Checklist

**Artifact:** *Attention-Based Multiple Instance Learning for Whole-Slide Image Classification: Project A* (17-page IEEE-style PDF report submitted separately via Quercus).  
**Companion code:** [GitHub repository](https://github.com/Sotiropoulos-Anastasios/ece1512-project-a-wsi-mil).  
**Basis:** [Machine Learning Reproducibility Checklist v2.0](https://www.cs.mcgill.ca/~jpineau/ReproducibilityChecklist-v2.0.pdf), also supplied with the assignment.  
**Scope:** This checklist concerns claims and experimental reporting in the PDF, **not** whether all code artifacts are downloadable. See [the separate code checklist](CODE_REPRODUCIBILITY.md).

**Statuses:** Yes = explicitly documented; Partial = documented but incomplete or not fully verifiable; No = not provided; N/A = outside this empirical project.

| Paper checklist question | Status | Evidence / qualifications |
|---|---|---|
| Clear mathematical/model description | **Yes** | Report §§III–IV and VI: feature projection, gated ABMIL, class-conditional heads, prototype guidance, losses, restricted context. |
| Assumptions clearly explained | **Yes** | Weak slide-level labels, fixed pretrained HDF5 features, the distinction between attention weights and tumor probabilities (§§I, III–IV). |
| Algorithm complexity (time/space/sample size) analyzed | **Partial** | Discussion of WSI patch count, quadratic full self-attention cost, restricted context; no systematic complexity bound for every proposed variant. |
| Theoretical claims and complete proofs | **N/A** | Empirical study; no new theorem or formal proof is claimed. |
| Dataset statistics provided | **Yes** | Report Tables II–III: C16 398 slides (242/27/129), C17 499 (269/30/200), BRACS 537 (389/62/86); class distributions provided. |
| Train/validation/test splits documented | **Yes** | §III: C16 named test slides, C17 held-out centers, BRACS metadata assignments; random seed 42 for C16/C17 train/val. |
| Exclusions and preprocessing explained | **Partial** | Fixed pre-extracted patch features and slide partitioning documented. Upstream patch extraction and exact encoder checkpoint settings are not fully available. |
| Downloadable dataset link supplied | **Yes** | [Zenodo ECE1512 WSI features](https://doi.org/10.5281/zenodo.23130543); large feature files are not copied into Git. |
| New data collection/annotation procedure | **N/A** | No new clinical dataset collected for this project. |
| Code dependencies available | **Partial** | `requirements.txt` exists, but tested package versions are not pinned; details belong chiefly to code artifact. |
| Training and evaluation code shared | **Yes** | Dataset-specific notebooks plus the combined ablation notebook are published in the accompanying repository. |
| Pretrained or trained model weights made available | **No** | `.pt`/`.pth` weights were excluded from the initial commit; rerun training or separately supply checked, permitted weights. |
| Results linked to exact reproduction instructions | **Partial** | Saved metrics and notebook execution guidance are provided, but not a tested one-command end-to-end reproduction script. |
| Hyperparameter range, final choices, and selection explained | **Partial** | A0 settings in Table IV and design choices in §VI; selection via validation macro-AUROC + macro-F1. Not all exploratory search bounds are enumerated. |
| Exact number of runs specified | **Partial** | Main staged analysis comprises A0–A4 on 3 datasets; report describes 18 focused variants × 3 datasets (54 additional runs). Additional informal/exploratory runs require explicit accounting if included in claims. |
| Evaluation statistics clearly defined | **Yes** | §IV-C: accuracy (%), macro-F1, macro one-versus-rest AUROC, cross-entropy test loss, classwise recall/confusion matrices. |
| Central tendency and variability reported | **No** | Main comparisons are primarily single-seed results. The report explicitly avoids confidence intervals or significance claims. |
| Runtime or estimated energy cost | **Partial** | Stage comparisons include runtime figures (`training_minutes.png`); not every reported experimental result has a consolidated compute-cost summary. |
| Computing infrastructure described | **Partial** | Notebook records device availability at execution, but report does not give a consolidated, reproducible OS/GPU/CUDA/package-version inventory. |

## Interpretation and evidence boundaries

- The report should be cited for **scientific conclusions**, the repository for **code and experimental artifacts**.
- Baseline comparisons and focused ablations are **exploratory**. Validating checkpoint epochs on held-out validation data does not make architecture selection based on inspected test scores confirmatory.
- The **second proposal** (full-bag versus reduced-bag prediction consistency) was **not fully implemented**. X-series `S` experiments modify evidence selection/retention but are not an implementation of that entire proposal.
