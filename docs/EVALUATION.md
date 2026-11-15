# Evaluation

## Setup
- Test set: 17 reports (13 PDF, 4 PNG). Two further reports were excluded from the set because their text could not be read reliably.
- Ground truth: annotation files created with `create_ground_truth_annotations.py`.
- Runner: `evaluation_runner.py` and the scripts in `evaluation/`.

## Results
| Metric | Value |
|---|---|
| Parameters compared | 203 |
| Matching classification | 199 |
| Agreement | 98.03% |
| Mismatches | 4 (annotation = "Unknown") |

## Interpretation and caveats
- The annotation set was generated with a script from the project, so the figure shows the pipeline is **consistent with its annotation set**. It is not proof of clinical correctness.
- The 4 mismatches come from parameters with no usable reference range, which the pipeline cannot classify.
- Earlier internal write-ups used words like "100% extraction" or "publication-quality". Those are not claimed here, because extraction was not scored independently of the annotations.
- Next step for a stronger result: have a second person (or a clinician) label a fresh set of reports by hand and score against that.

## Privacy
The reports used for testing are real patient documents. They are intentionally excluded from this repository.
