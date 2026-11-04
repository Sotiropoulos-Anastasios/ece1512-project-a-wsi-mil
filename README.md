
# ECE1512 Project A
## Whole-Slide Image Classification with Multiple Instance Learning

### Project Overview
PyTorch implementation of Attention-Based Multiple
Instance Learning (ABMIL) for WSI classification.

### Datasets
- CAMELYON16
- CAMELYON17
- BRACS

Pre-extracted features are available at:
https://zenodo.org/records/22166570

Large datasets are not included in this repository.

Expected local data organization:
feature/
  C16/patch_feats_pretrain_medical_ssl.h5
  C17/patch_feats_pretrain_medical_ssl.h5
  BRACS/patch_feats_pretrain_medical_ssl.h5

### Implementation
- dataloader.py: Dataset loading and splitting
- utils.py: Utility functions and metrics
- wsi_visualization.py: WSI attention visualization
- ECE1512_ProjectA_Student_C16.ipynb
- ECE1512_ProjectA_Student_C17.ipynb
- ECE1512_ProjectA_Student_BRACS.ipynb
- ECE1512_ALL_DATASETS_ONE_NOTEBOOK.ipynb

### Installation
pip install -r requirements.txt

### Experiments
ABMIL baseline and research-extension experiments
on the three datasets.

See the dataset-specific notebooks and experiment
results folders for implementation and evaluation.

### Reproducibility
See the experiment configurations, saved results,
and reproducibility documentation.
