# Architecture

The system is organised in three phases that map to the folders in the repository.

## Phase 1 - Ingestion and OCR (`core_phase1/`)
- `ingestion/` loads PDF, image or JSON input (`load_input`).
- `ocr/pdf_ocr.py`, `ocr/image_ocr.py` run Tesseract; `ocr/text_parser.py` cleans the text and pulls out parameter lines and patient metadata (age, sex).
- `extraction/parser.py` turns raw lines into `{parameter: {value, unit}}`.
- `validation/validator.py` rejects impossible or malformed values.

## Phase 2 - Structured extraction and first classifier (`core_phase2/`)
- `extraction/parameter_extractor.py` normalises parameter names and units.
- `models/rule_based_classifier.py` is the initial rule-based classifier that Phase 3 replaced with reference-range evaluation.

## Phase 3 - Evaluation, risk and recommendations (`core_phase3/`)

| Stage | Module | Output |
|---|---|---|
| 3a Evaluation (Model 1) | `evaluation/evaluator.py`, `knowledge_base/unified_reference_manager.py` | status, severity, deviation % per parameter |
| 3b Patterns (Model 2) | `contextual_model.py`, `health_risk_engine.py` | multi-parameter patterns (cardiovascular, diabetes, kidney, thyroid, anaemia) |
| 3c Recommendations (Model 3) | `recommendations/` (`diet`, `exercise`, `lifestyle`, `recommender`) | diet / exercise / lifestyle guidance |
| 3e Risk scoring | `risk_scoring_engine.py` | overall score and risk category |
| Orchestration | `orchestrator.py`, `main.py` (`Phase3RecommendationPipeline`) | combined report |

### Reference data
`knowledge_base/` holds `reference_ranges.json`, `nhanes_reference_ranges.json`, `abim_reference_ranges.json`, `medical_guidelines.json` and `food_guidelines.json`. The `UnifiedReferenceManager` merges the sources and chooses the most appropriate range (by sex, age and source priority). `nhanes_processor.py` and `build_reference_ranges.py` show how the NHANES-derived ranges were built.

### Safety layer
Every recommendation set passes through a `SafetyValidator` that attaches a medical disclaimer and avoids diagnostic wording.

## API and UI
`api/main.py` exposes upload / status / report endpoints. Processing runs as a background task and updates a progress value that the React dashboard (`blood-report-ui/`) polls. Results are mapped from the Phase 3 report to the UI schema (parameters, classification, health risk score, recommendations, summary).
