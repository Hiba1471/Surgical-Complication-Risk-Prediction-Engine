# Surgical-Complication-Risk-Prediction-Engine

A machine learning project for predicting post-surgical complication risk before surgery using patient demographics, baseline health conditions, procedure information, and clinical risk indicators.

This project is designed as a **pre-operative risk prediction engine**. The goal is to help identify patients who may need closer monitoring, preventive planning, or additional clinical review before surgery.

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

The **extended predictor set** was selected for the final model because it provided strong performance without relying heavily on pre-built risk index scores.

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

This model offered the best balance between predictive performance, complexity, and practical usefulness.

## Threshold Selection

A threshold of **0.25** was selected instead of the default 0.50 threshold.

This decision was made because the clinical cost of missing a true complication is higher than the cost of creating an unnecessary alert. Lowering the threshold allowed the model to identify more high-risk patients.

At the selected threshold, the model favored sensitivity, making it more appropriate for a patient-safety use case.

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

The model generalized well from validation to test data, with no major performance drop on unseen records.

## Fairness Assessment

Fairness was assessed across gender and race indicator variables.

Although the model performed strongly overall, fairness analysis identified disparities in false positive rates across demographic groups. The most notable disparity appeared in the Race 2, category 1 subgroup, where the false positive rate was **0.244**, more than double the overall false positive rate of **0.113**.

Because of these disparities, the model should **not** be deployed as a clinical decision tool without additional fairness mitigation, subgroup monitoring, and clinical validation.

## Key Findings

- Random Forest and XGBoost produced stronger class separation than simpler linear models.
- The extended Random Forest model provided the best balance of performance and complexity.
- The 0.25 threshold helped prioritize sensitivity for patient safety.
- Test-set results were stable compared with validation results.
- Fairness disparities were identified and reported transparently.
- The model is technically promising but not ready for autonomous clinical deployment.

## Recommended Next Steps

Before real-world use, the following steps are recommended:

- Clarify race and gender category labels in the source data
- Investigate causes of subgroup false positive rate disparities
- Apply fairness-aware threshold calibration
- Consider fairness-constrained model training
- Compare the model against existing clinical tools such as NSQIP
- Add interpretability tools such as SHAP or partial dependence plots
- Conduct retrospective clinical validation
- Conduct a monitored prospective pilot before deployment
- Review regulatory and legal considerations around demographic variables

## Technologies Used

- R
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

## Repository Structure

```text
.
├── README.md
├── data/                 # Dataset or data access instructions
├── notebooks/            # Exploratory analysis and modeling notebooks
├── src/                  # Reusable preprocessing/modeling scripts
├── figures/              # ROC, PR, calibration, and threshold visuals
├── reports/              # Final report and supporting documentation
└── results/              # Model outputs and evaluation summaries
```

## Why This Project Matters

This project demonstrates an end-to-end healthcare machine learning workflow, including preprocessing, feature engineering, model comparison, threshold optimization, test-set evaluation, and fairness assessment.

It is especially relevant for healthcare analytics because it does not only focus on high predictive performance. It also considers practical deployment limits, fairness concerns, and clinical validation requirements.

## Disclaimer

This project is for educational and research purposes only. It is not a medical diagnostic tool and should not be used for clinical decision-making without further validation, fairness review, and approval from qualified healthcare professionals.
