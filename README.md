# Employee Performance Prediction (R, Decision Tree)

**In one line:** I helped build a decision tree model in R that predicts whether an employee is a **High** or **Normal** performer, and used it to find which factors best separate the two groups.

---

## The problem

Spotting strong performers early helps a company decide who to develop, promote, and keep. Performance reviews happen only once in a while, so it is useful to know whether everyday workforce and engagement data can point to high performers too. This project asks two questions:

1. Can we predict which employees are rated as High performers?
2. Which factors most clearly separate High performers from Normal performers?

## What I did

1. **Explored the data.** I reviewed 700 employee records and confirmed that no values were missing. About 21% of employees (148 of 700) are rated High.
2. **Visualized key factors.** I compared satisfaction score, leadership score, number of projects, and training hours between High and Normal performers using boxplots.
3. **Held data back for testing.** I trained the model on 80% of the employees (560) and tested it on the other 20% (140), which the model never saw during training.
4. **Built a classification tree.** A decision tree is easy to explain: it splits employees into groups using simple yes/no rules (for example, "satisfaction score of 7.5 or higher").
5. **Evaluated and interpreted it.** I measured performance with a confusion matrix, checked which variables mattered most, and tested whether pruning the tree would improve it.

## Key results

The test set has 140 employees: 113 Normal and 27 High performers.

| Metric (test set, 140 employees) | Value |
|---|---:|
| **Accuracy** | **93.57%** (95% CI: 88.15%–97.02%) |
| No-information rate ("predict everyone is Normal") | 80.71% |
| Kappa | 0.80 |
| Normal performers correctly identified (sensitivity) | 107 of 113 (94.69%) |
| High performers correctly identified (specificity) | 24 of 27 (88.89%) |
| When the model predicts "High," it is right | 24 of 30 (80.0%) |
| Balanced accuracy | 91.79% |

**Confusion matrix** (rows = predicted, columns = actual):

| | Actual Normal | Actual High |
|---|---:|---:|
| **Predicted Normal** | 107 | 3 |
| **Predicted High** | 6 | 24 |

**How to read sensitivity and specificity here:** the analysis treats **Normal (0) as the positive class**. So *sensitivity* is the share of Normal performers the model got right, and *specificity* is the share of **High** performers it got right.

**A note on the baseline:** because about 81% of the test employees are Normal performers, a model that labeled everyone "Normal" would already score 80.71% accuracy. The tree's 93.57% accuracy is significantly better than that baseline (p < 0.001), and it correctly found 24 of the 27 High performers.

**Variable importance** (from the decision tree):

| Variable | Importance |
|---|---:|
| SatisfactionScore | 62.7 |
| LeadershipScore | 51.4 |
| NumProjects | 29.0 |
| TrainingHours | 9.5 |
| WorkHoursPerWeek | 6.6 |
| Age | 4.3 |
| YearsExperience | 3.8 |
| Department | 2.7 |
| EducationLevel | 2.4 |
| PromotionLast3Years | 0.9 |

## What the model found

- **Three factors drive the model.** The final tree uses only **satisfaction score**, **leadership score**, and **number of projects** to make its splits. The other variables add little or nothing.
- **Satisfaction comes first.** The tree's first split is a satisfaction score of 7.5 or higher. In the training data, about 54% of employees above that line were High performers, compared with about 12% below it.
- **Workload and leadership sharpen the picture.** Among highly satisfied employees, those handling 7 or more projects were mostly High performers (about 81% in training). Higher leadership scores pushed the share higher still.
- **Lower satisfaction, fewer projects usually means Normal.** Employees below the satisfaction cutoff who handled fewer than 7 projects were almost all rated Normal.

> These are patterns in this dataset, not proof that raising satisfaction or adding projects causes higher performance. High performers may simply be given more projects, for example.

## Business recommendations

These are starting points for an HR team to explore, not tested interventions:

