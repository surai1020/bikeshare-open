# Decisions

One line per methodological decision: what you chose, why, and what it changed.
Agents append here via the `/new-feature` skill; you append here by hand.

- Exclude `casual`, `registered`: they sum to `cnt` exactly. Leaky model R^2 1.000 -> honest ridge 0.583.
- Exclude `cnt_2d_bfr`: a shipped 2-day lag we cannot test the alignment of. Build our own lags instead.
- Exclude `instant`, `yr`, `days_since_2011`: time indexes let a tree predict a flat constant past 2011 and a linear model extrapolate a trend we have no grounds for.
- Split 2011 / H1 2012 / H2 2012 by date. Test set touched once, after all feature work.
- Report mean and persistence baselines with every score. Persistence (test R^2 0.510, MAE 840) is within 0.07 R^2 of ridge and beats it on MAE; that caveat goes in the report.
- 2026-11-25: Add `cnt_roll_std_7`, the sample std (ddof=1) of `cnt` over days d-7..d-1, built as `shift(1)` then `rolling(7).std()`. Availability: all seven values are known by the end of d-1, so nothing from day d enters. Validation R^2 (ridge) 0.6475 -> 0.7224, MAE 832.9 -> 700.4; (hgb) 0.2185 -> 0.0977, MAE 1296.6 -> 1381.6. Kept because the ridge gain is far above the 0.01 noise floor; hgb got worse and I report that rather than hide it. Decision made on validation only.
- 2026-11-25: Rejected the agent's first draft of this feature, which used `rolling(7, min_periods=0)`. That would compute a std over 6 days across the missing 2011-03-10 and silently reintroduce the calendar-gap bug. Final line written by hand with the default `min_periods`.
- 2026-11-25: Agent errors caught so far: it claimed to have "added weather_severity" (already in the starter; it never wrote the requested tests or ran make test), and it ran `make data` unprompted. Neither claim or command was trusted without a printed check.
- 2026-11-25: Tested target transformations for HGB on validation only. `log1p(cnt)` worsened performance (R² -0.0282, MAE 1493.3). Modeling `log(cnt / cnt_lag_1)` and reconstructing counts with `cnt_lag_1 * exp(prediction)` improved HGB to R² 0.5672 and MAE 893.3, but Ridge remained better (R² 0.7224, MAE 700.4). Kept as a defensible relative-growth experiment, not a replacement for the original pipeline; it still systematically underpredicted (mean error -590.5).
