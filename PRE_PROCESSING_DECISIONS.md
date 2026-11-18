# PathSnap preprocessing decisions

## Purpose and status
One shared candidate dataset supports Logistic Regression, Histogram Gradient Boosting,
Random Forest, Extra Trees, and MLP. No model has been trained. Labels derived below
are **proxy labels**, not organiser-supplied observation-level ground truth.
Raw files are preserved. Only train and student_test are read; evaluator data is excluded.

## Representation
Each trip produces one compressed NumPy file. `X` has shape (GPS observations, 8 candidates,
10 features); `y` is binary candidate correctness. Train a binary candidate classifier on
flattened rows, then reshape positive-class scores and choose the best candidate per GPS
observation. Candidate IDs map scores back to actual road segments and remain metadata.
Arc IDs are local to their trip. Network file hashes differ across all trips in each partition.
Never use trip IDs, raw row IDs, reference IDs, or label diagnostics as predictive features.

## Geometry and candidates
Coordinates are longitude then latitude. Each trip uses a local equirectangular projection
centred on its median road-node longitude/latitude, with mean Earth radius 6371008.8 m.
x = R*cos(latitude_origin)*(longitude-longitude_origin) in radians; y = R*(latitude-latitude_origin).
Units are metres and the origin is stored per trip. This is a spherical local approximation,
not a surveyed ellipsoidal projection. Scale distortion grows away from the origin; the
validation report compares segment lengths with haversine distances. Revisit projection
for geographically broad future trips. PyPI DNS was unavailable, so cached dependencies
were used and no unavailable projection library was required.
Arcs are treated as directed straight lines between supplied nodes; richer geometry is absent.
Eight nearest segments are selected by exact point-to-segment distance, with arc ID breaking ties.
A midpoint spatial index seeds the search; an expanded bound includes all potentially closer
segments, including long segments whose midpoints are far from the GPS observation.
There is no maximum candidate radius: sparse-network observations still receive candidates,
and distances expose the mismatch. K=8 is an initial engineering choice, not a tuned optimum.
Candidate miss rates must be reported and K may need revisiting using permitted development data.

## Cleaning
Reject malformed dimensions, invalid node/arc/reference arrays, fractional indices and out-of-range
indices. Remove nonfinite GPS rows or invalid GPS coordinate ranges and record the count.
Stable-sort timestamps, retain original row numbers, retain duplicate timestamps and GPS points.
Do not interpolate gaps, smooth GPS, trim speed outliers, or delete difficult trips.
At zero/negative elapsed time, speed is zero. Heading is unavailable for the first observation,
steps shorter than 2 m, duplicate timestamps, or gaps over 30 s. These are fixed initial heuristics.
Large gaps are retained and exposed in features/diagnostics for later decoder handling.

## Feature order
1. distance_m: GPS-to-candidate distance.
2. heading_cos: agreement of previous GPS displacement with directed segment; zero when unavailable.
3. heading_available: flag distinguishing unknown heading from perpendicular movement.
4. segment_length_m.
5. projection_fraction: position of closest point along segment, clipped to [0,1].
6. speed_mps: previous GPS displacement / elapsed time.
7. previous_gap_s: elapsed time; zero for first point.
8. gps_step_m: previous GPS displacement; zero for first point.
9. candidate_distance_rank: zero-based rank among candidates.
10. distance_minus_nearest_m: distance relative to closest candidate.
All inference features depend only on GPS and network, never the reference route.
Only current and previous GPS are used, allowing the same feature computation at inference.

## Reference-label derivation and limitations
.route is an ordered traversal, not one arc per GPS point. Align observations to route
**occurrences** by dynamic programming: minimise sum of squared point-to-reference-segment
distances, subject to nondecreasing occurrence indices. Start/end are free and occurrences
may be skipped because GPS does not necessarily observe every segment. Repeated route
arc IDs retain separate occurrences. Store both occurrence and resulting arc ID.

This is an assumption, not an official scoring convention. There is no timestamp ground truth
for route occurrences. Stops, parallel roads, loops, gaps, and inaccurate reference routes can
produce wrong proxy labels. The alignment permits free forward skips; it is not a physical
speed-constrained reconstruction. Squared distances can let GPS outliers influence alignment.
Do not present derived-label accuracy as independently verified physical accuracy.
Before final scoring, confirm the organisers' observation-level reference convention.

For each point retain aligned distance and margin to the nearest *different* reference arc.
A label is flagged reliable only if distance <=50 m and margin >=3 m. These fixed thresholds
are conservative starting heuristics and are not calibrated confidence probabilities.
Training additionally requires that the derived reference arc is among the eight candidates.
Only those observations enter the default fitting loader; all eight candidate rows are retained
for each eligible observation. No class balancing, resampling, or negative sampling is applied.
This excludes ambiguous intersection/opposite-direction examples and biases the fitting set;
report coverage and do not claim filtering solves label uncertainty.

## Evaluation and leakage
All valid student-test and validation GPS observations remain in evaluation, including candidate
misses and ambiguous labels. Never silently filter evaluation to the easy training subset.
Report full proxy-label metrics, reliable-label subset metrics with coverage, and candidate miss
rate separately. Candidate misses count as observation-level selection errors. Binary candidate
F1 is not a substitute for observation-level arc-selection Macro F1.
Observation-level log loss needs an explicit scoring convention for reference arcs absent from
the candidate set; do not simply drop those points. Candidate binary probabilities are not an
intrinsically normalised categorical distribution over candidates: define and document any
normalisation/calibration used for HMM emissions and categorical log loss.

