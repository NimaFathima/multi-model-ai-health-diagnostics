# Multi-Model AI Agent for Automated Health Diagnostics

An end-to-end pipeline that reads a blood-test report (PDF or image), extracts the lab parameters, checks them against reference ranges, scores overall health risk, and produces diet / exercise / lifestyle guidance. It comes with a FastAPI backend and a React dashboard.

> **Disclaimer:** This is an academic project. It gives general information only and is **not** a medical device or a substitute for professional medical advice.

Built during the Infosys Springboard internship programme. This is my individual work. The original working repository (with mentor review) is here: [mentor-pranaya/Design-and-Implementation-of-a-Multi-Model-AI-Agent-for-Automated-Health-Diagnostics, branch `Nima_Fathima`](https://github.com/mentor-pranaya/Design-and-Implementation-of-a-Multi-Model-AI-Agent-for-Automated-Health-Diagnostics/tree/Nima_Fathima).

---

## What it does

```
PDF / image / JSON
      |
      v
  OCR + text parsing            (core_phase1: pytesseract, pdf2image)
      |
      v
  Parameter extraction + validation     (core_phase2 / core_phase1)
      |
      v
  Model 1: reference-range evaluation   Normal / Low / High / Critical, severity, % deviation
      |
      v
  Model 2: multi-parameter patterns     cardiovascular, diabetes, kidney, thyroid, anaemia ...
      |
      v
  Risk scoring engine                   overall + per-system scores
      |
      v
  Model 3: recommendations              diet, exercise, lifestyle (knowledge-base driven)
      |
      v
  FastAPI  ->  React dashboard
```

See [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) for the module-by-module breakdown.

## Key features

- **Multi-format input:** PDF, PNG/JPG (via OCR) and JSON.
- **Data-driven evaluation:** no hard-coded thresholds in the logic. Ranges live in JSON files and are loaded by a reference manager, so adding a parameter does not mean changing code.
- **Several reference sources** combined by a `UnifiedReferenceManager`: standard laboratory ranges, NHANES-derived ranges, Indian-population data (ILPD liver dataset, published Indian reference-interval studies) and ABIM ranges.
- **Sex- and age-aware** evaluation where the reference data supports it.
- **Severity grading** (mild / moderate / severe) and percentage deviation from the range.
- **Pattern detection** across parameters, not just single values.
- **Risk scoring** with cardiovascular, diabetes and kidney components.
- **Knowledge-base recommendations** (JSON) with a safety layer that always attaches a medical disclaimer.
- **Evaluation framework** that compares pipeline output with ground-truth annotations.

## Tech stack

| Layer | Tools |
|---|---|
| OCR / ingestion | Tesseract (`pytesseract`), `pdf2image`, Pillow |
| Core logic | Python 3.9+, JSON knowledge bases |
| Reference data | NHANES, ILPD (Indian Liver Patient Dataset), laboratory and ABIM ranges |
| API | FastAPI, Pydantic, Uvicorn |
| Frontend | React + TypeScript, Vite, Tailwind CSS |

## Project structure

```
core_phase1/        OCR, ingestion, text parsing, validation
core_phase2/        parameter extraction, rule-based classifier
core_phase3/        evaluation, risk scoring, recommendations, knowledge base
  knowledge_base/   reference-range and guideline JSON files, reference managers
  recommendations/  diet / exercise / lifestyle modules
api/                FastAPI backend (upload -> process -> fetch report)
blood-report-ui/    React dashboard
config/             configuration
evaluation/         evaluation framework and results
docs/               architecture, evaluation write-up, data notes
```

## Getting started

### Prerequisites
- Python 3.9+
- [Tesseract OCR](https://github.com/tesseract-ocr/tesseract) installed and on your PATH
- Poppler (needed by `pdf2image` for PDFs)
- Node.js 18+ (for the dashboard)

### Backend
```bash
git clone https://github.com/NimaFathima/multi-model-ai-health-diagnostics.git
cd multi-model-ai-health-diagnostics
python -m venv .venv
# Windows: .venv\Scripts\activate     macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt
cp .env.example .env
uvicorn api.main:app --reload --port 8000
```

### Frontend
```bash
cd blood-report-ui
npm install
npm run dev
```
The API allows CORS from `http://localhost:3000`; change it in `api/main.py` if your dev server uses another port.

### Reference data (NHANES)
The raw NHANES CSVs are large and are **not** stored in this repo. Download them from the [CDC NHANES site](https://wwwn.cdc.gov/nchs/nhanes/) into the `NHANES/` folder, and see [docs/DATA.md](docs/DATA.md). The processed reference ranges used by the pipeline are already included in `core_phase3/knowledge_base/`.

## API

| Method | Endpoint | Purpose |
|---|---|---|
| POST | `/api/reports/upload` | Upload a report; returns a `reportId` and starts processing in the background |
| GET | `/api/reports/{id}/status` | Processing progress (0-100) |
| GET | `/api/reports/{id}` | Parameters, classifications, risk scores, recommendations |
| GET | `/api/reports/history` | Paginated list of processed reports |
| POST | `/api/reports/{id}/export` | Export placeholder (see limitations) |

## Evaluation

The evaluation framework runs the pipeline on a set of 17 de-identified-style test reports (13 PDF, 4 PNG) and compares the output with ground-truth annotations.

| Metric | Result |
|---|---|
| Reports evaluated | 17 |
| Parameters compared | 203 |
| Classification agreement with annotations | 199 / 203 (**98.03%**) |
| Disagreements | 4, all where the annotation was "Unknown" (no reference range available) |

**How to read this number:** the annotations were produced with the helper script `create_ground_truth_annotations.py` in this project, so this is a *consistency / regression check of the pipeline against its own annotation set*, not an independent clinical validation. A proper validation would need labels made by hand or by a clinician on a separate set of reports. Details are in [docs/EVALUATION.md](docs/EVALUATION.md). The test reports themselves are real patient documents and are not published.

## Limitations (honest status)

- Report storage in the API is **in-memory**; restarting the server clears it. No database is wired in even though `.env.example` lists PostgreSQL/Redis placeholders for future work.
- If a report has no readable age/sex, the API falls back to default values, which can change which reference range applies.
- Patient name/ID in API responses are placeholders; the export endpoint is a stub.
- OCR accuracy depends on scan quality; heavily formatted or low-resolution reports can lose parameters.
- Not clinically validated. Recommendations are generic guidance from a JSON knowledge base.

## Roadmap

- Persistent storage and user accounts
- Independent, hand-verified evaluation set
- Real PDF export of the analysed report
- Broader parameter coverage and unit conversion

## Author

**Nima Fathima** - B.Tech CSE (Artificial Intelligence), Adi Shankara Institute of Engineering and Technology, Kalady.
GitHub: [@NimaFathima](https://github.com/NimaFathima)

## License

MIT - see [LICENSE](LICENSE). Datasets (NHANES, ILPD) keep their own terms of use.