- **Track satisfaction regularly.** It was the strongest signal separating High from Normal performers.
- **Look at leadership ability** when identifying people for development or succession planning.
- **Review how projects are assigned.** Project load was closely tied to performance rating, so it is worth checking whether opportunities are spread fairly.
- **Use the model as a screening aid**, not an automatic decision tool. It still mislabeled 6 Normal performers as High and missed 3 High performers in the test set.

## Tools & methods

| Area | What I used |
|---|---|
| Language | R, R Markdown |
| Data split | `caret::createDataPartition` (80/20, `set.seed(123)`) |
| Model | Classification tree (`rpart`, `method = "class"`) |
| Evaluation | `caret::confusionMatrix` (accuracy, 95% CI, kappa, sensitivity, specificity) |
| Interpretation | `rpart` variable importance; tree plots with `rattle::fancyRpartPlot` and `rpart.plot` |
| Pruning | Cost-complexity pruning, choosing the CP with the lowest cross-validated error (`xerror`) |

**Note on pruning:** the lowest cross-validated error came at CP = 0.01, the smallest value tried, which is the full tree. Pruning therefore chose the same CP as the full tree and returned the same model, with an identical confusion matrix. Pruning did not change the results.

## Repository structure

```
├── Employee_Performance_Prediction.Rmd     # Full analysis (R Markdown)
├── data/
│   └── employee_performance.csv            # 700 employees, 13 variables
├── reports/
│   └── Employee_Performance_Prediction.html  # Knitted report with outputs and charts
├── .gitignore
└── README.md
```

## How to reproduce

1. Install R and these packages:
   ```r
   install.packages(c("caret", "rpart", "rattle", "rpart.plot", "rmarkdown"))
   ```
2. Clone the repository:
   ```bash
   git clone https://github.com/rajvi669/Employee-Performance-Prediction.git
   ```
3. Set the project folder as your working directory. The R Markdown file reads `data/employee_performance.csv`, so no files need to be moved.
4. Open `Employee_Performance_Prediction.Rmd` in RStudio and click **Knit**, or run:
   ```r
   rmarkdown::render("Employee_Performance_Prediction.Rmd", output_dir = "reports")
   ```
   With `set.seed(123)`, you should get the results shown above.

## About the dataset

- **Size:** 700 employees and 13 variables, with no missing values.
- **Outcome:** `PerformanceRating` (0 = Normal, 1 = High). 148 employees (about 21%) are High performers and 552 are Normal.
- **Variables:** Age, EducationLevel, Department, YearsExperience, WorkHoursPerWeek, Overtime, NumProjects, TrainingHours, SatisfactionScore, PromotionLast3Years, LeadershipScore, Salary, and PerformanceRating. Salary is in the file but was not used in the model.
- **Source:** not documented. The data may be synthetic or created for teaching, so it should **not** be treated as real company data. The findings show the method working, not real-world workforce facts.

## Limitations & next steps

**Limitations**

- **Class imbalance.** Only about 21% of employees are High performers, so accuracy alone overstates how well the model works. The no-information rate is already 80.71%.
- **One train/test split.** The results come from a single 140-employee test set (27 High performers), so they could shift with a different split.
- **No cross-validated evaluation.** Cross-validation was used only inside `rpart` to choose the pruning level. The model's performance was not checked with k-fold cross-validation.
- **Default settings.** The tree uses `rpart`'s default settings, with no further tuning.
- **Undocumented data.** The findings may not carry over to a real organization.

**Next steps**

- Use k-fold cross-validation to get a more stable estimate of performance.
- Handle the imbalance with class weights or resampling, and report ROC-AUC and precision-recall.
- Compare the tree with other models, such as logistic regression or random forest.
- Test the approach on real, well-documented HR data.

## Author

**Rajvi Soni**, contributor to this group project at Kent State University (MS Business Analytics)
LinkedIn: [linkedin.com/in/sonirajvi18](https://www.linkedin.com/in/sonirajvi18)
