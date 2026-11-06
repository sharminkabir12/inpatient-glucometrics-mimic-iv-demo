# Inpatient Glucose Patterns and Length of Stay: A Machine-Learning Learning Project (MIMIC-IV Demo)

**Author:** Sharmin Kabir (MBBS, MSc Diabetes)
**Status:** Learning project. This is not a clinical study, not a validated prediction tool, and must not be used for clinical decisions.

## Purpose

I built this project to practise a careful, reproducible workflow for clinical data science: defining a cohort from diagnosis codes, cleaning laboratory data, turning repeated measurements into one row per hospital stay, avoiding data leakage, and evaluating simple machine-learning models against a baseline.

## Question

Among hospital stays with a recorded diabetes diagnosis, can blood glucose readings from the **first 48 hours** of admission help predict which stays will be **long** (more than 7 days)?

## Data

- **MIMIC-IV Clinical Database Demo, version 2.2** (open access; 100 patients, 275 admissions; Beth Israel Deaconess Medical Center, USA).
- Johnson A, Bulgarelli L, Pollard T, Horng S, Celi LA, Mark R. PhysioNet, 2023. <https://doi.org/10.13026/dp1f-ex47>
- Tables used: `admissions`, `patients`, `diagnoses_icd`, `labevents`, `d_labitems`.
- The data are not included in this repository. The notebook downloads them directly from PhysioNet. No credentialed access is needed for the demo.

## Methods

1. **Diabetes cohort.** Admissions with an ICD-9 code starting `250` or an ICD-10 code starting `E08`, `E09`, `E10`, `E11` or `E13`. Diagnosis codes are recorded for the whole admission, so the cohort is defined retrospectively, not at the 48-hour point.
2. **Glucose readings.** Laboratory results labelled "Glucose" in blood, kept only in mg/dL, with values outside 10 to 1500 mg/dL removed as a basic data-quality filter.
3. **Prediction window.** Only readings from hour 0 to hour 48 after admission were used, and only stays still in hospital at 48 hours, so that no glucose information from after the prediction time could leak into the features. Each stay needed at least 3 readings in the window.
4. **Features (one row per stay).** Number of readings, mean, standard deviation, coefficient of variation, minimum, maximum, percentage of readings below 70 mg/dL, above 180 mg/dL and between 70 and 180 mg/dL, **age at admission** and sex. Age at admission is calculated as `anchor_age + (admission year - anchor_year)`.
5. **Outcome.** Long stay, defined as length of stay greater than 7 days.
6. **Models.** A baseline that predicts the class proportion, logistic regression, and a small random forest. Imputation and scaling were done inside a pipeline so that they were learned from training data only.
7. **Evaluation.** Patient-grouped 5-fold cross-validation (all admissions from one patient kept in the same fold), with AUROC as the metric.

The thresholds (48 hours, 3 readings, 7 days) were fixed before looking at model results. They are reasonable but arbitrary choices.

## Results (version 2, current)

**Cohort flow**

| Step                                           | Admissions remaining |
|------------------------------------------------|----------------------|
| All admissions in the demo                     | 275                  |
| With a diabetes diagnosis code                 | 112                  |
| With at least one valid glucose reading        | 103                  |
| Still in hospital at 48 hours                  | 89                   |
| With at least 3 readings in the first 48 hours | **48**               |

The final analysis set contains **48 admissions from 27 patients**: 27 long stays and 21 not-long stays.

**Model performance (AUROC, patient-grouped 5-fold cross-validation)**

| Model                       | Mean AUROC | Fold AUROCs                    | Range       |
|-----------------------------|------------|--------------------------------|-------------|
| Baseline (class proportion) | 0.500      | 0.50 in every fold             | 0.50        |
| Logistic regression         | 0.304      | 0.54, 0.24, 0.20, 0.15, 0.39   | 0.15 to 0.54 |
| Random forest               | 0.585      | 0.79, 0.67, 0.32, 0.70, 0.44   | 0.32 to 0.79 |

Each held-out fold contained 9 to 10 admissions from 5 to 6 patients, with 4 to 7 long stays.

Neither model reliably outperformed the baseline. Logistic regression performed below chance on average, while the random forest showed considerable variation between folds. The random forest ranged from 0.32 to 0.79 across folds, which shows how unstable results are with so few admissions. No conclusion about predictive ability can be drawn. I report this as a finding in its own right: the demo dataset is too small to support a prediction model of this kind.

## Version history

**Version 1** (the version I described in my applications) used `anchor_age` directly as the age feature. Results: logistic regression mean AUROC 0.333 and random forest 0.572.

**Version 2** (this version) corrects the age feature to age at admission and adds fold-by-fold reporting. The cohort is unchanged (48 admissions, 27 patients). Mean AUROCs changed slightly (logistic regression 0.304, random forest 0.585). The conclusion is the same: neither model clearly beat the baseline.

## Figures

Figures are produced by the notebook and saved to `figures/`:

- `glucose_readings_histogram.png`: all glucose readings in the first 48 hours
- `clues_by_stay_length.png`: selected features by long versus not-long stay (descriptive only)
- `model_auroc_comparison.png`: cross-validated AUROC by model
- `logistic_regression_weights.png`: illustrative coefficients (explanatory only, not validated evidence)

## Limitations

- **Very small sample.** 48 admissions from 27 patients. Estimates are unstable and no statistical conclusions can be drawn.
- **Laboratory glucose only.** Readings are intermittent. Bedside fingerstick glucose and continuous glucose monitoring data are not included.
- **Number of readings may reflect illness.** Sicker patients are tested more often, so `n_readings` may act as a marker of severity rather than a glucose effect.
- **Retrospective cohort definition.** Diabetes status comes from diagnosis codes recorded for the whole admission, which may include information that was not known at 48 hours. Codes may also miss people with diabetes and carry no information on severity.
- **Many features for few patients.** Eleven features were used with only 27 patients. No feature selection was done, but a smaller, clinically motivated feature set would be more appropriate in a larger study.
- **No adjustment** for treatment, comorbidities, intensive care stay or illness severity.
- **No external validation.** Cross-validation uses a single small dataset from one US hospital, which may differ from NHS settings.
- **Mortality was not modelled** because there are too few in-hospital deaths in the demo.
- **Associations are not causal.**

## Possible next steps

- Repeat the analysis on the full MIMIC-IV dataset (credentialed access) with far more admissions.
- Add illness severity, medication, other diagnoses and ICU data.
- Use time-aware glucose features such as trends and time outside range.
- Validate on a separate set of patients before drawing any conclusions.

## How to run

1. Open `inpatient_glucometrics_mimic_iv_demo.ipynb` in Google Colab or Jupyter.
2. Run all cells from top to bottom. The notebook downloads the five required files from PhysioNet into a local `data/` folder (not tracked in this repository).
3. Required packages: `pandas`, `numpy`, `matplotlib`, `scikit-learn`.

## Note on tools

I wrote this project as a learning exercise with the help of AI coding assistance. I ran the analysis myself, checked the outputs, and take responsibility for the interpretation and for any errors.

## Repository contents

- `inpatient_glucometrics_mimic_iv_demo.ipynb`: the analysis notebook
- `figures/`: figures produced by the notebook
- `README.md`: this file
