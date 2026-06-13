# Surgical-Complication-Risk-Prediction-Engine

A machine learning project for predicting post-surgical complication risk before surgery using patient demographics, baseline health conditions, procedure information, and clinical risk indicators.

This project is designed as a **pre-operative risk prediction engine** using  the R Language. The goal is to help identify patients who may need closer monitoring, preventive planning, or additional clinical review before surgery.

## About This Project

Surgical complications can create serious risks for patients and increase hospital resource burden. This project models whether a patient is likely to experience a post-operative complication using information available before or at the time of surgery planning.

The final selected model was a **Random Forest** trained on an extended predictor set. A classification threshold of **0.25** was chosen to prioritize patient safety by reducing missed complication cases.

The final model achieved:

- **ROC-AUC:** 0.925
- **PR-AUC:** 0.876
- **Sensitivity / Recall:** 0.785
- **Precision / PPV:** 0.701
- **F1 Score:** 0.741
- **False Positive Rate:** 0.113

In practical terms, the model identified approximately **4 out of 5 patients** who truly experienced a complication, and about **7 out of 10 patients flagged as high-risk** genuinely experienced a complication.

## Project Objective

The objective of this project was to build a predictive analytics model for **post-surgical complications** while keeping the problem realistic for clinical decision support.

The model was scoped as a **pre-operative risk assessment tool**, meaning it only uses data that would be available before or during surgery planning. Post-operative outcome variables were excluded to prevent data leakage.

A reliable pre-operative risk estimate could help clinicians:

- Allocate monitoring resources more effectively
- Plan preventive interventions
- Counsel patients before procedures
- Identify patients who may require additional review
- Support safer surgical planning workflows

## Dataset

The project uses a surgical procedures dataset containing:

- **14,635 surgical procedures**
- **25 variables**
- Patient demographics
- Baseline comorbidity indicators
- Procedure category information
- Historical complication and mortality rates
- Existing risk stratification index scores

The target variable is:

```text
complication
```

Where:

- `0` = no post-surgical complication
- `1` = post-surgical complication

The dataset showed moderate class imbalance:

| Outcome | Count | Percent |
|---|---:|---:|
| No complication | 10,945 | 74.8% |
| Complication | 3,690 | 25.2% |

Because the positive class was less common and clinically important, **PR-AUC** and sensitivity-focused evaluation were prioritized.

## Key Variables

The model used variables related to:

- Age
- BMI
- ASA status
- Gender
- Race
- Baseline cancer
- Cardiovascular disease
- Dementia
- Diabetes
- Digestive conditions
- Osteoarthritis
- Psychiatric conditions
- Pulmonary conditions
- Charlson Comorbidity Index
- AHRQ CCS procedure category
- Historical CCS complication rate
- Procedure timing

The variable `mort30` was excluded because it represents a post-operative mortality outcome and would not be available during pre-operative prediction.

## Data Preparation

The data preparation process included:

- Stratified 70/30 train-test split
- One-hot encoding for categorical variables
- Dummy column alignment across train and test sets
- Standardization of continuous predictors
- Leakage prevention by excluding post-operative outcomes
- Feature engineering for clinically meaningful risk patterns

The train-test split was:

| Split | Observations |
|---|---:|
| Training set | 10,244 |
| Test set | 4,391 |
| Total | 14,635 |

## Feature Engineering

Four new features were created to capture meaningful clinical risk patterns:

| Feature | Description |
|---|---|
| `comorbidity_count` | Sum of baseline condition indicators |
| `obese` | Indicates BMI ≥ 30 |
| `weekend_surgery` | Indicates surgery on Saturday or Sunday |
| `night_surgery` | Indicates surgery during evening or overnight hours |

These features helped summarize disease burden, obesity-related risk, and potential timing-related differences in care.

## Predictor Sets

Three predictor sets were tested to compare model performance and complexity.

### Benchmark

A simple interpretable baseline using:

- BMI
- Age
- ASA status
- Baseline comorbidities
- Charlson score

### Extended

The benchmark set plus:

- AHRQ CCS procedure categories
- Weekend surgery indicator
- Gender
- Race indicators
- CCS complication rate
- Comorbidity count

### Kitchen Sink

The extended set plus:

- Complication RSI
- Mortality RSI

## Modeling Approach

Multiple model types were evaluated:

- Logistic Regression
- Lasso Regression
- Random Forest
- XGBoost

Models were compared using stratified cross-validation and PR-AUC as the primary tuning metric.

The final selected model was:

```text
random_forest_predSet_extended
```


## Threshold Selection

A threshold of **0.25** was selected instead of the default 0.50 threshold.


## Model Performance

The final model was evaluated on the held-out test set.

| Metric | Test Set Value |
|---|---:|
| ROC-AUC | 0.925 |
| PR-AUC | 0.876 |
| Sensitivity / Recall | 0.785 |
| Precision / PPV | 0.701 |
| F1 Score | 0.741 |
| False Positive Rate | 0.113 |

## Key Findings

- Random Forest and XGBoost produced stronger class separation than simpler linear models.
- The extended Random Forest model provided the best balance of performance and complexity.
- The 0.25 threshold helped prioritize sensitivity for patient safety.
- Test-set results were stable compared with validation results.
- Fairness disparities were identified and reported transparently.
- The model is technically promising but not ready for autonomous clinical deployment.



## Methods Used
- Random Forest
- XGBoost
- Logistic Regression
- Lasso Regression
- Cross-validation
- PR-AUC and ROC-AUC evaluation
- Threshold tuning
- Fairness metrics
- One-hot encoding
- Feature engineering


## Additional Files

This repository also includes supporting files for reviewing the full project:

* **Final PDF Report** — A polished project report summarizing the objective, dataset, modeling strategy, performance results, fairness assessment, and final recommendations.
* **R Markdown Source Code (`.Rmd`)** — The source analysis file containing the code used for data preparation, feature engineering, model training, evaluation, and reporting.
* **Markdown Report Version** — A GitHub-friendly version of the PDF report, including images and visual outputs, so the full analysis can be viewed directly in the repository.

Together, these files provide both a high-level explanation of the project and the technical workflow behind the final results.

## Disclaimer

This project is for educational and research purposes only. It is not a medical diagnostic tool and should not be used for clinical decision-making without further validation, fairness review, and approval from qualified healthcare professionals.



