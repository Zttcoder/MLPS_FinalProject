# How to read a `run_TIMESTAMP` folder

This document explains how to navigate the outputs of a training run for the two-stage outage forecasting notebooks.

It is written so that someone opening the repository for the first time can understand:

- what each file and subfolder is for
- where to look first
- which files correspond to model selection, validation, and final prediction export
- how candidate models, submission models, and refit models differ

Throughout this guide, the example run is:

- [run_20260418_214955](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955>)

In that example:

- the run timestamp is `20260418_214955`
- the run came from a notebook using a two-stage direct tabular model
- the run evaluated horizons `24` and `48`
- the selected best pair for both horizons was `xgb_classifier + xgb_regressor`

## 1. Big picture: what a run folder is

A `run_TIMESTAMP` folder is a self-contained experiment record.

The goal is that, after a run finishes, you can answer all of the following without reopening the notebook:

- Which configuration produced these results?
- What environment was used?
- Which train/validation split was used?
- Which weather features were selected?
- Which candidate models were trained?
- Which model won for each horizon?
- What validation metrics did each candidate get?
- Which exact model generated the final submission CSVs?
- What plots and diagnostics were produced?

In other words, each run folder is meant to be an experiment snapshot.

## 2. Typical run folder structure

Using `run_20260418_214955` as the example, the run root contains:

```text
run_20260418_214955/
├── checkpoints/
├── metrics/
├── models/
├── plots/
├── predictions/
├── tables/
├── artifact_manifest.json
├── config.json
├── environment.json
└── selected_weather_features.json
```

Important note about `checkpoints/`:

- the notebooks use `checkpoints/` to support resume/restart workflows during long training sessions
- this subfolder is useful while a run is in progress
- however, it is **not expected to be uploaded to GitHub**
- so if you browse a shared run folder in the repository, `checkpoints/` may be missing on purpose

That is normal.

## 3. Recommended reading order

If you want the fastest way to understand a run, read files in this order:

1. `config.json`
2. `environment.json`
3. `tables/run_summary.csv`
4. `tables/backtest_folds.csv`
5. `tables/best_pair_by_horizon.csv`
6. `tables/submission_model_summary.csv`
7. `predictions/two_stage_pred_24h.csv` and `predictions/two_stage_pred_48h.csv`
8. `models/submission_bundle_h24_metadata.json` and `models/submission_bundle_h48_metadata.json`

If you want a deeper audit, continue with:

1. `tables/pair_summary.csv`
2. `tables/candidate_fold_summary.csv`
3. `metrics/`
4. `plots/`
5. `artifact_manifest.json`

## 4. What the top-level files mean

### `config.json`

This is the most important reproducibility file.

It contains the exact notebook configuration used for the run, such as:

- random seed
- horizons
- CPU/GPU settings
- stride
- lag settings
- rolling-window settings
- weather-feature selection settings
- hyperparameter grids
- resume settings
- manual split definitions when applicable

Example:

- [config.json](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/config.json:1>)

In the example run, this file tells us that:

- `origin_stride_hours = 1`
- `n_backtest_folds = 1`
- `handle_weather_outliers = true`
- `weather_outlier_strategy = winsorize_iqr`
- `manual_backtest_splits` were used

So before looking at any metric, always check `config.json`.

### `environment.json`

This records the execution environment:

- Python version
- OS / platform
- package versions
- GPU availability
- XGBoost device actually used
- whether the run resumed an existing folder

Example:

- [environment.json](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/environment.json:1>)

This is especially useful when comparing runs done locally, in Colab, or on a cloud VM.

### `selected_weather_features.json`

This stores the final weather features retained after correlation-based feature screening.

Example:

- [selected_weather_features.json](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/selected_weather_features.json:1>)

In the example run, the selected features include:

- `gh_4`
- `cape_1`
- `pwat`
- `sh2`
- `mslma`
- `sdlwrf`
- `mcc`
- `d2m`
- `cape`
- `lcc`

### `artifact_manifest.json`

This is a run-level index file.

It summarizes:

- the run timestamp
- the project root used when training
- selected weather features
- best model pair(s) by horizon
- other run-level summary information

Example:

- [artifact_manifest.json](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/artifact_manifest.json:1>)

This file is very handy when you want one compact snapshot instead of opening many CSVs individually.

## 5. Understanding the subfolders

## `tables/`

This is the main analytical summary folder.

If you only have time to inspect one subfolder, inspect `tables/`.

### Key files in `tables/`

#### `run_summary.csv`

High-level summary of the entire run.

Example:

- [tables/run_summary.csv](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/tables/run_summary.csv:1>)

