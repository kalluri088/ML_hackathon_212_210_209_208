# Notebook training guide

Install requirements.txt in a local Python 3.12+ environment (`python -m venv .venv`, activate,
then `python -m pip install -r requirements.txt`). Launch `jupyter lab` from the repository root.
Place the team’s data archives at the paths listed in the README and run notebooks/01_unpack_data.ipynb. If you already have the full processed/ folder, open your assigned model notebook directly. Each executable workflow
is a notebook, with shared definitions in 00_shared_training.ipynb. No .py files are required.

First set RUN_SMOKE=True to check the interface; smoke outputs are not experiment evidence.
The full training cell performs five fixed trip CV fits and a final fit/student-test evaluation.
Use a fresh RUN_ID when rerunning; results folders cannot silently be overwritten.
Save actual notebook outputs and your own algorithm/objective/configuration/result analysis.
Models must use the same features, labels, folds and scoring; do not randomly split candidate rows.
Logistic Regression and MLP use fold-fitted StandardScaler pipelines; tree models use raw features.
MLP must preserve convergence warnings and save real loss histories if plotting training loss;
do not invent validation loss curves. Initial settings are baselines, not optimal hyperparameters.
Do not perform Bayesian optimisation until all five initial model comparisons are complete.

## Shared evaluation
Evaluate all held-out observations, including candidate misses and ambiguous proxy labels.
The default fitting loader filters uncertain proxy labels; it does not filter evaluation points.
Report all-point scores and reliable-subset scores with coverage. Reference labels/diagnostics
must never become predictive features. Candidate-level binary metrics are not observation metrics.
Arc Macro F1 uses (trip ID, arc ID) classes, including the union of true and predicted classes.
CV means average the five fold scores equally; std is population std. Top-1 chooses the highest
positive-class candidate score and maps it back to its arc ID.
Categorical log loss floors scores at 1e-12 and normalises with a fixed 1e-12 missing-candidate
state. A missed reference uses that state's probability. This is an explicit scoring convention,
not calibrated missing-candidate probability. Binary candidate log loss is reported separately.
Inference timing measures predict_proba on prepared features; full system latency comes later.
Route continuity is N/A until agreed graph-based scoring exists; retain probabilities for rescoring.
The fitting subset is easier than full validation, so their metric gap alone cannot prove overfitting.

## Required outputs
Deliver your executed notebook(s), configuration.json, provenance.json, metrics.json, summary.json,
per-split prediction/probability NPZ files, final model.joblib (including scaling), warnings and your
analysis. Preserve all attempted configurations and exact run details. Keep large artifacts outside
notebook outputs and retain them locally for the final submission. Do not fabricate results.
Do not commit/push/merge automatically; prepare personal-branch work for owner review.
Each member must understand their code and actual findings. See TEAM_TASKS.md and prompts/.