Five fixed trip-based folds are assigned by shuffling sorted training trip IDs using seed
212210209208 and assigning round-robin. Student-test fold is -1. All rows of each trip stay
together. Fit scalers, imputers, feature selectors, calibration, and resampling only on the fitting
trips inside each CV fold. Hyperparameter optimisation also uses these training folds.
Reference alignment is fixed, label-only, per trip; it does not fit feature transforms across splits.

## Different models need different transforms, not datasets
Use this same dataset, folds, and scoring for everyone. Logistic Regression and MLP should
use StandardScaler in a training pipeline. Random Forest, Extra Trees, and Histogram Gradient
Boosting normally use raw features. Do not distribute globally standardised data: that would
leak validation statistics. Class weights are a model-training decision to record separately.
The agreed fifth model is Extra Trees, replacing the expensive RBF SVM discussed earlier.

## Reproduce and load
```bash
uv venv .venv
uv pip install --python .venv/bin/python -r requirements-preprocessing.txt
.venv/bin/python scripts/preprocess.py
.venv/bin/python -m unittest discover -s tests -v
```
See `scripts/load_dataset.py`: `load_training(validation_fold=0)` returns training and held-out
trip dictionaries; `load_training()` returns all fitting trips and student-test observations.
Use X.reshape(-1, X.shape[-1]) and y.ravel() for classifier fitting. Keep the observation/candidate
shape to reconstruct predictions. Load NPZ with allow_pickle=False. Feature order is schema.json.
Training arrays can be large: load one split/fold at a time and avoid unnecessary copies.

## Artifacts
- processed/train and processed/student_test: per-trip arrays and diagnostics.
- processed/inventory.csv: per-trip data quality, network and alignment summaries.
- processed/schema.json: feature order and fixed configuration.
- processed/raw_sha256.json: raw-file provenance hashes.
- reports: validation and geographical exploration outputs.
- exports/pathsnap_shared_preprocessed.zip: common dataset, scripts, docs, and checks.
The archive does not contain the raw road networks; retain the original Student folder package
for HMM development and road-network rendering. One shared archive avoids five redundant copies.

## Changes after this version
If candidate generation, alignment, features, thresholds, or folds change, regenerate the shared
package and have every teammate rerun comparisons. Do not mix results from different versions.

## Local tooling and repository state
The local workspace is not currently an active Git checkout (`git status` reports no repository).
During the initial preprocessing stage, no Git initialisation, commit, push, or remote modification was performed. A later, separately authorised handoff publishes only the selected training materials. The original package
installation encountered unavailable DNS; locally cached NumPy, SciPy and Folium packages
were copied into the ignored environment to execute the pipeline. The shared ZIP contains no
machine-specific environment paths and can be installed normally using the requirements file.

## Verified run results
- 80 training trips: 208,417 observations, 1,667,336 candidate rows.
- Default fitting mask: 136,582 observations (65.53%); 1,092,656 candidate training rows.
- 10 student-test trips: 19,160 observations, 153,280 candidate rows; all retained for evaluation.
- Candidate misses against derived labels: training 18,967 (9.10%); student-test 67 (0.35%).
- Training trip 00000054 accounts for 13,162 misses out of 19,313 observations;
  trip 00000033 contributes 3,867 out of 5,844. Their aligned-distance 95th percentiles
  are approximately 209 m and 142 m. These are priority alignment/data-quality investigations,
  not proof that the candidate search is incorrect. No trip was silently removed.
- 10 training reference routes and 1 student-test reference route have at least one consecutive
  pair whose directed endpoints do not join. This checks direct adjacency only; do not infer
  that every pair is unreachable through intermediate roads.
- No nonfinite or out-of-range GPS observations were removed.
- Five folds contain 16 training trips each.
- Maximum segment-length difference between the local projection and spherical haversine
  is 1.14% for segments longer than 1 m. This is not an ellipsoidal accuracy guarantee.
- Five focused tests passed, including exact candidate search against brute force and alignment
  optimality against exhaustive monotone paths on a small synthetic example.
- The real-data validator checked all 90 artifacts, exact candidates for 10 real observations,
  schema/finite arrays, sorted timestamps, candidate-label correspondence, fitting masks,
  fold assignments, and SHA-256 of all 360 original files.
- A permitted training-trip Folium map includes raw GPS, road candidates, supplied reference,
  and an explicitly labelled untrained nearest-road baseline.

These results validate implementation consistency, not physical truth of the derived labels.
Resolve the organiser's scoring convention and investigate high-miss trips before claiming
final observation-level accuracy. Do not tune thresholds using evaluator data.

## Public training-only handoff
Automatic approval review blocked publishing geographic bounds from the full inventory.
A separate export omits GPS coordinates/absolute timestamps, projected coordinates, projection
origins, original row IDs and geographic inventory columns. Full originals remain local.
scripts/export_training_data.py preserves X, y, candidate IDs, label diagnostics, folds and
fitting masks. The optional geographic validator requires full regenerated artifacts and raw
files; it is not a prerequisite for training with the stripped public export.

## Notebook handoff
The team’s local handoff contains the original and preprocessed ZIPs. Executable code in the repository is provided as notebooks. References to scripts above identify the original implementation, now embedded in 00_shared_training.ipynb and 02_preprocessing.ipynb. For archive setup, use requirements.txt and 01_unpack_data.ipynb.
