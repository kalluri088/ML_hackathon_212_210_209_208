# PathSnap — Team 6

Shared preprocessing and initial model-training handoff. No full model experiments,
Bayesian search, HMM or application results have been completed yet.

## Team assignments

| Member | Model(s) | Later ownership |
|---|---|---|
| Yaswanth | Logistic Regression, Histogram Gradient Boosting | Comparison, Bayesian tuning, HMM/Viterbi and integration |
| Madhu | Random Forest | FastAPI backend and server-side evaluation |
| Hrisheek | Extra Trees | Evaluator UI on Laptop 2 |
| Joshwin | MLP | Folium maps, experiment plots and report assembly |

Start with [training instructions](docs/TRAINING_GUIDE.md),
[team responsibilities](docs/TEAM_TASKS.md), and your copy/paste prompt:
[Yaswanth](docs/prompts/yaswanth.md), [Madhu](docs/prompts/madhu.md),
[Hrisheek](docs/prompts/hrisheek.md), [Joshwin](docs/prompts/joshwin.md).

## Quick start

Python 3.12+; data is already checked in, so no raw-data download or preprocessing run is needed.

```bash
python -m venv .venv
# Linux/macOS: source .venv/bin/activate
# Windows PowerShell: .venv\Scripts\Activate.ps1
python -m pip install -r requirements-training.txt
python -m unittest discover -s tests -v
# Replace random_forest with your assigned model name:
python scripts/model_training.py --model random_forest --folds 0 --smoke --out scratch/smoke
python scripts/model_training.py --model random_forest --final --out results/run_01
```

Use .venv's Python after activation. The five choices are logistic,
hist_gradient_boosting, random_forest, extra_trees, and mlp.

## Shared dataset

80 training trips / 208,417 GPS observations; 10 student-test trips / 19,160 observations.
Each observation has 8 candidate segments and 10 features. Default fitting uses 136,582
quality-filtered observations. All evaluation observations remain present.
Five fixed validation folds keep whole trips together. All models use the same dataset;
Logistic Regression and MLP scale features inside each fitted training pipeline.

**Labels are derived reference proxies.** Read [preprocessing decisions](PRE_PROCESSING_DECISIONS.md)
and [validation findings](reports/PREPROCESSING_SUMMARY.md). Two high-miss training trips
need review before final claims. Route continuity is N/A until shared graph scoring is added.

`processed/` contains per-trip NPZ files, schema, source-file hashes and inventory.
`scripts/` contains preprocessing, loading, scoring and training code; `tests/` checks geometry,
alignment and scoring. No evaluator data is included or used.

## Reproduce preprocessing (optional)

Requires the original raw package extracted as `Student folder/train/<id>/...` and
`Student folder/student_test/<id>/...`. It is deliberately not duplicated in this repository.

```bash
python -m pip install -r requirements-preprocessing.txt
python scripts/preprocess.py
python scripts/validate_dataset.py
```

Raw archives, original handout PDFs, generated HTML maps, virtual environments, duplicate ZIPs,
smoke runs and large trained artifacts are excluded. Every member retains actual logs, predictions,
model files and analysis for final submission; see the training guide for the handoff contract.
