# PathSnap HMM and Viterbi handoff

The HMM notebook was executed on the ten student-test trips with the tuned Extra Trees scores. Local source, data and results are under the Team 6 project folder. GitHub main was inspected at `afd03dadb7637982d13d45c8167107712b4ea0d8`. The local folder has no active Git checkout, and Git cloning could not resolve github.com. This branch contains decoder source, configuration and aggregate evidence. Per-trip files and the map are in the local team handoff. The separate Bayesian evidence was published to `feature/bayesian-optimisation`.

We read the project README, preprocessing decisions, training guide, team tasks, requirements, the shared/unpack/preprocessing and model notebooks, assignment PDFs, and existing tests. The real raw road graph files and prepared arrays for 80 training and 10 student-test trips were present. The tuning artifacts contain student-test probabilities, rather than tuned out-of-fold predictions. The HMM design was fixed before this experiment and HMM settings were not tuned on student test. A CV decoding diagnostic requires regenerating all five held-out score arrays with the selected model configuration; selection used those same folds, so such a diagnostic would not be an unbiased estimate of hyperparameter selection.

## Decoder and API

The hidden state is a trip-local candidate arc per observation. The uniformly floored positive-class scores are normalised over valid states and converted to logs. This is emission evidence derived from classifier scores, without a calibration model. For an arc pair, the decoder measures the remaining length of the first arc, a directed Dijkstra path to the second arc's start, and the length along the second arc. Forward travel on the same arc also has a direct partial-distance option. The selected distance must satisfy `20 m + 55 m/s × elapsed seconds`. The transition log potential is `−abs(network metres − GPS step metres) / (20 m + 0.25 × GPS step metres)`. Invalid transitions get negative infinity. The decoder uses bounded distance caches, stores dynamic-programming backpointers, and restarts at gaps over 30 seconds or where all paths fail. A point with no valid state emits arc ID -1 and records a break.

Use the notebook's first code cell as the source of `DecoderConfig`, `RoadGraph` and `viterbi_decode`. Move that definition cell into the trusted server bootstrap or a shared module when wiring the API. The same decoder does not need to run notebook experiment cells at server startup.

- Build/cache `RoadGraph(nodes_xy, arcs)` per trip. Use the same local metre projection and keep original directed arc IDs as row indices.
- Call `viterbi_decode(trip_id, xy, times, arc_ids, scores, graph, DecoderConfig())`. Inputs are ordered `N × 2` metre coordinates, `N` time values in seconds, and aligned `N × K` integer arc IDs and positive-class score arrays. Preserve sorted order and carry along original row indices if the caller needs to restore indexing. Score arrays for the current classifier have K=8. Out-of-range or missing arc candidates and invalid scores are masked. Malformed arrays raise ValueError.
- Return `arc_ids`, `candidate_indices`, `breaks`, and timing. Treat a nonempty break map as a segmented decode status. A predicted -1 indicates an observation without a state. The callable does not read reference files. Server scoring may read references separately after inference.
- The map plots actual selected road geometries. It does not export bridging arc geometry for paths between observations. For that later API feature, retain Dijkstra predecessors and reconstruct the chosen intermediate paths.

## Measured tradeoff

| Student-test metric | Tuned classifier | Classifier + Viterbi |
|---|---:|---:|
| Top-1 | 81.18% | 75.99% |
| Macro precision | 0.6772 | 0.7478 |
| Macro recall | 0.6191 | 0.7180 |
| Macro F1 | 0.6358 | 0.7194 |
| Reliable-subset Top-1 | 88.68% | 80.37% |
| Reliable-subset Macro F1 | 0.7584 | 0.7752 |
| Same arc or directed endpoint adjacency | 89.67% | 95.64% |
| Directed path within motion bound without restart | 86.94% | 99.56% |
| Mean trip collapsed-sequence similarity | 0.4856 | 0.6722 |

Continuity rates divide by all 19,150 within-trip GPS-point pairs, including sequence breaks. Motion-feasible continuity is a model-defined feasibility measure, not proof of physical route accuracy. Both variants evaluate all 19,160 points and retain the same 67 candidate misses. Reliable coverage is 69.45%. Sequence similarity uses run-collapsed edit distance against aligned proxy labels. Student-test Top-1 decreased on five trips and increased on five. See per_trip.csv for all ten and for decoder timing. The model's empirical tradeoff suggests treating decoder constraints as provisional and evaluating the effect of emission weight and transition scale on permitted training trips before deployment.

There were 83 unreachable restarts and two large-gap restarts. Measured decoding took 5.54 seconds total, and graph building took 0.24 seconds. The largest combined decoder array allocation reported was 886,592 bytes. The graph, Python objects, cache and library allocation are outside that figure. Both classifiers and decoder results use derived proxy labels.

## Reproduction and evidence

Install the repository's requirements, supply the original Student folder archive and processed dataset, and obtain `results/bayesian_extra_trees_01/tuned_student_test_predictions.npz`. Launch `jupyter lab` from the project root and run `notebooks/06_hmm_viterbi.ipynb` top to bottom. The experiment needs the local outputs of the tuning notebook, which are kept outside public Git. Existing tests were run with `python -m unittest discover -s tests -v` in the project workspace. The HMM notebook has a standalone synthetic correctness cell and a 40-observation real smoke check. The initial run failed to find the raw archive from its staged notebook folder. Linking the existing raw data there resolved it. All ten full-trip outputs were checked for original observation IDs, candidate ordering and score correspondence. Local outputs include configuration, graph metadata, break list, aggregate and per-trip scores, decoded observations, plots and a Folium map.

## Viva in ten answers

1. **What is an HMM?** It relates a sequence of observed GPS points to unobserved road states while modelling movement between states.
2. **What are the states?** The candidate directed arcs at each GPS observation. IDs only identify arcs within one trip.
3. **What are emissions?** Normalised and logged classifier scores for each valid candidate at a point.
4. **What are transitions?** A directed-route movement score constrained by elapsed time and network distance.
5. **How does Viterbi work?** Store the best prefix score ending at each candidate, record the predecessor, then follow those pointers backward.
6. **Why use it?** Independent classifiers can jump between roads. Viterbi prefers a sequence that can be driven.
7. **What about disconnected roads?** They have impossible transitions. Restart when no preceding state can reach any candidate.
8. **What about GPS gaps?** A gap over 30 seconds starts a new segment. Zero time permits only the positional slack.
9. **Did it improve?** It improved Macro F1, graph continuity and collapsed-sequence similarity. It reduced observation Top-1 by 5.19 percentage points.
10. **What are the limits?** Proxy labels, the fixed candidate pool, uncalibrated classifier scores, and provisional movement constraints. Held-out fold decoding remains to be run.
