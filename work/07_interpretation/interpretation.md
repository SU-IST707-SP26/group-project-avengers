# Milestone 7: Interpretation & Implications

## Part A: Stakeholder Interpretation

Taxi trip duration and congestion-fee exposure are practical forecasting problems. Drivers, fleet operators, dispatch systems, riders, and transportation agencies all benefit from knowing whether a trip is likely to take longer than expected or trigger a congestion-related charge. The evaluation results show that simple baseline models provide useful reference points, but the strongest results come from ensemble methods that can learn nonlinear relationships among distance, location, time of day, airport activity, and route context.

One comparison caveat applies across the results: most baseline and classification models were evaluated on a 500,000-trip sample, while the tuned XGBoost duration model used a 150,000-trip stratified sample with a separate validation split and early stopping. Some Logistic Regression comparisons also used balanced class weights, so the logistic accuracy and F1 values differ by setup. The ranking of models is still clear, but very small differences should not be overinterpreted as perfectly apples-to-apples comparisons.

### 1. Baseline Models

#### 1a. Regression - Trip Duration Prediction

The regression baselines estimated trip duration in minutes. The Linear Regression baseline learned broad average relationships between the engineered features and trip time, while the Random Forest baseline captured more flexible patterns such as nonlinear distance effects and interactions between location and time.

| Model | Test RMSE | Test MAE | Test R^2 |
|---|---:|---:|---:|
| Naive mean prediction | 10.59 | - | - |
| Linear Regression baseline | 5.32 | 3.72 | 0.7477 |
| Random Forest baseline | 3.99 | 2.61 | 0.8579 |

Both regression models exceeded the stakeholder goal of keeping RMSE below 10 minutes, but the Random Forest baseline was materially stronger. It reduced test RMSE by about 25.0% and MAE by about 29.9% relative to Linear Regression. This matters because RMSE penalizes large misses more heavily than MAE; lowering both metrics means the Random Forest improved typical errors while also reducing larger trip-duration mistakes.

The train-test comparison also shows the tradeoff. Linear Regression had a very small RMSE gap of 0.08 minutes, suggesting low variance but limited flexibility. Random Forest had a larger gap of 0.89 minutes, with train RMSE of 3.10 minutes and test RMSE of 3.99 minutes. That gap indicates some additional model complexity, but the test performance still improved substantially.

Feature-importance results identify trip distance as the dominant driver of trip duration. That finding is expected but important: time forecasts are most reliable when trip distance is measured cleanly and when trips resemble common distance patterns in the training data. Location and time features add value because two trips with similar mileage can take different amounts of time depending on borough, route context, airport involvement, and pickup period.

From a stakeholder perspective, the Random Forest baseline is useful for operational planning. Drivers and dispatch systems could use its predictions to estimate trip time more accurately than a linear model, especially when trips involve combinations of features that do not follow a simple straight-line relationship. The model is most reliable for ordinary trips with familiar distance and location patterns. It is less reliable when a short recorded distance corresponds to an unusually long duration or when a trip has unusual route conditions.

#### 1b. Classification - Congestion Fee Prediction

The classification baselines estimated whether a trip would include a congestion fee. The Logistic Regression baseline learned a simple decision boundary, while the Random Forest baseline learned more complex spatial and temporal patterns.

| Model | Accuracy | AUC | F1 | Precision | Recall |
|---|---:|---:|---:|---:|---:|
| Naive predict-fee baseline | 0.7438 | - | - | - | - |
| Logistic Regression baseline | 0.7750 | 0.7336 | 0.8661 | 0.7770 | 0.9783 |
| Random Forest baseline | 0.9533 | 0.9794 | 0.9684 | 0.9759 | 0.9609 |

The Logistic Regression baseline improved only modestly over the naive approach in accuracy, although it had very high recall for fee trips. The class-level metrics show the main weakness: it identified fee trips often, but it struggled to separate no-fee trips, with recall of only 0.18 for the no-fee class. In practical terms, this means it would frequently warn of a congestion fee when one may not apply.

The Random Forest baseline performed much better across all classification metrics. Its accuracy reached 0.9533, AUC reached 0.9794, and F1 reached 0.9684. The balance between precision and recall is especially important for stakeholders. High precision means fewer false congestion-fee warnings; high recall means fewer missed fee exposures. The Random Forest achieved both, with precision of 0.9759 and recall of 0.9609 for the fee class.

Feature-importance results indicate that pickup and dropoff location features are central to congestion-fee prediction. This is consistent with the structure of the problem: congestion-fee exposure is fundamentally spatial, and the model needs to understand whether a trip enters, exits, or remains within fee-relevant areas. Time features also matter because fee patterns and trip purposes vary throughout the day.

