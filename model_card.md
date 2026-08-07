# Model Card

## Model Details
- Model developed: Ryan Monemian
- Model date: August 2026
- Model version: v1.0 
- Model type: `RandomForestClassifier` (scikit-learn), n_estimators=100, binary classification. Mitigated version wraps the same classifier in Fairlearn's `ExponentiatedGradient` reduction.
- Information about training algorithms, parameters, fairness constraints, or other applied approaches, and features: Baseline model trained with `sex`, `age`, `priors_count`, and `c_charge_degree` as features (one-hot encoded where categorical). Mitigated model trained using Fairlearn's `ExponentiatedGradient` with a `DemographicParity` constraint (difference_bound=0.01), wrapped around the same `RandomForestClassifier` framework.
- License: MIT License
- Questions or comments: Open an issue on the GitHub repository.

## Works Cited:
Larson, Jeff, et al. "Compas-Analysis." GitHub, ProPublica, 2016, 
github.com/propublica/compas-analysis.  

Angwin, Julia, et al. "How We Analyzed the COMPAS Recidivism Algorithm." 
ProPublica, 23 May 2016, 
www.propublica.org/article/how-we-analyzed-the-compas-recidivism-algorithm.

## Intended Use
- Primary intended uses: Auditing and research, demonstrating a bias-detection methodology, and potentially reusable to check whether similar disparities appear in newer datasets. **NOT** for generating real risk scores for defendants.
- Primary intended users: People studying algorithmic fairness, researchers, students, portfolio reviewers. Not judges, not courts, not anyone making real decisions about real defendants.
- Out-of-scope use cases: Not suitable for generating actual risk scores to inform bail, sentencing, parole, or reoffense predictions for real individuals. Using this model this way would violate principles of human-centered AI design, since even a model with a smaller measured bias than COMPAS's own deployed tool still carries meaningful racial disparity, and applying it to real decisions risks legitimizing that bias under the appearance of algorithmic objectivity.

## Factors
Factors could include demographic or phenotypic groups, environmental conditions, technical attributes, or others.

- Relevant factors: Race, as well as sex and age (both included as model features but not separately evaluated for fairness in this project). 
- Evaluation factors: African-American and Caucasian, the two groups with enough sample size (n=800 and n=521 respectively) for statistically meaningful comparison. Other race categories present in the data (Asian, Hispanic, Native American, Other) had sample sizes too small (n=5 to n=128) to make reliable conclusions.

Race was chosen as the primary evaluation factor because it was the central axis of ProPublica's original COMPAS investigation and the documented axis of concern in criminal justice risk assessment more broadly. Sex and age were included as model features but were outside this project's evaluation scope.

## Metrics
- Model performance measures: Accuracy, false positive rate, true positive rate, selection rate, demographic parity difference, and equalized odds difference. FPR, DPD, and EOD were the primary focus since they directly measure error and prediction-rate disparities across groups. Which is the actual central focus of this audit rather than just overall correctness. 
- Decision thresholds: My models used scikit-learn's default probability threshold of 0.5. COMPAS's score_text was converted to binary using a category threshold: "Medium" or "High" as positive, "Low" as negative. 
- Variation approaches: Trained two versions of the same RandomForestClassifier. One was a normal baseline model with no fairness constraint. The other was a mitigated version, trained using Fairlearn's ExponentiatedGradient with a DemographicParity constraint. Comparing these two versions shows the tradeoff between accuracy and fairness.

## Evaluation Data
Details on the dataset(s) used for the quantitative analyses in the card.

- Datasets: ProPublica's COMPAS recidivism dataset (compas-scores-two-years.csv), covering defendants in Broward County, Florida from 2013 to 2014. 
- Motivation: This is the dataset used in ProPublica's original 2016 investigation into racial bias in the COMPAS algorithm. It is a fixed, well documented dataset, so results from this project can be checked against ProPublica's own published findings and other academic work that has used the same data.
- Preprocessing: 
  1. Reviewed all columns and checked for missing or duplicate values.
  2. Applied ProPublica's own filtering criteria. I excluded cases where the charge date was more than 30 days from the arrest date, cases with no matched COMPAS record (is_recid == -1), and minor ordinance violations (c_charge_degree == 'O').
  3. Dropped columns representing post-outcome recidivism data (target leakage), duplicate columns, and identifier columns like `name` and `id`.
  
  This reduced the dataset from 7,214 rows to 6,172 rows, and from 53 columns 
  to 35 columns.

## Training Data 
Same source and preprocessing as Evaluation Data above. The cleaned dataset was split into training and validation sets using scikit-learn's `train_test_split` (default 75/25 split, fixed `random_state` for reproducibility). Only the training portion was used to fit the models.

## Quantitative Analyses

### Unitary Results

Race was the only single feature evaluated in this project. Metrics were broken down by race using Fairlearn's `MetricFrame`. Two groups had large enough sample sizes for reliable results: African American (n=800) and Caucasian (n=521).

| Metric | My Baseline Model | My Mitigated Model | COMPAS's Score |
|---|---|---|---|
| Accuracy | 64.4% | 62.2% | 65.5% |
| False positive rate (African American) | 36.3% | 36.8% | 42.1% |
| False positive rate (Caucasian) | 20.2% | 24.3% | 20.6% |
| Demographic parity difference | 0.189 | 0.150 | 0.259 |
| Equalized odds difference | 0.161 | 0.125 | 0.225 |

The false positive rate gap was large and consistent across all three models. This means African American defendants who did not reoffend were misclassified as high risk much more often than Caucasian defendants who also did not reoffend. This pattern held even for the group with the largest sample size (n=800), which makes the finding more credible than a result based on a small group.

### Intersectional Results

This project did not evaluate combinations of factors, such as race and sex together. Combining factors would have likely reduced sample sizes further and make some subgroups too small to evaluate reliably. This is also noted as a limitation in the Caveats section below.

## Ethical Considerations
The dataset itself raises ethical concerns beyond the model. Everyone in this dataset is here because they were arrested, and arrests are not a neutral or evenly applied measurement of behavior. Communities with heavier police presence tend to produce more arrests for similar underlying behavior, and policing has a documented history of racial disparity. This means a feature like `priors_count`, used in this model, may partly reflect how heavily a person's community was policed, not only their own individual actions. Even though race was excluded as a direct input, this kind of feature can carry some of that same historical bias indirectly. 

This concern is supported by a permutation importance analysis of the model. `priors_count` had by far the highest importance score (0.105) of any feature, meaning the model depends on it more than any other input to make correct predictions. `age` had some importance (0.035), while `sex` and `c_charge_degree` had almost no measurable effect. This does not prove the model is biased through this feature alone, but it does confirm that `priors_count` is the main driver of the model's decisions, which strengthens the concern that uneven policing history may be shaping the model's predictions even without race being used directly.

Beyond the modeling choices, using a system like this to influence real decisions removes human judgment and accountability from decisions that seriously affect someone's freedom, without any guarantee the underlying data itself is a fair or accurate reflection of that person's actual behavior.

## Caveats and Recommendations
- Some race categories in the dataset (Asian, Native American) had very small sample sizes and were excluded from fairness comparisons. Results might look different with more data for these groups. 
- This project only evaluated race on its own. Sex and age were used as model features, but not evaluated as fairness factors, and intersectional combinations (like race and sex together) were not tested.
- Permutation importance was run to check whether `priors_count` was doing a large share of the model's predictive work. See Ethical Considerations for the result.
- This dataset reflects arrests in Broward County, Florida from 2013 to 2014. Findings may not reflect how COMPAS or similar tools perform today, or in other locations.