This tells you:

- how many artifacts were produced
- the best pair for 24h
- the best pair for 48h
- which files are the exported submission predictions
- which files are the refit predictions

In the example run:

- best 24h pair: `h24__clf_xgbc01__reg_xgbr01`
- best 48h pair: `h48__clf_xgbc01__reg_xgbr01`

#### `backtest_folds.csv`

This is the train/validation split map.

Example:

- [tables/backtest_folds.csv](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/tables/backtest_folds.csv:1>)

It records, for each horizon and fold:

- validation origin index
- validation origin timestamp
- training origin start
- last eligible training origin
- validation window start
- validation window end

This is the file to inspect if you want to know exactly which timestamps were used for training and validation.

For the example run:

- 24h validation window: `2023-06-12 01:00:00` to `2023-06-13 00:00:00`
- 48h validation window: `2023-06-12 01:00:00` to `2023-06-14 00:00:00`

#### `pair_summary.csv`

This is the main model comparison table.

Example:

- [tables/pair_summary.csv](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/tables/pair_summary.csv:1>)

Each row is one classifier-regressor pair for one horizon.

Typical columns include:

- `pair_tag`
- `horizon`
- `classifier_name`
- `regressor_name`
- `mean_avg_county_rmse_expected`
- `mean_overall_rmse_expected`
- `mean_stage1_roc_auc`
- `mean_stage2_positive_rmse`
- zero baseline metrics

If you want to compare all two-stage model combinations, start here.

#### `candidate_fold_summary.csv`

This is the fold-level version of `pair_summary.csv`.

Example:

- [tables/candidate_fold_summary.csv](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/tables/candidate_fold_summary.csv:1>)

Each row is one specific:

- horizon
- candidate pair
- fold

This file is especially useful when there are multiple folds, because it lets you inspect variability across folds.

In the example run there is only one fold, so `candidate_fold_summary.csv` and `pair_summary.csv` are very similar.

#### `best_pair_by_horizon.csv`

This stores the winning pair for each horizon after model comparison.

Example:

- [tables/best_pair_by_horizon.csv](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/tables/best_pair_by_horizon.csv:1>)

This file is the answer to:

- “Which candidate won for 24h?”
- “Which candidate won for 48h?”

#### `best_candidate_model_by_horizon.csv`

This identifies the exact trained candidate model selected for each horizon.

Example:

- [tables/best_candidate_model_by_horizon.csv](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/tables/best_candidate_model_by_horizon.csv:1>)

It includes:

- `fold_id`
- paths to the selected candidate model files
- the validation metrics of the winning candidate fold

This file is important because it connects model selection to the concrete `.joblib` artifacts in `models/`.

#### `submission_model_summary.csv`

This describes the model actually used to generate the exported submission CSVs.

Example:

- [tables/submission_model_summary.csv](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/tables/submission_model_summary.csv:1>)

This is one of the most important files in the whole run folder.

It answers:

- Which artifact generated `two_stage_pred_24h.csv`?
- Was the submission generated from a validation-winning candidate or from a refit?
- Where are the stage-1 and stage-2 pieces for that submission model?

In the example run, the submission models come from:

- `best_candidate_validation_model`

That means the exported submission files were produced by the best validation candidate model, not by the refit bundle.

#### `final_refit_validation_summary.csv`

This summarizes the final refit models and links them to their exported files.

Example:

- [tables/final_refit_validation_summary.csv](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/tables/final_refit_validation_summary.csv:1>)

This file is useful when you want to inspect the “refit” artifacts separately from the “submission” artifacts.

In practice, it helps answer:

- Which final refit bundle corresponds to horizon 24?
- Which final refit bundle corresponds to horizon 48?
- Which pair was refit?

#### `classifier_stage_summary.csv`

Stage-1-only comparison table.

Example:

- [tables/classifier_stage_summary.csv](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/tables/classifier_stage_summary.csv:1>)

This aggregates classifier-only metrics such as:

- ROC AUC
- average precision
- Brier score
- F1

Use this when you want to compare the classification component independently of the regression component.

#### `regressor_stage_summary.csv`

Stage-2-only comparison table.

Example:

- [tables/regressor_stage_summary.csv](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/tables/regressor_stage_summary.csv:1>)

This aggregates regression-on-positive-cases metrics such as:

- positive-only RMSE
- positive-only MAE
- number of positive training rows

Use this when you want to evaluate the regression stage on its own.

#### `selected_weather_features.csv`

CSV form of the selected weather features with correlations.

Example:

- [tables/selected_weather_features.csv](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/tables/selected_weather_features.csv:1>)

#### `feature_frame_summary.csv`

