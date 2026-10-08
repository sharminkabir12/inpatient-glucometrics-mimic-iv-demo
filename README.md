# Inpatient Glucose Patterns and Length of Stay

**An exploratory Python and machine-learning learning project using MIMIC-IV Demo v2.2.**

## Research question

Can glucose measurements recorded in the **first 48 hours** of admission help explore hospital stays lasting **more than seven days** among admissions with recorded diabetes diagnosis codes?

## Data and methods

- Publicly available [MIMIC-IV Demo v2.2](https://physionet.org/content/mimic-iv-demo/2.2/) hospital data.
- Identify admissions with diabetes diagnosis codes (ICD-9 `250`; ICD-10 `E08`, `E09`, `E10`, `E11`, `E13`).
- Select blood glucose laboratory readings in mg/dL, remove missing or invalid readings, and use only measurements taken in the first 48 hours.
- Include stays lasting at least 48 hours with at least three eligible glucose readings.
- Calculate glucose summaries per admission, including mean, variability, minimum, maximum, and percentages below 70 mg/dL, above 180 mg/dL, and between 70–180 mg/dL.
- Compare a prevalence baseline, logistic regression, and random forest using **patient-grouped** five-fold cross-validation. Both outcome classes are checked in training and test folds before AUROC is calculated.

## Results from the executed Colab notebook

| Measure | Result |
|---|---:|
| Hospital admissions in the demo | 275 |
| Admissions with a recorded diabetes diagnosis code | 112 |
| Admissions with at least one valid glucose reading | 103 |
| Diabetes admissions still in hospital at 48 hours, with glucose readings | 89 |
| Final admissions with at least three readings in the first 48 hours | **48** |
| Distinct patients in the final cohort | **27** |
| Admissions with stay >7 days | 27 |
| Admissions with stay ≤7 days | 21 |

### Exploratory model results

| Model | Mean cross-validation AUROC |
|---|---:|
| Baseline (class prevalence) | 0.500 |
| Logistic regression | 0.333 |
| Random forest | 0.572 |

All five grouped folds contained both outcome classes in their training and test subsets in this run. These values are **exploratory only**: the small, selected demo cohort cannot establish reliable predictive performance, and the apparent differences between models should not be interpreted as clinical evidence.

## How to run

1. Open `inpatient-glucometrics.ipynb` in Google Colab.
2. Choose **Runtime → Run all**. The notebook downloads the public demo files automatically.
3. Review the cohort flow, plots and model output. Figures are written to `figures/`.

For local use, install packages with `pip install -r requirements.txt` and run the notebook in Jupyter.

## Limitations

- Small demo cohort; not representative of clinical populations and not suitable for clinical deployment.
- Diabetes is identified through recorded diagnosis codes, which may miss cases.
- Uses laboratory glucose readings, not all bedside point-of-care measurements.
- Does not adjust for illness severity, treatment, comorbidities or hospital context.
- Length of stay is associated with many factors; this analysis does **not** establish causality.
- Patient-grouped cross-validation reduces overlap between training and test patients but does not replace external validation.

## Reproducibility and data

The notebook contains the full workflow and saves plots locally. Do not commit downloaded patient-level datasets or other restricted MIMIC data to GitHub. The open demo can be downloaded from PhysioNet by running the notebook.

**Status:** Executed in Google Colab with outputs saved in the included notebook. Not independently re-run in this package-building step.

