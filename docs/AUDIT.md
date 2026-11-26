# Leakage Audit

## Scope and method

Reviewed the current repository implementation, configuration, tests, existing metrics, report, and methodological decisions. The audit covers target leakage, temporal validity, calendar gaps, silent row loss, and baseline reporting.

No pipeline or training commands were run during this review. Existing metrics were inspected without regeneration.

## Results

### 1. Target leakage — PASS

`src/config.py` defines forbidden target-derived columns (`casual`, `registered`, `cnt`) and unverified or time-index columns. `feature_columns` excludes forbidden columns. The leakage tests check that forbidden columns are absent and that raw columns are classified.

The target identity `casual + registered = cnt` is also tested. The constructed lag and rolling features use historical target values rather than the current day's target.

### 2. Temporal validity — PASS, WITH A TESTING CAVEAT

`src/features.py` shifts target values before calculating rolling features. `src/train.py` uses chronological date boundaries for training, validation, and testing, and fits models using training rows.

The Ridge pipeline includes its scaler, so preprocessing is fitted as part of the training pipeline.

Caveat: `tests/test_contract.py:46-52` fits Ridge and evaluates its score on the test set. Running the test suite therefore consults test performance repeatedly. This weakens the stated intention in `docs/DECISIONS.md` to touch the test set only once. The performance assertion should use validation data instead, leaving the test set for final evaluation.

### 3. Calendar gaps — PASS

`src/features.py` reindexes the observations to a complete daily calendar before calculating lag and rolling features. Tests check calendar completeness and verify that lag calculations respect actual calendar days. Rolling standard deviation is also checked across the known calendar gap.

### 4. Silent row loss — PASS, WITH A CAVEAT

Unknown weather labels raise a `ValueError` rather than silently mapping to missing values. The feature pipeline drops incomplete rows after constructing lag and rolling features. `tests/test_contract.py:39-43` places an upper bound on allowable row loss based on lag warmup and missing calendar days.

Caveat: the test checks that row loss does not exceed the bound; it does not assert the exact expected number of retained rows. An exact row-count or explicit accounting assertion would make this safeguard stronger.

### 5. Baseline reporting — PASS

`src/train.py` includes both a training-mean baseline and a persistence baseline. Existing `reports/metrics.json` contains validation and test results for both baselines alongside Ridge and HGB. `REPORT.md` discusses the Ridge-versus-persistence comparison and acknowledges the negligible MAE difference.

## Overall assessment

No confirmed target leakage was identified in the reviewed implementation. Temporal ordering, calendar handling, and baseline inclusion are sound. The principal methodological concern is the test-set assertion inside the test suite, which should be moved to validation evaluation. Row-loss accounting could also be made more explicit.

The audit was based on static code and existing artifacts; the full pipeline was not rerun.