Compact summary of the engineered feature frame.

Example:

- [tables/feature_frame_summary.csv](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/tables/feature_frame_summary.csv:1>)

Typical values include:

- number of rows in the assembled modeling dataframe
- number of base numeric features

#### `eda_summary.csv`

Small descriptive summary of the training target distribution.

Example:

- [tables/eda_summary.csv](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/tables/eda_summary.csv:1>)

#### `*_feature_signal.csv`

These tables summarize feature importance / feature signal for stage 1 and stage 2.

Examples:

- `final_stage1_feature_signal_h24.csv`
- `final_stage2_feature_signal_h48.csv`
- `pair_h24__clf_xgbc01__reg_xgbr01_stage1_feature_signal.csv`

Use the `final_*` files for the chosen final artifacts and the `pair_*` files for candidate-level analysis.

## `predictions/`

This folder contains the actual prediction CSVs.

### Most important files

#### `two_stage_pred_24h.csv` and `two_stage_pred_48h.csv`

These are the exported submission-style predictions produced by the selected submission models.

Examples:

- [predictions/two_stage_pred_24h.csv](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/predictions/two_stage_pred_24h.csv:1>)
- [predictions/two_stage_pred_48h.csv](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/predictions/two_stage_pred_48h.csv:1>)

Their schema is intentionally simple:

- `timestamp`
- `location`
- `pred`

These are usually the files you would submit or use for downstream inference analysis.

#### `final_refit_pred_24h.csv` and `final_refit_pred_48h.csv`

These are the predictions generated by the refit bundles.

Examples:

- [predictions/final_refit_pred_24h.csv](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/predictions/final_refit_pred_24h.csv:1>)
- [predictions/final_refit_pred_48h.csv](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/predictions/final_refit_pred_48h.csv:1>)

These are useful for comparing:

- the best candidate validation model
- the final refit version of the best pair

#### `pair_*_val_predictions.csv`

These are the detailed validation predictions for each candidate pair and fold.

Example:

- [predictions/pair_h24__clf_xgbc01__reg_xgbr01_f01_val_predictions.csv](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/predictions/pair_h24__clf_xgbc01__reg_xgbr01_f01_val_predictions.csv:1>)

These files are extremely useful for analysis because they include richer columns such as:

- `location`
- `timestamp`
- `target_timestamp`
- `horizon_step`
- `target_out`
- `target_nonzero`
- `stage1_proba`
- `stage1_label`
- `stage2_positive_pred`
- `final_pred`
- `final_pred_hard_gate`

Use them when you want to inspect:

- how the stage-1 classifier behaved
- how large the positive prediction was
- how the combined prediction was formed
- which cases were false positives or false negatives

## `models/`

This folder contains the reusable trained artifacts.

If you want to load models again later, this is the folder to use.

### Three model families live here

#### 1. Candidate fold models

Examples:

- `pair_h24__clf_xgbc01__reg_xgbr01_f01_two_stage.joblib`
- `pair_h24__clf_xgbc01__reg_xgbr01_f01_stage1_classifier.joblib`
- `pair_h24__clf_xgbc01__reg_xgbr01_f01_stage2_regressor.joblib`

These are the models trained inside model selection.

They correspond to:

- one horizon
- one candidate pair
- one fold

Use these if you want to reproduce the exact validation-time artifact that produced a given fold score.

#### 2. Submission models

Examples:

- `submission_bundle_h24.joblib`
- `submission_stage1_h24_classifier.joblib`
- `submission_stage2_h24_regressor.joblib`

These are the artifacts actually used to generate:

- `two_stage_pred_24h.csv`
- `two_stage_pred_48h.csv`

They are the safest choice when you want to reuse exactly the same model that generated the exported submission file.

The metadata files explain their origin:

- [models/submission_bundle_h24_metadata.json](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/models/submission_bundle_h24_metadata.json:1>)
- [models/submission_bundle_h48_metadata.json](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/models/submission_bundle_h48_metadata.json:1>)

#### 3. Final refit models

Examples:

- `final_bundle_h24.joblib`
- `final_stage1_h24_classifier.joblib`
- `final_stage2_h24_regressor.joblib`

These correspond to the chosen pair after selection, retrained as final “refit” artifacts.

Their metadata files are:

- [models/final_bundle_h24_metadata.json](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/models/final_bundle_h24_metadata.json:1>)
- [models/final_bundle_h48_metadata.json](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/models/final_bundle_h48_metadata.json:1>)

### Which model should I use?

Use:

- `submission_bundle_h24.joblib` / `submission_bundle_h48.joblib`
  - if you want to recreate the exported submission predictions exactly

