# Predictive Modeling for Surgical Complications

**Final Report**  
**Author:** Hiba Khan  
**Date:** May 30, 2026  

This report describes a predictive analytics project that modeled the risk of post-surgical complications using a dataset of 14,635 procedures. The final selected model — a random forest trained on an extended predictor set with a classification threshold of 0.25 — achieved a test ROC-AUC of 0.925 and PR-AUC of 0.876. While discrimination performance is strong, statistically significant fairness disparities across demographic groups mean the model should not be deployed in clinical practice without further bias mitigation, subgroup monitoring, and clinical validation.

## Table of Contents

1. [Project Objective](#1-project-objective)
2. [Dataset Overview](#2-dataset-overview)
   - [Outcome Distribution](#21-outcome-distribution)
   - [Key Variables](#22-key-variables)
3. [Data Preparation](#3-data-preparation)
   - [Train/Test Split](#31-traintest-split)
   - [One-Hot Encoding](#32-one-hot-encoding)
   - [Standardization](#33-standardization)
4. [Feature Engineering](#4-feature-engineering)
5. [Predictor Sets](#5-predictor-sets)
6. [Modeling Strategy](#6-modeling-strategy)
7. [Validation Findings](#7-validation-findings)
8. [Test-Set Evaluation](#8-test-set-evaluation)
9. [Fairness Assessment](#9-fairness-assessment)
10. [Final Recommendation](#10-final-recommendation)
11. [Conclusion](#11-conclusion)
12. [Appendix A: Variable Reference](#appendix-a-variable-reference)
13. [Appendix B: Visual Evidence from Milestones](#appendix-b-visual-evidence-from-milestones)

---

## 1. Project Objective

The goal of this project was to predict whether a surgical patient would experience a post-operative complication. The problem was scoped as a pre-operative risk assessment task: all predictions must rely only on information available before or at the time of surgery planning, making the model practically deployable as a clinical decision-support tool.

A reliable pre-operative risk estimate could help clinicians allocate monitoring resources, plan preventive interventions, and counsel patients before their procedure. The binary target variable `complication` takes the value `1` when a post-surgical complication occurred and `0` otherwise.

Although early scoping considered filtering to elderly or high-ASA-status patients, the final modeling scope used the full surgical case population to maximise training data and generalisability.

---

## 2. Dataset Overview

The dataset contains 14,635 surgical procedures and 25 columns covering patient demographics, pre-existing conditions, procedure information, and risk index scores. The dataset is publicly available on Kaggle and contains no missing values, which simplified preprocessing.

### 2.1 Outcome Distribution

The outcome variable shows moderate class imbalance: approximately 74.8% of records had no complication and 25.2% had a complication. Because the positive class is both less common and clinically important, precision-recall performance was treated as the primary tuning metric throughout.

| Outcome | Count | Percent |
|---|---:|---:|
| No complication (0) | 10,945 | 74.8% |
| Complication (1) | 3,690 | 25.2% |

**Table 1:** Distribution of the outcome variable.

### 2.2 Key Variables

Variables included patient characteristics such as age, BMI, gender, and race; baseline comorbidity indicators such as cancer, cardiovascular disease, diabetes, dementia, and others; a Charlson Comorbidity Index score; procedure category codes (AHRQ CCS); historical complication and mortality rates for each procedure category; and pre-built risk stratification index scores.

The variable `mort30` (30-day mortality) was excluded because it is a post-operative outcome and cannot be used for pre-operative prediction.

---

## 3. Data Preparation

### 3.1 Train/Test Split

The data was split into training and testing sets using a 70/30 random split stratified by `complication`. Stratification ensured both splits maintained similar positive-class proportions, which is important under class imbalance.

| Split Detail | Value |
|---|---:|
| Training observations | 10,244 |
| Testing observations | 4,391 |
| Total | 14,635 |
| Missing values | None — imputation not required |

**Table 2:** Train/test split summary.

### 3.2 One-Hot Encoding

Three integer-coded categorical variables — `asa_status`, `ahrq_ccs`, and `race` — were expanded into dummy columns using `fastDummies`. Original columns were retained alongside the dummies. Dummy column alignment was performed across splits to handle any categories appearing in only one split.

### 3.3 Standardization

Eight continuous predictors were standardized using training-set means and standard deviations. The same parameters were applied to the test set to prevent data leakage. Standardized variables included BMI, age, hour, baseline Charlson score, mortality RSI, CCS mortality rate, CCS complication rate, and complication RSI.

---

## 4. Feature Engineering

Four new features were constructed to capture clinically meaningful risk patterns:

| Feature | Definition | Clinical rationale |
|---|---|---|
| `comorbidity_count` | Sum of all eight baseline condition indicators | Compact summary of total disease burden |
| `obese` | 1 if BMI ≥ 30 | Obesity is an established surgical risk factor |
| `weekend_surgery` | 1 if surgery on Saturday or Sunday | Potential staffing and resource differences |
| `night_surgery` | 1 if hour ≥ 17 or hour ≤ 7 | Potential fatigue and oversight differences |

**Table 3:** Engineered features.

---

## 5. Predictor Sets

Three predictor sets of increasing complexity were specified to allow systematic comparison of model performance and complexity trade-offs.

| Set | Contents | Purpose |
|---|---|---|
| Benchmark | BMI, age, ASA status dummies, baseline comorbidity indicators, Charlson score | Simple interpretable baseline |
| Extended | Benchmark + AHRQ CCS dummies, weekend surgery, gender, CCS complication rate, race dummies, comorbidity count | Tests whether procedure type and demographics improve prediction |
| Kitchen sink | Extended + complication RSI + mortality RSI | Tests whether pre-built risk indices add value beyond clinical predictors |

**Table 4:** Predictor set design.

The kitchen-sink models include `complication_rsi` and `mortality_rsi`, which are outputs of existing Cleveland Clinic risk models built on ICD billing codes from the year prior to admission. Including these is somewhat circular — essentially asking whether a model that predicts complications can predict complications. The extended set was preferred for the final model as it tests what raw clinical and procedural variables can achieve without relying on pre-built scores.

---

## 6. Modeling Strategy

Four learner types were evaluated across the predictor sets:

- **Logistic regression (GLM)** — simple interpretable benchmark.
- **Lasso** — logistic regression with L1 regularization to shrink less useful coefficients.
- **Random forest** — captures non-linear relationships and variable interactions.
- **XGBoost** — gradient-boosted trees, often strong on tabular predictive tasks.

| Modeling Detail | Value |
|---|---|
| Cross-validation folds | 5, stratified by complication |
| Hyperparameter grid size | 5 (balance between search quality and compute) |
| Tuning metric | PR-AUC (appropriate for imbalanced positive class) |
| Learners specified | GLM on benchmark; GLM, Lasso, RF on extended; Lasso, RF, XGBoost on kitchen sink |

**Table 5:** Cross-validation and tuning parameters.

---

## 7. Validation Findings

Seven learners were trained and compared. Random forest and XGBoost showed the widest predicted probability ranges and clearest class separation. Lasso produced a much narrower probability range and weaker discrimination. Logistic regression was a useful middle-ground but did not match the tree-based models.

The selected validation model was `random_forest_predSet_extended`. The kitchen-sink models were competitive but the extended random forest offered a strong balance of performance and complexity without relying on the additional risk-index variables.

A classification threshold of 0.25 was selected because the clinical cost of missing a true complication (false negative) is considered higher than the cost of an unnecessary alert (false positive). At 0.25 the model flags more patients as high-risk, catching more true complications at the cost of some additional false alarms.

| Metric | Value | Metric | Value |
|---|---:|---|---:|
| PR-AUC | 0.865 | F1 score | 0.734 |
| ROC-AUC | 0.914 | Accuracy | 0.857 |
| Sensitivity (TPR) | 0.782 | Threshold | 0.25 |
| Specificity (TNR) | 0.883 | False positive rate | 0.117 |
| Precision (PPV) | 0.692 | False negative rate | 0.218 |

**Table 6:** Selected validation model performance — `random_forest_predSet_extended` at threshold 0.25.

---

## 8. Test-Set Evaluation

The final model was applied to the held-out test set. Performance was stable and marginally higher than validation across all key metrics, providing no evidence of overfitting or performance degradation on unseen data.

| Metric | Validation | Test set | Change |
|---|---:|---:|---:|
| ROC-AUC | 0.914 | 0.925 | +0.011 |
| PR-AUC | 0.865 | 0.876 | +0.011 |
| Sensitivity (TPR) | 0.782 | 0.785 | +0.003 |
| False positive rate | 0.117 | 0.113 | −0.004 |
| Precision (PPV) | 0.692 | 0.701 | +0.009 |
| F1 score | 0.734 | 0.741 | +0.007 |

**Table 7:** Validation vs. test-set performance comparison.

In plain terms: of patients who truly experienced a complication, the model correctly identified approximately 4 in 5. Of patients flagged as high-risk, approximately 7 in 10 genuinely experienced a complication. The small improvement from validation to test suggests the model generalises well to the held-out population.

---

## 9. Fairness Assessment

Fairness was assessed across gender and race indicator variables. Disparities observed during validation were confirmed on the test set. The most consistent and statistically significant issue was unequal false positive rates across demographic groups.

A higher false positive rate for a subgroup means patients in that group are more likely to be unnecessarily flagged as high-risk, potentially leading to extra monitoring, additional interventions, and increased resource use without clinical benefit. A lower true positive rate means patients in that group are less likely to receive preventive attention when they genuinely need it. Both types of disparity can erode patient trust and affect quality of care.

| Variable | Category | TPR | FPR | PPV | FNR |
|---|---:|---:|---:|---:|---:|---:|
| Gender | 0 | 0.757 | 0.150 | 0.677 | 0.243 |
| Gender | 1 | 0.816 | 0.085 | 0.728 | 0.184 |
| Race 0 | 0 | 0.790 | 0.107 | 0.712 | 0.210 |
| Race 0 | 1 | 0.742 | 0.163 | 0.618 | 0.258 |
| Race 1 | 0 | 0.753 | 0.179 | 0.625 | 0.247 |
| Race 1 | 1 | 0.791 | 0.103 | 0.716 | 0.209 |
| Race 2 | 0 | 0.785 | 0.109 | 0.704 | 0.215 |
| Race 2 | 1 ⚠ | 0.783 | **0.244** | 0.643 | 0.217 |

**Table 8:** Test-set fairness metrics at threshold 0.25. The highlighted row (Race 2, category 1) shows an FPR of 0.244 — more than double the overall rate of 0.113.

**Note:** Race is coded as 0/1/2 and gender as 0/1 in the source dataset. The data dictionary does not provide labels for these codes, and the dataset is heavily skewed (approximately 92% of patients are coded as race = 1). The clinical and data teams must obtain the original source documentation to identify which real-world demographic groups these codes represent before the fairness findings can be fully interpreted.

---

## 10. Final Recommendation

From a predictive performance perspective, the random forest extended model is strong. It achieves high ROC-AUC and PR-AUC, stable validation-to-test performance, and useful sensitivity at the selected threshold. However, the fairness disparities described above raise important ethical and practical concerns that preclude deployment in its current form.

The model should not be deployed as an autonomous or production clinical decision tool. It may be appropriate for further research, retrospective analysis, or carefully supervised pilot testing, but only with the following safeguards in place.

### Recommended next steps

- **Clarify demographic group labels.** The data dictionary does not specify what race codes 0, 1, and 2 represent. This must be resolved before the fairness findings can be acted on.
- **Investigate root causes of FPR disparity.** Understand whether the disparities are driven by differences in case mix, procedure type, or other systematic factors.
- **Apply fairness-aware threshold calibration.** Group-specific thresholds can equalize FPR across subgroups. This is the lowest-cost first step and should be evaluated before more involved model changes.
- **Consider fairness-constrained model training.** Libraries such as Fairlearn can enforce FPR parity as a constraint during training, potentially achieving a better overall performance/fairness trade-off.
- **Benchmark against current standard of care.** The model has not been compared against existing risk tools such as NSQIP scores or clinical judgment. This comparison is necessary to determine whether the model represents a genuine improvement.
- **Add interpretability tooling.** Feature importance plots, SHAP values, and partial dependence plots would support clinical review and help identify unexpected drivers of risk scores.
- **Conduct clinical validation.** The model must be reviewed with clinical staff before any operational use. A monitored retrospective pilot should precede any prospective deployment.
- **Address regulatory and legal considerations.** The use of race and gender as model inputs in a clinical decision-support tool may require legal review under applicable health equity frameworks.

---

## 11. Conclusion

This project successfully developed and evaluated a predictive model for post-surgical complications. The final model — a random forest using the extended predictor set at a threshold of 0.25 — demonstrates strong discrimination and generalises well to held-out data, achieving a test ROC-AUC of 0.925 and PR-AUC of 0.876.

The selected threshold reflects the clinical priority of minimising missed complications. At this threshold the model identifies approximately 4 in 5 true complications with a false positive rate of 11.3%.

However, statistically significant fairness disparities — particularly in false positive rates across gender and race subgroups — limit the model's readiness for clinical deployment. The most notable disparity is in the race 2, category 1 subgroup, where the false positive rate of 0.244 is more than double the overall rate.

The overall conclusion is that the model is technically promising and the fairness concerns were identified and reported transparently. With focused remediation — demographic label resolution, fairness-aware threshold calibration or constrained retraining, interpretability enhancements, and clinical validation — this model could be a candidate for a closely monitored prospective pilot.

---

## Appendix A: Variable Reference

| Variable | Type | Description | Used as predictor |
|---|---|---|---|
| `bmi` | Numeric | Body mass index | Yes (benchmark+) |
| `age` | Numeric | Patient age | Yes (benchmark+) |
| `asa_status` | Categorical | ASA physical status (0/1/2) | Yes — dummy encoded |
| `baseline_cancer` | Binary | Pre-existing cancer | Yes (benchmark+) |
| `baseline_charlson` | Numeric | Charlson Comorbidity Index score | Yes (benchmark+) |
| `baseline_cvd` | Binary | Pre-existing cardiovascular disease | Yes (benchmark+) |
| `baseline_dementia` | Binary | Pre-existing dementia | Yes (benchmark+) |
| `baseline_diabetes` | Binary | Pre-existing diabetes | Yes (benchmark+) |
| `baseline_digestive` | Binary | Pre-existing digestive condition | Yes (benchmark+) |
| `baseline_osteoart` | Binary | Pre-existing osteoarthritis | Yes (benchmark+) |
| `baseline_psych` | Binary | Pre-existing psychiatric condition | Yes (benchmark+) |
| `baseline_pulmonary` | Binary | Pre-existing pulmonary condition | Yes (benchmark+) |
| `ahrq_ccs` | Categorical | AHRQ CCS procedure category | Yes — dummy encoded (extended+) |
| `ccs_complication_rate` | Numeric | Historical complication rate for CCS category | Yes (extended+) |
| `ccs_mort30rate` | Numeric | Historical 30-day mortality rate for CCS category | No |
| `complication_rsi` | Numeric | Cleveland Clinic complication risk stratification index | Yes (kitchen sink only) |
| `mortality_rsi` | Numeric | Cleveland Clinic mortality risk stratification index | Yes (kitchen sink only) |
| `dow` | Categorical | Day of week of surgery (0–6) | Indirectly via `weekend_surgery` |
| `gender` | Binary | Gender (0/1, unlabelled) | Yes (extended+) |
| `hour` | Numeric | Hour of surgery | Indirectly via `night_surgery` |
| `month` | Categorical | Month of surgery (0–11) | No |
| `moonphase` | Categorical | Moon phase (0–3) | No |
| `mort30` | Binary | 30-day mortality — post-operative outcome | Excluded |
| `race` | Categorical | Race (0/1/2, unlabelled; 92% coded as 1) | Yes — dummy encoded (extended+) |
| `complication` | Binary | Target: post-surgical complication (1=yes) | Outcome variable |

**Table A1:** Complete variable reference.

---

## Appendix B: Visual Evidence from Milestones

The following figures were added from the milestone analysis to support the report findings visually. They are kept in an appendix so the main report remains readable for non-technical stakeholders, while still giving reviewers evidence for model selection, discrimination, calibration, and threshold trade-offs.

### Figure B1. Validation predicted probability distributions by learner

![Figure B1. Validation predicted probability distributions by learner](assets/figure-b1.png)

**Stakeholder takeaway:** Models with a wider spread of risk scores, especially random forest and XGBoost, separate low-risk and high-risk patients more clearly than models whose scores cluster tightly.

### Figure B2. Validation predicted probabilities split by actual outcome

![Figure B2. Validation predicted probabilities split by actual outcome](assets/figure-b2.png)

**Stakeholder takeaway:** The selected tree-based models show clearer separation between patients with and without complications, supporting the choice of a random forest model.

### Figure B3. ROC curves by prediction set

![Figure B3. ROC curves by prediction set](assets/figure-b3.png)

**Stakeholder takeaway:** Curves closer to the top-left indicate better discrimination. The extended and kitchen-sink tree-based models perform better than the simple benchmark model.

### Figure B4. Precision-recall curves by prediction set

![Figure B4. Precision-recall curves by prediction set](assets/figure-b4.png)

**Stakeholder takeaway:** Precision-recall is important because complications are the minority class. This view focuses more directly on how well the model identifies true complication cases.

### Figure B5. Calibration plot across learners

![Figure B5. Calibration plot across learners](assets/figure-b5.png)

**Stakeholder takeaway:** Calibration checks whether predicted risks behave like real probabilities. The selected model is reasonably aligned but still needs clinical calibration review before deployment.

### Figure B6. Classification performance across thresholds

![Figure B6. Classification performance across thresholds](assets/figure-b6.png)

**Stakeholder takeaway:** The threshold choice controls the trade-off between catching more true complications and creating more false alerts. The selected 0.25 threshold favors sensitivity, which fits a patient-safety use case.
