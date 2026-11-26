# Bike Share Modeling Report

## Exercise 4: Target Transformation

The original HistGradientBoostingRegressor performed poorly when predicting raw daily ridership. On the validation set, it achieved an R² of 0.0977 and an MAE of 1,381.6 bikes, compared with Ridge's R² of 0.7224 and MAE of 700.4 bikes.

The training target ranged from 431 to 6,043 rides, while the validation target ranged from 1,027 to 8,362. There were 71 validation days with ridership above the training maximum. This suggested that the tree model struggled to capture the upward ridership trend because tree predictions based on leaf averages do not naturally extrapolate beyond the training target range.

I first tested a log transformation, fitting the model on log(1 + cnt) and converting predictions back with the exponential function. This did not help: validation R² decreased to -0.0282 and MAE increased to 1,493.3.

I then transformed the target into the log day-over-day growth ratio:

    log(cnt / cnt_lag_1)

After predicting the growth ratio, I exponentiated the prediction and multiplied it by the observed previous day's count (`cnt_lag_1`) to recover a ridership prediction. This formulation allows the model to predict counts above the maximum training target because predictions are scaled by the previous day's observed count.

The relative-growth HGB model achieved validation R² of 0.5672 and MAE of 893.3. This was a substantial improvement over the original HGB, but it did not outperform Ridge. Its mean prediction error was -590.5 rides, indicating systematic underprediction. Its predicted validation mean was 4,561.6, compared with the actual mean of 5,152.1.

I consider the relative-growth transformation defensible because it models proportional day-over-day change rather than assuming ridership is stationary in absolute terms. However, it changes the forecasting formulation by explicitly using yesterday's count as a scaling baseline. It should therefore be understood as an alternative modeling approach, not evidence that HGB is generally superior to Ridge. Ridge remained the best validation model among those tested.

All Exercise 4 model comparisons and transformation decisions were made using the training and validation periods only. The test set was not used for these experiments.

## Exercise 5: Ridge vs. Persistence

On the test set, Ridge achieved an R² of 0.6356 and an MAE of 840.3 rides, compared with persistence at an R² of 0.5099 and an MAE of 840.1 rides. Ridge improved R² by 0.1257, meaning it explained more of the variation in daily ridership. However, persistence had a marginally lower MAE by just 0.2 rides, which is practically negligible. This shows that Ridge captures patterns beyond yesterday's ridership, but the simple persistence baseline remains remarkably competitive. The additional model complexity does not translate into a meaningful improvement in average absolute error.
