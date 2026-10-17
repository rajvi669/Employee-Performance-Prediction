# Employee Performance Prediction (R)

A classification-tree analysis in R Markdown that predicts whether an employee is a **High** or **Normal** performer.

## Business question
Can we flag high performers from workforce and engagement data, and which factors distinguish "High" from "Normal" performers?

## Dataset
- **File:** `data/employee_performance.csv`
- **Size:** 700 employees, 13 variables, no missing values.
- **Target:** `PerformanceRating` (0 = Normal, 1 = High): 552 Normal vs 148 High (~21% High performers).
- **Predictors used:** Age, EducationLevel, Department, YearsExperience, WorkHoursPerWeek, Overtime, NumProjects, TrainingHours, SatisfactionScore, PromotionLast3Years, LeadershipScore. (The file has one further numeric column that the model does not use.)
- **Source:** not documented. The data may be synthetic or educational; it should **not** be treated as real company data, and results should not be generalised to real workforces.

## Method
- Classification tree (`rpart`, `method = "class"`).
- 80/20 stratified train/test split with `caret::createDataPartition` (seed 123): 560 train / 140 test.
- Evaluation with a confusion matrix on the test set; variable importance from the tree; cost-complexity pruning (CP) via cross-validated `xerror`.

## Results (test set, n = 140)
| Metric | Value |
|---|---|
| Accuracy | **93.57%** (95% CI 88.15–97.02%) |
| No-information rate | 80.71% |
| Kappa | 0.802 |
| Sensitivity | 94.69% |
| Specificity | 88.89% |

Sensitivity and specificity use **Normal (0)** as the positive class, so sensitivity is the share of Normal performers correctly identified and specificity is the share of High performers correctly identified.

Confusion matrix (rows = predicted, columns = actual):

| | Actual Normal | Actual High |
|---|---|---|
| Pred Normal | 107 | 3 |
| Pred High | 6 | 24 |

**Most important variables:** SatisfactionScore, LeadershipScore, NumProjects (then TrainingHours, WorkHoursPerWeek, and smaller contributions from the rest).

**Note on pruning:** the CP with the lowest cross-validated error was 0.01, the smallest CP tried, which is the full tree. Pruning therefore returned the same tree, and the pruned and unpruned confusion matrices are identical. Pruning did not improve this model.

## How to reproduce
1. Install R and RStudio (optional).
2. Install packages:
   ```r
   install.packages(c("caret", "rpart", "rattle", "rpart.plot"))
   ```
3. Open the project folder as the working directory (so `data/employee_performance.csv` resolves), then:
   ```r
   rmarkdown::render("Employee_Performance_Prediction.Rmd", output_dir = "reports")
   ```

## Structure
```
employee-performance-prediction/
├── Employee_Performance_Prediction.Rmd
├── README.md
├── .gitignore
├── data/
│   └── employee_performance.csv
└── reports/
    └── Employee_Performance_Prediction.html
```

## Author
Rajvi Soni — contributor; completed as a group project at Kent State University (MS Business Analytics)
