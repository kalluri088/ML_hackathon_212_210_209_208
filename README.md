# PathSnap — Team 6

Notebook-only shared workflow. The Logistic Regression and Histogram Gradient Boosting notebooks include measured initial results. The three teammate notebooks are ready to run.

## Start training
1. Clone/download this repository and install requirements.txt in a Python 3.12+ environment.
2. Launch JupyterLab from the repository root.
3. Place the team’s data handoff archives in the paths listed below, then run notebooks/01_unpack_data.ipynb once.
4. Open your assigned notebook and run its real training cells; save actual outputs.

| Member | Model notebook(s) |
|---|---|
| Yaswanth | logistic.ipynb, hist_gradient_boosting.ipynb |
| Madhu | random_forest.ipynb |
| Hrisheek | extra_trees.ipynb |
| Joshwin | mlp.ipynb |

All notebooks live in notebooks/. Read docs/TRAINING_GUIDE.md and your prompt in docs/prompts/.
The original question/guidelines are the five PDFs in docs/given/.

## Local datasets

The notebooks can use an existing `processed/` directory or unpack the team’s dataset archives. For archive setup, place the original ZIP at `data/raw/student_data.zip` and the full preprocessing ZIP at `data/processed/preprocessed_data.zip`. `data/SHA256SUMS.json` records the expected files. Get the archives from the team’s local handoff. The original student package contains 80 training and 10 student-test trips. Each model uses the same data and folds.

## Reproducibility and limitations
notebooks/00_shared_training.ipynb defines the common loader/trainer/scoring; model notebooks run it.
notebooks/02_preprocessing.ipynb reproduces preprocessing and validation if needed.
Read PRE_PROCESSING_DECISIONS.md: observation labels are derived proxies, and high-miss training
trips need investigation. The two owner notebooks include five-fold and student-test results. Bayesian tuning and HMM evaluation follow the five-model comparison.
The notebook definitions derive from the tested pipeline: seven focused tests passed, five model
factories passed synthetic probability checks, and a small logistic smoke run completed. The two owner notebooks have also completed all five folds and their final student-test evaluation. Their code cells were executed locally without a Jupyter UI.