From a stakeholder perspective, this model could support fare transparency, dispatch planning, and policy analysis. Riders benefit from better advance warning about likely charges. Drivers and fleet operators benefit from more predictable trip economics. Transportation agencies can use the same pattern to understand where fee exposure concentrates across the city.

### 2. XGBoost Classifier

The XGBoost classifier learned the congestion-fee task more effectively than the Logistic Regression and Random Forest classifiers in the advanced classification comparison. It used tree boosting to combine many small decision rules, allowing the model to learn nonlinear location, time, and route interactions.

| Model | Accuracy | F1 Score | ROC-AUC |
|---|---:|---:|---:|
| Logistic Regression comparison model | 0.6673 | 0.7469 | 0.7336 |
| Random Forest baseline | 0.9535 | 0.9685 | 0.9794 |
| XGBoost classifier | 0.9639 | 0.9755 | 0.9850 |

The XGBoost classifier had the strongest classification results, with accuracy of 0.9639, F1 score of 0.9755, and ROC-AUC of 0.9850. The ROC-AUC result is important because it measures ranking quality across thresholds, not only the final yes/no classification. A value of 0.9850 means the classifier ranked fee and no-fee trips very effectively.

Compared with Random Forest, XGBoost delivered a smaller but still meaningful improvement. Random Forest was already strong, with F1 around 0.968 and ROC-AUC of 0.9794. XGBoost improved those values to 0.9755 and 0.9850. Compared with Logistic Regression, the difference was much larger, confirming that congestion-fee prediction depends on nonlinear combinations of location and time rather than a single linear score.

Given the high F1 score, the remaining misclassification concern is less about a broad failure to detect fee trips and more about harder borderline contexts. Hourly checks show that model performance varied slightly across pickup hours, with lower accuracy during late afternoon and evening peak periods. This suggests that congestion-fee patterns are harder to classify when travel demand and route choices are more complex. The model remains strong overall, but the time-of-day pattern matters because peak periods are also when stakeholders most need reliable guidance.

The model is most reliable when pickup and dropoff geography clearly align with typical fee patterns. The model struggles more when trips occur during heavier traffic periods or when the spatial context is less clear. From a stakeholder perspective, the XGBoost classifier is the best congestion-fee model because it provides both accurate binary predictions and strong probability ranking. That combination is useful for driver decision support, dispatch systems, rider-facing fare estimates, and agency-level monitoring.

### 3. XGBoost Regressor

The tuned XGBoost regressor was the strongest trip-duration model overall. It used boosted trees with early stopping, a learning rate of 0.05, maximum tree depth of 6, minimum child weight of 3, subsampling of 0.80, column sampling of 0.80, and a best boosting round of 423. This setup allowed the model to learn complex duration patterns while using validation performance to limit unnecessary additional boosting.

| Model | Test RMSE | Test MAE | Test R^2 |
|---|---:|---:|---:|
| Linear Regression baseline | 5.32 | 3.72 | 0.7477 |
| Random Forest baseline | 3.99 | 2.61 | 0.8579 |
| Tuned XGBoost regressor | 3.896 | 2.502 | 0.866 |

The tuned XGBoost regressor achieved MAE of 2.502 minutes, RMSE of 3.896 minutes, and R^2 of 0.866 on its held-out test split. It also recorded a best validation RMSE of 3.916. These metrics show that the model explains most trip-duration variation and slightly improves on the Random Forest baseline. The improvement over Random Forest is modest, but it is directionally consistent across RMSE, MAE, and R^2.

Residual analysis shows that the model is well calibrated for many ordinary trips. Residuals stayed close to zero for much of the test set, meaning predicted and actual durations were often similar. The model struggles when trip duration becomes unusually large. As duration increases, the residual spread becomes wider, and the largest errors are mostly underpredictions. Among the 20 largest absolute errors, 95% were underpredictions.

The group-based checks provide a clearer interpretation of where the model is dependable. Rush hour did not create a major performance drop: MAE was 2.45 minutes during rush hour and 2.53 minutes outside rush hour. The average residuals were also close to zero in both groups. This means rush-hour status alone was not the main weakness.

Trip length mattered much more. Short trips had MAE of 1.79 minutes and RMSE of 2.63 minutes. Medium trips had MAE of 2.83 minutes and RMSE of 3.86 minutes. Long trips had MAE of 4.97 minutes and RMSE of 7.14 minutes. The model is most reliable when trips are short or moderate in length. It struggles when trips are long, unusual, or affected by conditions that are not fully captured by the engineered features.

