# Prompt for Madhu

Copy the following into your coding assistant from this repository checkout.

```text
I am Madhu, a member of Team 6 working on PathSnap.
Read README.md, PRE_PROCESSING_DECISIONS.md, docs/TRAINING_GUIDE.md,
and docs/TEAM_TASKS.md before changing anything.

My assigned model(s): random_forest.
Work on branch models/madhu-random-forest; preserve other team members' changes.
Implement and run my model training in executed Jupyter .ipynb notebooks: notebooks/random_forest.ipynb.
Use the supplied model notebook in my branch and save my executed output there.
Preserve actual outputs, configurations, warnings, fold metrics, applicable plots, and
my own analysis. Import shared loader/trainer/scoring functions instead of duplicating
the evaluation contract. Install requirements.txt.
First implement/verify and perform my model training only. Do not start later application
work until the training outputs are complete and the team agrees to proceed.
Run notebooks/01_unpack_data.ipynb once to prepare the shared dataset. My model notebook loads notebooks/00_shared_training.ipynb for the common loader, training and metrics.
Do not regenerate data or change labels/features/folds/metrics independently.
Use the model notebook’s optional smoke cell, then run its full training cell for all five fixed trip CV folds and the final student-test evaluation.
Use a fresh result directory for each run. Keep the student-test observations out of fitting.
Retain the scaler inside the fitted pipeline when applicable. Do not split candidate rows
randomly. Keep missing reference candidates in evaluation. Labels are derived proxies.

Deliver executed .ipynb notebooks, configuration, provenance, actual metrics and warnings, saved per-split candidate
probabilities/predictions, the final pipeline/model, exact run commands, and analysis.md.
Explain my algorithm's suitability, objective, settings, measured limitations and fit; do not
fabricate outcomes, training curves, or route continuity values. Report timing scope and
proxy-label limitations. Use the shared scoring conventions. Record every experiment tried.
If resource problems or alignment defects appear, record evidence and coordinate a shared
fix rather than silently changing the benchmark. Do not fabricate completed folds.

My later project responsibility: Implement FastAPI model-loading/inference endpoints and server-side scoring as separate steps; expose trip choices/results without reference labels; retain receiving-laptop API logs; coordinate graph-based transition/continuity work with Yaswanth.

Do not commit, push, merge, force-push, or contact anyone automatically. Prepare work
for team review; I will explicitly authorise publication when appropriate. Do not share
this team's work with other teams. Help me understand and explain every submitted claim.
```

Repository executable workflows are .ipynb only. Open 01_unpack_data.ipynb first, then your assigned notebook. Do not add .py training files. Preserve the shared notebook scoring contract.
