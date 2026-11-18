# Team 6 responsibilities

Assignments use the names supplied by the team owner, not unverified GitHub handles.
Everyone works within this same team; do not share code, weights, data derivatives or results
with other teams. Each member must understand and defend their own work and shared pipeline.

| Person | Initial models | Later project ownership | Deliverable branch |
|---|---|---|---|
| Yaswanth (owner) | Logistic Regression + Histogram Gradient Boosting | Review high-miss label alignment; consolidate five-model comparison; Bayesian tune the evidence-selected winner; HMM/Viterbi integration; final two-laptop integration | models/owner-baselines |
| Madhu | Random Forest | FastAPI backend/model loading; server-side scoring separated from inference; API logs; support road-network transition implementation with owner | models/madhu-random-forest |
| Hrisheek | Extra Trees | Laptop 2 UI: ten evaluation trip choices, API calls, predicted route/results, loading and failure states; integration with Madhu | models/hrisheek-extra-trees |
| Joshwin | MLP | Development Folium maps and result plots; collect measured per-model analysis into the consolidated PDF; report/repository reproducibility review | models/joshwin-mlp |

## Order of work
1. Everyone reads PRE_PROCESSING_DECISIONS.md and TRAINING_GUIDE.md and runs checks.
2. Each model owner performs a smoke check, five fixed CV folds and a student-test run.
3. Submit executed .ipynb notebook(s), model code/config, measured metrics, predictions, version records, warnings, and analysis.
4. Owner compares five models, reviews alignment limitations, and selects the Bayesian tuning target.
5. Implement and evaluate HMM, compare to classifier-only, then integrate FastAPI and UI.
6. Each member writes their actual findings; Joshwin assembles one PDF with team review.

## Shared responsibilities
- Do not change candidates, features, labels, folds or metric definitions independently.
  Flag problems and agree a dataset version change; all model comparisons must use one version.
- Candidate misses remain in evaluation; report uncertainty and reliable-subset coverage.
- No evaluator data is used for development. Evaluator labels never reach inference or the UI.
- No fabricated plots, scores, loss curves, citations, or claims of work not actually performed.
- Record assistance as required by event policy; every member must be able to explain their work.
- Work on personal branches and submit reviewable changes. Do not overwrite another member's
  results or merge/force-push main as part of model training. Only push when the owner authorises.
- API/UI work follows the training handoff; each prompt includes the later scope for planning.

Individual copy/paste prompts are in docs/prompts/.

Repository executable workflows are .ipynb only. Open 01_unpack_data.ipynb first, then your assigned notebook. Do not add .py training files. Preserve the shared notebook scoring contract.
