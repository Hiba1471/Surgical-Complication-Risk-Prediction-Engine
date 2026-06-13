# Surgical Complication Risk Prediction Engine

## About This Project

This project builds a machine learning risk prediction engine for identifying patients who may be at higher risk of post-surgical complications. The goal is to support safer pre-operative planning by using patient demographics, comorbidities, procedure information, and clinical risk indicators to estimate complication risk before surgery.

The final selected model was a random forest trained on an extended predictor set. Using a classification threshold of 0.25, the model achieved strong test-set performance with a ROC-AUC of 0.925 and PR-AUC of 0.876.

## Project Goal

The main goal of this project is to create a pre-operative decision-support model that can help identify patients who may need closer monitoring, additional preventive care, or more detailed clinical review before surgery.

Because the project is focused on patient safety, the model prioritizes sensitivity and precision-recall performance rather than accuracy alone. Missing a patient who later experiences a complication can be more harmful than flagging a patient who may not ultimately experience one.

## Dataset

The project uses a surgical procedures dataset containing 14,635 records and 25 variables. The data includes patient demographics, baseline health conditions, procedure categories, and existing risk index scores.

The target variable is binary:

- `0`: No post-surgical complication
- `1`: Post-surgical complication

The outcome distribution showed moderate class imbalance, with approximately 25.2% of cases involving a complication and 74.8% having no complication.

## Data Preparation

The data preparation process included:

- Stratified 70/30 train-test split
- One-hot encoding of categorical variables
- Alignment of dummy columns across train and test sets
- Standardization of continuous predictors
- Exclusion of post-operative variables that would cause data leakage

The variable `mort30` was excluded because it represents a post-operative mortality outcome and would not be available during pre-operative risk assessment.

## Feature Engineering

Several clinically meaningful features were engineered to improve prediction quality:

- `comorbidity_count`: total number of baseline conditions
- `obese`: indicator for BMI ≥ 30
- `weekend_surgery`: indicator for procedures scheduled on Saturday or Sunday
- `night_surgery`: indicator for procedures occurring during evening or overnight hours

These features were designed to capture patient risk burden, obesity-related surgical risk, and possible timing-related care differences.

## Modeling Approach

The project compared multiple learner types across different predictor sets:

- Logistic regression
- Lasso regression
- Random forest
- XGBoost

Three predictor sets were evaluated:

- **Benchmark:** simple clinical baseline variables
- **Extended:** benchmark variables plus procedure type, demographics, CCS complication rate, and engineered features
- **Kitchen sink:** extended variables plus pre-built risk stratification index scores

The final model used the extended predictor set because it offered a strong balance between predictive performance and practical interpretability without relying heavily on pre-built risk scores.

## Final Model

The selected model was:

```text
random_forest_predSet_extended
```

A classification threshold of 0.25 was selected because the clinical cost of missing a true complication was considered higher than the cost of creating an unnecessary alert.

At this threshold, the model was designed to flag more patients as high-risk in order to catch more true complications.

## Model Performance

The final model performed strongly on the held-out test set:

| Metric | Test Set Value |
|---|---:|
| ROC-AUC | 0.925 |
| PR-AUC | 0.876 |
| Sensitivity / Recall | 0.785 |
| Precision / PPV | 0.701 |
| F1 Score | 0.741 |
| False Positive Rate | 0.113 |

In practical terms, the model correctly identified approximately 4 out of 5 patients who truly experienced a post-surgical complication. Of the patients flagged as high-risk, approximately 7 out of 10 genuinely experienced a complication.

## Fairness Assessment

The project also evaluated fairness across gender and race indicator variables. Although the model achieved strong predictive performance, fairness analysis showed unequal false positive rates across demographic subgroups.

The most notable disparity was observed in the Race 2, category 1 subgroup, where the false positive rate was 0.244 — more than double the overall false positive rate of 0.113.

Because of these disparities, the model should not be deployed as a clinical decision tool without further bias mitigation, subgroup monitoring, and clinical validation.

## Key Findings

- Random forest and XGBoost showed stronger class separation than simpler linear models.
- The extended random forest model provided the best balance between performance and complexity.
- The selected threshold of 0.25 improved sensitivity for a patient-safety use case.
- Test performance was stable compared with validation, suggesting good generalization.
- Fairness concerns were identified and reported transparently.
- The model is technically promising but not ready for autonomous clinical deployment.

## Recommended Next Steps

Before real-world deployment, the following steps are recommended:

- Clarify demographic group labels in the source data
- Investigate causes of false positive rate disparities
- Apply fairness-aware threshold calibration
- Consider fairness-constrained model training
- Compare performance against existing clinical risk tools
- Add SHAP or other interpretability methods
- Conduct retrospective and prospective clinical validation
- Review regulatory and legal implications of using demographic variables

## Technologies Used

- R
- Random Forest
- XGBoost
- Logistic Regression
- Lasso Regression
- PR-AUC / ROC-AUC evaluation
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

Post-surgical complications can affect patient outcomes, hospital resource use, and care planning. A reliable pre-operative risk prediction model could help clinicians identify patients who may benefit from additional monitoring or preventive intervention.

This project demonstrates not only predictive modeling ability, but also responsible machine learning practices such as leakage prevention, threshold tuning, fairness assessment, and cautious deployment recommendations.

## Disclaimer

This project is for educational and research purposes only. It is not a medical diagnostic tool and should not be used for clinical decision-making without further validation, fairness mitigation, and review by qualified healthcare professionals.