- `final_bundle_h24.joblib` / `final_bundle_h48.joblib`
  - if you want the final refit artifact associated with the winning pair

- `pair_*_two_stage.joblib`
  - if you want one specific validation candidate artifact

## `metrics/`

This folder stores numeric diagnostics and training histories.

### `pair_*_fold_metrics.csv`

Each file is the fold-level metric record for one candidate pair.

Example:

- [metrics/pair_h24__clf_xgbc01__reg_xgbr01_fold_metrics.csv](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/metrics/pair_h24__clf_xgbc01__reg_xgbr01_fold_metrics.csv:1>)

These are especially useful when you want a single CSV per candidate rather than one large merged summary table.

### `pair_*_stage1_history.json` and `pair_*_stage2_history.json`

These record training/evaluation trajectories when the estimator exposes them.

Examples:

- [metrics/pair_h24__clf_xgbc01__reg_xgbr01_f01_stage1_history.json](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/metrics/pair_h24__clf_xgbc01__reg_xgbr01_f01_stage1_history.json:1>)
- [metrics/pair_h24__clf_xgbc01__reg_xgbr01_f01_stage2_history.json](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/metrics/pair_h24__clf_xgbc01__reg_xgbr01_f01_stage2_history.json:1>)

For tree boosters such as XGBoost, these JSON files can contain per-iteration validation curves like:

- log loss
- RMSE

For non-iterative models such as logistic regression or ElasticNet, these files may be very short or nearly empty.

That is expected and does not indicate a failure.

### `demo_eval_*.json`

These are optional demo-only evaluation files.

Examples:

- [metrics/demo_eval_24h.json](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/metrics/demo_eval_24h.json:1>)
- `demo_eval_48h.json`
- `demo_eval_final_refit_24h.json`
- `demo_eval_final_refit_48h.json`

These use the provided demo labels if they exist.

They are useful for sanity checks, but should not be treated as the official hidden-test evaluation.

## `plots/`

This folder stores visual diagnostics for quick interpretation.

Examples from the sample run:

- `backtest_layout.png`
- `pair_comparison_h24.png`
- `pair_comparison_h48.png`
- `best_pair_confusion_h24.png`
- `best_pair_confusion_h48.png`
- `best_pair_curves_h24.png`
- `best_pair_curves_h48.png`
- `best_pair_validation_trajectories_h24.png`
- `best_pair_validation_trajectories_h48.png`
- `prediction_distributions_h24.png`
- `prediction_distributions_h48.png`
- `selected_weather_features.png`
- `final_stage1_feature_signal_h24.png`
- `final_stage2_feature_signal_h48.png`

These plots are intended to make run inspection much faster.

In practice:

- look at `backtest_layout.png` to understand the split
- look at `pair_comparison_*.png` to compare candidate performance
- look at `best_pair_confusion_*.png` and `best_pair_curves_*.png` to understand stage-1 classification quality
- look at `prediction_distributions_*.png` to see whether predictions look reasonable
- look at `*_feature_signal_*.png` to see which engineered inputs were influential

## 6. Naming conventions

Run artifacts are heavily name-driven, so understanding the filename pattern helps a lot.

## Horizon prefixes

- `h24` means 24-hour horizon
- `h48` means 48-hour horizon

## Candidate tags

Examples:

- `clf_logen01`
- `clf_xgbc01`
- `reg_enet01`
- `reg_xgbr01`

These are short identifiers for the estimator family and hyperparameter preset used in the grid.

In the example notebooks:

- `clf_logen01` = logistic regression with elastic-net-style regularization
- `clf_xgbc01` = XGBoost classifier candidate
- `reg_enet01` = ElasticNet regression candidate
- `reg_xgbr01` = XGBoost regressor candidate

## Fold identifiers

- `f01` means fold 1
- if there were more folds, you would see `f02`, `f03`, and so on

## Full example

Take:

- `pair_h24__clf_xgbc01__reg_xgbr01_f01_two_stage.joblib`

This means:

- `pair_...`
  - candidate artifact
- `h24`
  - 24-hour horizon
- `clf_xgbc01`
  - XGBoost classifier candidate
- `reg_xgbr01`
  - XGBoost regressor candidate
- `f01`
  - fold 1
- `two_stage.joblib`
  - combined two-stage artifact

## 7. Submission model vs final refit model

This distinction is important.

### Submission model

The submission model is the artifact used to generate the exported submission-style CSVs:

- `two_stage_pred_24h.csv`
- `two_stage_pred_48h.csv`

In the example run, the submission model was:

- the best validation candidate model

This is documented in:

