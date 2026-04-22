# XGBoost Interpretation - Stage 7

## Overview

For this stage, I interpreted my tuned XGBoost regressor for predicting taxi trip
duration in minutes. I focused on what the model learned, how it compared with the
Random Forest and Linear Regression baselines, and where it still made its biggest
mistakes.

My XGBoost model was evaluated on a 150,000-row sample with separate train,
validation, and test sets. The baseline models were run on a larger 500,000-row
sample, so the comparison is not perfectly identical. Even so, I still think the
comparison is useful for understanding overall model performance. I am just a little
careful about making too much of very small differences between XGBoost and Random
Forest.

## What my tuned XGBoost regressor learned

My tuned XGBoost regressor performed well overall on the held-out test set:

| Metric | Value | What it means |
|---|---:|---|
| MAE | 2.50 minutes | On average, my prediction was about 2.5 minutes away from the real trip time. |
| RMSE | 3.90 minutes | This is also measured in minutes, but it penalizes large errors more than MAE. |
| R^2 | 0.866 | The model explained about 86.6% of the variation in trip duration. |

The best validation RMSE was 3.916, and the test RMSE was 3.896. Since those two
numbers are very close, the model seems to generalize well instead of just memorizing
the training data. The best boosting round was 423.

The feature importance results also make sense. The strongest signals were mostly
about distance and trip length:

| Feature | Importance |
|---|---:|
| `is_extreme_distance` | 0.382 |
| `distance_category` | 0.295 |
| `trip_distance` | 0.165 |
| `is_same_borough` | 0.080 |
| `is_weekend` | 0.014 |
| `time_slot` | 0.011 |
| `is_airport_trip` | 0.009 |

To me, this shows that trip distance and distance category were the biggest clues for
predicting duration. After that, the model also used location and trip context, like
whether the ride stayed in the same borough, whether it involved an airport, and what
time period it happened in. I would not call these causal factors, but they do show
what the model relied on most.

## Comparison with the Random Forest and Linear Regression baselines

Here is the main comparison for trip-duration regression:

| Model | RMSE | MAE | R^2 |
|---|---:|---:|---:|
| My tuned XGBoost regressor | 3.90 | 2.50 | 0.866 |
| Random Forest baseline | 3.99 | 2.61 | 0.858 |
| Linear Regression baseline | 5.45 | 3.75 | 0.739 |

My XGBoost model clearly did better than the Linear Regression baseline. The RMSE
dropped from 5.45 minutes to 3.90 minutes, and the MAE dropped from 3.75 minutes to
2.50 minutes. The R^2 also improved from about 0.739 to 0.866. This tells me that
XGBoost captured patterns that a straight-line model could not capture as well.

Compared with the Random Forest baseline, XGBoost was slightly better in the reported
results. The RMSE was lower by about 0.09 minutes, the MAE was lower by about 0.11
minutes, and the R^2 was a little higher. Since the sample sizes and splits were not
exactly the same, I would describe the two models as very close overall, with XGBoost
having a small edge.

## Residual analysis: where the model is well-calibrated vs biased

I used residuals to check where the predictions were too high or too low:

`residual = actual trip time - predicted trip time`

So if the residual is positive, the model predicted the trip too short. If the
residual is negative, the model predicted the trip too long.

Overall, the model looks fairly well-calibrated for normal trips. Many residuals stay
close to zero, and the average residuals in the group checks are also very close to
zero. That means the model does not seem to strongly overpredict or underpredict
ordinary trips on average.

One thing I noticed is that the errors get wider as the actual trip duration gets
larger. The biggest misses were usually underpredictions. In the 20 largest errors,
95% were positive residuals, which means the model predicted those trips as too short.
So the main issue is not average bias. The bigger problem is that the model sometimes
misses unusually long trips by a lot.

There were small average overpredictions for medium and long trips because their mean
residuals were slightly negative. However, those mean residuals were tiny compared
with the overall size of the errors, so I do not think that is the main practical
problem. The more important concern is the large underpredictions in the worst cases.

## Group-based error checks

### Rush hour vs non-rush hour

Rush hour means pickups from 7-9 AM or 5-7 PM.

| Group | Trips | MAE | RMSE | Mean residual |
|---|---:|---:|---:|---:|
| Non-rush hour | 20,359 | 2.527 | 3.992 | -0.019 |
| Rush hour | 9,641 | 2.450 | 3.686 | -0.001 |

