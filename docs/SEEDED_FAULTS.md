# Exercise 2: Seeded Faults

## Fault 1 — Forbidden feature

**Change:** Added `casual` to `feature_columns`, even though it is a forbidden target-derived column.

**Detection:** The PostToolUse hook ran `tests/test_leakage.py::test_no_forbidden_features` and reported `forbidden columns in feature set: {'casual'}`.

**Fix:** Removed `casual` from the feature list. The relevant test passed afterward.
**Claude responses to fix:** 1
## Fault 2 — Rolling feature leakage

**Change:** Removed `.shift(1)` from the `cnt_roll_7` calculation, causing the rolling mean to include the current day's target.

**Detection:** The PostToolUse hook caught the issue with `tests/test_leakage.py::test_rolling_mean_excludes_today`. The expected value was `4414.86`, but the faulty calculation returned `4389.0`.

**Fix:** Restored `.shift(1)` before `.rolling(LAG_WARMUP).mean()`. The test passed afterward.
**Claude responses to fix: 2**
## Fault 3 — Missing weather-label validation

**Change:** Removed the validation that raises a `ValueError` when `encode_weather()` encounters an unrecognized label.

**Detection:** No visible PostToolUse hook output appeared. Running `python -m pytest -q` with the project virtual environment activated produced `18 passed`. The existing tests did not catch the missing validation.

**Risk:** An unrecognized weather label could silently map to `NaN` and later be removed by `dropna()`. `BAD` is a valid label in `WEATHER_ORDER`; it is not itself an unknown label.

**Fix:** Restored the unknown-label check and `ValueError`. All 18 tests passed afterward.
**Claude responses to fix: 1**
## Fault 4 — Shell command bypass

**Change:** Used Bash `cp` to copy a harmless probe file into `data/raw/hook_probe.txt`.

**Detection:** No visible PostToolUse hook output appeared after the copy.

**Cleanup and verification:** Removed the probe file. The SHA-256 hash of `data/raw/bike_daily.csv` remained:

`a6bed1577f6e4de56dcfae6d8315985073cc0ad3446c8c99ed60c86a0b5ad9dc`

**Takeaway:** A hook limited to Claude's Edit/Write tools may not detect filesystem changes made through Bash commands. A PreToolUse Bash guard, filesystem watcher, or Git pre-commit/CI check could provide additional protection.
**Claude responses to fix: N/A**
## Response counts

The exact number of Claude responses required to fix each fault was not consistently recorded, so no counts are asserted here.
