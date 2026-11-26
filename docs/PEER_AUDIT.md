# Peer Repository Leakage Audit

**Repository:** `vanshika-s/bikeshare-open`
**Reviewed commit:** `f656469` — Add weather severity rolling feature with tests

## Scope and Method

I manually reviewed the partner repository's configuration, feature engineering implementation, training code, tests, ingestion logic, report, and saved metrics. The review focused on target leakage, temporal validity, calendar gaps, silent row loss, and baseline reporting.

No pipeline or test commands were run during this review, and no repository files were modified.

## Results

### 1. Target Leakage — PASS

`src/config.py` defines `casual`, `registered`, and `cnt` as forbidden columns. The `feature_columns()` function in `src/features.py` excludes forbidden columns from the model feature set.

The leakage tests verify that the target identity holds and that forbidden columns are excluded. Lagged target features use historical values rather than the current day's target.

No confirmed target leakage was identified.

### 2. Temporal Validity — PASS, WITH A CAVEAT

`src/train.py` uses chronological date boundaries to create training, validation, and test sets. Models are fitted using training data, and Ridge preprocessing is included in a pipeline.

The new `weather_severity_roll_3` feature uses `.shift(1).rolling(3)`, so it uses the previous three days and excludes the current day.

However, `tests/test_contract.py` contains `test_beats_the_mean_baseline`, which evaluates Ridge performance on the test set whenever the test suite runs. This repeatedly consults test performance and weakens the intention to reserve the test set for final evaluation.

**Recommendation:** Use validation performance for the test assertion and reserve the test set for final evaluation.

### 3. Calendar Gaps — PASS

`src/features.py` reindexes the data to a complete daily calendar before calculating lagged and rolling features. This ensures that shifts represent calendar days rather than simply previous available observations.

The tests verify calendar lags and check that the new weather rolling feature behaves correctly when a calendar date is missing.

### 4. Silent Row Loss — PASS, WITH A CAVEAT

The weather encoder checks for unknown labels and raises a `ValueError` rather than silently converting unexpected labels into missing values.

The feature pipeline drops incomplete rows after constructing lagged and rolling features. `tests/test_contract.py` also places an upper bound on allowable row loss based on the lag warmup period and missing calendar days.

However, the test checks only that row loss does not exceed the upper bound. It does not verify the exact expected number of retained rows.

**Recommendation:** Add an exact row-count or explicit row-loss accounting assertion.

### 5. Baseline Reporting — PASS

`src/train.py` includes both a training-mean baseline and a persistence baseline. The saved `reports/metrics.json` includes validation and test results for both baselines alongside Ridge and HGB.

The report also discusses the difference between R² and MAE and recognizes that Ridge has a higher test MAE than persistence despite its higher test R².

## Overall Assessment

No confirmed target leakage was identified in the reviewed implementation. The new weather severity rolling feature correctly uses historical weather observations, and the repository handles calendar gaps and chronological splits appropriately.

The main methodological concern is that a test-suite assertion repeatedly evaluates performance on the test set. Row-loss accounting could also be strengthened by checking the exact expected number of retained rows.

The review was conducted manually using source code and existing artifacts. The full pipeline and test suite were not rerun.
