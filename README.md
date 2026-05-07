# Student Academic Performance Prediction

![R](https://img.shields.io/badge/R-4.2-blue)
![caret](https://img.shields.io/badge/caret-6.0-green)
![glmnet](https://img.shields.io/badge/glmnet-4.1-orange)
![randomForest](https://img.shields.io/badge/randomForest-4.7-red)

## 📋 Project Overview

**Objective:** Predict student pass/fail outcomes using behavioral, contextual, and academic predictors to support early academic intervention.

**Dataset:** 10,000 student records from Kaggle  
**Context:** Academic project (DAT402)  
**Date:** April 2026

**Contributors:**
- **Willis Dongmo** – Data analysis, modeling, evaluation 
- **Emily Chiu** – Literature review, model tuning adjustments, explanatory videos

---

## 🎯 Business Problem

Educational institutions need predictive tools to:
- **Identify at-risk students** before final exams
- **Allocate resources** (tutoring, counseling) effectively
- **Understand key drivers** of academic success

This project compares two machine learning approaches:
1. **Regularized Logistic Regression (LASSO)** – Interpretable linear classifier
2. **Random Forest** – Non-linear ensemble model

---

## 📊 Dataset

- **Source:** [Kaggle Student Academic Performance Dataset](https://www.kaggle.com/datasets/sadiajavedd/student-academic-performance)
- **Size:** 10,000 students
- **Features:** 23 variables (18 used for modeling)
- **Target:** Binary (Pass/Fail)

### Feature Categories

**Behavioral:**
- Study hours per day
- Attendance rate
- Sleep hours
- Social media hours
- Assignment completion rate
- Participation score

**Contextual:**
- Gender
- Parental education
- Family income
- Internet access
- Study environment
- Tutoring (Yes/No)

**Academic:**
- Math score
- Reading score
- Writing score
- Science score
- Online courses completed

### Excluded Variables (Data Leakage)
❌ `final_exam_score` – Direct outcome proxy  
❌ `previous_gpa` – Too strongly correlated with pass/fail  
❌ `grade_category` – Encodes final outcome  
❌ `student_id` – No predictive value  
❌ `age` – Unrealistic range in dataset  

---

## 🔧 Methodology

### 1. Data Preprocessing

#### Missing Values
- **Result:** 0 missing values detected
- **Action:** No imputation required

#### Outlier Screening
- **Strategy:** Range validation for bounded variables
- **Screened variables:** Study hours, sleep hours, social media hours
- **Decision:** All values within plausible ranges → retained

#### Feature Engineering
- Conversion of categorical variables to factors
- One-hot encoding for Logistic Regression
- Feature scaling (z-score standardization)

### 2. Train/Test Split
- **Strategy:** 75/25 stratified split
- **Preservation:** Pass/fail ratio maintained
- **Train set:** 7,501 students
- **Test set:** 2,499 students

### 3. Model Training

#### **Model 1: LASSO Logistic Regression**
```r
cv.glmnet(
    x = X_scaled,
    y = y_binary,
    family = "binomial",
    alpha = 1,           # LASSO penalty
    nfolds = 10          # Cross-validation
)
```

**Hyperparameter tuning:** 10-fold CV to select optimal λ (regularization strength)  
**Best λ:** 0.000619

#### **Model 2: Random Forest**
```r
randomForest(
    pass_fail ~ .,
    data = train_data,
    mtry = 2,            # Tuned via CV
    importance = TRUE
)
```

**Hyperparameter tuning:** Grid search over `mtry` (2, 6, 11, 16, 21)  
**Best mtry:** 2 (optimal balance between tree diversity and accuracy)

---

## 📈 Results

### Model Comparison

| Metric | Logistic Regression | Random Forest |
|--------|---------------------|---------------|
| **Accuracy** | **94.36%** | 94.32% |
| **Balanced Accuracy** | **94.37%** | 94.33% |
| **Sensitivity** | **94.09%** | 94.01% |
| **Specificity** | **94.65%** | 94.65% |
| **AUC-ROC** | **0.9904** | 0.9875 |
| **Kappa** | 0.8871 | 0.8863 |

**Pass detection rate:** 94.65% of actual passes correctly identified  
**Fail detection rate:** 94.09% of actual failures correctly identified

---

## 🎯 Key Findings

### Variable Importance (Random Forest)

**Top 5 Predictors:**
1. **Reading score** (highest importance)
2. **Science score**
3. **Writing score**
4. **Math score**
5. **Study hours per day**

**Weaker predictors:**
- Tutoring
- Internet access
- Gender
- Family income
- Study environment

### Insights from EDA

✅ **Study hours:** Pass students study ~3.6 hrs/day vs ~2.5 hrs/day (fail)  
✅ **Attendance:** Pass students ~87% vs ~84% (fail)  
✅ **Social media:** Fail students use ~3 hrs/day vs ~2.3 hrs/day (pass)  
✅ **Math score:** Strongest individual separator (median 60 vs 37)  

⚠️ **Tutoring paradox:** No strong isolated effect on pass rate  
⚠️ **Internet access:** Minimal difference between groups  

---

## 🏆 Model Selection: Logistic Regression

**Why Logistic Regression is preferred despite near-identical performance:**

1. ✅ **Interpretability:** Coefficients directly show feature impact
2. ✅ **Simplicity:** Easier to deploy and explain to stakeholders
3. ✅ **Transparency:** Clear decision boundary
4. ✅ **Slightly better AUC:** 0.9904 vs 0.9875

**When Random Forest might be better:**
- If non-linear interactions are critical
- If interpretability is not a priority
- If overfitting is a concern (ensemble averaging)

---

## 🚀 Technical Implementation

### Installation

```bash
# Clone repository
git clone https://github.com/willisdongmo/student-performance-prediction.git
cd student-performance-prediction

# Open in RStudio
# Or run from command line
```

### Requirements

```r
# Install required packages
install.packages(c(
    "here",
    "caret",
    "glmnet",
    "randomForest",
    "ggplot2",
    "tidyverse",
    "corrplot",
    "pROC"
))
```

### Usage

```r
# Run complete analysis
source("student_performance_prediction.R")

# Or knit R Markdown
rmarkdown::render("SAPA.Rmd")
```

---

---

## 📊 Visualizations Included

1. **Target distribution** (Pass vs Fail)
2. **Boxplots:** Study hours, attendance, social media by outcome
3. **Proportional bar charts:** Tutoring, internet access effects
4. **Correlation heatmap** (numerical features)
5. **ROC curves** (both models)
6. **Variable importance plot** (Random Forest)

---

## 💡 Limitations & Future Work

### Limitations

1. **Dataset characteristics:**
   - Academic scores highly correlated with outcome (makes task relatively easy)
   - No temporal dimension (cannot track student progress over time)
   - Synthetic/cleaned data may not reflect real-world noise

2. **Model limitations:**
   - Very high performance suggests possible data leakage risk
   - Subject scores may proxy for final outcome too closely
   - External validity unknown (single institution/context)

### Future Improvements

1. **Early prediction:** Use only behavioral/contextual features (exclude subject scores)
2. **Temporal modeling:** Predict outcome at mid-semester using early data
3. **Intervention simulation:** A/B test framework for tutoring effectiveness
4. **Fairness analysis:** Ensure no discrimination by gender, income, parental education
5. **Ensemble methods:** Combine Logistic Regression + Random Forest predictions
6. **Calibration:** Adjust predicted probabilities for decision thresholds

---

## 📚 References

- **Dataset:** [Kaggle Student Academic Performance](https://www.kaggle.com/datasets/sadiajavedd/student-academic-performance)
- **LASSO:** Tibshirani (1996) – Regression Shrinkage and Selection via the Lasso
- **Random Forest:** Breiman (2001) – Random Forests
- **Imbalanced Learning:** caret package documentation

---

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---

## 👥 Authors

**Primary Author: Willis Dongmo**  
- Data analysis, preprocessing, modeling, evaluation
- GitHub: [@willisdongmo](https://github.com/willisdongmo)  
- LinkedIn: [Willis Dongmo](https://linkedin.com/in/willisdongmo)

**Contributing Author: Emily Chiu**  
- Literature review, model tuning adjustments, explanatory videos

---

## 🙏 Acknowledgments

- Kaggle for providing the dataset
- Academic context: DAT402 coursework
- Emily Chiu for collaborative contributions

---

**⭐ If you found this project useful, please consider giving it a star!**
