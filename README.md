# OCTA Parkinson Progression (public)

De-identified OCTA analysis for Parkinson’s progression: patient-level models (Logit), feature screening (RF/ANOVA), reproducible Jupyter notebook. **No raw data** is published.

> **TL;DR**  
> Public, de-identified notebook to explore OCTA metrics in PD vs. controls, build deltas across visits, define a progression composite, and fit patient-level logistic models (with figures and tidy outputs).

---

## Contents

- [What’s here](#whats-here)
- [Privacy & de-identification](#privacy--de-identification)
- [Environment](#environment)
- [How to run](#how-to-run)
- [Repo structure](#repo-structure)
- [Notebook outline](#notebook-outline)
- [Outputs](#outputs)
- [Reproducibility notes](#reproducibility-notes)
- [Troubleshooting](#troubleshooting)
- [License](#license)
- [Citation](#citation)
- [Acknowledgments](#acknowledgments)

---

## What’s here

- **Jupyter notebook** with clear, English comments and clean code.
- **Patient-level** modeling (Logit) adjusted by **Age** and **disease duration**.
- **Feature screening** (Random Forest + ANOVA F) and **multicollinearity** checks (VIF).
- **Figures** (ROC, logistic curve, quick distributions) and **tidy CSVs** in `data/derived/`.

---

## Privacy & de-identification

- Participants are pseudonymized as `PatientID` (hashed).
- Direct identifiers (`NombrePaciente`, `NombreControl`, `FechaNac`, etc.) are **dropped**.
- **Raw data is not published**. Place your private Excel locally (see below).

**De-identification salt**

The notebook reads an environment variable for pseudonymization:
`os.getenv("PII_SALT", "CHANGE-ME")`.

If you re-run the de-identification step yourself, set a salt **locally** before running and **never commit a real salt**.

```bash
# macOS / Linux
export PII_SALT="your-local-secret-salt"
```
```bash
# Windows PowerShell
$env:PII_SALT="your-local-secret-salt"
```

> Tip: leave the default "CHANGE-ME" in the codebase; set the real value only in your local environment or CI secrets.

---

## Environment

Python ≥ 3.9 is recommended.

**Quick setup (venv)**

```bash
python -m venv .venv
source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt
```

**Conda (optional)**

```bash
conda create -n octa python=3.10 -y
conda activate octa
pip install -r requirements.txt
```

requirements.txt
```
numpy
pandas
matplotlib
seaborn
scipy
statsmodels
scikit-learn
openpyxl
```

---

## How to run

1.   Clone this repo or download it as a ZIP.
2.   Place your private Excel (e.g., octa_parkinson2.xlsx) outside the repo or keep it untracked (see .gitignore).
3.   Activate the environment (see above).
4.   Launch Jupyter and open the notebook:
```bash
jupyter lab  # or: jupyter notebook
```
5.   Run cells top-to-bottom. Outputs land in:
   *   data/derived/ (CSV tables)
   *   figures/ (PNG figures)

> The published octa_parkinson.html lets you read the results without executing code.

---

## Repo structure

```bash
.
├─ README.md
├─ LICENSE
├─ .gitignore
├─ requirements.txt
├─ OCTA_Parkinson_public.ipynb     # main notebook (de-identified)
├─ octa_parkinson.html              # optional HTML export (de-identified)
├─ data/
│  └─ derived/
│     └─ .gitkeep                  # keep folder in git; CSVs are written here
└─ figures/
   └─ .gitkeep                     # figures are written here (see .gitignore note)
```

**.gitignore highlights**

* Ignores all of data/ except data/derived/**.

* You can choose to version or ignore figures/ (see comments inside .gitignore).

---

## Notebook outline

High-level steps (with clear, documented code cells):

**1.   Imports & config**

   Single source of truth for packages and plot styles.

**2.   Load data & quick peek**

   Load Excel; basic shape; no head with PII (uses PatientID).

**3.   Utilities**

   Helpers: safe filenames, coercion to numeric, suffix families (e.g., V0/v0, EPv0, Cv0).

**4.   Manual PD vs Control comparisons**

   Explicit pairs (provided list), Mann–Whitney U, Bonferroni correction.

**5.   Automatic deltas across families**

   Build v1/v2 deltas (e.g., V0→V1, V0→V2) and P1→P3 deltas.

**6.   Progression composite (MCID rules)**

   From v0→v2 using predefined thresholds (UPDRS/HY/NMS, etc. per code comments).

**7.   Group comparisons by progression**

   Numeric (Mann–Whitney U) & categorical (Chi²/Fisher).

**8.   NaN audit & KNN imputation**

   Report missingness; impute columns under a threshold (≤34%)—kept explicit.

**9.   Feature screening**

   Random Forest + ANOVA F; combined score; top variables plot.

**10.   VIF check**

   Multicollinearity review on selected features.

**11.   Patient-level logistic models**

   Per-variable models with covariates; robust fallbacks for separation.

**12.   Focused model**

   Logit(progression_composite) ~ PapPInfEPv0 + Edad + AñosSx, patient-level.

**13.   ROC/AUC + Youden’s J + confusion**

   ROC figure and CSV confusion tables (patient-level).

**14.   Logistic curve**

   Probability vs. baseline inferior peripapillary vessel perfusion (BIPVP), adjusted by covariates.

---

## Outputs

* Figures:

   *   ```figures/roc_focus_PapPInfEPv0.png```

   *   ```figures/logistic_curve_PapPInfEPv0.png (styled bands, mean, 0.5 cutoff)```

   *   Optional quick hist/box plots based on your runs.

*   Derived tables (CSV):

*   ```data/derived/OR_table_focus_PapPInfEPv0.csv```

*   ```data/derived/confusion_focus_0p50_patient.csv```

*   ```data/derived/confusion_focus_optimal_patient.csv```

*   ```data/derived/logit_<var>_coef.csv (per-variable models)```

*   Any additional delta summaries/stats depending on active sections.

---

## Reproducibility notes

*   Random seeds
Models that use randomness (e.g., Random Forest) set ```random_state=42``` in code.

*   Apparent performance
ROC/AUC and confusion matrices are in-sample (apparent).
For unbiased estimates, adapt the notebook to use CV/holdout.

*   Exact replication
To replicate the same numbers:

   *   Use the same ```df_grouped``` construction and drop rules (complete cases).

   *   Ensure the same imputation order (if used).

   *   Keep identical suffix mappings for deltas (see code comments).

---

## Troubleshooting

“Missing column …”
Check the header names in your Excel match the notebook (e.g., PapPInfEPv0, Edad, AñosSx).

“Outcome has a single class…”
After dropna(), you may be left with all 0s/1s. Verify the progression composite was computed and that both classes remain at patient level.

“Patient-level table still has duplicate PatientIDs.”
Ensure de-identification produced a single PatientID per participant and that aggregation uses groupby('PatientID', dropna=False).

Different N than before
Usually due to: imputation order, a renamed column, or dropna() removing rows newly missing after coercion to numeric.

---

## License

Code: MIT © 2025 Ana Jimena Hernández-Medrano

See LICENSE for details.

---

## Citation

If you use this code, please cite the repository:

```java
Ana Jimena Hernández-Medrano (2025). OCTA Parkinson Progression (public).
GitHub repository: https://github.com/jimenahmedrano/octa.parkinson-progression
```

(Optional) Add a ```CITATION.cff``` to enable GitHub’s “Cite this repository” button:

```yaml
cff-version: 1.2.0
title: "OCTA Parkinson Progression (public)"
authors:
  - family-names: YourLastName
    given-names: YourFirstName
date-released: 2025-09-27
version: "0.1.0"
license: MIT
```

---

## Acknowledgments

This notebook was prepared for public sharing with strict de-identification.
It is research code and not medical advice.

```css

