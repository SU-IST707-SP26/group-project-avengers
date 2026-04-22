# Stage 7 Interpretation: XGBoost Trip Duration Regressor

## Overview

This interpretation focuses on my tuned XGBoost regressor for predicting NYC yellow taxi trip duration. The model predicts `trip_duration_min` using engineered trip features such as distance, pickup/dropoff location, time of day, borough information, rush-hour flags, and airport-trip indicators. Post-trip variables were removed, so the model is based on information that would be available before or during dispatch.

Overall, the model performed well on the held-out test data. The main test metrics were:

| Metric | Value |
|---|---:|
| RMSE | 3.896 minutes |
| MAE | 2.502 minutes |
| R^2 | 0.866 |

The average error is low enough to be useful for real trip-duration planning, but the residual checks show that the model is not equally reliable for every type of trip.

## What my tuned XGBoost regressor learned

The tuned XGBoost model learned that trip duration is mostly driven by distance and route context. The strongest patterns were tied to distance-related features, whether the trip stayed in the same borough, location IDs, boroughs, time features, and airport-trip status.

The RMSE of 3.896 minutes means larger misses are still present, but the MAE of 2.502 minutes shows that a typical prediction error is only about two and a half minutes. The R^2 value of 0.866 means the model explains most of the variation in trip duration on the test set. In plain terms, the model learned the normal relationship between where a trip starts, where it ends, how far it goes, and how long it usually takes.

The best validation RMSE was about 3.916 at boosting round 423, which also suggests that the tuned model was not just running for a fixed number of trees blindly. Early stopping helped choose a strong point in training.

## Comparison with the Random Forest and Linear Regression baselines

The tuned XGBoost regressor had the strongest overall regression results in the project comparison:

| Model | RMSE | MAE | R^2 |
|---|---:|---:|---:|
| Tuned XGBoost regressor | 3.896 | 2.502 | 0.866 |
| Random Forest baseline | 3.99 | 2.61 | 0.8579 |
| Linear Regression baseline | 5.32 | 3.72 | 0.7477 |

Compared with the Linear Regression baseline, XGBoost was clearly better. The linear model still did a reasonable job, but it could not capture the nonlinear patterns in taxi trips as well as the tree-based models. For example, the effect of distance is not exactly the same for every route, hour, borough, or airport trip.

Compared with the Random Forest baseline, XGBoost was only slightly better. Its RMSE and MAE were lower, and its R^2 was slightly higher. I would describe this as a meaningful but small improvement, not a huge gap.

One thing to keep in mind is that the comparison is useful at the project level, but it is not perfectly apples-to-apples. The Linear Regression and Random Forest baselines were evaluated on a larger 500,000-row sample with a 100,000-row test set, while the XGBoost regressor used a 150,000-row sample with a 30,000-row test set and a validation split for early stopping. Because of that, I would trust the overall ranking, but I would not overread tiny differences between XGBoost and Random Forest.

## Residual analysis: where the model is well-calibrated vs biased

Residuals were measured as:

`actual duration - predicted duration`

So a positive residual means the model underpredicted the trip time, and a negative residual means it overpredicted.

The residual plots show that the model is well-calibrated for many ordinary trips. The residual cloud stays close to zero for common trip durations, and the average residuals in the group checks are also close to zero. For example, the mean residual was -0.019 minutes for non-rush-hour trips and -0.001 minutes for rush-hour trips. By trip length, the mean residuals were also small: 0.011 for short trips, -0.055 for medium trips, and -0.044 for long trips.

The main bias appears in the worst errors, not in the average trip. The residual spread gets wider as trips get longer, and 95% of the 20 largest errors were underpredictions. This means the model usually handles normal trips well, but when it misses badly, it often predicts a trip as much shorter than it actually was.

## Group-based error checks

The rush-hour check did not show a major drop in performance during rush hour. The model actually had a slightly lower MAE during rush hour:

| Group | Trips | MAE | RMSE | Mean residual |
|---|---:|---:|---:|---:|
| Non-rush hour | 20,359 | 2.527 | 3.992 | -0.019 |
| Rush hour | 9,641 | 2.450 | 3.686 | -0.001 |

This suggests that the rush-hour feature and time/location patterns helped the model handle regular traffic periods. Rush hour alone was not the biggest weakness.

Trip length mattered much more:

| Trip length | Trips | MAE | RMSE | Mean residual |
|---|---:|---:|---:|---:|
| Short | 18,302 | 1.793 | 2.632 | 0.011 |
| Medium | 7,424 | 2.826 | 3.864 | -0.055 |
| Long | 4,274 | 4.974 | 7.139 | -0.044 |

The model was most reliable for short trips. Error increased for medium trips and became much larger for long trips. This makes sense because long trips have more chances for unusual traffic, route changes, airport delays, or other conditions that are hard to capture from the available features.

## Worst-case predictions

The worst-case predictions show where the average metrics hide the model's biggest problems. The 20 worst cases averaged 84.2 actual minutes, but the model predicted only 42.6 minutes on average. Among those 20 worst errors, 85% were long trips, 70% involved an airport, and only 25% happened during rush hour.

The most common routes in the worst cases were Queens to Manhattan, Manhattan to Manhattan, and Manhattan to Queens. Several of these were airport-related or unusually long for their distance. One of the largest misses was a short Manhattan-to-Manhattan trip that lasted over 110 minutes but was predicted under 9 minutes, which looks like an unusual delay or outlier that the model could not reasonably infer from the normal features.

The hardest trips usually had one or more of these traits:

- unusually long actual duration
- long distance or airport involvement
- Queens/Manhattan route patterns
- actual trip time much higher than typical for the distance
- delays that were not fully explained by rush-hour status

## Practical findings for Maria and fleet operators

The model is useful for estimating normal taxi trip durations, especially short and ordinary city trips. For Maria, this means the model can support trip planning, customer expectations, and better estimates of how long a ride should take.

The model struggles when a trip turns into an unusually long ride, especially for long-distance, airport-related, or Queens/Manhattan trips. In those cases, a single point prediction may be too optimistic. Maria and fleet operators should treat those estimates with extra caution and add more buffer time.

For fleet operators, the biggest practical lesson is that dispatch and scheduling should not rely only on the average predicted duration. Long trips and airport trips should have extra slack because those are where the largest underpredictions happen. The model can still help with planning, but the riskiest trips should be flagged as higher uncertainty.

## Conclusion

My tuned XGBoost regressor was the strongest trip-duration model in the project comparison. It achieved an RMSE of 3.896 minutes, MAE of 2.502 minutes, and R^2 of 0.866, which shows strong overall fit.

The model learned the normal relationship between distance, route, location, time, and trip duration. It is well-calibrated for many ordinary trips, and rush hour by itself was not a major weakness. The main limitation is that the model underpredicts some extreme long trips, especially airport-related and Queens/Manhattan rides. For real use, I would trust the model for general planning, but I would add caution or buffer time for the trip types where the worst errors are concentrated.
