# Evaluation

## Setup
- Test set: 17 reports (13 PDF, 4 PNG). Two further reports were excluded because their text could not be read reliably.
- Ground-truth files live in `evaluation/test_dataset/ground_truth/`. They were created with `create_ground_truth_annotations.py` / `ground_truth_generator.py`, which **pre-fill each file from the pipeline's own extraction output** and mark it "Auto-generated template - requires manual verification" (`verified: false`).
- Runner: `evaluation_runner.py` and the scripts in `evaluation/`.

## Results
| Metric | Value |
|---|---|
| Parameters compared | 203 |
| Matching classification | 199 |
| Agreement | 98.03% |
| Mismatches | 4 (annotation = "Unknown") |

## How to read this (important)
- Because the labels were generated from the pipeline's own output and not hand-verified, the 98.03% measures **self-consistency**: it shows the classification step behaves the same way on re-runs. It is **not** an accuracy measurement against independent truth.
- It can hide extraction errors. For example, in the stored labels some units are wrong (a unit read as "years" for AST, or flag words like "Low" captured as units), and the LDL/HDL values in one report look swapped. These are OCR/parsing mistakes the labels simply copied.
- Earlier internal write-ups used words such as "100% extraction" or "publication-quality". Those are **not** claimed here.

## What a real evaluation would need
1. Hand-check (or have a clinician check) the values, units and classifications for each report and set `verified: true`.
2. Score extraction and classification separately against those verified labels.
3. Use a held-out set of reports not used while developing the parsing patterns.

## Privacy
The test reports are real patient documents and are intentionally excluded. The label files contain only parameter values and file names.
