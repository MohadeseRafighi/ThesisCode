# ThesisCode

Code accompanying the M.Sc. thesis **"Hybrid Statistical and Machine Learning Modeling of Cognitive Neuroscience Data"** (University of Tehran, Faculty of Mathematics, Statistics and Computer Science, 2025).

The thesis analyzes **fNIRS data with a nested (hierarchical) structure**: repeated measurements from several brain channels within each participant. It compares classical statistical models and tree-based / boosting mixed-effects methods that account for this dependence, and evaluates their prediction accuracy and robustness to contaminated data.

- Advisor: Dr. Zahra Rezaei Ghahroodi
- Author: Mohadese Rafighi

---

## Data

The analysis uses the open simultaneous EEG–fNIRS dataset of Shin et al. (2018) (n-back task; 26 participants, 3 sessions each, conditions: 0-back, 2-back, 3-back). The raw data is **not** included in this repository.

Preprocessing (`Preprocessing.m`) produces one long-format table with the following columns:

| Column | Description |
|---|---|
| `Subject` | Participant ID (1–26) |
| `Session` | Session number (1–3) |
| `Age`, `Gender` | Demographics |
| `n_back` | Task condition (`0-back`, `2-back`, `3-back`) |
| `Indices` | fNIRS channel |
| `value` | Mean HbO in the 10–20 s window after series onset (response variable) |
| `Accuracy` | Behavioral accuracy (%) |
| `MeanRT` | Mean reaction time of correct responses |
| `SDRT` | Standard deviation of reaction time |

All R scripts analyze only the **2-back and 3-back** conditions.

---

## Repository structure

| File | Language | Purpose |
|---|---|---|
| `Preprocessing.m` | MATLAB | fNIRS preprocessing and merging with behavioral data; writes the CSV used by all R scripts |
| `LMM.R` | R | Linear mixed-effects models (`nlme`) with different within-group correlation structures (none, AR(1), continuous AR(1), general symmetric), compared by AIC |
| `GLMMtree.R` | R | GLMM trees (`glmertree::lmertree`), with random intercept for subject and with nested subject/channel structure |
| `Reemtree.R` | R | RE-EM trees (`REEMtree`), with and without nested structure, default and AR(1)/CAR(1) correlation |
| `UnbiasedREEMtree.R` | R | Unbiased RE-EM tree (conditional inference trees from `party` combined with mixed-effects estimation), with and without nested structure and different correlation structures |
| `LongCARTtree.R` | R | Longitudinal CART (`LongCART`) with Age and Gender as partitioning variables |
| `Gpboost.R` | R | GPBoost (`gpboost`) with grouped random effects, with and without nested structure; also reproduces the fitted-vs-actual figure |
| `Evaluation.R` | R | Final comparison of all methods: fit metrics, robustness to contamination, and cross-validation (see below) |

---

## Pipeline

### 1. Preprocessing (`Preprocessing.m`)

Requires MATLAB and the [BBCI Toolbox](https://github.com/bbci/bbci_public).

- 6th-order Butterworth low-pass filter (0.2 Hz) applied to oxy and deoxy signals
- Epoching from −5 to 60 s around the series marker, baseline correction using −5 to −2 s
- Averaging per condition within each session, then extracting mean HbO between 10 and 20 s per channel
- Computing accuracy, mean RT and SD of RT per participant, condition and session from the behavioral summary files
- Merging everything into a single table and writing it to CSV

### 2. Modeling (R scripts)

Each model is fitted in two versions:

- **Without nested structure**: random effect for `Subject` only
- **With nested structure**: random effects for `Subject/Indices` (channel nested within participant)

Metrics reported: MSE, RMSE, MAE, and execution time.

### 3. Evaluation (`Evaluation.R`)

- **Contamination experiment**: 10% of observations of `value` are randomly selected and shifted by 10 SD and by 15 SD; the relative change (%) in RMSE and MAE of each method is reported.
- **Cross-validation**: subject-level split (90% of participants for training, 10% for testing), with responses scaled by a factor of 10⁴ for numerical stability; test RMSE, MSE and MAE are reported for every method.

---

## Requirements

**MATLAB**: BBCI Toolbox, Signal Processing Toolbox

**R packages**

```r
install.packages(c(
  "readr", "dplyr", "ggplot2", "gridExtra", "patchwork", "knitr",
  "nlme", "lme4", "glmertree", "partykit", "party",
  "REEMtree", "rpart", "LongCART", "gpboost"
))
```

---

## Usage

1. Download the dataset and run `Preprocessing.m` (edit the paths at the top of the script first).
2. Run any R script on the resulting CSV, or run `Evaluation.R` to reproduce the full comparison.

> **Paths:** all scripts use hard-coded Windows paths (for example `D:/payanname/Matlabcodes/...`). Change them to your local paths before running.
>
> **File name:** `Preprocessing.m` saves `HbO_Behavior.csv`, while the R scripts read `HbO_Behavior_fnirs.csv`. Rename the file or update the path in the R scripts.
>
> **Random seeds** are set inside the scripts (for example 12 for contamination, 40 for cross-validation, 123 for GPBoost) so results are reproducible.

---

## Citation

If you use this code, please cite the thesis:

```
Rafighi, M. (2025). Hybrid Statistical and Machine Learning Modeling of Cognitive
Neuroscience Data. M.Sc. thesis, University of Tehran.
```

Dataset:

```
Shin, J., von Lühmann, A., Kim, D.-W., Mehnert, J., Hwang, H.-J., & Müller, K.-R. (2018).
Simultaneous acquisition of EEG and NIRS during cognitive tasks for an open access dataset.
Scientific Data, 5, 180003.
```