- [tables/submission_model_summary.csv](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/tables/submission_model_summary.csv:1>)

### Final refit model

The final refit model is the chosen pair retrained as the final bundle.

It generated:

- `final_refit_pred_24h.csv`
- `final_refit_pred_48h.csv`

This is documented in:

- [tables/final_refit_validation_summary.csv](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/tables/final_refit_validation_summary.csv:1>)

So if you ever wonder:

- “Which predictions should I treat as the submission outputs?”
  - use `two_stage_pred_*.csv`

- “Where is the final refit version of the winning pair?”
  - use `final_bundle_*.joblib` and `final_refit_pred_*.csv`

## 8. How to answer common questions using the run folder

### “What was the exact split?”

Open:

- `config.json`
- `tables/backtest_folds.csv`

### “Which model won?”

Open:

- `tables/best_pair_by_horizon.csv`

### “Which artifact produced the submission CSV?”

Open:

- `tables/submission_model_summary.csv`
- `models/submission_bundle_h24_metadata.json`
- `models/submission_bundle_h48_metadata.json`

### “Where are the reusable model files?”

Open:

- `models/`

### “Where are the per-row validation predictions?”

Open:

- `predictions/pair_*_val_predictions.csv`

### “Where are the overall comparison metrics?”

Open:

- `tables/pair_summary.csv`
- `tables/candidate_fold_summary.csv`

### “Where can I inspect stage-1 and stage-2 separately?”

Open:

- `tables/classifier_stage_summary.csv`
- `tables/regressor_stage_summary.csv`
- `metrics/pair_*_stage1_history.json`
- `metrics/pair_*_stage2_history.json`

### “Where can I see the chosen weather features?”

Open:

- `selected_weather_features.json`
- `tables/selected_weather_features.csv`
- `plots/selected_weather_features.png`

## 9. A practical walkthrough with `run_20260418_214955`

If someone gave you this run folder and asked, “What happened here?”, a good workflow would be:

1. Open [config.json](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/config.json:1>)
   - confirm horizons, split strategy, feature settings, and whether GPU was enabled

2. Open [environment.json](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/environment.json:1>)
   - confirm the platform and package versions

3. Open [tables/backtest_folds.csv](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/tables/backtest_folds.csv:1>)
   - confirm the exact train/validation windows

4. Open [tables/pair_summary.csv](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/tables/pair_summary.csv:1>)
   - compare all candidate pairs

5. Open [tables/best_pair_by_horizon.csv](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/tables/best_pair_by_horizon.csv:1>)
   - identify the winners

6. Open [tables/submission_model_summary.csv](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/tables/submission_model_summary.csv:1>)
   - see which concrete artifacts generated the exported predictions

7. Open [predictions/two_stage_pred_24h.csv](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/predictions/two_stage_pred_24h.csv:1>) and [predictions/two_stage_pred_48h.csv](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/predictions/two_stage_pred_48h.csv:1>)
   - inspect the final exported outputs

8. Open [models/submission_bundle_h24_metadata.json](</c:/Users/Arturo Arias/OneDrive/Documentos en OneDrive/CMU/2026-1 Spring/Machine Learning for Problem Solving/Project/run_20260418_214955/models/submission_bundle_h24_metadata.json:1>)
   - connect the exported prediction file to the exact trained model

9. Open the key plots in `plots/`
   - visually verify the split, model comparison, confusion matrix, and prediction distributions

## 10. A few practical caveats

### Some paths may look like Colab paths

You may see paths such as:

- `/content/drive/MyDrive/MLPS Project/...`
- `results/run_20260418_214955/...`

This is normal if the run was executed in Colab or on a machine where the notebook used that root.

These paths are still useful for understanding provenance, even if your local copy lives somewhere else.

### Demo evaluation is not the official challenge evaluation

Files like:

- `demo_eval_24h.json`
- `demo_eval_final_refit_48h.json`

are only sanity-check evaluations against any demo targets available in the repo.

They are helpful, but they are not the hidden-test leaderboard metric.

### Checkpoints may be missing from GitHub

Again, `checkpoints/` is runtime infrastructure, not a required archive artifact.

If it is missing from a shared run folder, that does not mean the run is incomplete.

## 11. Short version: if you only remember three files

If you only remember three places to look, make them these:

1. `config.json`
2. `tables/best_pair_by_horizon.csv`
3. `tables/submission_model_summary.csv`

Together, those three files already tell you:

- how the run was configured
- which model won
- which exact artifact produced the exported prediction files

---

If you are browsing this repository and want a concrete example, start with:

- [run_20260418_214955](run_20260418_214955)

and then follow the reading order from Section 3.
