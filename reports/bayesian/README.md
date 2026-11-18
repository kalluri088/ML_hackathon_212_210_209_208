# Extra Trees Bayesian optimisation

The five-model baseline table is in [baseline_comparison.csv](baseline_comparison.csv). Extra Trees leads the fixed-fold arc Macro F1 comparison at 0.65852. Random Forest's all-fold mean includes its separately executed fold 0. The [shared-protocol MLP rerun](../../notebooks/03_mlp_shared_baseline.ipynb) scored 0.64321 and completed after the existing Extra Trees study. It confirmed the choice retrospectively. The study was run before five directly comparable baselines were available. No earlier run is represented as satisfying that chronological gate.

GitHub base was `afd03dadb7637982d13d45c8167107712b4ea0d8` on `main`. The local project folder has no active Git checkout, and git transport could not resolve GitHub. The authenticated GitHub connector can create a review branch without changing main. We reviewed the GitHub README, preprocessing decisions, training guide, team tasks, dependency manifest, notebooks and original local assignment PDFs. All 80 training and 10 student-test trips are available locally. Random Forest’s 90 recorded file hashes match the training-only export. Its feature, label, candidate, fold and mask arrays were checked against the full local data. Extra Trees’ published per-fold scores were reproduced on the local files to displayed precision. Logistic, Boosting and the MLP rerun record identical full dataset hashes. Evaluator data was outside the workflow.

## Study and findings

The [executed study](../../notebooks/04_bayesian_extra_trees.ipynb) uses a fixed-seed Matérn Gaussian process with expected improvement over a discrete pool. This sequential Bayesian method reuses installed scikit-learn/SciPy and stores every completed trial in an atomic JSON checkpoint. The four initial points include the published baseline. Two later points were proposed by the surrogate. The user capped the study at six completed trials. Trial 7 started and was interrupted, with no partial fold score included. Neither pruning nor random row splits were used. The six trials took 805 seconds, excluding the interrupted trial and final model fits.

The objective was mean all-observation trip-and-arc Macro F1 over the five existing held-out trip folds, maximised equally across folds. Tuning varied tree count for computation and ensemble stability, depth and leaf size for model complexity, and features per split for randomness. The winning settings were 150 trees, depth 16, minimum leaf 8, and seven features per split. The 64-tree trial scored 0.65645 and the 96-tree depth-28 trial scored 0.65697, both below baseline. Trial 6 was close at 0.65912. See [all trial and fold results](trials.json).

| Metric | Untuned | Tuned |
|---|---:|---:|
| Mean grouped CV Macro F1 | 0.65852 | 0.65915 |
| Student-test Top-1 | 81.1065% | 81.1795% |
| Student-test arc Macro F1 | 0.63245 | 0.63579 |
| Student-test categorical log loss | 0.62583 | 0.62239 |
| Final fit time, seconds | 33.37 | 48.87 |
| Prediction time, ms per GPS point | 0.0231 | 0.0187 |

The gain is modest, amounting to 14 additional student-test matches. CV selection can favour noise on the same folds, and the labels are proxies. Lower log loss under the shared candidate normalisation shows better measured probability assignments on this sample. It does not establish calibration or physical route accuracy. No evaluator data was used. Small benchmark differences should be rechecked on the serving laptop.

[Report notebook and chart code](../../notebooks/05_bayesian_results.ipynb) generate PNGs from the recorded evidence. Six trials with four jointly changing inputs are insufficient for a defensible global hyperparameter-importance ranking. The relationship plots show measured associations. The shared classifier model gives candidate scores, not route continuity.

## Reproduce or deploy locally

Install `requirements.txt`, unpack the team datasets with `01_unpack_data.ipynb`, then execute `04_bayesian_extra_trees.ipynb` from the repo root. The executed copy shows the completed search. For fresh runs use a new `RUN_ID`. An identical run can resume from its checkpoint and verified configuration in `results/bayesian_extra_trees_01/`. Read the chart notebook without rerunning training. Seven existing geometry/alignment/scoring tests passed locally. Both saved Extra Trees models reloaded and matched the saved student-test predictions.

The model binaries, per-observation probability arrays and datasets stay in the local handoff. Load a trusted `tuned_model.joblib` with joblib in the recorded Python/scikit-learn environment. Verify the ten feature names in `processed/schema.json`. Flatten the input from `N × 8 × 10` to `(N*8) × 10`, call `predict_proba`, and select the column whose class label equals 1. Reshape to `N × 8` and keep each score aligned to its trip-local arc ID. Viterbi should clip and normalise valid candidate scores per observation before using logs. Ground-truth fields are for scoring after prediction.

## Viva

**60 seconds.** Extra Trees had the strongest comparable grouped CV score. We measured the baseline and three initial Extra Trees configurations, then used a Gaussian process to predict which settings might improve Macro F1. Expected improvement selected two more configurations. We used five trip-based folds per completed trial, then retrained the winner and baseline on all eligible training trips. CV F1 went from 0.65852 to 0.65915. Student Top-1 rose from 81.1065% to 81.1795%, so the benefit is small. The tuned model supplies candidate scores to Viterbi.

**Three minutes.** Start with the difference between candidate binary classification and selecting one arc for a GPS observation. Explain why trips stay grouped and why scaling is fitted in each fold for the linear and neural baselines. Explain Extra Trees' baseline and the four searched hyperparameters. Describe how the Gaussian process updates after each completed trial and how expected improvement balances promising areas and uncertainty. Explain the six-trial budget, the interrupted seventh trial and why student test was excluded from tuning. Give the table results, explain CV selection optimism and proxy labels, and describe the FastAPI and Viterbi interface.

| Question | Answer |
|---|---|
| What is Bayesian optimisation? | Sequentially learn a model of measured validation scores and use it to choose trials. |
| How is it different from grid or random search? | The choice depends on earlier measured scores and model uncertainty. |
| What are Optuna and TPE? | Optuna is a tuning framework. Its TPE sampler models parameter regions associated with better versus worse outcomes. Here we used a Gaussian process with expected improvement. |
| What is a trial? | One Extra Trees hyperparameter configuration evaluated across all five fixed folds. |
| What is the objective? | Maximise mean all-observation arc Macro F1 across the five grouped folds. |
| What is the search space? | Discrete choices of tree count, depth, leaf size and features per split recorded in the search configuration. |
| Parameters versus hyperparameters? | Tree splits are learned parameters. The number of trees and their limits are hyperparameters set before fitting. |
| Why group by trip? | Nearby observations from a trip are correlated. Grouping keeps a held-out trip out of fitting, including preprocessing fits. |
| Did performance improve? | Yes, slightly. The CV gain was 0.00062 and the student-test Top-1 gain was about 0.073 percentage points. |
| How does the model feed the API and HMM? | The server scores eight candidates per GPS point. Viterbi receives scores and arc IDs, combines them with graph transitions and outputs a sequence. |

The tuned model’s impurity-based feature importance reflects how the trees split the features. Correlated distance features can share or trade importance. It is not causal proof. See [feature importance](plots/09_feature_importance.png).