The rush-hour results surprised me a little because rush hour was not harder for the
model in this sample. The MAE was 2.45 minutes during rush hour and 2.53 minutes
outside rush hour. That is a small difference, and both groups had mean residuals
very close to zero.

Based on this, I would not say rush hour is a major weakness for my model. It
performed about the same during rush hour and non-rush hour, with a slight advantage
for rush hour in this test sample.

### Short trips vs long trips

Trip length was grouped as short = 0-2 miles, medium = 2-5 miles, and long = 5+
miles.

| Group | Trips | Mean actual | Mean predicted | MAE | RMSE | Mean residual |
|---|---:|---:|---:|---:|---:|---:|
| Short | 18,302 | 8.618 | 8.607 | 1.793 | 2.632 | 0.011 |
| Medium | 7,424 | 17.305 | 17.360 | 2.826 | 3.864 | -0.055 |
| Long | 4,274 | 32.024 | 32.068 | 4.974 | 7.139 | -0.044 |

Trip length made a much bigger difference than rush hour. Short trips were clearly
the easiest group for the model, with MAE around 1.79 minutes. Long trips were much
harder, with MAE around 4.97 minutes and RMSE around 7.14 minutes.

This is a noticeable gap. Long-trip MAE was almost 2.8 times the short-trip MAE. The
model predicted the average long-trip duration pretty closely, but the individual
long-trip predictions had much larger errors.

One practical reason could be that long trips have more room for delays, route
differences, airport effects, and borough-to-borough variation. The results also
showed that many of the worst errors were long trips, airport-related trips, or
Queens/Manhattan routes. I cannot say exactly what caused every delay, but the pattern
is still clear enough to be useful.

## Worst-case predictions

The hardest trips were the ones with the largest absolute errors. The biggest misses
were mostly underpredictions, meaning the model predicted the trips as much shorter
than they actually were.

The two largest individual errors were:

- Row `69573`: actual 110.30 minutes, predicted 8.97 minutes, absolute error 101.33
  minutes. This was labeled as a short 1.60-mile Manhattan to Manhattan non-rush-hour
  trip.
- Row `172395`: actual 152.83 minutes, predicted 54.83 minutes, absolute error 98.00
  minutes. This was a long 18.33-mile Manhattan to Manhattan non-rush-hour trip.

The first case is a good reminder that a short-distance trip can still take a very
long time. I cannot say exactly why that happened, but distance clearly did not
explain it well.

For the 20 worst errors:

- 95% were underpredictions.
- The average actual duration was 84.2 minutes.
- The average predicted duration was only 42.6 minutes.
- 85% were long trips.
- 70% involved an airport.
- Only 25% happened during rush hour.
- The most common borough routes were Queens to Manhattan, Manhattan to Manhattan,
  and Manhattan to Queens.

So the worst-case pattern was not mainly about rush hour. The hardest trips were more
often unusually long rides, airport-related rides, and Queens/Manhattan routes. My
model often recognized that these trips should be longer than normal, but it still did
not add enough time for the most extreme cases.

## Practical findings for Maria and fleet operators

The model struggles when a trip becomes unusually long. This is especially true for
long trips, airport-related trips, and some Queens/Manhattan routes.

The model does better when the trip is a normal short ride. Short trips had much
lower MAE and RMSE than long trips, so I would trust those predictions more.

The predictions are more reliable when the trip is not an extreme case. For ordinary
trips, the residuals stay close to zero and there is very little average bias.

For Maria and fleet operators, I would treat the model prediction as a useful estimate
instead of a guaranteed trip time. If the trip is long, airport-related, or between
Queens and Manhattan, it would be smart to add extra buffer time because those are the
types of trips where the model had its biggest underpredictions.

Rush hour by itself was not the biggest issue in this evaluation. I would focus more
on trip length, airport status, and route type when deciding how much confidence to
place in the prediction.

## Conclusion

My tuned XGBoost regressor performed strongly for trip-duration prediction. It reached
MAE = 2.50 minutes, RMSE = 3.90 minutes, and R^2 = 0.866 on the test set. It clearly
improved over the Linear Regression baseline and was slightly better than the Random
Forest baseline, although the two tree-based models were pretty close overall.

My main takeaway is that the model learned normal trip-duration patterns well,
especially from distance and trip-length features. The biggest weakness is the unusual
long-trip cases. For regular short and medium trips, I would trust the model more. For
long, airport-related, or Queens/Manhattan trips, I would be more careful because that
is where the worst underpredictions happened.