Worst-case predictions show the same pattern. The 20 largest errors averaged 84.2 actual minutes but only 42.6 predicted minutes. Among those worst errors, 85% were long trips and 70% involved an airport. Only 25% occurred during rush hour. The most common routes among the worst cases were Queens to Manhattan, Manhattan to Manhattan, and Manhattan to Queens. These are practically important cases because they can lead to major underestimates of driver time, rider wait expectations, and dispatch availability.

From a stakeholder perspective, the tuned XGBoost regressor should be treated as a strong duration estimator with a known limitation. It is appropriate for routine trip-time forecasting and fleet planning, but long airport-related trips and Queens-Manhattan routes should receive extra uncertainty buffers. The model output is most useful when paired with a flag for high-risk trip contexts rather than treated as a perfectly certain travel-time estimate.

### 4. Tuned Random Forest

The Random Forest models were constrained rather than left fully unconstrained. The regression and classification versions used 100 trees with maximum depth of 30 and minimum leaf size of 5. The classification model also used balanced class weights. These settings are intended to preserve the flexibility of a tree ensemble while reducing the risk that individual trees memorize rare trip patterns.

The tuned Random Forest regressor substantially improved over Linear Regression, reaching test RMSE of 3.99 minutes, MAE of 2.61 minutes, and R^2 of 0.8579. Its train-test RMSE gap was 0.89 minutes, compared with 0.08 minutes for Linear Regression. This means the Random Forest was more flexible and showed more gap between training and test performance, but the test-set gain was large enough to justify the added complexity.

The tuned Random Forest classifier was also very strong. It reached accuracy of 0.9533, AUC of 0.9794, F1 of 0.9684, precision of 0.9759, and recall of 0.9609. Its train accuracy was 0.9637 and test accuracy was 0.9533, for a gap of 0.0104. That gap indicates limited overfitting for a high-capacity model. It also shows that the depth and leaf-size constraints helped keep the model general enough for held-out trips.

The results compare the tuned Random Forest models against linear and logistic baselines rather than against an untuned Random Forest. Therefore, the improvement should be interpreted as the value of the final constrained Random Forest approach, not as a direct before-and-after tuning effect. The clearest supported conclusion is that the constrained forest kept overfitting manageable while producing much stronger test performance than the simpler baselines.

Feature-importance patterns are consistent with the rest of the project. Trip distance drives duration prediction, while pickup and dropoff locations drive congestion-fee prediction. This makes the tuned Random Forest useful as both a predictive model and an interpretable benchmark. From a stakeholder perspective, it offers a strong balance between accuracy, stability, and explainability. It may be especially useful when decision-makers want a model that is easier to explain than boosted trees but much stronger than linear baselines.

### 5. Cross-Task Insights - The Dual-Model Advantage

Across the project, the best regression model was the tuned XGBoost regressor, with RMSE of 3.896 minutes, MAE of 2.502 minutes, and R^2 of 0.866. The best classification model was the XGBoost classifier, with accuracy of 0.9639, F1 score of 0.9755, and ROC-AUC of 0.9850. Random Forest remained a strong second choice for both tasks and substantially outperformed the linear and logistic baselines.

The strongest shared pattern is that taxi outcomes depend on interactions among distance, geography, and time. Distance is the clearest driver of trip duration, but location and context determine whether the same distance becomes a short routine ride or a much longer trip. Congestion-fee exposure is even more spatial: pickup and dropoff locations are central because the fee depends on where a trip starts, ends, or travels.

The dual-model approach is useful because trip duration and congestion-fee exposure answer different stakeholder questions. A driver or fleet operator needs to know both how long a trip is likely to take and whether it is likely to trigger an added fee. A rider benefits from more transparent expectations about time and cost. A transportation agency can use the same outputs to understand how traffic burden and pricing exposure vary by location, time, and trip type.

The combined evidence also shows where caution is needed. The models are strongest for common, well-represented trip patterns. They are less reliable for unusually long trips, airport-related rides, Queens-Manhattan routes, and some peak-period classification cases. This matters because those trips can have the highest operational impact. A practical deployment should therefore report both predictions and uncertainty flags, especially for long-distance or airport-related trips.

Overall, the project supports using ensemble models as the final modeling strategy. XGBoost provides the best overall performance, while tuned Random Forest offers a strong and more interpretable benchmark. Together, the regression and classification models provide a fuller decision-support system: one model estimates the time burden of a trip, and the other estimates congestion-pricing exposure. That combination is more useful to stakeholders than either prediction alone.
